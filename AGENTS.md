# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

ZMK firmware configuration for a custom Corne split keyboard (42-key) running on nice!nano v2 controllers. The keyboard uses BLE split communication, SSD1306 OLED displays (128×32, I2C), and WS2812 RGB underglow (27 LEDs, SPI). ZMK Studio is enabled for runtime keymap editing over USB.

## Build

Firmware is built via GitHub Actions — push to `main` triggers the workflow defined in `.github/workflows/build.yml`, which delegates entirely to `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`. There is no local build toolchain in this repo.

Artifacts (`.uf2` files) land in `firmware/` and are flashed by double-tapping the reset button to enter bootloader mode, then dragging the `.uf2` onto the USB mass-storage device.

The build matrix is declared in `build.yaml`:
- `corne_left` — central half, BLE + USB, RGB on
- `corne_right` — peripheral half, BLE only, RGB off
- `settings_reset` — utility firmware to clear BLE pairing state

To temporarily add or swap a display shield, edit the `shield:` lines in `build.yaml`. The `nice_oled` shield is currently commented out pending upstream fixes.

## Repository layout

```
config/
  corne.conf          # shared Kconfig options (BLE, sleep, Studio, debounce)
  corne.keymap        # keymap (DTS syntax): all layers, behaviors, combos, macros
  west.yml            # ZMK west manifest — pins ZMK revision and lists external modules
  boards/shields/corne/
    corne.dtsi        # shared hardware: matrix transform, kscan, OLED (I2C), RGB (SPI)
    corne.conf        # shield-level display Kconfig (custom vs built-in screen)
    corne_left.overlay / corne_right.overlay  # per-side GPIO column assignments, battery
    corne_left.conf   # left-side Kconfig: OLED built-in screen, RGB on
    corne_right.conf  # right-side Kconfig: OLED built-in screen, RGB off
    Kconfig.defconfig # assigns keyboard name, enables ZMK_SPLIT + display drivers
    Kconfig.shield    # shield selection symbols
    corne.zmk.yml     # ZMK shield metadata
firmware/             # pre-built .uf2 artifacts (committed for convenience)
build.yaml            # build matrix consumed by the CI workflow
clean.sh              # GitHub Actions run-deletion script (admin utility, not build-related)
```

## Key config relationships

- `config/corne.conf` applies to both halves.
- `config/boards/shields/corne/corne.conf` applies at the shield level (display subsystem).
- `config/boards/shields/corne/corne_left.conf` and `corne_right.conf` override per side.
- The shield's `corne.dtsi` defines the physical hardware (GPIO pins, I2C, SPI). Changes to pin assignments go here.
- External ZMK modules (nice-oled, hammerbeam-slideshow, nice-view-*, etc.) are declared in `config/west.yml`. Adding a new display module requires adding a remote + project entry there.

## Keymap layers

| # | Name | Purpose |
|---|------|---------|
| 0 | default | QWERTY + homerow mods (LGUI/ALT/CTRL on A/S/D; RSHIFT/RCTRL/RALT/RGUI on J/K/L/;) |
| 1 | lower | Navigation (arrows, PgUp/Dn) + numpad |
| 2 | raise | Mouse scroll/move, symbols, F-keys |
| 3 | layer_3 | BLE profile select, RGB controls, media keys |
| 4 | gaming | Gaming layout (no homerow mods) |
| 5 | gaming_2 | Gaming numbers + arrow cluster |

Custom behaviors: `hm` (homerow mod, tap-preferred, 150 ms), `ltq` (layer-tap with quick-tap, balanced, 150 ms).

## Display status

The `nice_oled` shield (third-party, `mctechnology17/zmk-nice-oled`) is currently disabled in `build.yaml` due to upstream build issues. The built-in ZMK status screen (`CONFIG_ZMK_DISPLAY_STATUS_SCREEN_BUILT_IN=y`) is active on both sides. To re-enable `nice_oled`, un-comment the shield lines in `build.yaml` and set `CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y` / `CONFIG_ZMK_DISPLAY_STATUS_SCREEN_BUILT_IN=n` in the relevant `.conf` files.
