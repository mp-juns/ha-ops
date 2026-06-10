# 전체 시스템 분석 (System Analysis)

이 문서는 실제 운영 중인 Home Assistant 스마트홈 시스템을 Docker 컨테이너, 애드온, 통합(Integration), 기기, 엔티티 수준까지 전수 분석한 결과를 **민감정보를 제거하여** 정리한 것입니다.

분석 기준일: 2026-06-11 / Home Assistant Core `2026.5.4` / Home Assistant OS 기반.

---

## 1. 호스트 / 런타임 환경

| 항목 | 값 |
|------|----|
| 플랫폼 | Home Assistant OS (Supervised, Docker 기반) |
| CPU | Intel N100 (4 cores) |
| 메모리 | 약 12GB (11Gi) |
| Docker | v29.x |
| HA Core | 2026.5.4 |
| Supervisor | amd64-hassio-supervisor |
| 모니터링 | Glances 애드온 (CPU/메모리/네트워크/컨테이너 상태) |

Home Assistant OS는 모든 구성 요소를 Docker 컨테이너로 실행합니다. 크게 **HA 코어 시스템 컨테이너(hassio_*)**, **사용자 설치 애드온(addon_*)**, **메인 homeassistant 컨테이너**로 나뉩니다.

---

## 2. Docker 컨테이너 전수 분석

### 2.1 코어 시스템 컨테이너 (hassio_*)

| 컨테이너 | 역할 |
|----------|------|
| `hassio_supervisor` | 애드온/백업/업데이트/네트워크를 총괄하는 슈퍼바이저 |
| `hassio_dns` | 내부 DNS 리졸버 |
| `hassio_audio` | PulseAudio 기반 오디오 스택 (음성 비서용) |
| `hassio_multicast` | mDNS/멀티캐스트 브리지 |
| `hassio_cli` | `ha` CLI 백엔드 |
| `hassio_observer` | 헬스 체크 / 관측 (포트 4357) |
| `homeassistant` | Home Assistant Core 본체 |

### 2.2 설치된 애드온 (16개)

| 애드온 | 버전 | 분류 | 역할 |
|--------|------|------|------|
| **Mosquitto broker** | 7.1.0 | 통신 | MQTT 브로커. Zigbee2MQTT·Frigate·기타 기기 메시지 허브 (1883/8883) |
| **Zigbee2MQTT** | 2.11.0 | 통신 | Zigbee 기기를 MQTT로 브리지 (ember 어댑터, /dev/ttyUSB0) |
| **Matter Server** | 8.5.0 | 통신 | Matter/Thread 기기 연동 서버 |
| **Frigate (Full Access)** | 0.17.1 | AI 영상 | NVR + 실시간 객체 감지(사람 등). go2rtc/RTSP/WebRTC 스트리밍 (5000/8554/8555/8971) |
| **Whisper** | 3.1.0 | 음성 | 로컬 STT(음성→텍스트), Assist 음성 비서 파이프라인 |
| **AdGuard Home** | 6.1.3 | 네트워크 | 네트워크 광고/추적 차단 DNS |
| **Tailscale** | 0.28.1 | 네트워크 | 메시 VPN 원격 접속 (healthy) |
| **Nginx Proxy Manager** | 2.1.0 | 네트워크 | 리버스 프록시 / SSL 관리 (81/82/443) |
| **Duck DNS** | 2.0.0 | 네트워크 | 동적 DNS + Let's Encrypt 인증서 갱신 |
| **SFTPGo** | 2.7.0 | 파일 | SFTP/FTP/WebDAV 파일 서버 (80/2022/8080/10443 등) |
| **Advanced SSH & Web Terminal** | 23.0.9 | 운영 | SSH/웹 터미널 관리 접속 |
| **Studio Code Server** | 6.0.1 | 운영 | 브라우저 VS Code 설정 편집기 |
| **Glances** | 0.22.0 | 모니터링 | 시스템 리소스 모니터링 |
| **Firefox** | 1.10.1 | 유틸 | 컨테이너 내 브라우저(스트림 디스플레이용) |
| **Samba share** | 12.6.1 | 파일 | 윈도우 네트워크 공유로 config 접근 |
| **HassOS SSH Configurator** | 0.9.3 | 운영 | 호스트 SSH(22222) 설정 (현재 중지) |

> 애드온은 모두 독립 Docker 컨테이너로 격리 실행되며, 슈퍼바이저가 생명주기를 관리합니다.

---

## 3. 통합(Integration) 분석

`core.config_entries` 기준, 활성 통합 도메인 분포:

