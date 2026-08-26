# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZMK firmware configuration for a Lily58 split keyboard with nice!nano v2 controllers, connected via Bluetooth.

## Build

Firmware is built via GitHub Actions — push to `master` triggers the workflow (`.github/workflows/build.yml`), which calls ZMK's reusable `build-user-config.yml`. Build matrix is defined in `build.yaml` and produces firmware for three targets:
- `nice_nano_v2` + `lily58_left`
- `nice_nano_v2` + `lily58_right`
- `nice_nano_v2` + `settings_reset`

ZMK is pinned to release `v0.3.0` in two coupled places that must always match: the firmware source (`config/west.yml` → `revision`) and the reusable workflow (`.github/workflows/build.yml` → `build-user-config.yml@<ref>`). Newer ZMK (Zephyr 4.1+) renames the board to the `nice_nano//zmk` variant — bumping the pin requires updating `board:` in `build.yaml` too.

There is no local build setup. Download firmware artifacts from the GitHub Actions run.

## Key Files

- `config/lily58.keymap` — keymap definition (layers, combos, behaviors)
- `config/lily58.conf` — board feature toggles (OLED, encoder)
- `config/west.yml` — ZMK west manifest (pins ZMK revision)
- `build.yaml` — GitHub Actions build matrix

## Keymap Architecture

6 layers, accessed via `mo` (momentary), `lt` (layer-tap), and custom hold-tap behaviors:

| # | Name | Purpose | Activation |
|---|------|---------|------------|
| 0 | default_layer | Mac layout, home-row `mo 2`, arrows on right thumb cluster | Default |
| 1 | win_layer | Windows variant — swaps GUI/Alt positions | `to 1` from layer 4 |
| 2 | fn2_layer | Navigation (WASD arrows), symbols row, media, delete | `mo 2` (left home-row hold) |
| 3 | fn3_layer | Numpad (left), F-keys (right) | `lt 3 SPACE` / `lt 3 ENTER` |
| 4 | fn4_layer | BT profile select/disconnect, BT clear, layer switch (`to 0`/`to 1`) | `mo 4` from layers 2/3 |
| 5 | caps_cancel_layer | Internal transparent layer used to cancel one-shot Caps on a second tap | Activated together with one-shot Caps |

Notable custom behaviors:
- **hrm** — home-row mod (hold-tap, tap-preferred, 200ms tapping-term, 125ms prior-idle)
- **ss_hs_modmorph** — sends `M` normally, `]` with shift
- **kh_hs_tapdance** — tap `[`, double-tap `]`
- **fn2_caps / caps_oneshot** — on the Mac layer, hold for layer 2; tap holds Caps through the next key; tap again to cancel
- **combo_esc** — keys 0+1 → ESC (50ms timeout)
- Shift keys double as parentheses via `mt` (mod-tap)

## ZMK Conventions

- Keymap uses devicetree syntax (`.dtsi` includes, `compatible` properties)
- Behaviors are defined under `/ { behaviors { ... } }` node
- Combos under `/ { combos { ... } }` node
- Key positions are 0-indexed left-to-right, top-to-bottom across both halves
- `&kp` = key press, `&mo` = momentary layer, `&lt` = layer-tap, `&mt` = mod-tap, `&bt` = bluetooth, `&trans` = transparent (fall through)
