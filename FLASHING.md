# Flashing BRODY's motor boards

Notes and status for reflashing the two hoverboard motor boards. Companion to the
firmware section in the [README](README.md).

## The board
- Chip: **MM32SPIN27PF** (ARM Cortex-M0, an MM32SPIN2X). Confirmed alive: CPUID = 0x410CC200.
- Board markings: H217776A / E472363 / JK-1 94V-0 / YK189, dated 2021-05.
- Two identical boards, one per wheel. Each has its own MM32 chip. Flash both, one at a time.

## Tools
- **Raspberry Pi Debug Probe** (CMSIS-DAP) over SWD.
- **pyOCD** (installed). Device pack installed: `MindMotion.MM32SPIN2x_DFP`
  (`pyocd pack install mm32spin27pf`).

## Wiring
SWD header pads on the board: **3.3V / SWCLK / SWDIO / GND**, plus a separate **nRST**
(reset) pad that's needed for the unlock step (below).

Plug the probe cable into the **"D" (SWD) port, NOT the "U" (UART) port.**
This cost a whole session of "No ACK". U is a serial port, it has no SWD on it.

| Probe wire | Board pad |
|---|---|
| orange | SWCLK |
| yellow | SWDIO |
| black  | GND |

For the actual flash, also find the **nRST** pad plus a jumper wire to touch it to GND.

## Powering it to flash
- **Do NOT** back-power the board from an external 3.3V (Arduino / programmer). The
  community warns this has killed boards. Fine for a quick read, not for writing.
- **Do** power from the ~36V battery (or a current-limited bench supply) and **hold the
  power button** the whole time, or the chip drops its power latch and switches off mid-write.

## Firmware (found 2026-07-16)
- Project: **RoboDurden Hoverboard-Firmware-Hack-Gen2.x** and its **MM32 fork**
  (SourceForge "hoverboard-hack-gen2-mm32", gitlab ailife8881). Supports MM32SPIN0X and
  **MM32SPIN2X** (your SPIN27 is a SPIN2X).
- **The safe way that protects the MOSFETs: the AutoDetect / PinFinder firmware.** You
  flash a special autodetect firmware first; it probes the board and reports the correct
  pin layout WITHOUT the wrong-pinmap risk. Then you flash the real firmware with the
  matching `#define LAYOUT x` in `Inc/config.h`.
  - Autodetect binary: **`HoverboardOutput.hex`** in the SourceForge `/autodetect` folder.
    Confirm it's built for SPIN2x/SPIN27 (or build from the source there) before flashing.
- Wrong LAYOUT can "shortcut the battery and kill the mosfets", so the autodetect step is
  the whole point: never guess the layout.

## Unlock + flash procedure (pyocd)
The chip ships read-protected; unlock it with a mass erase using the nRST trick:
1. Connect the board's **nRST pad to GND** with a jumper.
2. Run: `pyocd erase -t mm32spin27pf --chip`
3. The instant you see "Erasing chip...", **pull the nRST jumper off GND.**
4. Flash the autodetect firmware: `pyocd flash -t mm32spin27pf HoverboardOutput.hex`
5. Run it, read back the detected layout, then flash the final firmware (built with the
   matched LAYOUT) the same way: `pyocd flash -t mm32spin27pf yourfirmware.hex`

Do all of this on proper battery power, power button held.

## Status (2026-07-16)
- SWD connect to board #1 works (`pyocd reset -t cortex_m`, CPUID reads OK).
- Chip is **read-protected** (flash read fails, core read works), so it needs the unlock above.
- `-t mm32spin27pf` errored on its vendor debug sequence; likely fixed by the nRST unlock
  trick (the read-protected chip needs nRST grounded to let the programmer in).
- Not yet flashed. Board #2 not wired yet.

## What's still needed before flashing
1. Wire the **nRST** pad (a 5th connection).
2. Power from the **battery** (button held), not the Arduino 3.3V.
3. Confirm or build the **SPIN2x autodetect binary**.
4. Then: unlock, flash autodetect, read the layout, flash the final firmware.

## Commands that work today
```
pyocd list
pyocd reset -t cortex_m --frequency 500000
pyocd commander -t cortex_m -c "halt" -c "read32 0xe000ed00"
```

## Links
- RoboDurden main project: https://github.com/RoboDurden/Hoverboard-Firmware-Hack-Gen2.x
- MM32 fork (gitlab): https://gitlab.com/ailife8881/Hoverboard-Firmware-Hack-Gen2.x-MM32
- MM32 binaries (SourceForge): https://sourceforge.net/projects/hoverboard-hack-gen2-mm32/files/
