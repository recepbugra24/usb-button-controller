# USB Button Controller

A minimal USB HID keyboard board that reads two externally wired buttons and
sends keystrokes to a host computer. No drivers required — the host enumerates
it as a standard keyboard.

Designed around the WCH CH552G, an 8-bit MCU with an integrated USB device
controller, on-chip 24 MHz oscillator and on-chip 5 V to 3.3 V regulator. This
removes the crystal, load capacitors, reset network and programming header that
a comparable AVR design would need, and cuts the controller cost from roughly
$4.70 to $0.31 per unit.

## Specifications

| | |
|---|---|
| MCU | WCH CH552G (SOP-16) |
| Interface | USB 2.0 Full Speed, USB-C connector |
| Inputs | 2 buttons, wired off-board via screw terminal |
| Power | Bus powered, ~20 mA |
| Board | 41.8 × 25.25 mm, 2 layers, 1.6 mm FR-4 |
| Assembly | Single sided (top), SMD |

## Design notes

**USB.** Both differential pairs of the USB-C receptacle are tied together so
the cable works in either orientation. CC1 and CC2 each carry a 5.1 kΩ pulldown
to advertise the board as a sink; without them the host supplies no power.
A USBLC6-2SC6 clamps ESD on D+/D- as close to the connector as layout allows.

**Button inputs.** Each channel has a 4.7 kΩ pull-up so the input never floats,
a 1 kΩ series resistor and a 100 nF capacitor forming a low-pass filter with a
0.1 ms time constant, and a second USBLC6-2SC6 protecting against discharge
carried in on the button wiring. The filter suppresses contact bounce in
hardware; firmware adds a short debounce window on top.

**Programming.** The CH552G ships with a USB bootloader in ROM. Holding SW1
while plugging in pulls D+ to V33 through a 10 kΩ resistor, which puts the chip
into bootloader mode. No external programmer is needed.

**Ground.** Copper pours on both layers, stitched with vias. The pour under the
USB pair is kept unbroken to hold the differential impedance steady.

## Repository layout

```
usb-button-controller.kicad_sch     schematic
usb-button-controller.kicad_pcb     board layout
usb-button-controller.kicad_pro     project file
gerber/                 fabrication output
  *.gbr                 gerber layers
  *.drl                 drill file
  BOM_jlcpcb.csv        bill of materials with LCSC part numbers
  usb-button-controller-all-pos.csv component placement
```

## Manufacturing

Built for JLCPCB with assembly. Upload the gerber archive, then the BOM and
placement files from `gerber/`.

Settings: 2 layers, 1.6 mm, HASL, top-side assembly. Enable **Confirm Parts
Placement** — KiCad and JLCPCB do not always agree on the zero-rotation
reference for SOT-23 and SOP packages, and the preview is where a misoriented
part gets caught.

All passives, the LED and the tactile switch are JLCPCB Basic parts. The
CH552G, USB-C receptacle, USBLC6-2SC6 and screw terminal are Extended parts and
carry a per-part setup fee.

## Firmware

Built with [ch55xduino](https://github.com/DeqingSun/ch55xduino) in the Arduino
IDE. Add this board manager URL:

```
https://raw.githubusercontent.com/DeqingSun/ch55xduino/ch55xduino/package_ch55xduino_mcs51_index.json
```

Install **CH55x Boards**, select **CH552 Board**, and pick a USB setting that
reserves RAM for HID. Pin mapping:

| Function | Port | Arduino pin |
|---|---|---|
| Button 1 | P3.3 | 33 |
| Button 2 | P3.4 | 34 |
| Status LED | P1.7 | 17 |

A blank chip enters the bootloader on its own, so the first upload needs no
button press. Later uploads are triggered automatically unless the running
firmware has locked up the USB stack, in which case hold SW1 and replug.

## Verification

- ERC clean
- DRC clean, zero unconnected items
- Schematic parity between board and schematic confirmed
- Netlist reviewed pin by pin against the CH552G datasheet

Not yet verified in hardware — no board has been built from these files.

## License

Not yet chosen. Add a LICENSE file before sharing this publicly.
