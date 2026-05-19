# AI HomeOps: Home Assistant 기반 AI 스마트홈 자동화

AI HomeOps는 Home Assistant를 기반으로 IoT 기기, 사람 감지 센서, 카메라 스냅샷, LLM Vision 분석, PC/WSL 원격 제어, 서비스 상태 모니터링, 일일 생활 리포트를 통합한 개인 스마트홈 자동화 프로젝트입니다.

이 저장소는 실제 Home Assistant 설정을 그대로 공개한 것이 아니라, **민감정보를 제거한 공개용 포트폴리오 템플릿**입니다. API Key, MAC 주소, 내부 IP, SSH 사용자명, 카메라 엔티티, provider ID 등은 모두 placeholder 또는 `!secret` 참조 형태로 대체했습니다.

## 프로젝트 목적

일반적인 스마트홈 자동화는 보통 단순 규칙 기반입니다.

> 움직임이 감지되면 조명을 켠다.

AI HomeOps는 여기서 한 단계 더 나아가, 집 안의 상황을 인식하고 생활 데이터를 구조화합니다.

> 사람이 방에 일정 시간 머무르면 카메라 스냅샷을 촬영하고, LLM Vision으로 활동을 분류한 뒤, Home Assistant helper entity에 상태를 저장하고, 누적 데이터를 기반으로 일일 리포트를 생성한다.

즉, 단순 IoT 제어가 아니라 **Context-Aware Smart Home Automation**을 목표로 한 프로젝트입니다.

## 주요 기능

- Home Assistant 기반 스마트홈 자동화 구성
- 사람 감지 센서 기반 거실/부엌 활동 인식
- 카메라 스냅샷 + LLM Vision 기반 활동 분류
- 요리, 식사, 음료, 컴퓨터 작업, 수면, 폰 사용 등 생활 활동 구조화
- PTZ 카메라 프리셋을 활용한 다각도 방 청결도 평가
- LLM API를 활용한 일일 생활 리포트 생성
- Wake-on-LAN 기반 PC 전원 제어
- SSH shell command 기반 Windows/WSL 원격 제어
- 로컬 AI/보안 서비스 상태 모니터링
- Home Assistant helper entity 기반 상태 저장 모델링
- History Stats 기반 일일 활동 시간 누적
- 공개 저장소 업로드를 위한 민감정보 분리 및 보안 문서화

## 시스템 흐름

```text
사람 감지 센서
      ↓
Home Assistant 자동화 트리거
      ↓
카메라 스냅샷 촬영
      ↓
LLM Vision 활동 분석
      ↓
JSON 응답 파싱
      ↓
Helper Entity 상태 저장
      ↓
History Stats / 활동 카운터 / 알림
      ↓
일일 LLM 리포트 + 캘린더 기록
```

## 저장소 구조

```text
ha-ops/
├── README.md
├── automations/
│   ├── living_room_activity_detection.yaml
│   ├── kitchen_activity_detection.yaml
│   ├── daily_activity_report.yaml
│   └── light_helper_control.yaml
├── packages/
│   └── activity_tracking_helpers.yaml
├── examples/
│   ├── secrets.example.yaml
│   └── sanitized_entities.md
├── docs/
│   ├── architecture.md
│   ├── privacy.md
│   └── resume-description.md
└── .gitignore
```

## 핵심 자동화

### 1. 거실 활동 인식

거실에 사람이 일정 시간 이상 머무르면 카메라 스냅샷을 촬영하고, LLM Vision으로 현재 위치와 활동을 분류합니다.

분류 예시는 다음과 같습니다.

```json
{
  "location": "desk",
  "activity": "computer_work",
  "detail": "책상에서 작업 중",
  "detail_long": ""
}
```

분석 결과는 다음 Home Assistant helper entity에 저장됩니다.

- `input_text.current_location`
- `input_text.current_activity`
- `input_text.current_activity_detail`

이를 통해 다른 자동화, 대시보드, History Stats 센서가 현재 상태를 재사용할 수 있습니다.

