# Remote Lighting

A wireless dimmer system for LED lighting built from Arduinos and **nRF24L01 2.4 GHz radios**. A wall-mounted button **Controller**, built into a standard single-gang switch plate, sends commands to one or more **Receivers**. Each receiver drives up to three dimmable lighting zones with PWM.

<!-- PHOTOS: add photos of the remote and a receiver here, e.g. ![Remote](docs/remote.jpg) -->

## Features

- **On / Off** for the selected zones. Pressing *On* when already on turns all zones on, and pressing *Off* when already off turns all zones off.
- **Zones 1, 2, 3, or All**
- **Four brightness levels:** High, Medium, Low, and Night light
- Supports up to 4 receivers out of the box (more with a small config change)

## How it works

Every command is a single byte sent over the radio:

| Bits 7-6 | Bit 5 | Bits 4-2 | Bits 1-0 |
|---|---|---|---|
| unused | State: 1 = on, 0 = off | Zone: `001` = 1, `010` = 2, `100` = 3, `111` = all | Brightness: `11` High, `10` Med, `01` Low, `00` Night light |

Receivers map the brightness level to a PWM value (`5, 50, 150, 255`) on each selected zone's pin.

## Hardware

**Controller**
- Arduino (Uno/Nano-class) + nRF24L01 (CE pin 9, CSN pin 10, SPI)
- 10 buttons:

| Button | Pin |
|---|---|
| On | 7 |
| Off | 8 |
| High | A0 |
| Medium | A1 |
| Low | A2 |
| Night light | 6 |
| Zone 1 | 5 |
| Zone 2 | 4 |
| Zone 3 | 3 |
| All zones | 2 |

**Each receiver**
- Arduino + nRF24L01 (CE 9, CSN 10)
- MOSFETs driving the LED zones from PWM pins **3, 5 and 6** (zones 1-3)

## 3D-printed parts

![The printed parts](docs/parts.png)

The STL files are in `models/`:

| File | Part |
|---|---|
| `Wall_Plate.STL` | Controller face plate. Standard single-gang switch-plate size (113 × 69 mm), with letter-shaped cut-outs: `I` / `O` (on/off), `H` `M` `L` `N` (high, medium, low, night light) and `1` `2` `3` `A` (zones) |
| `Buttons.STL` | The matching letter-shaped buttons, printed as one sheet that sits behind the wall plate |
| `Arduino_holder.STL` | Mount for the controller's Arduino |
| `Back_Plate.STL` | Controller back plate |
| `Spacer.STL` | Small round spacer (9 mm) |
| `Light_Receiver_-_Box.STL` | Enclosure for a receiver |

GitHub can show the STL files in 3D: open one there to rotate it.

## Setup

1. Install the [RF24 library](https://github.com/nRF24/RF24).
2. `Transmitter.h` holds the radio addresses. **Keep the copies in `Controller/` and `Receiver/` identical.**
3. Flash the **Controller** sketch once.
4. For **each receiver**, set `RECEIVER_NUMBER` in `Receiver/Transmitter.h` to a unique value, then flash it.
5. For more than 4 receivers, add addresses to `recieverAddress[]` and update `RECEIVER_COUNT`.

Both sides use radio channel 8 and low TX power. Change `radio.setChannel()` in both sketches if channel 8 is busy where you are.

## Files

| Path | Purpose |
|---|---|
| `Controller/Controller.ino` | Remote: reads buttons, sends commands |
| `Receiver/Receiver.ino` | Receiver: listens and sets zone brightness |
| `*/Transmitter.h` | Shared radio addresses and receiver count |
| `models/` | 3D-printable parts for the controller and receiver |

## Known issues

- In `Controller.ino` `setup()`, the loop that configures the button pins is written `for(int i = 0; i++; i < 9)`. Its condition is `i++`, which is `0` on the first pass, so the loop never runs. The buttons still work because pins default to inputs, but the loop should be `for (int i = 0; i < 10; i++)`.
