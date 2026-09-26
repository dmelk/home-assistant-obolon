# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Configuration for a personal Home Assistant install. It is a collection of config files copied into or pasted into a running HA instance, not an application: there is no build, lint, or test suite. Checking a change means validating it in Home Assistant / ESPHome.

## Layout and how pieces connect

- **Lovelace dashboards** (`*_dashboard.yml` at repo root) are raw-config YAML for the HA dashboard editor: `wallscreen` (wall tablet), `mobile` (phone), `childroom`. They rely heavily on HACS frontend cards: `custom:button-card` (with shared `button_card_templates` at the top of each file and JS templates in `[[[ ... ]]]`), `custom:floorplan-card`, `custom:xiaomi-vacuum-map-card`, `custom:more-info-card`, `custom:clock-weather-card`.
- **`floorplan/`** holds assets served by HA from `/config/www/floorplan/` and referenced as `/local/floorplan/...`. `wallscreen.svg` + `wallscreen.css` drive the floorplan card in `wallscreen_dashboard.yml`. That card's rules call `floorplan.class_set` / `style_set` / `text_set` on SVG element IDs, so renaming an element in the SVG breaks the dashboard rules that target it (and the other way round). The `vacuum-plan.*` files serve the vacuum view.
- **Vacuum segments**: `vacuum_segments.txt` maps rooms to Roborock segment IDs (hallway 19, bedroom 17, living_room 16, kitchen 18). `mobile_dashboard.yml` hardcodes these IDs in `roborock.vacuum_clean_segment` calls and in the JS that builds the segment list from `input_boolean.vacuum_<room>` helpers. Keep all three in sync.
- **`esphome/`** has one ESPHome device config per file. Wall switches are wired locally on the device (e.g. `hallway-controller.yaml`: PCF8574 I2C expanders, `pcf8574_hub_out` relays at 0x24 and `pcf8574_hub_in` switch inputs at 0x22, with `on_click` → `switch.toggle`), so lights keep working without HA. Wi-Fi credentials come from `!secret` in `esphome/secrets.yaml`, which is gitignored and must never be committed. Compiled `*.bin` firmware is also gitignored. `intercom.yaml` is adapted from https://github.com/Anonym-tsk/smart-domofon/blob/master/esphome/domofon.yaml.
- **`custom_zha_quirks/`** contains a zigpy/zhaquirks `CustomDevice` quirk for a Tuya TS0601 air-quality sensor. It goes in HA's ZHA custom quirks path.

## Validating ESPHome changes

With the ESPHome CLI installed (`pip install esphome`), run from `esphome/`:

```sh
esphome config <device>.yaml    # validate
esphome compile <device>.yaml   # build firmware
esphome run <device>.yaml       # build + flash (OTA or serial)
```

These commands need `esphome/secrets.yaml` to exist and define `wifi_ssid`, `wifi_password`, etc.

## Conventions

Commit messages use the form `[feature] <description>`.
