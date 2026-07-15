# BRODY — Full Parts List & Shopping Checklist

Companion to the [README](README.md). This is the detailed bill of materials — what's
salvaged, what's on order, and what still needs buying — grouped by subsystem. Prices are
rough USD.

**Status key:** ✅ have · 🟡 on order / buying · 🛒 still need · ⚪ optional

---

## The power tree (how it all connects)

```
36V e-bike battery  (~42V charged, 5.8 lb, key-lock rail)
        │
     [ 40A fuse ]   ← on the + lead, right at the pack
        │
        ├─ 36V ──→ motor board (L) ──→ hub motor (L)
        ├─ 36V ──→ motor board (R) ──→ hub motor (R)
        │
        └─ 36V ──→ buck converters  (MUST be rated for 42V+ input!)
                     ├─→ 5V  ──→ Raspberry Pi ──(USB)──→ Arduino
                     ├─→ 5V  ──→ speaker amp, sensors
                     └─→ 12V ──→ salvaged LED bars, LED face
```

\* Servos get their **own** 5V rail, not the Pi's — otherwise they brown out and reboot the Pi. (Moot now: no servos in the build, so there is no servo rail to worry about.)
Everything shares a **common ground**. High-voltage (36V) and logic (5V) stay separate; only
GND and signal wires cross between them.

---

## ✅ Salvaged / already have

| Item | Notes |
|---|---|
| 2× hub motors + 6.5" wheels | From the dead hoverboard. The "wheels." |
| 2× motor boards | MM32SPIN27PF chips, marked YK189 / 210412. These **are** the motor drivers. |
| 36V e-bike battery | 10.4Ah, 374Wh, 5.8 lb. Slides on a **key-lock rail — reuse that as the battery lock.** The slide-in dock is not coming, so tap the pack's + / - blades directly (meter first, fuse the + lead, dad present). |
| 2× LED light bars | Hoverboard turn signals → accent lights. 2-wire, single color (test at **12V**). |
| Monitor speaker | 4Ω 2W (salvaged stereo pair — only need **one** for a mono voice). |
| CMSIS-DAP debug probe | $8. Flashes the motor boards over SWD (3 wires). |
| Arduino kit | Real-time motor/servo/light control; talks to the Pi over USB. |
| Soldering iron, calipers, 3D printer | Already owned. |
| PETG (4kg) | Skeleton + shell panels. |

## 🟡 On order / already buying

| Item | ~Cost | Notes |
|---|---|---|
| Header pins, spade connectors, wire, fuse, heat shrink | $15 | Spades = **6.3mm / 0.25", yellow** (10–12 AWG). Fuse ~40A. Have now: cable/wire, heat shrink, fuses, and butt connectors. Still need header pins and spade/faston terminals (butt connectors splice wire-to-wire; the battery blades want spades). |
| 8×8 LED matrix (face) | $5–10 | Brody's face. |
| M3 heat-set inserts | $10 | Fastening skeleton + shell. |

## 🛒 Still need — Power distribution

| Item | ~Cost | Notes |
|---|---|---|
| Buck converter → **5V** | $6–12 | **Rated 12–80V input** (the e-bike / EV type). A basic LM2596 tops out at 40V and will **pop on a 42V pack** — don't use one. Get 5V @ **5A** for the Pi. HAVE ONE: got a 10-60V in / 5V 5A unit; its 60V input ceiling clears the ~42V pack. |
| Buck converter → **12V** | $5–8 | Same 12–80V input rating. Runs the salvaged LED bars + face. |
| Thick wire, 10–12 AWG (red + black) | $10 | Main battery / motor power runs. |
| Crimp tool (if not owned) | $15–20 | For clean, safe spade crimps. |

## 🛒 Still need — Brain & comms

