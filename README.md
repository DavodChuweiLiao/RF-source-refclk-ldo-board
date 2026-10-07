# AD9959 Supply and Reference Clock Board

A USB-C powered supply and 125 MHz reference clock board for the Analog Devices
**AD9959/PCBZ** quad DDS evaluation platform.

The eval board needs four externally supplied rails and an external reference
clock, and provides neither. The usual answer is four bench supplies plus a
signal generator. This board replaces all of that with one USB-C cable.

Built for RF generation in the 100–200 MHz band driving acousto-optic
modulators in a cold-atom experiment, where close-to-carrier phase noise is the
governing spec.

![3D render](Screenshot%202026-10-07%20162503.png)

![PCB layout](Screenshot%202026-10-07%20162719.png)

---

## What it does

| Output | Voltage | Current | To |
|---|---|---|---|
| AVDD | 1.8 V | 225 mA | eval J10/J16–J20 (SMA) |
| DVDD | 1.8 V | 145 mA | eval TB1.4 |
| DVDD_I/O | 3.3 V | 40 mA | eval TB1.2 |
| VCC_USB | 3.3 V | ~300 mA | eval TB1.1 |
| REFCLK | 125 MHz, +5.3 dBm | — | eval J9 (SMA) |

Total draw 735 mA from VBUS, 1.80 W dissipated. All rails linearly regulated —
no switching converter anywhere.

## Why 125 MHz at 4×

The AD9959's own PLL is the phase-noise limit, not the oscillator. At 20×
multiplication its residual is −107 dBc/Hz at 1 kHz offset; a good 25 MHz TCXO
multiplied up contributes −123 dBc/Hz, so the chip dominates by 16 dB and a
better oscillator buys nothing.

Dropping to 4× — the minimum the part allows, requiring a 125 MHz reference,
which is also the maximum multiplier input — recovers about 13 dB. The
oscillator is a **Crystek CCHD-575-50-125.000** at −141.9 dBc/Hz, leaving the
chip as the limit with 24 dB of headroom.

Same change moves the first Nyquist image for a 200 MHz output from 130 MHz to
300 MHz, which makes the output reconstruction filter far easier.

## Board

4 layer, 60 × 60 mm, 1.6 mm, ENIG. L2 is an unbroken ground plane with no
routing. 36 components, 145 vias.

## Verification

Simulated in ngspice using component values and routed trace resistances
extracted directly from the KiCad board file.

- **Hot-plug:** 5.87 V worst-case peak against a 6.5 V regulator maximum.
  Cable inductance rings against the on-board ceramics; undamped this reaches
  8.46 V. Fixed with a 100 µF polymer cap in series with 0.235 Ω.
- **Reference clock:** 489 mV/pin at the AD9959 (spec 200–1000 mV), oscillator
  driver at 20.2 mA against its 24 mA rating.
- **Oscillator supply:** 12.1 kHz RC corner, −40 dB at 1 MHz, no peaking.

Full analysis in [`ad9959_supply_board.pdf`](refclk_board_doc.pdf).

## Gotchas

- **U2's tab is VIN, not ground** — the only regulator on the board like that.
  No ground pour near it.
- **C2 must be C0G**, not X7R. It sits in the 125 MHz path.
- **C6 must be conductive polymer.** Damping is set by R8‖R9; a high-ESR
  substitute returns the hot-plug peak to a failing 6.9 V.
- **Both R8 and R9 required.** With one, VBUS peaks at 6.77 V — over the limit.
- **Label the J4 harness.** DVDD's absolute max is 2.0 V and it sits beside two
  3.3 V rails. Nothing on either board prevents a swapped wire.
- **Set the AD9959 VCO gain control bit high.** It defaults low (100–160 MHz);
  a 500 MHz system clock needs the 255–500 MHz range.

## Status

Schematic, layout and gerbers complete and verified. In fabrication.

## Licence

MIT
