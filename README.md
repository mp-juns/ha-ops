# AI HomeOps: Context-Aware Smart Home Automation

AI HomeOps is a Home Assistant-based personal automation lab that integrates IoT devices, occupancy sensors, camera snapshots, LLM Vision analysis, PC/WSL remote control, service monitoring, and daily lifestyle reporting.

This repository contains a **sanitized public template** of my Home Assistant automation setup. Sensitive values such as API keys, MAC addresses, internal IPs, SSH usernames, camera entity names, and provider IDs are replaced with placeholders or `!secret` references.

## Why this project exists

Most smart home automations are rule-based:

> If motion is detected, turn on a light.

AI HomeOps extends that idea into context-aware automation:

> If a person stays in a room, capture a snapshot, classify the activity with LLM Vision, store the result as structured Home Assistant state, accumulate activity statistics, and generate a daily report.

## Key Features

- Home Assistant-based smart home automation
- Occupancy-triggered room activity detection
- LLM Vision-based living room and kitchen activity classification
- Multi-angle room cleanliness scoring using camera presets
- Daily activity summary generation through an LLM API
- Wake-on-LAN PC power control
- Windows/WSL remote control through SSH shell commands
- Service status monitoring for a local AI/security stack
- Helper entity-based state modeling
- History Stats-based daily time tracking
- Privacy-first public configuration structure

## System Overview

```text
Occupancy Sensor
      ↓
Camera Snapshot
      ↓
LLM Vision Analysis
      ↓
JSON Parsing
      ↓
Home Assistant Helper Entities
      ↓
History Stats / Counters / Notifications
      ↓
Daily LLM Report + Calendar Log
```

## Repository Structure

```text
ha-ops/
├── README.md
├── automations/
│   ├── living_room_activity_detection.yaml
│   ├── kitchen_activity_detection.yaml
│   ├── daily_activity_report.yaml
│   └── light_helper_control.yaml
├── scripts/
│   ├── pc_power_control.yaml
│   └── room_cleanliness_scoring.yaml
├── packages/
│   └── activity_tracking_helpers.yaml
├── config/
│   └── configuration.example.yaml
├── examples/
│   ├── secrets.example.yaml
│   └── sanitized_entities.md
├── docs/
│   ├── architecture.md
│   ├── privacy.md
│   └── resume-description.md
└── .gitignore
```

## Main Automations

### 1. Living Room Activity Detection

Detects whether a person is present in the living room. When presence is detected for a configured duration, Home Assistant captures a camera snapshot and sends it to an LLM Vision analyzer.

The model returns structured JSON such as:

```json
{
  "location": "desk",
  "activity": "computer_work",
  "detail": "책상에서 작업 중",
  "detail_long": ""
}
```

The parsed result is stored in Home Assistant helper entities:

- `input_text.current_location`
- `input_text.current_activity`
- `input_text.current_activity_detail`

### 2. Kitchen Activity Detection

Detects kitchen presence and classifies activities such as cooking, eating, drinking, laundry, cleaning, passing, and other. The result is stored as a recent activity log and daily/weekly counters.

### 3. Room Cleanliness Scoring

Moves a PTZ camera across multiple preset positions, captures multiple snapshots, and asks LLM Vision to score each area from 1 to 10. The automation calculates average cleanliness score, worst area score, and a short reason for each camera angle.

### 4. Daily Activity Report

At the end of the day, Home Assistant collects location time, activity time, kitchen activity counts, room cleanliness score, and latest detected activity. Then an LLM generates a short daily report and stores it in a Home Assistant notification, `input_text.daily_report`, and a calendar event.

### 5. PC / WSL Remote Control

Home Assistant can control a local development machine through Wake-on-LAN, SSH-based WSL shutdown, shell command-based hibernation, and optional web terminal startup.

## Security Notice

This repository is intentionally sanitized.

Do **not** commit:

- `secrets.yaml`
- API keys
- real MAC addresses
- internal IP addresses
- SSH private keys
- camera RTSP URLs
- `.storage/`
- database files
- logs
- SSL certificates
- access tokens

Use `examples/secrets.example.yaml` as a template.

## Portfolio Summary

AI HomeOps demonstrates practical experience with IoT automation, event-driven system design, Home Assistant configuration, LLM Vision integration, structured state modeling, local infrastructure control, privacy-aware public documentation, and real-world personal automation operations.