### 2. 부엌 활동 인식

부엌에 사람이 30초 이상 머무르면 카메라 스냅샷을 촬영하고, LLM Vision으로 다음 활동 중 하나로 분류합니다.

- cooking
- eating
- drinking
- laundry
- cleaning
- passing
- other

분류 결과는 최근 부엌 활동 로그와 일일/주간 활동 카운터에 반영됩니다.

### 3. 방 청결도 평가

PTZ 카메라를 여러 프리셋 위치로 이동시키며 방을 다각도로 촬영하고, LLM Vision을 활용해 각 구역의 청결도를 1~10점으로 평가하는 자동화입니다.

계산 항목은 다음과 같습니다.

- 평균 청결도 점수
- 가장 지저분한 구역의 점수
- 각 구역별 짧은 평가 이유

### 4. 자정 일과 리포트

매일 밤 Home Assistant에 누적된 생활 데이터를 모아 LLM API에 전달하고, 하루 요약 리포트를 생성합니다.

수집 데이터 예시는 다음과 같습니다.

- 위치별 누적 시간
- 활동별 누적 시간
- 부엌 활동 횟수
- 방 청결도 점수
- 마지막 감지 활동

생성된 리포트는 Home Assistant 알림, `input_text.daily_report`, 캘린더 이벤트에 기록됩니다.

### 5. PC / WSL 원격 제어

Home Assistant를 단순 스마트홈 허브가 아니라 개인 운영 패널처럼 사용하기 위해, 로컬 개발 PC와 WSL 환경을 원격 제어하는 구성을 포함했습니다.

구성 요소는 다음과 같습니다.

- Wake-on-LAN 기반 PC 켜기
- shell command 기반 최대절전
- SSH 기반 WSL 종료
- 로컬 AI/보안 서비스 상태 확인

## 보안 처리

이 저장소는 공개용으로 정리된 버전입니다.

다음 정보는 절대 커밋하지 않습니다.

- `secrets.yaml`
- API Key
- 실제 MAC 주소
- 내부 IP 주소
- SSH private key
- 카메라 RTSP URL
- Home Assistant `.storage/`
- DB 파일
- 로그 파일
- SSL 인증서
- access token
- 실제 생활공간 스냅샷

실제 값은 `secrets.yaml`에만 보관하고, 공개 저장소에는 `examples/secrets.example.yaml`만 포함합니다.

## 포트폴리오 관점에서의 의미

이 프로젝트는 단순히 스마트홈 장비를 연결한 것이 아니라, 실제 생활 공간에 AI 기반 인식/기록/리포팅 파이프라인을 적용한 개인 HomeOps 실험입니다.

보여줄 수 있는 역량은 다음과 같습니다.

- IoT 자동화 구성 능력
- 이벤트 기반 시스템 설계
- Home Assistant YAML 구성 능력
- LLM Vision 연동 경험
- 비정형 AI 응답을 JSON으로 구조화하는 처리 흐름
- helper entity 기반 상태 모델링
- History Stats 기반 생활 데이터 누적
- 로컬 PC/WSL 개발환경 운영 자동화
- 공개 저장소 업로드를 위한 민감정보 분리 및 보안 의식

## 이력서 요약

```text
Home Assistant 기반 AI HomeOps 자동화 시스템을 구축했습니다. Occupancy sensor와 카메라 스냅샷을 연동해 거실/부엌 활동을 감지하고, LLM Vision으로 생활 활동을 분류한 뒤 helper entity와 History Stats를 통해 누적했습니다. 또한 PTZ 카메라 기반 방 청결도 평가, 일일 생활 리포트 생성, PC/WSL 원격 제어, 로컬 AI 서비스 상태 모니터링을 구성했습니다.
```

## English Summary

AI HomeOps is a sanitized public portfolio version of my Home Assistant-based smart home automation setup. It integrates occupancy sensors, camera snapshots, LLM Vision activity classification, helper entity state modeling, History Stats-based time tracking, daily LLM reports, and local PC/WSL remote operations.
