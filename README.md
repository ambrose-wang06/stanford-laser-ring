# Stanford Laser Ring

Real-Time Laser Collision Visualization System — Department of Radiation Oncology, Stanford University.

This repository contains the STL files needed to 3D print the laser ring, the wiring/assembly reference, and the Arduino code used to run the system.

> Developed by Ambrose Wang, Piotr Dubrowski, Aisha Burns, Amy Yu, and Lawrie Skinner — Department of Radiation Oncology, Stanford University, Palo Alto, CA.

## Overview

The laser ring is an isometric ring of 40 laser diodes (5V, 650nm, 5mW) that forms a visible boundary around the linac gantry to help radiation therapists spot potential collisions between the patient/immobilization devices and the gantry or on-board imaging system — without needing a manual dry run.

The 40 diodes are split into **8 independently controlled segments** (5 diodes each). If a segment's beam is broken by the patient or equipment, therapists get an immediate visual cue. The system can also flash a specific segment based on predicted collisions coming from the treatment planning system.

## Bill of Materials

### 3D Printed Parts

| Part | Quantity |
|---|---|
| `control_box_back_bracket_left` | 1 |
| `control_box_back_bracket_right` | 1 |
| `control_box_bottom` | 1 |
| `control_box_top` | 1 |
| `control_box_connector_left` | 1 |
| `control_box_connector_right` | 1 |
| `diode_ball_joint` | 40 |
| `laser_ring_back_bracket` | 7 |
| `laser_ring_segment_bottom` | 8 |
| `laser_ring_segment_top` | 8 |

All parts were designed in Onshape and printed in PLA on a Prusa i3 MK3 for structural rigidity. STL files are in the [`/STL`](./STL) folder.

### Other Materials

| Item | Quantity | Notes |
|---|---|---|
| Laser diodes (5V, 650nm, 5mW) | 40 | 5 per segment, 8 segments |
| Arduino (microcontroller) | 1 | Controls the 8 segments independently |
| Wiring | — | Each group of 8 diodes wired in its own parallel branch for independent segment control and easy fault isolation |
| Screws (ball-joint reinforcement) | — | Secures each diode's alignment after calibration |
| Remote sensor / power switch | 1 | Mounts through the aperture in the control box for remote on/off |

## Assembly

1. **Print all parts** listed in the Bill of Materials above using the STL files in this repo.
2. **Assemble the ring**: mount each laser diode into a `diode_ball_joint`, then secure the ball joints into the `laser_ring_segment_top`/`laser_ring_segment_bottom` pairs to build all 8 segments.
3. **Join the segments**: connect adjacent segments using the `laser_ring_back_bracket` pieces (7 total) to form the full ring.
4. **Build the control box**: assemble `control_box_top`, `control_box_bottom`, and the left/right back brackets and connectors around the Arduino and wiring.
5. **Wire the diodes**: wire each segment's 5 diodes in their own parallel branch back to the control box, so each segment can be individually controlled and faulty diodes can be isolated without affecting other segments.
6. **Mount the remote sensor** through the aperture in the upper-left corner of the control box for power on/off.
7. **Mount the assembled ring** vertically on the wall facing the linac, with the beams projected horizontally toward the gantry.

## Calibration / QA

1. Mount the calibration ring on the gantry head, parallel to the laser ring.
2. Power on the system and check that each laser beam passes through its corresponding aperture on the calibration ring.
3. For any diode off by more than 2 mm at the calibration ring, use the adjustment screw on that diode's ball-joint housing to re-align it until the beam passes through its aperture.
4. Repeat daily until stability is established; the system can move to monthly QA checks once alignment drift is confirmed to stay under 1 mm over time.

## Software

The predictive warning system has **three parts** that hand data off to each other in sequence:

1. **`RingCollisionDetection_Script/`** — an Eclipse Scripting API (ESAPI) plugin that runs *inside Varian Eclipse*. When run against an open plan, it checks the patient's BODY/EXTERNAL/SUPPORT structures against the linac geometry, works out which of the 8 laser segments (if any) are predicted to collide, and writes the result to a shared JSON file named `{MRN}.json`.
2. **`RingCollCommunicator/`** — a Windows console app (.NET) that runs on the PC connected to the Arduino via USB. It auto-detects the Arduino's COM port, looks up which patient is currently "In Progress" on the treatment machine in ARIA, reads that patient's `{MRN}.json` file, and sends the list of colliding segment numbers to the Arduino over serial.
3. **`Arduino Code/SerialOnly_5_29_2026/`** — the sketch that runs on the Arduino itself, described below.

