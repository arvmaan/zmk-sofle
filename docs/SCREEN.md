# Customising the nice!view screens

## What's actually on each screen

The two halves run completely different display code.

| | Left half (central) | Right half (peripheral) |
|---|---|---|
| Source | `widgets/status.c` | `widgets/peripheral_status.c` + `widgets/art.c` |
| Shows | battery, BLE profile circles, output glyph, WPM graph, layer name | **art image (140×68)** + battery + connection glyph |
| Art? | none — it's all live status | yes, this is the one with the picture |

So **"custom art" means the right-hand screen.** On boot it picks `balloon` or `mountain` at
random (`sys_rand32_get() & 1` in `peripheral_status.c`), which is why it seems to change on its own.

## How this repo is set up

`boards/shields/nice_view_custom/` is a copy of the nice!view shield from the ZMK fork this repo
builds against, with every Kconfig symbol namespaced (`NICE_VIEW_CUSTOM_*`) so it can never collide
with the stock shield. `build.yaml` builds **both**, so each CI run gives you:

| Artifact | Use |
|---|---|
| `eyelash_sofle_left nice_view-…uf2` | stock left — **fallback** |
| `eyelash_sofle_right nice_view-…uf2` | stock right — **fallback** |
| `eyelash_sofle_left nice_view_custom-…uf2` | custom left |
| `eyelash_sofle_right nice_view_custom-…uf2` | custom right |
| `settings_reset-…uf2` | wipe BLE + split pairing |

If a custom build ever looks wrong, drag the stock `.uf2` back on. Nothing to recover from.

## Image format

The screen is a 1-bit Sharp memory LCD. Art is stored as an LVGL `LV_IMG_CF_INDEXED_1BIT` array:

- **140 × 68 pixels**, hard black and white — no grey, no anti-aliasing
- 18 bytes per row (140 bits padded to a byte boundary) × 68 rows = 1224 bytes, plus an
  8-byte palette = the `data_size = 1232` you'll see in `art.c`
- bit set = palette index 1 = white (in a non-inverted build)

High-contrast line art, silhouettes and chunky logos work. Photos and fine gradients turn to mush
unless you dither.

## Swapping in your own art

One-time setup:

```sh
python3 -m venv tools/.venv && tools/.venv/bin/pip install Pillow
```

Drop your images in `art/`, then:

```sh
tools/.venv/bin/python tools/img2art.py \
    boards/shields/nice_view_custom/widgets/art.c \
    balloon=art/one.png mountain=art/two.png
```

The names `balloon` and `mountain` are what `peripheral_status.c` declares via `LV_IMG_DECLARE()`.
Keep them, or rename in both files together. Two images = the screen alternates on boot; point both
names at the same file if you want one fixed image.

Useful flags:

| Flag | When |
|---|---|
| `--dither` | photos and anything with shading — Floyd-Steinberg instead of a hard cut |
| `--threshold N` | tune the black/white cutoff, 0–255, default 128 |
| `--invert` | your art came out as a negative |
| `--fit cover` | crop to fill 140×68 instead of letterboxing |
| `--fit stretch` | squash to fit, ignoring aspect ratio |

### Preview before you flash

```sh
python3 tools/art2png.py boards/shields/nice_view_custom/widgets/art.c art/preview/
```

Writes exactly what the screen will show, pixel for pixel. Look at it before committing — a
five-second check beats a five-minute build-and-flash cycle. `tools/art2png.py` is pure stdlib,
no venv required.

> The round trip is lossless: decoding ZMK's own `art.c` to PNG and re-encoding it reproduces the
> original bytes exactly, so the preview is not an approximation.

## Other things you could change later

Not needed for art, but this is the menu once the shield is yours:

- **The "WiFi" glyph.** In `widgets/status.c` and `widgets/peripheral_status.c`, `LV_SYMBOL_WIFI`
  means *BLE connected*. Swap it for `LV_SYMBOL_BLUETOOTH` and it stops lying.
- **Static art instead of random.** Delete the `sys_rand32_get() & 1` pick in
  `peripheral_status.c` to always show one image.
- **Animation.** `lv_animimg` can cycle frames. Costs battery — the vendor removed the original
  animations for exactly that reason.
- **Drop the WPM graph** on the left for something else — layer name in a bigger font, a clock,
  a custom logo.
- **Invert the whole display** with `CONFIG_NICE_VIEW_CUSTOM_WIDGET_INVERTED=y`.