| 분류 | 통합 |
|------|------|
| **AI / LLM** | `ollama`(로컬 LLM), `llmvision`(x2, 카메라 이미지 분석), `google_translate` |
| **음성** | `wyoming`(Whisper 연결), `voice_satellite`(음성 위성), `assist_satellite` |
| **영상/카메라** | `frigate`, `go2rtc`, `webrtc`, `generic`(범용 카메라), `tapo_control`(TP-Link) |
| **기기 생태계** | `zha`(Zigbee), `mqtt`, `matter`, `xiaomi_home`(제습기 등), `switchbot`(x2, 봇/스위치), `smartthings`, `broadlink`(IR 리모컨) |
| **모바일/존재감지** | `mobile_app`(x3), `ibeacon`, `ping`, `bluetooth`, `person` |
| **네트워크/인프라** | `adguard`, `synology_dsm`(NAS), `upnp`, `hassio` |
| **음성비서 연동** | `google_assistant` |
| **UI/관리** | `browser_mod`, `hacs`(커뮤니티 스토어), `backup`, `shopping_list`, `radio_browser`, `sun`, `met`(날씨) |

총 **약 40개 통합**이 활성 상태입니다.

### Custom Components (HACS 설치)

`browser_mod`, `clova`, `frigate`, `hacs`, `llmvision`, `smartir`, `tapo_control`, `voice_satellite`, `webrtc`, `xiaomi_home`

---

## 4. 기기 / 엔티티 인벤토리

| 항목 | 수량 |
|------|------|
| 등록 기기(devices) | **98** |
| 등록 엔티티(entities) | **919** |

### 도메인별 엔티티 분포 (상위)

| 도메인 | 수 | 도메인 | 수 |
|--------|----|--------|----|
| sensor | 495 | select | 30 |
| binary_sensor | 109 | device_tracker | 24 |
| switch | 97 | light | 19 |
| update | 34 | media_player | 19 |
| button | 18 | number | 14 |
| camera | 9 | automation | 7 |

그 외 `humidifier`, `climate`(스마트IR 에어컨), `fan`, `remote`, `siren`, `calendar`, `weather`, `todo`, `conversation`, `stt`, `tts`, `assist_satellite` 등 음성/제어 도메인 포함.

---

## 5. Zigbee 네트워크

- 어댑터: **ember** (EmberZNet), `/dev/ttyUSB0`, 115200 baud
- 채널: 11
- 브로커: `mqtt://core-mosquitto:1883`
- ZHA 통합과 Zigbee2MQTT 애드온이 함께 존재 (이중 Zigbee 스택)

---

## 6. 제어 / 자동화 핵심 구성

### 6.1 조명 제어
- 물리 벽 스위치를 **SwitchBot 봇**(`switch.switch_1` 켜기 / `switch.bot_0c49` 끄기)으로 누르는 방식
- `input_boolean.living_room_light` 헬퍼로 상태 추상화 → 일몰 15분 전 자동 점등, 자정 자동 소등

### 6.2 냉난방 / 습도
- **SmartIR climate**: BroadLink IR 리모컨(`remote.universal_remote`)으로 거실 에어컨 제어 (device_code 3020), 온습도 센서 연동
- **제습기**(Xiaomi `humidifier`): 습도 60% 초과 시 ON / 46% 미만 시 OFF, 물탱크 가득 차면 자동 정지 + 모바일 알림, 의류건조 모드 감지

### 6.3 PC / WSL 원격 운영
- **Wake-on-LAN**: 매직 패킷으로 PC 전원 ON (`switch.my_pc_switch` 템플릿)
- **최대절전/종료**: `shell_command.hibernate_pc`, SSH로 `wsl --shutdown`
- `ttyd` 기반 Windows PowerShell 웹 터미널 노출
- **ScamGuardian** 로컬 보안 서비스 상태를 funnel status 센서로 모니터링

### 6.4 네트워크 제어
- `command_line` 스위치로 **OpenWrt 라우터의 일본 VPN**을 SSH 스크립트로 ON/OFF/상태조회

---

## 7. AI 활동 추적 파이프라인 (핵심)

```text
사람 감지(occupancy) → 자동화 트리거 → 카메라 스냅샷
  → LLM Vision 활동 분류(JSON) → helper entity 상태 저장
  → History Stats 시간 누적 / 활동 카운터 → 알림
  → 자정 LLM 일과 리포트 → 캘린더 기록
```

- **거실 활동 인식**: 30초 이상 체류 시 스냅샷 → 위치(chair/bed/desk/floor/standing/away) + 활동(computer_work/sleep/phone/reading/eating/exercise) 분류
- **부엌 활동 인식**: cooking/eating/drinking/laundry/cleaning/passing/other 분류 후 일·주 카운터 증가
- **방 청결도 평가**: PTZ 카메라 4개 프리셋 다각도 촬영 → 각 구역 1~10점 평가 → 평균/최악 점수 알림
- **자정 일과 리포트**: 누적 데이터를 LLM API로 요약 → `input_text.daily_report` + 캘린더 이벤트 + 알림

