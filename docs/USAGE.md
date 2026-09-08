# Using the Eyelash Sofle

Distilled from the vendor manual (`S794a4200b8414a0ca3e536b06ce397a0y.pdf`) plus what's actually
in this repo's config. Where the two disagree, this repo wins.

## Physical controls

Each half has two things on its side:

- **Slide switch** — battery ↔ MCU. **Down = on.** Must be on to charge.
  Only turn it off if the keyboard will sit unused for 6+ months (protects the LiPo).
- **Push button** — reset. **Double-tap within 0.5 s = bootloader** (USB drive appears).

## LEDs (near the USB-C port, dimly visible through the case)

| LED | State | Meaning |
|---|---|---|
| Green | solid | charging |
| Green | off | charge complete |
| Green | blinking | battery not connected, **or** power switch off |
| Blue | breathing | bootloader, USB connected — ready for a `.uf2` |
| Blue | fast blink | bootloader, no USB data connection (suspect a charge-only cable) |
| Blue | off | normal operation |

Charge time 6–8 h. Battery is 2000 mAh (505060), PH2.0 connector — mind the polarity if you replace it.

## Screen (nice!view)

Both halves have one. The **left** half is the central and shows real status:

| Element | Meaning |
|---|---|
| Battery bar + % | left half's own battery |
| **WiFi arcs glyph** | **Bluetooth connected.** There is no WiFi in this keyboard — see below |
| ✕ glyph | BLE profile is bonded but currently disconnected |
| Gear glyph | BLE profile is unbonded (nothing paired to this slot) |
| USB glyph | output is going over USB, not Bluetooth |
| Row of 5 circles | BLE profiles 1–5. Filled = active, outlined = connected, dashed = bonded only |
| WPM line graph | words per minute, last 10 samples |
| Bottom text | current layer's `display-name` (`LAYER0`…`LAYER4`) |

> **Why it says "WiFi":** ZMK's stock nice!view widget uses LVGL's `LV_SYMBOL_WIFI` glyph
> to mean "BLE bonded and connected" — purely a font choice in
> [`app/boards/shields/nice_view/widgets/status.c`](https://github.com/zmkfirmware/zmk/blob/v0.3-branch/app/boards/shields/nice_view/widgets/status.c).
> The MCU is an nRF52840: Bluetooth LE only, no WiFi radio, no WiFi stack compiled in.
> Changing the glyph means forking the nice_view shield into a module.

## Connecting

The **left half is the keyboard** as far as your computer is concerned. The right half only talks
to the left over BLE; its USB-C port is for charging and flashing only.

1. Turn both halves on. They pair with each other automatically.
2. On the host, pair with the device named **Eyelash Sofle**.
3. Five independent host profiles are available — `&bt BT_SEL 0` … `&bt BT_SEL 4` on layer 2.
   Each holds one host, so you can hop between laptop / phone / work machine.

**If typing stops working:** you probably nudged a `BT_SEL` key and switched to an empty profile.
Switch back, or check which circle is filled on the screen.

**If BLE misbehaves:** `&bt BT_CLR_ALL` (layer 2, `W` position) wipes every profile, then re-pair.

**Plugging in USB** silently steals the output — keycodes go to USB, not Bluetooth. Force it back
with `&out OUT_BLE` (layer 2, `S` position).

## Split pairing reset

Only if the halves refuse to talk to each other:

1. Flash `settings_reset-nice_nano_v2-zmk.uf2` to **both** halves.
2. Flash the normal left/right firmware back. Order doesn't matter — they re-pair on boot.

Normal firmware updates do **not** clear the split pairing.

## Power notes

- The WS2812 underglow draws current even when "off", so this config gates its power rail
  (`CONFIG_ZMK_RGB_UNDERGLOW_EXT_POWER=y`, `EP_OFF` on `P0.13`). **Do not press `&ext_power EP_ON`
  and leave it** — that re-energises the rail and quietly eats your battery.
- Underglow is off at boot, auto-off on idle and on USB. Backlight is on at boot, brightness 100.
- Deep sleep after 1 hour idle.
- **Soft-off:** hold key positions 14 / 28 / 40 for 2 s (`Q` + `S` + `Z`) → deep sleep that key
  presses won't wake. Press the reset button once to wake. Good for travel.
- The left half always dies first. It's the central; that's expected.

## Layers as built

| Layer | How to reach | What's on it |
|---|---|---|
| 0 | default | QWERTY, arrows in the centre column, encoder = volume |
| 1 | hold `&mo 1` (left thumb) | F-keys, mouse keys + mouse move, nav cluster, RGB controls |
| 2 | hold `&mo 2` (right thumb) | Bluetooth profiles, USB/BLE output, `sys_reset`, `bootloader`, `soft_off` |
| 3, 4 | not bound | empty, yours to claim |

The **encoder** (left knob) is volume up/down on every layer; pressing it sends `C_MUTE`.

See the [rendered keymap](../keymap-drawer/eyelash_sofle.svg) for the exact positions.

## Troubleshooting quick table

| Symptom | Cause |
|---|---|
| Totally dead | power switch not down |
| Won't charge, green LED blinks forever | switch off, or the switch itself is dead (MKS12C02) |
| Connected but nothing types | wrong BLE profile, or USB stole the output |
| One key dead | switch pin bent instead of seated in the hot-swap socket |
| Right half not responding | it has no independent mode — check the left half is on and paired |
| Laggy / dropping | 2.4 GHz interference. Halves need to be within ~0.6–0.8 m. Move WiFi to 5 GHz |
| New keycode won't compile | it needs its `#include` header in the keymap |
