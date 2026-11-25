# My QMK Userspace – Keychron Q6 (original, ANSI, no knob)

## What this does
Custom firmware for the original Keychron Q6 ANSI (not Pro/Max/HE):
- **Num Lock indicator is inverted**: lit only when Num Lock is OFF.
- **Caps Lock indicator** unchanged: lit when Caps Lock is ON.
- **Backlight off = indicators only**: with the backlight off, the board is dark except
  for the lock indicators. With the backlight on, effects work normally and the
  indicators draw on top.
- **VIA enabled**, so Keychron Launcher still works for remapping and lighting.

## Files
`keyboards/keychron/q6/ansi/keymaps/custom/`
- `keymap.c`
  - Default layers (Launcher overrides these after flashing).
  - `#error` guard: build fails if either lock LED index is undefined.
  - `process_record_user()`: RGB toggle (`RM_TOGG`/`UG_TOGG`) switches RGB Matrix
    flags between `LED_FLAG_ALL` (backlight on) and `LED_FLAG_NONE` (indicators only)
    instead of disabling RGB Matrix.
  - `housekeeping_task_user()`: if anything (e.g. Launcher) disables RGB Matrix,
    converts that to `LED_FLAG_NONE` and re-enables it.
  - `rgb_matrix_indicators_advanced_user()`: blanks all LEDs when flags are NONE, then
    draws the lock indicators. Returns `false` to skip the stock code in `q6.c`.
- `config.h` – defines `NUM_LOCK_LED_INDEX 37` (official QMK ships it commented out
  for the Q6 ANSI; `CAPS_LOCK_LED_INDEX 61` comes from the keyboard's own config).
- `rules.mk` – `VIA_ENABLE = yes`.

`qmk.json` lists the build target: `keychron/q6/ansi` + keymap `custom`.

## Why it works this way
In official QMK, disabling RGB Matrix skips the indicator hooks entirely, so "backlight
off" would also hide the lock indicators. Keeping RGB Matrix enabled and using
`LED_FLAG_NONE` as "off" (the same approach Keychron's firmware uses) keeps the
indicators visible.

## Usage notes
- Toggle backlight: Fn+Tab in the default layers. When remapping in Launcher, keep an
  "RGB Toggle" key on the Fn layer.
- Launcher's lighting on/off control: turning **off** works (becomes indicators only).
  To turn the backlight back **on**, use the RGB toggle key; Launcher may show "on"
  while the board is in indicators-only mode.
- The backlight state is saved and survives unplugging.

## How to change things
In `keymap.c`, `rgb_matrix_indicators_advanced_user()`:
- Indicator color/brightness: change the `255, 255, 255` (R, G, B) values.
- Num Lock logic: `!led_state.num_lock` = lit when OFF. Remove the `!` for stock behavior.

## How to build (GitHub Codespaces + desktop VS Code)
1. GitHub → this repo → Code → Codespaces → create/open a codespace.
2. VS Code → "Codespaces: Connect to Codespace".
3. Terminal: `qmk compile -kb keychron/q6/ansi -km custom`
4. If the .bin isn't in the repo folder:
   `cp /workspaces/qmk_firmware/keychron_q6_ansi_custom.bin .`
5. Explorer → right-click the .bin → Download.
6. Stop the codespace when done ("Codespaces: Stop Current Codespace").

## How to flash
1. QMK Toolbox → select the .bin.
2. Unplug keyboard, remove spacebar keycap, hold the reset button, plug in.
3. Flash. Afterwards, reapply the keymap in Keychron Launcher if it was reset.

## Releases
Each release has the built .bin attached; its tag marks the exact source used.
- v1.0 – Inverted Num Lock indicator; backlight-off keeps indicators visible.

## Base
Built against official QMK firmware (qmk/qmk_firmware master), not Keychron's fork.