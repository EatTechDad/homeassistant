# LoraTap SS6400ZB · 4-Button Zigbee Remote Zigbee2MQTT ONLY 

Maps all eight actions on a LoraTap SS6400ZB (single and double press on four
buttons) to whatever you want. Paired via Zigbee2MQTT, no Tuya hub required. Blueprint will not work with ZHA. 

[![Open your Home Assistant instance and show the blueprint import dialog with this blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2FEatTechDad%2Fhomeassistant%2Fmain%2Fblueprints%2Fautomation%2Floratap_ss6400zb%2Floratap_ss6400zb.yaml)
---

## Read this first, or the blueprint won't work

**Press every button before you create the automation.**

Zigbee2MQTT only publishes a device trigger *after* it has seen that exact
action at least once. Fresh out of the box, your remote has zero triggers
registered, and the blueprint will fail to resolve them.

So before anything else: press each of the four buttons once, then
double-press each of the four buttons once. Eight presses total. Now the
triggers exist.

You can verify in **Settings → Devices & Services → MQTT → your device →
MQTT Info**. Under **Triggers** you should see eight entries ending in
`action_1_single` through `action_4_double`.

---

## Two more things that cost me an evening

**Don't use hyphens in the device name.** I originally named mine
`Zigbee-Sprinkler-Remote` and Home Assistant refused to list the triggers in
the automation editor's device picker, even though the discovery messages were
correctly retained on the broker and the triggers fired fine when referenced by
`device_id`. Renaming to underscores fixed it. Use
`Zigbee_Sprinkler_Remote`, not `Zigbee-Sprinkler-Remote`.

**Renaming in Z2M changes the MQTT topic.** If you have existing automations
pointed at `zigbee2mqtt/Old-Name/action`, they'll go silent. Check with the
MQTT listen panel and a wildcard: `zigbee2mqtt/+/action`

---

## Where did my `sensor.*_action` entity go?

It's gone on purpose. Zigbee2MQTT 2.0 removed the legacy action sensors as
part of a cleanup of features that had been deprecated for years. The action
data still flows, it's just delivered as MQTT device triggers now, which is
what this blueprint uses.

If you have an older blueprint or automation that needs the sensor back, you
can re-enable it in Z2M under **Settings → Home Assistant integration →
Home Assistant legacy action sensors**, or in `configuration.yaml`:

```yaml
homeassistant:
  legacy_action_sensor: true
```

It's deprecated and will eventually disappear, so treat that as a bridge, not
a destination.

---

## Supported actions

| Button | Single | Double |
|---|---|---|
| 1 | `1_single` | `1_double` |
| 2 | `2_single` | `2_double` |
| 3 | `3_single` | `3_double` |
| 4 | `4_single` | `4_double` |

Hold gestures aren't wired into the blueprint. If your unit reports them,
open an issue and I'll add them.

---

## Prefer raw YAML?

If the device picker gives you trouble, you can skip the blueprint entirely
and trigger straight off the MQTT topic. This doesn't depend on device trigger
discovery at all:

```yaml
triggers:
  - trigger: mqtt
    topic: zigbee2mqtt/Zigbee_Sprinkler_Remote/action
conditions:
  - condition: template
    value_template: "{{ trigger.payload | trim != '' }}"
actions:
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ trigger.payload == '1_single' }}"
        sequence:
          - action: switch.turn_on
            target:
              entity_id: switch.sprinkler_zone_1
```

The condition filters out the empty payload Zigbee2MQTT publishes right after
each action to reset state. Without it, every press fires twice.

---

## Hardware

The SS6400ZB listing claims it requires a Tuya hub. It does not. It pairs
with Zigbee2MQTT like any other Zigbee 3.0 device.

I bought mine on AliExpress. Full write-up and Amazon-available alternatives
on [eattechdad.com](https://eattechdad.com).