### How a treatment session runs

1. Therapist points an IR remote at the ring and presses the power button.
2. The Arduino turns **all 8 segments on solid** and sends `ReadyForSegments` over serial.
3. The Communicator app receives that, looks up the in-progress patient in ARIA, reads their collision JSON, and sends back either `CLEAR` (no predicted collisions — lasers stay solid) or a comma-separated list of segment numbers (e.g. `3,4`).
4. The Arduino sets those specific segments to **blink** while the rest stay solid, giving the therapist a location-specific warning.
5. Pressing the IR remote again turns everything off.

### Setup

**1. Eclipse Scripting API script**
- Add `RingCollisionDetection.cs` and `CollisonDetection.cs` to your Eclipse Scripting API project (references `VMS.TPS.Common.Model.API`, `Newtonsoft.Json`) and get it approved/verified per your institution's Eclipse scripting workflow.
- Set the shared `FolderPath` in `CollisionFileWriter` (top of `RingCollisionDetection.cs`) to the network location the Communicator app will also read from.

**2. RingCollCommunicator (PC app)**
- Requires **Newtonsoft.Json**, **System.Management**, and your institution's ARIA connectivity library (referenced as `RingAriaQ` / `RingAria` — not included in this repo; link against your own ARIA API library).
- Set `machineId` in `Program.cs` to the treatment machine's ID in ARIA.
- Set `FolderPath` in `CollisionFileReader` to the **same** shared folder used by the Eclipse script above.
- Build and run this app on the PC that's plugged into the Arduino via USB. It will wait until it detects an Arduino, then listen continuously.

**3. Arduino**
- Install the [Arduino IDE](https://www.arduino.cc/en/software) and the **IRremote** library (`IRremote.hpp`, by Armin Joachimsmeyer — install via Library Manager).
- Open `Arduino Code/SerialOnly_5_29_2026/SerialOnly_5_29_2026.ino`.
- Under **Tools > Board**, select your board; under **Tools > Port**, select the Arduino's port.
- Click **Upload**.
- Connect an IR receiver to pin 14, and the 8 segment relay lines to the pins listed in `relayPins[]` in the sketch.

### ⚠️ Known issues to resolve before deployment

- **`FolderPath` is a placeholder (redacted) in both `RingCollisionDetection.cs` and `Program.cs`.** These must be filled in with a real, matching path before the two apps can hand off data.
- **`machineId` in `Program.cs` is also a placeholder** — set this to your treatment machine's actual ARIA ID.
- **`Program.cs` references a variable `inProgressLA2Pt`** when logging the "in-progress patient found" message, but the variable that's actually populated is `inProgressPt`. As written, this will not compile — needs a fix (likely just renaming to `inProgressPt`).
- **Relay pins 0 and 1 are used as digital outputs** in the Arduino sketch (`relayPins[] = {5, 4, 3, 2, 1, 0, A6, A5}`), but pins 0/1 are the hardware Serial RX/TX lines on most Arduino boards. Since this sketch also uses `Serial` for communication with the Communicator app, double-check this doesn't cause conflicts on your specific board.

## Repository Structure

```
├── STL/                                    # 3D printable parts (see Bill of Materials)
├── RingCollisionDetection_Script/          # Eclipse Scripting API plugin (runs in Varian Eclipse)
│   ├── RingCollisionDetection.cs
│   └── CollisonDetection.cs
├── RingCollCommunicator/                   # Windows console app — bridges Eclipse/ARIA data to the Arduino
│   └── Program.cs
├── Arduino Code/
│   └── SerialOnly_5_29_2026/
│       └── SerialOnly_5_29_2026.ino        # Arduino sketch — relay/segment control + IR power toggle
└── README.md
```

## Contact

Questions about this project can be directed to the Stanford Radiation Oncology team.
