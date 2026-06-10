# 네트워크 토폴로지 (Network Topology)

이 문서는 Home Assistant 호스트와 NAS(Synology BeeStation)의 네트워크 구성을 정리한 것입니다.
저장소 정책에 따라 실제 서브넷 3옥텟·MAC·인증정보는 placeholder(`A`, `B`)로 대체했습니다.

- `192.168.A.0/24` : 메인 LAN (일반 클라이언트, 인터넷 게이트웨이)
- `192.168.B.0/24` : NAS 전용 격리 세그먼트 (BeeStation 전용)

## 토폴로지

```text
                 Internet (WAN)
                       │
            ┌──────────┴───────────┐
            │  OpenWrt Router       │  192.168.A.1
            │  (custom firmware,    │  SSH: key-only, password 로그인 차단
            │   default gateway)    │
            └──────────┬───────────┘
                       │  메인 LAN  192.168.A.0/24
        ┌──────────────┼───────────────────────────┐
        │              │                            │
   일반 클라이언트   ...                       ┌────┴───────────────────────┐
   (PC / 모바일 /                              │  Home Assistant Host (N100) │
    IoT 등)                                    │                             │
                                               │  enp1s0: 192.168.A.69       │ ← 메인 LAN
                                               │  enp3s0: 192.168.B.1        │ ← NAS 세그먼트 게이트웨이
                                               └────┬────────────────────────┘
                                                    │  격리 세그먼트 192.168.B.0/24
                                                    │  (전용 물리 NIC / 전용 회선)
                                              ┌─────┴──────────────┐
                                              │  Synology BeeStation│  192.168.B.23
                                              │  SMB (CIFS v3.1.1)  │  share: /home
                                              └────────────────────┘
```

## 핵심 설계 의도

| 설계 | 내용 | 효과 |
|------|------|------|
| **물리 NIC 분리** | HA 호스트에 NIC 2장 (`enp1s0` 메인 / `enp3s0` NAS) | NAS 트래픽을 메인 LAN과 L2 레벨에서 분리 |
| **NAS 전용 세그먼트** | BeeStation을 `192.168.B.0/24` 에만 배치, HA 호스트가 이 대역 게이트웨이(`.B.1`) | BeeStation이 메인 LAN·WAN에 **직접 노출되지 않음** |
| **SMB v3.1.1** | `//192.168.B.23/home` 을 CIFS 3.1.1 + `soft` 마운트 | 최신 암호화 프로토콜, NAS 장애 시 HA 비차단 |
| **상위 라우터 무경로** | 메인 라우터에 `192.168.B.0/24` 경로 미설정 | 외부/메인 LAN 클라이언트가 NAS 세그먼트로 라우팅 불가 |

## BeeStation 접근 경로

```text
BeeStation (192.168.B.23, SMB)
   ├─ /share/mp_beestation   ← rclone serve webdav (:10081) 의 소스
   └─ (HA 미디어/백업 연동)
```

상세 WebDAV 브리지 구성은 [`system-analysis.md`](system-analysis.md) §8 참고.

## 운영 메모 / 하드닝 체크리스트

- ✅ BeeStation WAN 미노출 — 전용 세그먼트 격리 확인
- ✅ OpenWrt 라우터 SSH password 로그인 차단 (key-only)
- ✅ **세그먼트 간 포워딩 차단 확인** — HA 호스트 `FORWARD` 기본정책이 `DROP`이고
  Docker 체인(`DOCKER-FORWARD`)은 도커 브리지(`docker0`/`hassio`)만 ACCEPT.
  메인 LAN ↔ NAS 세그먼트 라우팅은 기본 차단 상태.
- ✅ **명시적 격리 룰 추가** — 암묵적 기본값에 의존하지 않도록 `DOCKER-USER` 체인에
  양방향 DROP(`enp1s0 ↔ enp3s0`)을 명시. SSH 애드온 `init_commands`로 부팅 시 재적용(영속).
- ✅ **레거시 마운트 정리 완료** — 과거 메인 LAN 시절의 죽은 SMB 마운트(`//192.168.A.23`,
  `Host is down`) 언마운트 + 마운트포인트 제거. 현재는 격리 세그먼트(`//192.168.B.23`)만 active.
- ✅ **오프사이트 백업 동기화 가동** — "serve(노출)"와 별개로 HA 백업을 실제로 BeeStation에
  복사하는 작업을 추가. 상세는 [`system-analysis.md`](system-analysis.md) §8 참고.

> 참고: `ip_forward=1` 은 Docker 컨테이너 네트워킹에 필요해 끌 수 없으므로,
> 세그먼트 격리는 전역 포워딩 비활성화가 아니라 **인터페이스 단위 FORWARD 차단**으로 처리.