LLM 백엔드는 LLM Vision 통합(이미지)과 REST 기반 텍스트 API(요약)를 병행 사용합니다.

---

## 8. 오프사이트 백업 / BeeStation WebDAV 브리지

Home Assistant 설정 파일에는 드러나지 않지만, SSH 애드온 호스트에서 **Synology BeeStation을 WebDAV로 연결·재노출하는 rclone 서비스**가 상시 동작합니다. (config 스캔이 아닌 프로세스/호스트 레벨 구성)

```text
Synology BeeStation
   │  (네트워크/FUSE 마운트)
   ▼
/share/mp_beestation         ← BeeStation 표준 폴더
   │  (A-EYE / Backups / Photos / Files / Computers / Cloud services / USB backup)
   ▼
rclone serve webdav  --addr 0.0.0.0:10081  --vfs-cache-mode writes
   ▼
WebDAV 엔드포인트 (:10081)   ← HA / Frigate / 클라이언트가 읽기·쓰기
```

| 항목 | 값 |
|------|----|
| 도구 | `rclone serve webdav` (호스트 상주 프로세스) |
| 소스 경로 | `/share/mp_beestation` (BeeStation 마운트) |
| 노출 포트 | `10081` (WebDAV) |
| VFS 캐시 | `writes` 모드, `dir-cache-time=10s`, `poll-interval=0` |
| 로그 | `/share/rclone-webdav.log` |
| rclone.conf | 없음 — remote 설정 없이 로컬 마운트 폴더를 그대로 serve |

용도: HA 백업/스냅샷/미디어를 NAS급 BeeStation 저장소로 오프사이트 보관하고, WebDAV 표준 프로토콜로 여러 클라이언트가 접근할 수 있게 하는 **저장소 게이트웨이** 역할.

> ⚠️ 운영 메모(보안): WebDAV 사용자/비밀번호를 `--user`/`--pass` 플래그로 넘기면 `ps`(프로세스 목록)에 평문 노출됩니다. 환경변수(`RCLONE_PASS`)나 obscure 값 사용을 권장하고, 외부 노출이 불필요하면 `--addr`를 `127.0.0.1:10081`로 제한하세요. 실제 자격증명은 공개 저장소에 포함하지 않습니다.

### 8.1 HA 백업 → BeeStation 자동 동기화

WebDAV serve(저장소를 "여는" 쪽)와 별개로, **HA 슈퍼바이저 백업을 실제로 BeeStation에 복사**하는 작업을 구성했습니다. 코어 컨테이너는 `/backup`을 보지 못하므로, `/backup`·`rclone`·BeeStation 마운트가 모두 보이는 **SSH 애드온**에서 실행합니다.

```text
/backup (HA 자동백업 tar, 로컬)
   │  rclone copy --include "*.tar"  (멱등)
   ▼
/share/mp_beestation/Backups/HA/   (BeeStation, SMB)
   +  30일 경과분만 prune (이 폴더 한정 — 다른 BeeStation 데이터 미접근)
```

| 항목 | 값 |
|------|----|
| 스크립트 | `/share/scripts/ha_backup_sync.sh` (락·로깅·마운트 생존확인 포함) |
| 스케줄 | 매일 **05:30** (04:45 자동백업 이후), busybox `crond` |
| 영속화 | SSH 애드온 `init_commands`(crond 등록) — 부팅/애드온 재시작 시 재적용 |
| 보관정책 | BeeStation 측 `Backups/HA/*.tar` 30일 (`rclone delete --min-age 30d`) |
| 로그 | `/share/ha_backup_sync.log` |

이로써 단일 노드(N100) 장애 시에도 HA 백업이 오프사이트(NAS)에 남습니다.

## 9. 보안 / 민감정보 처리

이 분석 문서는 다음 정보를 **포함하지 않습니다**: API Key, 실제 MAC/내부 IP/RTSP URL, SSH 사용자명·키, provider ID, `.storage/`, DB·로그·SSL 인증서, 생활공간 스냅샷. 실제 값은 `secrets.yaml`에만 보관하며 `.gitignore`로 커밋에서 제외됩니다.

> ⚠️ 운영 메모: 실제 설정 파일(`configuration.yaml`, `packages/activity_tracking.yaml`)에는 일부 API 키가 평문으로 하드코딩되어 있습니다. 공개 저장소에는 절대 올리지 않으며, 키를 `!secret` 참조로 옮기고 노출된 키는 폐기/재발급할 것을 권장합니다.
