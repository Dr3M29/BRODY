# BRODY

A robot I'm building out of a dead hoverboard, my e-bike battery, and an Arduino. Two wheels, LED face, posable arms, about 28" tall. Budget: whatever's in my pocket, which is not much.

## The story

My hoverboard died a while ago (battery gave out, everything else is fine). Turns out the two hub motors inside are strong enough to carry a person, and the motor boards can be reflashed with open source firmware so an Arduino can control them over serial. So instead of buying motor controllers ($60+) I'm hacking the boards the hoverboard already came with. Total spend so far on the motor side: an $8 debug probe.

The battery is borrowed from my Qlife e-bike - it slides onto a rail and locks with a key, so BRODY gets a slide-in battery too. It'll load through a hatch in his chest.

## Parts

**Free (salvaged from the hoverboard):**
- 2x brushless hub motors, 6.5" wheels
- 2x motor boards - MM32SPIN27PF chips, board marked YK189 / 210412
- charger, power button, charge port
- 2x LED light bars (turn signals, will reuse for accent lights)

**Borrowed:**
- 36V 10.4Ah e-bike battery (374Wh, 5.8 lbs) - shared with the bike

**Bought / buying:**
- CMSIS-DAP debug probe - $8
- header pins, spade connectors, wire, fuse, heat shrink - ~$15
- 8x8 LED matrix for the face - $5-10
- M3 heat set inserts - $10
- 4kg PETG - $40 (already have)

**Already had:** Arduino kit, soldering iron, 3D printer, calipers.

## How it goes together

Battery slides down a slant into the lower torso (head tilts up, chest hatch opens, battery drops in, rail lock holds it). Power comes off the pack's +/- blades with spade connectors, through a fuse, then splits left and right - one motor board lives in each leg, right above the wheel mount. The Arduino sits in the torso and talks to the boards over UART. Skinny signal wires run up, thick power wires run down, and the whole skeleton is PETG with a metal core in the hip beam since the battery weighs almost 6 pounds.

Stance is 16" wheel to wheel with the legs angled inward so he doesn't look like a table. Height is 28-30" for now - the spine is one straight segment on purpose so I can swap it for a longer one and make him 3 feet tall later.

The shell is separate from the skeleton. Skeleton takes all the weight, shell panels just screw or snap on so I can pull them off to get at the guts.

Top speed will be capped around 2 mph in software. The motors could do 8. They will not be doing 8 in my house.

## Firmware

The boards use MindMotion MM32SPIN27 chips (ARM Cortex-M0), which is the less common path - most guides assume STM32. Useful repos:

- https://github.com/RoboDurden/Hoverboard-Firmware-Hack-Gen2.x - the Gen2 split-board firmware project
- https://gitlab.com/ailife8881/Hoverboard-Firmware-Hack-Gen2.x-MM32 - MM32 version
- https://github.com/trondin/MM32SPIN05_Hoberboard_hack - SPIN05 port, close cousin, UART control works

Flashing is over SWD (3 wires) with a CMSIS-DAP probe and pyOCD. The chips ship read-protected so first contact needs a full erase. Two gotchas I know about going in: the board layout has to be matched against the repo before flashing (wrong pin map = dead MOSFETs), and the chip controls its own power latch, so you have to physically hold the power button the entire time you flash or the board shuts off mid-write.

## Progress

Done:
- figured out the hoverboard's problem (battery, not boards)
- identified the chips and found the programming + UART pads (labels are on the back of the board)
- found the battery's + and - blades
- picked materials and the overall design

Next up:
- multimeter check on the battery blades before ANY wiring happens
- solder headers onto the boards
- match the board layout, flash both boards
- spin the motors from the PC, then from the Arduino
- CAD the skeleton, print the hip beam and legs
- rolling chassis (with a caster - self balancing is a someday problem)
- LED face, head tilt servo, ultrasonic sensors
- arms (posable now, servos later)
- Raspberry Pi for talking and roaming, eventually

## Ground rules

Lithium batteries are the one part of this that can actually hurt you, so: no charging the old dead pack (it's getting recycled), nothing connects to the battery until a multimeter confirms the pinout, one wire cut at a time, fuse on the positive lead, and my dad's around for battery work.

---

CAD files and Arduino code will land here as they exist.
