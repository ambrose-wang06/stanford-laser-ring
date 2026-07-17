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

## Running the Code

The Arduino code (in [`/code`](./code)) controls the 8 laser segments independently and listens for collision-warning signals.

1. Install the [Arduino IDE](https://www.arduino.cc/en/software).
2. Connect the Arduino to your computer via USB.
3. Open the sketch from the `/code` folder in this repo.
4. In the Arduino IDE, select your board under **Tools > Board** and the correct port under **Tools > Port**.
5. Click **Upload** to flash the code to the Arduino.
6. Once uploaded, power the ring on using the remote sensor. The ring should project as a complete circle on the gantry when there is no obstruction.
7. To enable predictive (TPS-driven) segment flashing, connect the Arduino to the PC running the collision-prediction integration and follow the setup steps in [`/code`](./code) for linking it to your data source.

> ⚠️ This section is a general outline — add the exact sketch filename, required libraries, and any pin mapping/config specific to your build once the `/code` folder is finalized, so future users can follow it exactly.

## Repository Structure

```
├── STL/           # 3D printable parts (see Bill of Materials)
├── code/          # Arduino sketch(es) for laser ring control
└── README.md
```

## Contact

Questions about this project can be directed to the Stanford Radiation Oncology team.
