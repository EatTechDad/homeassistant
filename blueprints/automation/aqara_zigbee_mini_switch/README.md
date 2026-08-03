# Aqara WXKG11LM · Wireless Mini Switch (Zigbee2MQTT)
Maps single, double, hold, and release on an Aqara WXKG11LM to whatever you
want. Paired via Zigbee2MQTT, no Aqara hub required. Uses MQTT device
triggers, so it will not work with ZHA.

[![Open your Home Assistant instance and show the blueprint import dialog with this blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2FEatTechDad%2Fhomeassistant%2Fmain%2Fblueprints%2Fautomation%2Faqara_zigbee_mini_switch%2Faqara_WXKG11LM.yaml)

---

## Credit

This started as a fork of [this community blueprint](https://community.home-assistant.io/t/zigbee2mqtt-aqara-wireless-mini-switch-wxkg11lm/810035/3),
which used an entity selector pointed at an `event.*` entity and read the
action off `trigger.to_state.attributes.event_type`. I ran into enough
problems getting that version working that I ended up rebuilding it from the
trigger up. Details below, in case you hit the same walls.

---

## Read this first, or the blueprint won't work

**Press every action before you create the automation.**
Zigbee2MQTT only publishes a device trigger *after* it has seen that exact
action at least once. Single click will likely already be registered since
you've probably used the switch, but double, hold, and release won't show up
in the device trigger picker until you've actually done each of them once.

Press the button single, double, hold, and release, once each. Now the
triggers exist and will show up in the automation editor.

---

## Why the original version didn't work for me

**The entity selector was empty no matter what I picked.**
The original blueprint filtered its entity selector with flat `domain:` /
`integration:` keys directly under `selector: entity:`. That's deprecated
syntax. Current Home Assistant wants filters nested under a `filter:` list:

```yaml
selector:
  entity:
    filter:
      - integration: mqtt
        domain: event
```

With the old flat syntax the picker just came up empty, which looked like a
missing entity rather than a selector bug.

**Even after fixing the selector, `event.*` entities didn't exist.**
Zigbee2MQTT doesn't create `event` domain entities unless you turn on
"Home Assistant experimental event entities" in the Z2M integration settings.
Without that, the domain: event filter will always return nothing, fixed
syntax or not.

**And even with an event entity, the logic depended on the right domain.**
The blueprint read `trigger.to_state.attributes.event_type`, which only
exists on `event` domain entities. Point that same code at a `sensor`
entity (the legacy `sensor.*_action` entity) and `command` is always `None`,
so no branch in the `choose` block ever matches. Nothing fires, with no
error to point at why.

**The fix I landed on: skip entities entirely, use MQTT device triggers.**
Device triggers (`type: action`, `subtype: single` / `double` / `hold` /
`release`) don't depend on an entity existing at all, don't require the
experimental event entities toggle, and don't require matching an attribute
that may or may not be present depending on which domain you picked. They're
also what Zigbee2MQTT is moving toward as it phases out the legacy action
sensor in Z2M 2.0. This blueprint uses `trigger.id` to branch instead of
reading state or attributes:

```yaml
triggers:
  - trigger: device
    device_id: !input switch
    domain: mqtt
    type: action
    subtype: single
    id: single
```

---

## Supported actions

| Action | Supported |
|---|---|
| Single | Yes |
| Double | Yes |
| Hold | Yes, hardware-dependent |
| Release | Yes, hardware-dependent |
| Triple / Quadruple | Not wired into this blueprint yet |

Not every WXKG11LM hardware revision reports triple, quadruple, hold, or
release. Check what your specific unit actually emits with the device
trigger dropdown (after pressing each gesture once) or by listening to
`zigbee2mqtt/<friendly_name>` in **Settings → Devices & Services → MQTT →
Listen to a topic**. If your unit supports triple or quadruple and you want
it added, open an issue and I'll wire it in.

---

## Hardware

Paired directly with Zigbee2MQTT, no Aqara hub or Aqara Home app needed.
