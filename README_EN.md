# Program for the breadboard build (version bb6)

Date: 2026-11-25. Status: **runs on the breadboard and has produced its first reading: a 1 kOhm resistor reads 1.002-1.003 kOhm at 1 kHz (not calibrated). Display, serial console, sine output, sampling and auto-ranging work. Capacitors, inductors and other resistor values have not been measured yet, and nothing is calibrated.**

This is a copy of the program in the RevC folder; the RevC folder itself was not changed. Its pins match the breadboard build guide, and it works for both the 74HC4051 version and the CD4051BE version.

| File | Content |
| --- | --- |
| `LCR_meter_bb6.bin` | the file to load into the module |
| `LCR_G431_breadboard/` | source code |
| `LCR_meter_bb6_program_2026-11-25.zip` | backup of the two items above plus the README files, to hand over in one piece to whoever writes the program |
| `empty_program.bin` | empty program: does nothing, every pin stays in its power-up state. Source in `empty_program/` |
| `lcd_blank.bin` | blank-display program: starts the display, clears it and then does nothing; no test signal. Source in `lcd_blank_program/` |

**The module now holds the blank-display program** (loaded 2026-11-25). The display keeps its last text for as long as it has power, and shows a row of blocks after a power cycle; only a program that sends it the "clear" command gives an empty screen, which the empty program cannot do. To get the meter back, drag `LCR_meter_bb6.bin` onto the module.

## 1. Loading the program

1. Connect the module to a PC with a USB cable. A new USB drive appears.
2. Drag `LCR_meter_bb6.bin` onto that drive.
3. The LED on the module flashes for a few seconds, then the program starts by itself.

## 2. What the display shows when all is well

| Display | Meaning |
| --- | --- |
| `LCR meter bb6` / `starting...` | flashes by |
| `LCR meter bb6` / `cal: none yet` | stays 1.5 s; no calibration done yet |
| `OPEN (no part)` | nothing between the two clips |
| `R*1.000kΩ  1kHz` / `ph +0.0°   1k` | a 1 kOhm resistor in the clips; `*` means this range is not calibrated yet |

## 3. If the display shows something else

| Display | Meaning | Check first |
| --- | --- | --- |
| a row of blocks on line 1, no text | the program is not running, or the six display signal wires are wrong | drag the .bin again; the wires of step 4 |
| garbage characters | one of the four display data wires is wrong | the wires of step 4 |
| `FATAL ERROR:` and a line of text | a part of the program failed while starting | write down the second line |
| `NO SIGNAL` | the test signal is not coming out | the filter of step 12, the yellow wire of step 19 |
| `ADC CLIPPING` | a signal hits 0 V or 3.3 V | the 1.65 V points (h41, g7) |
| `ADC TIMEOUT` | sampling did not finish | a program problem, not wiring |
| `OPEN` with a resistor fitted, or a value wrong by a large factor | range wires or reference resistors | steps 13, 14 and 20 |

## 4. Differences from the RevC program

1. **Averaging of several sample blocks corrected.** The original added the results of several blocks directly, but every block starts at a different phase, so the results could cancel: readings would jump about or `NO SIGNAL` would be reported wrongly. The new calculation does not depend on the starting phase. Check script: `LCR_G431_breadboard/tools/block_phase_sim.py`.
2. **The display starts first.** If anything fails later during start-up, the display shows `FATAL ERROR:` and where, so no serial connection is needed to see it.
3. **The start-up screen shows the version** (now `bb6`). Count it up with every change, so it is clear which program is in the module.
4. **`build.py` added**, so the program can be built without make.
5. **DAC output fault fixed (found on the real board).** The original fed the DAC with 16-bit transfers, which this chip does not accept: the transfer channel reported an error as soon as it started and switched itself off, so the DAC stayed at 0 V and the display showed `NO SIGNAL`. With 32-bit transfers the sine wave is correct: measured amplitude 621 (set value 620).
6. **New console command `raw`.** It prints the minimum, mean, maximum and signal amplitude of both sampled channels on every range, and of the DAC pin itself, to find where a signal is lost.
7. **An input stuck at 0 V or 3.3 V is reported as `ADC CLIPPING`**; before, that case was reported as `NO SIGNAL`.
8. **Auto-ranging no longer gets stuck on a range without signal (found on the real board).** On the breadboard the 1 MOhm range was open. The original moved to that range, saw no signal and stayed there, so `NO SIGNAL` was shown even with a 1 kOhm resistor in the clips. Now a range without signal is skipped and the other ranges are tried; `NO SIGNAL` is shown only if no range has a signal.
9. **New console command `scan`.** It switches on each of the eight multiplexer channels in turn (also the three unused ones) and prints mean and amplitude of both sampled channels, to find a misplaced wire or resistor or a dead channel.

## 5. Building again

- Command line: run `python build.py` in the `LCR_G431_breadboard` folder; it writes `build/lcr_g431.bin`.
- The compiler is in `C:\Users\64189\tools\arm-gcc` (xPack GNU Arm Embedded GCC 15.2.1, unpacked only, nothing installed in the system; delete that folder when it is no longer needed).
- STM32CubeIDE: File -> Import -> Existing Code as Makefile Project, select `LCR_G431_breadboard`, toolchain MCU ARM GCC.

## 6. Not done yet

- Only one 1 kOhm resistor has been checked; capacitors, inductors and other resistor values have not been measured.
- The measured reference resistor values are not stored yet, and the open / short calibration has not been done. See "第一次上电要做的校准" in `LCR_G431_breadboard/README.md`.
- The PCB (RevE / RevE1) cannot use this .bin as it is: five changes are needed first, see section 5 of the RevE README.