| Item | ~Cost | Notes |
|---|---|---|
| Raspberry Pi 4 (4GB) or Pi 5 | $55–80 | Onboard brain for voice + roaming. **Pi Zero 2 W (~$15)** is enough if the AI lives on a server and the Pi just relays commands + streams the camera. |
| microSD card, 32GB+ | $8 | The Pi's OS. |
| USB-C Pi power supply | $10 ⚪ | Handy for bench setup (on the robot it runs off the 5V buck). |
| USB cable (Arduino ↔ Pi) | — | Powers the Arduino **and** carries the commands. Likely already in your Arduino kit. |
| 1k resistor (UART level shift) | ~$0 | Single series resistor on the Arduino TX to board RX line (board TX to Arduino RX goes direct). Probably already in the kit. |

## 🛒 Still need — Voice

| Item | ~Cost | Notes |
|---|---|---|
| MAX98357A I2S amp | $6 | Cleanest amp for the Pi (no headphone jack needed). Drives the 4Ω speaker directly. |
| — or PAM8403 amp | $2 ⚪ | Analog alternative if you use a USB sound card / a Pi with a 3.5mm jack. |

*No DFPlayer needed — since the Pi is already onboard for the AI, it handles both the thinking
**and** the talking.*

## 🛒 Still need — Lights

| Item | ~Cost | Notes |
|---|---|---|
| Logic-level MOSFET (e.g. IRLZ44N) or MOSFET module | $1–3 | Lets the Arduino switch/dim the 12V LED bars — you can't drive them off a pin directly. |

## 🛒 Still need — Vision

| Item | ~Cost | Notes |
|---|---|---|
| USB webcam | $15 | **Main plan.** Plugs into the Pi; OpenCV reads it directly. Most include a **mic** — which Brody needs to listen / hold conversations anyway. Eyes + ears in one part. |
| — or Pi Camera Module | $15–25 ⚪ | Ribbon cable straight to the Pi's CSI port. Higher quality + lower latency, but no mic. |

*The old Android phone was going to be a WiFi IP-webcam, but it's water-damaged and dead. Don't
bother salvaging its camera sensor — phone cameras use proprietary MIPI connectors that won't
interface with a Pi.*

## 🛒 Still need — Motion & sensing

| Item | ~Cost | Notes |
|---|---|---|
| Caster wheel | $5–10 | The 3rd contact point (self-balancing is a someday problem). |
| Servo — head tilt (SG90 or MG996R) | dropped | Not doing servos. Head is fixed/posable, arms are posable and friction-held (nyloc + nylon washers, bigger bolts at the shoulders). |
| Ultrasonic sensors (HC-SR04) | $2 ea | Obstacle detection for roaming. Grab 2–3. |
| PCA9685 16-ch servo driver | not needed | No servos in the build. |

## ⚪ Software (all free)

- **pyOCD** — flash the MM32 motor boards (repos + gotchas are in the README).
- **OpenCV** — camera / vision so Brody can "see."
- **Piper** or **espeak** (offline TTS), or a cloud voice — the actual talking.
- **Arduino IDE** — motor / servo / light firmware.

---

## Buy-this-first order (to keep momentum)

1. **Finish the motor test** — fuse, spades, wire, headers (on order). Flash the boards with the
   probe → spin the wheels from the PC, then from the Arduino. *(Battery multimeter check first —
   per the ground rules.)*
2. **Power distribution** — the two buck converters (5V + 12V). Now everything runs off the one pack.
3. **Lights** — MOSFET + confirm the LED bars' voltage (the 12V test). Wire up the 8×8 face.
4. **Brain** — Raspberry Pi + microSD. Get it talking to the Arduino over USB.
5. **Voice** — MAX98357A amp + the salvaged speaker. Brody's first words.
6. **Vision + roaming** — phone-as-camera (or a webcam), ultrasonic sensors, head servo.

## Rough budget for what's left

- **Essentials** (2 bucks, Pi, amp, MOSFET, caster, a servo, sensors, USB webcam, wire): **~$120–180**
- **Optional / backup** (webcam, crimp tool, extra servos): another **~$40–60**

---

*Companion to the [README](README.md), which has the story, the design, and the firmware details.
Update this as parts land.*
