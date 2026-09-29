# zmk-config-from-eigatech - (modified) Custom wired TOTEM keymap/config

This is a [ZMK](https://zmk.dev/) `zmk-config` (keymap + hardware config, not a full firmware fork) for a [TOTEM](https://github.com/GEIGEIGEIST/TOTEM) split keyboard, running on Seeed XIAO BLE microcontrollers.

It started as a fork of [eigatech/zmk-config](https://github.com/eigatech/zmk-config) (a multi-keyboard collection of community ZMK configs, including the stock TOTEM shield definition) with the default branch set to `totem-dongle` - a fully wireless setup where both halves talk to a third XIAO BLE acting as a dongle. This branch (`totem-wiredbtmix`) removes the dongle: the left half is now the central and talks to the computer over USB, with the right half talking only to the left half, over BLE (never over USB, even if it's plugged in). Both halves can simply run on USB power - no battery required in either. The left half always needs its USB connection anyway (that's how it talks to the computer), but since the right half only needs power from USB, not data, it should also be able to run untethered on a battery instead, if you'd rather not have a cable on that side.

## Based on

- [eigatech/zmk-config](https://github.com/eigatech/zmk-config) - source of the original TOTEM shield hardware definition (`config/boards/shields/totem/`) this repo is built on.
- [GEIGEIGEIST/TOTEM](https://github.com/GEIGEIGEIST/TOTEM) - the physical keyboard design.
- [ZMK docs](https://zmk.dev/docs) and [ZMK Discord](https://discord.gg/8cfMkQksSB) - for ZMK itself.

**To change the keymap**, edit `config/totem.keymap` - that's the only file that needs to change for keymap edits; see `changes_to_make.md` for in-progress ideas/notes on it. Everything else in this repo is the hardware/BLE-topology setup (see "Major changes" below), which most people won't need to touch.

## Getting your own firmware

Push to this repo (or open a PR) and GitHub Actions builds automatically via ZMK's standard `build-user-config.yml` reusable workflow - no local toolchain needed. There's no local-build setup in this repo; building locally would mean installing `west` and the Zephyr SDK by hand, following [ZMK's manual setup docs](https://zmk.dev/docs/development/setup) - not documented here since the GitHub Actions path covers normal usage.

### Flashing

1. On GitHub, go to the **Actions** tab and open the most recent successful workflow run.
2. Under **Artifacts**, download and extract `firmware.zip`. You'll get three `.uf2` files: one for the left half (`totem_left`), one for the right half (`totem_right`), and one `settings_reset`.
3. For each half's microcontroller:
   1. Power it off (unplug/disconnect both halves from each other and the computer).
   2. Connect it to the computer via USB.
   3. Double-tap the reset button (the keyboard's own reset button if wired to one, or the button on the XIAO itself) to enter the UF2 bootloader. A new USB storage device should appear - if not, try again or try a different USB cable (some are power-only).
   4. **If anything other than the keymap changed** since this half was last flashed (e.g. switching to/from the dongle setup, or any non-keymap config change) - copy `settings_reset.uf2` onto that storage device first, wait for it to disappear, then repeat step 3 to get back into the bootloader. **If only `config/totem.keymap` changed**, skip straight to the next step.
   5. Copy that half's `.uf2` file (`totem_left.uf2` or `totem_right.uf2`, matching which physical half you're flashing) onto the storage device. It disappears once the copy finishes - the half is now flashed.

## Major changes from `eigatech/zmk-config`

The keymap (`config/totem.keymap`) is written from scratch for this layout preference - see `changes_to_make.md` for the history of what's been tried and what's still in progress there.

Beyond the keymap, the changes from the stock `eigatech/zmk-config` TOTEM setup, made while moving off the `totem-dongle` branch's fully-wireless topology to this wired-central one:

- **`build.yaml`** - builds `totem_left`/`totem_right`/`settings_reset` on the `xiao_ble//zmk` board directly (no third `totem_dongle` shield/board entry, and no ZMK Studio snippet/cmake-args the dongle branch had).
- **`config/boards/shields/totem/Kconfig.defconfig`** - `ZMK_SPLIT_ROLE_CENTRAL`/`ZMK_USB` defaults now apply to `SHIELD_TOTEM_LEFT` (the left half is the central, wired to the host over USB) instead of a separate dongle shield being central.
- **`config/boards/shields/totem/Kconfig.shield`** - the `SHIELD_TOTEM_DONGLE` option is gone along with the shield itself.
- **`config/totem.conf`** - adds `CONFIG_BT_CTLR_TX_PWR_PLUS_8` (max BLE TX power) and `CONFIG_ZMK_BLE_EXPERIMENTAL_CONN` (newer BLE connection parameters, for a more stable left/right link) and `CONFIG_ZMK_USB_BOOT` (wider BIOS/UEFI keyboard recognition over USB - trades away N-key rollover, which this keymap doesn't otherwise use).

For the exact diff, compare this branch to `totem-dongle`:
```sh
git diff origin/totem-dongle
```
