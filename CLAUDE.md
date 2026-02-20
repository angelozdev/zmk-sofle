# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZMK firmware for the **Eyelash Sofle** — a wireless split ergonomic keyboard (6x4 + 5-key extra panel) based on the nRF52840 MCU. Forked from `a741725193/zmk-sofle`, customized by `angelozdev`.

## Build System

This project builds via **GitHub Actions only** — there is no local build. Pushing to `main` triggers the workflows:

- **`build.yml`** — Compiles UF2 firmware for all targets (left, right, studio-left, settings-reset). Triggered on push (ignores `keymap-drawer/` changes).
- **`draw.yml`** — Generates SVG keymap visualizations via [keymap-drawer](https://github.com/caksoylar/keymap-drawer). Triggered on changes to `config/`, `keymap_drawer.config.yaml`, or the workflow itself. Auto-commits results to `keymap-drawer/`.

Build targets are defined in `build.yaml`:
| Target | Board | Notes |
|--------|-------|-------|
| Right half | `eyelash_sofle_right` | Nice!View display |
| Left half | `eyelash_sofle_left` | Nice!View, wireless central |
| Studio left | `eyelash_sofle_left` + `studio-rpc-usb-uart` | ZMK Studio enabled |
| Settings reset | `nice_nano_v2` + `settings_reset` | Factory reset utility |

Download firmware artifacts from the GitHub Actions build page.

## Architecture

### Key Files to Edit

| File                          | Purpose                                                                                      |
| ----------------------------- | -------------------------------------------------------------------------------------------- |
| `config/eyelash_sofle.keymap` | **Keymap definition** — layers, combos, behaviors, encoders (DTS format with C preprocessor) |
| `config/eyelash_sofle.conf`   | **Firmware features** — Kconfig toggles for BLE, RGB, sleep, backlight, debounce, etc.       |
| `keymap_drawer.config.yaml`   | **Visualization config** — styling, glyph mappings, custom key label rendering               |
| `build.yaml`                  | **Build matrix** — which board/shield/snippet combinations to compile                        |

### Board Definitions (`boards/arm/eyelash_sofle/`)

Hardware-level device tree files. Rarely need changes unless modifying pin assignments or adding peripherals:

- `eyelash_sofle.dtsi` — Shared: GPIO matrix, LED strip (WS2812 SPI), PWM backlight, battery ADC, display SPI
- `eyelash_sofle_left.dts` / `_right.dts` — Half-specific: encoder pins, column GPIOs, col-offset
- `eyelash_sofle-layouts.dtsi` — Physical key positions (x/y/rotation) for visualization tools
- `Kconfig.defconfig` — Default Kconfig for DCDC, display, USB, BLE split config
- `*_defconfig` — Per-half defaults (left has USB+display+sleep; right has no USB/display)

### Dependency Chain (`config/west.yml`)

```
Local boards/ → eyelash_sofle module (a741725193/zmk-sofle@main) → ZMK v0.3.0 → Zephyr RTOS
```

`zephyr/module.yml` registers `boards/` as a custom board root.

## Keymap Structure

Four layers defined in `config/eyelash_sofle.keymap`:

| #   | Name   | Purpose                                                             |
| --- | ------ | ------------------------------------------------------------------- |
| 0   | `BASE` | QWERTY + arrow keys on extra panel                                  |
| 1   | `NAV`  | F-keys, Vim arrows (HJKL), mouse buttons, word navigation, media    |
| 2   | `SYM`  | Sticky modifiers (left), numpad (right), brackets/symbols           |
| 3   | `ADJ`  | Bluetooth profiles, RGB control, system reset, bootloader, soft-off |

Layer access: `&sl NAV` and `&sl SYM` as sticky layers on thumb keys; `ADJ` via `&mo ADJ` on both thumbs simultaneously.

### Combos (50ms timeout, BASE layer)

- `J + K` → Escape
- `O + P` → Backspace
- `K + L` → Enter

### Notable Behaviors

- **Sticky keys** (`&sk`): 2s release, quick-release, chainable modifiers (used in SYM layer)
- **Sticky layers** (`&sl`): 1s activation window
- **Caps Word**: Continues through underscore, hyphen, backspace, delete
- **Soft Off**: Hold Q+S+Z for 2 seconds → deep sleep (wake via reset button only)
- **Mouse**: Linear acceleration, 100ms scroll ramp, 500ms movement smoothing

## Conventions

- **Commit messages**: Conventional Commits (`feat`, `style`, `fix`, etc.) with `(keymap)` or `(config)` scope
- **Visualization commits**: Auto-generated with prefix `[Draw] Updated keymap visualization`
- **Language**: READMEs are bilingual (Chinese `README.md`, English `README_EN.md`)
- The `keymap-drawer/` directory is auto-generated — do not edit SVG/YAML files there manually

## Common Workflow

1. Edit `config/eyelash_sofle.keymap` (and/or `.conf`)
2. Commit and push to `main`
3. GitHub Actions builds firmware and generates updated keymap visualization
4. Download UF2 artifacts from Actions, flash via USB mass storage (double-tap reset on each half)
