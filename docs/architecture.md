# Architecture

AI HomeOps is structured around an event-driven Home Assistant pipeline.

## Pipeline

```text
Sensor Event
  -> Automation Trigger
  -> Camera Snapshot
  -> LLM Vision Classification
  -> JSON Parsing
  -> Helper Entity State Update
  -> History Stats Accumulation
  -> Notification / Daily Report / Calendar Event
```

## Design Choices

### Helper entities as state storage

LLM output is converted into Home Assistant helper entities such as:

- `input_text.current_location`
- `input_text.current_activity`
- `input_number.cooking_today`

This makes model output queryable by other automations, dashboards, and history sensors.

### History Stats for time accumulation

Template binary sensors represent derived states such as:

- currently at desk
- currently sleeping
- currently working

`history_stats` sensors then accumulate time spent in each state.

### Sanitized public structure

The real Home Assistant setup contains private values. This repository uses placeholder entities and `!secret` references to keep the public version safe.
