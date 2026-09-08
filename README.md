# zmk-sofle — Arvind's Eyelash Sofle config

ZMK config for an **Eyelash Sofle** (wireless split, 5×6+5 per half, EC11 knob on the left,
nice!view screens on both halves, per-key backlight + WS2812 underglow, nice!nano v2 / nRF52840).

See [`docs/USAGE.md`](docs/USAGE.md) for how to actually operate the thing — LEDs, screen icons,
Bluetooth profiles, soft-off, troubleshooting.

Upstream vendor config: [a741725193/zmk-sofle](https://github.com/a741725193/zmk-sofle) — kept as the
`upstream` git remote. Vendor docs are preserved in [`docs/`](docs/).

## Keymap

![Keymap](keymap-drawer/eyelash_sofle.svg)

The SVG above is regenerated automatically by [keymap-drawer](https://github.com/caksoylar/keymap-drawer)
on every push that touches `config/` (see `.github/workflows/draw.yml`).

## Layout

| | |
|---|---|
| Halves | left = **central** (talks to the host), right = peripheral |
| MCU | nice!nano v2 clone (nRF52840), 2000 mAh battery |
| Screen | nice!view (Sharp memory LCD, 160×68, 1-bit) on SPI `P0.20` SCK / `P0.17` MOSI / `P0.25` MISO |
| Encoder | EC11 on the left half, `P1.10` / `P1.14`, 40 steps |
| Underglow | 7× WS2812 on `P1.12` (SPI3) |
| Backlight | PWM on `P1.13` |
| Ext power | `P0.13` (gates the WS2812 rail) |

## Flashing

1. Push to `main` → GitHub Actions builds. Download the `firmware` artifact from the run.
2. Plug the half into USB, **double-tap the reset button** (next to the power switch) within 0.5 s.
   A `NICENANO` USB drive appears; the blue LED breathes.
3. Drag the matching `.uf2` onto the drive. The drive dismounts and the half reboots.
4. Repeat for the other half. Order does not matter.

Artifacts produced by `build.yaml`:

| File | Goes on |
|---|---|
| `eyelash_sofle_left nice_view ...uf2` | left half |
| `eyelash_sofle_right nice_view ...uf2` | right half |
| `settings_reset ...uf2` | either half — wipes BLE bonds + split pairing (see below) |

### Re-pairing the halves

Flash `settings_reset.uf2` to **both** halves, then flash the normal left/right firmware back.
They re-discover each other automatically.

## Live keymap editing (no rebuild)

The left half is built with ZMK Studio enabled (`studio-rpc-usb-uart`, locking off), so you can
re-map keys over USB without recompiling:

- **[ZMK Studio](https://zmk.studio)** — plug the **left** half in via USB, open in Chrome/Edge, connect.
- **[DYA Studio](https://studio.dya.cormoran.works/)** — the vendor's preferred fork, matching the
  `cormoran/zmk v0.3-branch+dya` firmware this repo builds against. More features than stock Studio.

Studio changes live in the keyboard's settings storage and **override** this repo's keymap until you
reset settings. Treat this repo as the source of truth and mirror anything you like back into
`config/eyelash_sofle.keymap`.

## Repo map

| Path | What it is |
|---|---|
| `config/eyelash_sofle.keymap` | **the keymap** — layers, combos, macros, encoder bindings |
| `config/eyelash_sofle.conf` | Kconfig applied to both halves (RGB, backlight, sleep, debounce) |
| `config/eyelash_sofle.json` | physical layout, used by Studio + keymap-drawer |
| `config/west.yml` | dependency manifest (ZMK fork + cormoran modules) |
| `boards/shields/eyelash_sofle/` | hardware definition: matrix, encoder, LEDs, per-half Kconfig |
| `build.yaml` | what CI builds |
| `keymap_drawer.config.yaml` | how the keymap SVG is rendered |

## Staying current with upstream

```sh
git fetch upstream
git log --oneline HEAD..upstream/main   # see what changed
git merge upstream/main
```
