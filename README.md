# EatTechDad · Home Assistant

Blueprints, scripts, and config I actually run in my own house. Everything here
has a full write-up on [eattechdad.com](https://eattechdad.com).

Nothing in this repo is theoretical. If it's here, it's because I needed it,
built it, and it's still running.

---

## Blueprints

| Blueprint | What it does | Write-up |
|---|---|---|
| [LoraTap SS6400ZB](blueprints/automation/loratap_ss6400zb/) | Maps all 8 actions on a 4-button Zigbee remote (single + double press) to whatever you want | *coming soon* |

---

## Scripts

| Script | What it does | Write-up |
|---|---|---|
| *OpenSprinkler zone control* | Toggle individual OpenSprinkler zones from Home Assistant | *coming soon* |

---

## Installing a blueprint

Each blueprint folder has a one-click import button. Or do it manually:

1. In Home Assistant, go to **Settings → Automations & Scenes → Blueprints**
2. Click **Import Blueprint**
3. Paste the raw URL of the blueprint YAML
4. Click **Preview**, then **Import**

Once imported, blueprints update in place. When I push a fix, open the
blueprint and choose **Re-import blueprint** to pull it down.

---

## Contributing

Found a bug, or got a device that almost works with one of these? Open an
issue. If you've already fixed it, open a PR. I'd rather merge your fix than
buy hardware I don't need.

---

## License

MIT. Use it, fork it, ship it. Attribution appreciated but not required.
