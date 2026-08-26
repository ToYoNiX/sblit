# The sblit 36 keyboard

The sblit36 is a 36-key Corne-style split keyboard with a more ergonomic column layout, a
built-in rotary encoder, and a flat stacked-plate body — it looks like a slab, hence
*slab-split* → **SBLIT**.

- 3x5 matrix + 3 thumb keys per half (36 keys total)
- Rotary encoder on the right half
- Pro Micro controller, TRRS link between halves (I2C, with pull-ups on board)
- MX switches, through-hole diodes
- Laser-cut layered construction: bottom plate, PCB layer, switch plate, keycap layer
- M3 hardware, with nut pockets cut into the stack

## Generating the board

Everything in this repo is driven by a single [ergogen](https://github.com/ergogen/ergogen)
config: [`config.yaml`](config.yaml).

Ergogen already runs in the browser. I forked the GUI to add my own custom footprints —
the ones I need for the single-sided PCB attempt — so this config generates as-is, with no
install and no local toolchain:

**👉 https://toyonix.github.io/ergogen-gui/**

1. Open the link above.
2. Paste in the contents of [`config.yaml`](config.yaml) (or drop the file in).
3. Download the outputs:
   - **DXF** outlines → send straight to a laser cutting service.
   - **KiCad** PCB → for the (work-in-progress) PCB.

The plate outlines will generate on stock ergogen too, but the `pcbs` section won't — it
depends on footprints that only exist in the fork.

The relevant outlines are `bottom_layer`, `pcb_layer`, `switch_plate`, and `keycap_layer`.

## Why I built this

My first DIY keyboard was a 36-key Corne. It was my first DIY build *and* my first split,
and I loved it — but two things kept bothering me:

- **The pinky column.** The position was horrendous for me. On long sessions it genuinely
  hurt. The pinky is the weakest finger and it was being asked to reach the furthest.
- **The inner thumb key.** It sat so far in that I had to move my whole hand just to hit it
  with my thumb, which defeats the point of a thumb cluster.

So sblit36 is my fix for both: the pinky column is dropped and staggered to sit where my
pinky actually rests, and the thumb cluster is splayed outward so all three thumb keys are
reachable without shifting my hand.

The other problem was **cost**. My Corne was 3D printed, and I don't own a printer — every
prototype meant paying a service and waiting. Iterating that way is slow and expensive.
But I have several laser cutting services nearby, so I moved to a flat, stacked, laser-cut
design. That's what pushed me to ergogen: the whole board is one text file, and a new
revision is a re-export away.

I've learned a huge amount doing this, and I'm still learning.

## TRRS and hot-plug safety

A plain TRRS split link has a nasty failure mode: while the plug is being pulled out, the
moving contacts sweep across each other, and you can briefly short VCC into a data pin.
That's the worst case — it can fry the controller. It happens on an accidental yank, not
just on deliberate unplugging.

Two things in this design guard against it:

**Series resistors on the I2C lines.** I2C tolerates series resistance as long as the total
on each line stays under about 10k. So each half carries a 220R resistor in series on SDA
and on SCL (`series_sda` / `series_scl` in `config.yaml`) — 440R total per line once both
halves are connected. It works perfectly: a 1m TRRS cable, and countless hot-plugs with no
dead Pro Micros so far. When you pull the cable the controller resets because the I2C
connection drops, and it reconnects when you plug it back in, so in practice it behaves
like a hot-pluggable port.

**Pin order chosen so GND shields the data lines.** The jack is wired:

| Contact | Net |
| --- | --- |
| Tip | SCL |
| Ring 1 | SDA |
| Ring 2 | GND |
| Sleeve | VCC |

GND sits directly between VCC and the two data lines. As the plug is withdrawn the
contacts sweep across each other, so VCC meets GND before it can ever reach SDA or SCL —
a rail-to-rail short the supply shrugs off, instead of 5V landing on an MCU pin. The data
lines are the two contacts furthest from VCC, and the series resistors above are the
second line of defense behind that.

To be clear: this is **not** meant as a hot-swap feature. It's there so an accidental
removal doesn't cost you a controller.

## Status

- ✅ Layout and laser-cut plate stack — done and buildable.
- ✅ TRRS hot-plug protection — series resistors and a safe pin order, see above.
- 🚧 **PCB** — I want a single-sided, DIY-friendly PCB (something I can etch/mill myself
  rather than order). It isn't working yet, but I'm still working on it. The `pcbs` section
  of `config.yaml` is that ongoing attempt, and the custom footprints in my ergogen fork
  exist to serve it.
- 🚧 **Enclosed case** — I want to move away from the open sandwich stack I have now
  to a proper enclosed acrylic case. Still looking into how to do it and how everything
  fits inside.
- 🚧 **Display** — I want to go wireless, and a wireless build needs a display
  (battery level, layer state, connection status).
- 🚧 **M2 hardware** — switching the mounting holes and nut pockets from M3 to M2.
  M3 simply doesn't fit between switches at normal keyboard spacing, so anything screwed
  down inside the matrix has to be M2. On top of that, most of the small electronics I'd
  want to mount — breakout/test PCBs, small displays — already use M2 mounting, so
  standardizing the whole board on M2 is the right call.

## License

See [LICENSE](LICENSE).
