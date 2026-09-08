# Editing your keymap

Everything lives in one file: [`config/eyelash_sofle.keymap`](../config/eyelash_sofle.keymap).

## The one thing to know first

The vendor config is written for the **knob + centre-cluster** variant of this board. Yours has
neither. Five of the 64 positions are switches that **do not physically exist** on your keyboard:

```
row 0:  0   1   2   3   4   5   6X  7   8   9  10  11  12
row 1: 13  14  15  16  17  18  19X 20  21  22  23  24  25
row 2: 26  27  28  29  30  31  32X 33  34  35  36  37  38
row 3: 39  40  41  42  43  44  45X 46  47  48  49  50  51
row 4: 52  53  54  55  56  57  58X 59  60  61  62  63
        └──── left half ────┘  ↑   └──── right half ────┘
                            phantom
```

> **The 7th entry in every row is a phantom key.** It still needs a binding — the list must stay
> 64 long or the build breaks — but whatever you put there will never fire. Use `&none` so it's
> obvious.

Right now those five hold `UP` / `DOWN` / `LEFT` / `RIGHT` / `ENTER`, which is why **you have no
arrow keys on layer 0**. Rehoming those is the single highest-value change you can make.

**Also worth testing:** position 52 (`row 4`, first entry, currently `&kp C_MUTE`) is the knob's
push-switch on the knob variant. On yours it's either a normal key or nothing. Bind it to something
distinctive, flash, and see if it types — that tells you whether you get a free key.

## Behaviours you'll actually use

| Binding | Does |
|---|---|
| `&kp A` | taps A. Any [keycode](https://zmk.dev/docs/keymaps/list-of-keycodes) |
| `&kp LG(C)` | ⌘C — wrap with `LG()` `LC()` `LA()` `LS()` for gui/ctrl/alt/shift |
| `&mo 1` | layer 1 **while held** |
| `&to 1` | switch to layer 1 and **stay** |
| `&lt 2 SPACE` | layer 2 held, SPACE tapped ← thumb-key workhorse |
| `&mt LSHIFT F` | shift held, F tapped ← home-row mods |
| `&trans` | fall through to the layer underneath |
| `&none` | deliberately dead |
| `&bt BT_SEL 0` | Bluetooth profile 1 |
| `&caps_word` | caps until the next space — better than capslock |
| `&sk LSHIFT` | sticky: press once, applies to the next key |

## Layers

Five exist (`layer0`–`layer_4`); **3 and 4 are entirely `&trans`** and unreachable — nothing binds
`&mo 3` or `&mo 4`. They're yours. To reach one, put `&mo 3` on a key and fill the layer in.

Order matters: a layer only sees through `&trans` to layers *below* it.

## Recipes

**Arrows on the right hand, layer 1** — replace `HJKL` on layer 1 row 2:

```
&kp LEFT  &kp DOWN  &kp UP  &kp RIGHT
```

**Home-row mods** — hold for a modifier, tap for the letter. Row 2 of layer 0:

```
&mt LGUI A  &mt LALT S  &mt LCTRL D  &mt LSHFT F   ...   &mt RSHFT J  &mt RCTRL K  &mt RALT L  &mt RGUI SEMI
```

Add near the top of the file to stop it firing while typing fast:

```c
&mt { tapping-term-ms = <200>; flavor = "tap-preferred"; };
```

**A combo** — press two keys together. Positions come from the grid above:

```c
combos {
    compatible = "zmk,combos";
    esc_combo {
        bindings = <&kp ESC>;
        key-positions = <14 15>;   // Q + W
        timeout-ms = <40>;
    };
};
```

## How to actually make a change

**Fast, throwaway — ZMK Studio.** Plug the left half in, open [zmk.studio](https://zmk.studio),
drag keys around. Instant, no rebuild. But it lives in the keyboard's settings and **overrides this
repo** until you reset settings — so prototype there, then write the winner into the file.

**Permanent — edit the file.**

```sh
cd ~/_repos/zmk-sofle
# edit config/eyelash_sofle.keymap
git commit -am "keymap: arrows on layer 1" && git push
```

CI builds in ~4 min. Download the artifact, double-tap reset, drag the `.uf2` on. The keymap lives
on the **left** half — for keymap-only changes you don't need to reflash the right.

The rendered keymap SVG in the README regenerates automatically on every push, so you always have a
picture of what you built.

## When it won't compile

- **Wrong number of bindings.** Every layer needs exactly 64. Miscounting is the #1 cause.
- **Unknown keycode.** Some need an include — check the header list at the top of the file.
- **Studio is overriding you.** Keyboard ignoring your changes? That's Studio's stored layout
  winning. Reset settings to clear it.
