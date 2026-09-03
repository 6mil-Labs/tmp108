# TMP108 Qwiic

A 1" × 1" Qwiic-compatible breakout for the Texas Instruments [TMP108][ds] digital
temperature sensor.

The TMP108 is a 12-bit I²C/SMBus temperature sensor with a dynamically-programmable
alert window, packaged in a 6-ball WCSP. This board brings it out to two Qwiic
connectors for daisy-chaining, a 0.1" header for breadboard use, and a full set of
test points.

- **Resolution:** 12 bits (0.0625 °C)
- **Accuracy:** ±0.75 °C max (−20 °C to +85 °C), ±1 °C max (−40 °C to +125 °C)
- **Supply:** 1.4 V to 3.6 V (3.3 V from the Qwiic bus)
- **Current:** 6 µA max, active
- **Interface:** I²C / SMBus, four solder-selectable addresses
- **ALERT:** programmable over/under-temperature window, broken out to header and test point

## Board

| | |
|---|---|
| Dimensions | 25.4 × 25.4 mm (1.0" × 1.0"), 1 mm corner radius |
| Layers | 4 (F.Cu · In1.Cu GND · In2.Cu 3V3 · B.Cu) |
| Thickness | 1.6 mm |
| Finish | Lead-free HASL |
| Mounting | 4 × 2.54 mm holes on a 20.32 mm (0.8") grid |
| Signal track width | 0.1524 mm (6 mil) |
| Power track width | 0.4572 mm (18 mil) |

## Connectors

**J1, J2** — Qwiic (JST SH, 1 mm pitch, 4-pin right-angle SMD). Electrically identical
and wired in parallel, so the board can sit anywhere in a Qwiic chain.

| Pin | Signal |
|---|---|
| 1 | GND |
| 2 | 3V3 |
| 3 | SDA |
| 4 | SCL |

**J3** — 1 × 5 0.1" header. Not populated by the fab; solder your own.

| Pin | Signal |
|---|---|
| 1 | GND |
| 2 | 3V3 |
| 3 | SDA |
| 4 | SCL |
| 5 | ALERT |

`ALERT` is open-drain and has **no pull-up on this board** — provide one on the host
side if you use it.

## I²C address

The TMP108 address is set by the `A0` pin. Two jumpers select between the four
options; the silkscreen prints the resulting address next to each pad.

| Address | `A0` tied to | Jumper setting |
|---|---|---|
| **0x48** (default) | GND | JP2 pads 1–2 bridged (factory default) |
| 0x49 | 3V3 | JP2 pads 2–3 bridged |
| 0x4A | SDA | JP1 pads 1–2 bridged |
| 0x4B | SCL | JP1 pads 2–3 bridged |

Cut the existing JP2 bridge before closing any other option — only one may be
connected at a time.

R3 (10 kΩ) sits between `A0` and 3V3 and keeps the pin from floating while a jumper is
open. Note that in the default configuration it also draws roughly 330 µA through the
JP2 GND bridge, which is far more than the sensor's own 6 µA. If you are chasing
microamps, cut JP4 *and* remove R3, then hard-strap `A0` with JP1/JP2 alone.

## Jumpers

| Ref | Default | Function |
|---|---|---|
| JP1 | open | Ties `A0` to SDA (0x4A) or SCL (0x4B) |
| JP2 | 1–2 closed | Ties `A0` to GND (0x48) or 3V3 (0x49) |
| JP3 | closed | Connects the on-board 10 kΩ SDA/SCL pull-ups (R1, R2) to 3V3 |
| JP4 | closed | Powers the red LED (D1) via R4 |

Cut **JP3** when the bus already has pull-ups elsewhere. Every board added to a Qwiic
chain puts another 10 kΩ pair in parallel, and past a handful of boards the combined
pull-up starts to exceed what the bus drivers can comfortably sink.

Cut **JP4** to remove the LED current in battery-powered use — roughly 1.4 mA, assuming
a 1.9 V forward drop across D1 and the 1 kΩ R4. The sensor itself only needs 6 µA, so
the LED dominates the board's consumption by more than two orders of magnitude.

## Test points

Six 1.0 mm pads in two columns along the left edge:

| Pad | Signal | Pad | Signal |
|---|---|---|---|
| TP1 | 3V3 | TP4 | SCL |
| TP2 | GND | TP5 | ALERT |
| TP3 | A0 | TP6 | SDA |

## Bill of materials

| Ref | Value | Package | LCSC |
|---|---|---|---|
| U1 | TMP108 | DSBGA-6 (YFF), 0.5 mm pitch | C129486 |
| C1 | 0.1 µF | 0603 | C14663 |
| R1, R2 | 10 kΩ | 0603 | C25804 |
| R3 | 10 kΩ | 0603 | C25804 |
| R4 | 1 kΩ | 0603 | C21190 |
| D1 | Red LED | 0603 | C2286 |
| J1, J2 | Qwiic JST SH 1 mm RA | SMD | C51940130 |
| J3 | 1 × 5 header | 2.54 mm THT | — |
| TP1–TP6 | Test point | 1.0 mm pad | — |

## Repository layout

```
tmp108.kicad_pro          KiCad project
tmp108.kicad_sch          Schematic
tmp108.kicad_pcb          Board layout
SparkFun-Qwiic.kicad_sym  Local symbol library (Qwiic logo)
SparkFun-Qwiic.pretty/    Local footprint library (Qwiic logo, various sizes)
sym-lib-table             Project symbol library table
fp-lib-table              Project footprint library table
```

Fabrication outputs are generated into `production/` and are not tracked.

## Opening the project

Designed with **KiCad 10** (`generator_version "10.0"`). Older releases may refuse to
open these files or drop features on load.

`SparkFun-Qwiic.kicad_sym` and `SparkFun-Qwiic.pretty/` are vendored in this repository
and resolve through `${KIPRJMOD}`. Two dependencies are *not* vendored:

- The **SparkFun KiCad libraries** from the Plugin and Content Manager. Symbols come
  from `PCM_SparkFun-Aesthetic`, `-Capacitor`, `-Connector`, `-Jumper`, `-LED`,
  `-PowerSymbol` and `-Resistor`. Only `PCM_SparkFun-Aesthetic` and
  `PCM_SparkFun-Connector` supply footprints — the passives use the SparkFun 0402
  symbols with their footprints overridden to the standard KiCad `Resistor_SMD`,
  `Capacitor_SMD` and `LED_SMD` 0603 variants.
- The **6milLabs** symbol library, which provides the `TMP108` symbol. `sym-lib-table`
  references it by an absolute path
  (`/home/balbi/kicad/6mil-labs/6milLabs-KiCAD-Libraries/6milLabs.kicad_sym`); edit that
  entry to match your checkout.

The board itself only needs the two PCM footprint libraries above, since everything
else is either vendored or part of the stock KiCad footprint set.

## Manufacturing

Fabrication files are produced with the [Fabrication Toolkit][fabtk] plugin; its
settings live in `fabrication-toolkit-options.json`. Running the plugin writes a gerber
archive, `bom.csv`, `positions.csv`, `designators.csv` and `netlist.ipc` into
`production/`, formatted for JLCPCB.

Order as a 4-layer, 1.6 mm board with lead-free HASL.

`J3` is marked DNP and is left out of the generated BOM — solder your own header if you
want it. The six test points, however, *are* emitted into `bom.csv` with an empty part
number (they have no placement data and nothing to assemble); delete that row before
uploading.

## License

Released under the [Creative Commons Attribution-ShareAlike 4.0 International][cc]
license.

Designed by Felipe Balbi — Copper and Code.

[ds]: https://www.ti.com/lit/ds/symlink/tmp108.pdf
[fabtk]: https://github.com/bennymeg/Fabrication-Toolkit
[cc]: https://creativecommons.org/licenses/by-sa/4.0/
