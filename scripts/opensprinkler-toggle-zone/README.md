# opensprinkler-toggle-zone

A Home Assistant script that adds toggle behavior to [OpenSprinkler](https://www.home-assistant.io/integrations/opensprinkler/), which doesn't natively support it. Pass in a zone switch and its status sensor, and the script starts or stops that zone based on its current state.

Detailed writeup of my OpenSprinkler setup: [Eat+Tech+Dad](https://www.eattechdad.com/yd52)

## How it works

- If the zone is idle, it starts the zone using a configurable duration.
- If the zone is already running, it stops it.

This makes it easy to wire up a single tap action (dashboard icon, physical button, etc.) that toggles any zone on or off, instead of needing separate "start" and "stop" actions.

Because it uses `queue_option: append`, calling the toggle on multiple zones doesn't run them simultaneously, each new zone gets added to OpenSprinkler's run queue and starts once the previous one finishes. So tapping several zone icons in a row queues them up to run back-to-back instead of overlapping or fighting over water pressure.

## Requirements

- Home Assistant with the [OpenSprinkler integration](https://www.home-assistant.io/integrations/opensprinkler/) configured
- An `input_number` helper for irrigation duration (in minutes), e.g. `input_number.irrigation_duration`

## Script

See [`scripts/toggle_irrigation_zone.yaml`](scripts/toggle_irrigation_zone.yaml) for the full script.

At a glance, it takes two inputs (`zone_switch`, `zone_sensor`), checks the sensor state, and either runs the zone (queued) or stops it.

## Installation

1. Copy `scripts/toggle_irrigation_zone.yaml` into your Home Assistant `scripts.yaml`, or add it via **Settings → Automations & Scenes → Scripts → Add Script → Edit in YAML**.
2. Create the duration helper if you don't already have one: **Settings → Devices & Services → Helpers → Number**, and set the entity ID to `input_number.irrigation_duration`.
3. Reload scripts, or restart Home Assistant.

## Usage

Call the script from any dashboard tap action, automation, or the Developer Tools **Actions** panel, passing your zone's switch and sensor entities.

### Example: dashboard icon tap action

```yaml
icon_tap_action:
  action: perform-action
  perform_action: script.toggle_irrigation_zone
  target: {}
  data:
    zone_switch: switch.opensprinkler_zone_backyard
    zone_sensor: sensor.opensprinkler_s02_station_status
```

## Inputs

| Field | Type | Description |
|---|---|---|
| `zone_switch` | entity (switch) | The OpenSprinkler zone switch to start/stop |
| `zone_sensor` | entity (sensor) | The status sensor for that zone, used to determine idle vs. running |

## Notes

- **Queueing:** `queue_option: append` means zones don't overlap. If a zone is already running when you toggle another one on, the new zone queues up behind it instead of starting immediately.
- All zones currently share the same duration via `input_number.irrigation_duration`. To support per-zone durations, add a `duration` field to the script and reference it in `run_seconds` instead of the shared helper.
- The entity selectors are scoped to the OpenSprinkler integration, so only relevant switches/sensors show up when calling the script from the UI.

## License

MIT
