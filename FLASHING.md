# Flashing BRODY's motor boards

Notes and status for reflashing the two hoverboard motor boards. Companion to the
firmware section in the [README](README.md).

## The board
- Chip: **MM32SPIN27PF** (ARM Cortex-M0). Confirmed alive: read CPUID = 0x410CC200.
- Board markings: H217776A / E472363 / JK-1 94V-0 / YK189, dated 2021-05.
- Two identical boards, one per wheel. Each has its own MM32 chip. Flash both, one at a time.

## Tools
- **Raspberry Pi Debug Probe** (CMSIS-DAP) over SWD.
- **pyOCD** (installed). Device pack installed: `MindMotion.MM32SPIN2x_DFP`
  (`pyocd pack install mm32spin27pf`).

## Wiring (the hard-won part)
SWD header on the board = 4 pads: **3.3V / SWCLK / SWDIO / GND**.

Plug the probe cable into the **"D" (SWD) port, NOT the "U" (UART) port.**
This cost a whole session of "No ACK". U is a serial port, it has no SWD on it.

| Probe wire | Board pad |
|---|---|
| orange | SWCLK |
| yellow | SWDIO |
| black  | GND |

## Powering it to flash
- **Do NOT** back-power the board from an external 3.3V (Arduino / programmer). The
  hoverboard-hack community warns this has killed boards. It is fine for a quick read,
  but not the way to write firmware.
- **Do** power from the ~36V battery (or a current-limited bench supply) and **hold the
  power button** the whole time, or the chip drops its power latch and switches off
  mid-write.

## Status (2026-07-16)
- SWD connection to board #1 works. `pyocd reset -t cortex_m` connects; CPUID reads OK.
- Chip identified: MM32SPIN27PF / Cortex-M0.
- Chip is **read-protected**: reading flash (0x08000000) fails while the core reads fine.
  Needs a mass-erase to remove readout protection before flashing (this is expected).
- The `-t mm32spin27pf` target errors on its vendor "DebugPortSetup" sequence (No ACK);
  `-t cortex_m` connects fine. Need to resolve this (or use another tool) to reach the
  MM32 flash algorithm.

## Next steps
1. Power the board properly (battery + hold power button).
2. Get the MM32 flash target to connect (resolve the debug-sequence issue).
3. Unlock: mass-erase to disable readout protection.
4. Firmware: RoboDurden Hoverboard-Firmware-Hack-Gen2.x (+ the MM32 fork). **Match the
   board variant (defines_2-x.h) first. A wrong pin map fries the MOSFETs.**
5. Flash board #1, verify, then repeat for board #2.

## Commands that work today
```
pyocd list                                                    # confirm probe is seen
pyocd reset -t cortex_m --frequency 500000                    # test SWD connection
pyocd commander -t cortex_m -c "halt" -c "read32 0xe000ed00"  # read CPUID (should be 0x410cc200)
```
