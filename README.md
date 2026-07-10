# Ender 5 Plus / BIGTREETECH SKR Mini E3 V3.0 (Marlin config)

Marlin firmware configuration for a **Creality Ender 5 Plus** upgraded to a **BIGTREETECH SKR Mini E3 V3.0** (STM32G0B1), keeping the **stock DWIN touchscreen** via DGUS-Reloaded.

This is a config-only repo. It does not vendor Marlin's source. CI checks out a pinned Marlin release, drops this configuration in, applies a small patch, and builds. Every [release](../../releases) is reproducible from those pinned inputs.

- Marlin version: see [`MARLIN_VERSION`](MARLIN_VERSION) (currently `2.1.2.8`)
- Board: `BOARD_BTT_SKR_MINI_E3_V3_0`, TMC2209 (UART)
- Bed: 358 x 370, Z max 405, thermistor table 1, BLTouch on the Z-endstop input
- Screen: `DGUS_LCD_UI RELOADED`, `LCD_SERIAL_PORT 1`

## Download and flash

Grab the latest [release](../../releases). It contains two files:

**Board firmware (`firmware.bin`)**
1. Copy `firmware.bin` to a FAT32 microSD (32GB or smaller).
2. Insert into the SKR Mini E3's microSD slot, power on. It flashes and renames the file to `FIRMWARE.CUR`.

**Touchscreen (`DWIN_SET.tar.gz`)**
1. Extract it to get a `DWIN_SET` folder.
2. Copy that folder to the root of a FAT32 microSD formatted with **4096 (4K) allocation size**.
3. With the printer off and USB unplugged, insert into the slot on the **back of the touchscreen** and power on. Wait for the "END" message, then power cycle.

## Screen wiring (EXP1 header, cross TX/RX)

The stock screen connects to the board's EXP1 header on USART1:

| EXP1 pin | signal | to screen |
|---|---|---|
| 3 | PA9 (board TX) | RX |
| 5 | PA10 (board RX) | TX |
| 9 | GND | GND |
| 10 | 5V | 5V |

## Known working / notes

Confirmed on hardware: motion and homing (all axes), BLTouch probing and mesh leveling, heating with accurate temperature control, and the DGUS touchscreen including correct temperature display.

Known cosmetic limitation (upstream DGUS-Reloaded): when printing **from SD**, the on-screen progress bar and elapsed-time clock do not track correctly. The print itself is unaffected. This is not specific to this configuration.

## Safety

The 350mm bed draws roughly 15A through the board's onboard MOSFET, which is marginal. Add an external MOSFET or DC SSR for the bed before running unattended prints.

## Building locally

```sh
git clone https://github.com/MarlinFirmware/Marlin --branch $(cat MARLIN_VERSION) marlin
cp config/Configuration*.h marlin/Marlin/
( cd marlin && git apply ../patches/*.patch )
( cd marlin && pio run -e STM32G0B1RE_btt )
# result: marlin/.pio/build/STM32G0B1RE_btt/firmware.bin
```

## Updating the Marlin version

Change the tag in [`MARLIN_VERSION`](MARLIN_VERSION), push, and confirm CI builds green. If the DGUS patch no longer applies against a newer Marlin, regenerate it from the new source. Tag `vX.Y.Z` to cut a release.

## What's in here

- `config/Configuration.h`, `config/Configuration_adv.h` - the machine configuration
- `patches/dgus-fixed-point-temps.patch` - sends the current temperatures as fixed-point (1 decimal) so they match the bundled DWIN_SET; without it the home/temperature screens read 1/10 of the real value
- `screen/DWIN_SET.tar.gz` - the DGUS-Reloaded screen firmware (with a small fix so the probing screen's bed temperature reads correctly)
- `.github/workflows/build.yml` - the reproducible build + release pipeline

## Credits and license

Built on [Marlin](https://github.com/MarlinFirmware/Marlin) and its [DGUS-Reloaded](https://github.com/MarlinFirmware/Marlin/tree/2.1.2.8/Marlin/src/lcd/extui/dgus_reloaded) UI. The `DWIN_SET` originates from [MarlinFirmware/Configurations](https://github.com/MarlinFirmware/Configurations). Marlin is licensed under **GPLv3**; this repository inherits that license (see [`LICENSE`](LICENSE)).
