# Batocera ZeDMD 128x32 P4 LED Marquee

A USB-driven **128x32 P4 HUB75 LED marquee for Batocera**, using an **ESP32-WROOM-32U running ZeDMD**.

This project uses two 64x32 P4 LED panels side-by-side to create a 128x32 arcade marquee capable of displaying:

- System artwork
- Game-specific artwork
- Static PNG marquees
- Animated GIF marquees
- Pixelcade LED artwork
- Random startup animations
- Custom screensaver animation
- Custom shutdown animation
- Real DMD output for Virtual Pinball

Communication between Batocera and the ESP32 is entirely over **USB**.

**ESP32 Wi-Fi and WLED are not required.**

---

# Hardware

This project was developed and tested around the specific hardware below.

## P4 HUB75 LED Panels

**Quantity: 2**

Panels:

https://amzn.to/4wx3bSe

The software and ZeDMD configuration are set up for:

```text
Panel type:          P4 HUB75
Panel resolution:    64x32
Number of panels:    2
Layout:              Horizontal
Combined resolution: 128x32
Total pixels:        4096
Pixel pitch:         4 mm
```

Physical layout:

```text
+--------------------+--------------------+
|                    |                    |
|      PANEL 1       |      PANEL 2       |
|       64x32        |       64x32        |
|                    |                    |
+--------------------+--------------------+

              128x32 TOTAL
```

> [!IMPORTANT]
> This project is configured specifically for **two 64x32 P4 panels arranged horizontally**.
>
> Different panel resolutions, scan rates, densities, or panel counts may require changes to the ZeDMD configuration.

---

## HUB75 Cables

Cables:

https://amzn.to/4fWnVOa

The HUB75 connections are daisy-chained for **data**:

```text
+-------------+
|    ESP32    |
+------+------+
       |
       | HUB75
       v
+-------------+
|   PANEL 1   |
|     IN      |
+------+------+
       |
       | PANEL 1 OUT
       |     to
       | PANEL 2 IN
       v
+-------------+
|   PANEL 2   |
|     IN      |
+-------------+
```

The HUB75 cables carry the display signals and ground reference.

They do **not** provide the primary 5V power for the LED panels.

---

## ESP32 Controller

Controller:

https://amzn.to/4gdi1XE

Use the **ESP32-WROOM-32U board specified above**.

This is the controller used with this build and the ZeDMD firmware configuration documented here.

Connection to Batocera:

```text
+----------------+
|  BATOCERA PC   |
+-------+--------+
        |
        | USB
        v
+----------------+
| ESP32-WROOM-32U|
+-------+--------+
        |
        | HUB75
        v
+----------------+
|   LED PANELS   |
+----------------+
```

The USB connection provides:

- Power for the ESP32
- Serial communication between Batocera and ZeDMD

The ESP32 does **not** supply power to the LED panels.

---

# Power Supply

Power supply:

https://amzn.to/45uAn1z

This build uses the linked **BTF-LIGHTING Class 2 wall-plug power supply**.

Specifications:

```text
Input:           100-240V AC
Output:          5V DC
Maximum current: 10A
Maximum power:   50W
```

The power supply plugs directly into a standard wall receptacle.

There is:

- No exposed mains wiring
- No L/N terminal wiring
- No separate AC input wiring
- No open-frame power supply

The adapter provides a DC barrel plug.

The linked supply also includes a **barrel-jack-to-screw-terminal adapter**, which is used to connect the panel power wires.

---

# Panel Power Wiring

The power path is:

```text
+---------------------+
|   WALL RECEPTACLE   |
+----------+----------+
           |
           v
+---------------------+
| BTF-LIGHTING PSU    |
| 5V / 10A / 50W      |
+----------+----------+
           |
           | DC barrel plug
           v
+---------------------+
| BARREL-TO-SCREW     |
| TERMINAL ADAPTER    |
|                     |
|       +       -     |
+-------+-------+-----+
        |       |
       +5V     GND
```

> [!IMPORTANT]
> Check the `+` and `-` markings on the supplied screw-terminal adapter before connecting the panels.

---

## Connecting Both Panels

Both LED panels are powered from the same 5V supply.

The panels are connected **in parallel**.

```text
                +--------------------> PANEL 1 RED
                |
ADAPTER (+) ----+
                |
                +--------------------> PANEL 2 RED


                +--------------------> PANEL 1 BLACK
                |
ADAPTER (-) ----+
                |
                +--------------------> PANEL 2 BLACK
```

Connection table:

| Adapter | Connection |
|---|---|
| `+` | Panel 1 RED (+5V) |
| `+` | Panel 2 RED (+5V) |
| `-` | Panel 1 BLACK (GND) |
| `-` | Panel 2 BLACK (GND) |

Both red panel wires connect to the positive terminal.

Both black panel wires connect to the negative terminal.

Make sure the screw terminals securely clamp the conductors.

---

## ESP32 Power

The ESP32 is powered separately by the Batocera computer through USB:

```text
+----------------+
|  BATOCERA PC   |
+-------+--------+
        |
        | USB
        v
+----------------+
|     ESP32      |
+----------------+
```

Do **not** power the P4 panels from the ESP32:

```text
ESP32 5V ----X----> P4 PANELS
```

The panels receive their 5V power directly from the BTF-LIGHTING power supply.

---

## ESP32 Ground

No additional ESP32-to-power-supply ground jumper is required in this build.

The HUB75 connection already provides ground conductors between the controller and Panel 1.

The working arrangement is:

```text
DATA PATH

+-------------+
| BATOCERA PC |
+------+------+
       |
       | USB
       v
+-------------+
|    ESP32    |
+------+------+
       |
       | HUB75
       v
+-------------+
|   PANEL 1   |
+------+------+
       |
       | HUB75
       v
+-------------+
|   PANEL 2   |
+-------------+


POWER PATH

+-------------+
| 5V / 10A    |
| 50W PSU     |
+------+------+
       |
       | DC barrel
       v
+-------------+
| TERMINAL    |
| ADAPTER     |
+------+------+
       |
       +-------------> PANEL 1 POWER
       |
       +-------------> PANEL 2 POWER
```

No separate:

```text
ESP32 GND ----------> PSU NEGATIVE
```

jumper is used in the documented working configuration.

---

# Complete Hardware Wiring

```text
POWER

+-----------------------+
|    WALL RECEPTACLE    |
+-----------+-----------+
            |
            v
+-----------------------+
| BTF-LIGHTING          |
| 5V / 10A / 50W PSU    |
+-----------+-----------+
            |
            | DC barrel
            v
+-----------------------+
| BARREL-TO-SCREW       |
| TERMINAL ADAPTER      |
+-----------+-----------+
            |
       +----+----+
       |         |
       v         v
+-----------+ +-----------+
| PANEL 1   | | PANEL 2   |
| 5V POWER  | | 5V POWER  |
+-----------+ +-----------+


CONTROL / DATA

+-----------------------+
|      BATOCERA PC      |
+-----------+-----------+
            |
            | USB
            v
+-----------------------+
|   ESP32-WROOM-32U     |
+-----------+-----------+
            |
            | HUB75
            v
+-----------------------+
|       PANEL 1         |
|        64x32          |
+-----------+-----------+
            |
            | HUB75 OUT
            |     to
            | PANEL 2 IN
            v
+-----------------------+
|       PANEL 2         |
|        64x32          |
+-----------------------+

Combined display: 128x32
```

---

# ESP32 to HUB75 Pinout

The ZeDMD configuration uses the following ESP32 GPIO mapping:

| HUB75 Signal | ESP32 GPIO |
|---|---:|
| R1 | GPIO 25 |
| G1 | GPIO 26 |
| B1 | GPIO 27 |
| R2 | GPIO 14 |
| G2 | GPIO 12 |
| B2 | GPIO 13 |
| A | GPIO 23 |
| B | GPIO 19 |
| C | GPIO 5 |
| D | GPIO 17 |
| E | GPIO 22 |
| CLK | GPIO 16 |
| LAT / STROBE | GPIO 4 |
| OE | GPIO 15 |
| GND | ESP32 GND |

A typical HUB75 signal arrangement is:

```text
+-------+-------+
|  R1   |  G1   |
+-------+-------+
|  B1   |  GND  |
+-------+-------+
|  R2   |  G2   |
+-------+-------+
|  B2   |  GND  |
+-------+-------+
|  A    |  B    |
+-------+-------+
|  C    |  D    |
+-------+-------+
|  CLK  |  LAT  |
+-------+-------+
|  OE   |  GND  |
+-------+-------+
```

> [!CAUTION]
> Match the **printed signal labels on your actual panel/HUB75 connection**.
>
> Do not assume connector position alone, because HUB75 panel configurations can vary.

---

# Hardware Shopping List

| Component | Qty | Link |
|---|---:|---|
| 64x32 P4 HUB75 LED Panel | 2 | https://amzn.to/4wx3bSe |
| HUB75 Cables | As needed | https://amzn.to/4fWnVOa |
| ESP32-WROOM-32U | 1 | https://amzn.to/4gdi1XE |
| BTF-LIGHTING 5V / 10A / 50W Power Supply | 1 | https://amzn.to/45uAn1z |

---

# Software Architecture

```text
+--------------------------+
|         BATOCERA         |
|                          |
| - EmulationStation       |
| - Pixelcade artwork      |
| - dmd_real               |
| - dmd-play               |
| - Startup animations     |
| - Screensaver            |
| - Shutdown animation     |
+------------+-------------+
             |
             | USB
             v
+--------------------------+
|       ZeDMD 5.1.7        |
|     ESP32-WROOM-32U      |
+------------+-------------+
             |
             | HUB75
             v
+------------+-------------+
|            |             |
|  PANEL 1   |   PANEL 2   |
|   64x32    |    64x32    |
|            |             |
+------------+-------------+

          128x32 TOTAL
```

---

# Installation Files

The production installation uses **two ZIP files**.

## 1. Firmware Flash Package

```text
zedmd-5.1.7-128x32-flash-only.zip
```

This package handles the ESP32 firmware separately from the Batocera software installation.

It flashes the known-working:

```text
ZeDMD 5.1.7
128x32
USB transport
```

---

## 2. Batocera Marquee Package

```text
batocera-zedmd-marquee-working-combo.zip
```

This package combines the last known-working scripts from:

```text
batocera-zedmd-marquee-v3.0
+
batocera-zedmd-marquee-v3.2.1
```

It handles:

- Pixelcade artwork download
- Pixelcade artwork update checks
- Batocera native DMD artwork paths
- `dmd_real`
- ZeDMD USB configuration
- System marquees
- Game marquees
- Static artwork
- Animated artwork
- Startup animations
- Screensaver animation
- Shutdown animation
- Virtual Pinball Real DMD compatibility

The experimental custom-firmware boot-animation builds are **not included**.

---

# Installation Overview

Install in this order:

```text
+----------------------------+
| 1. FLASH ZeDMD FIRMWARE    |
+-------------+--------------+
              |
              v
+----------------------------+
| ZeDMD 5.1.7 / 128x32       |
+-------------+--------------+
              |
              v
+----------------------------+
| Set Transport = USB        |
| Exit ZeDMD setup           |
+-------------+--------------+
              |
              v
+----------------------------+
| 2. INSTALL BATOCERA COMBO  |
|                            |
| v3.0 base                  |
| +                          |
| v3.2.1 animations          |
+-------------+--------------+
              |
              v
+----------------------------+
| 3. REBOOT BATOCERA         |
+-------------+--------------+
              |
              v
+----------------------------+
| 4. TEST MARQUEE            |
+----------------------------+
```

> [!IMPORTANT]
> **The reboot after installing the Batocera package is required.**
>
> During the original working setup, the ESP32 was detected over USB but the marquee did not begin operating correctly until Batocera had been completely restarted.

---

# Step 1 - Connect the ESP32

Connect the ESP32 to the Batocera PC over USB.

Check detection:

```bash
ls -l /dev/ttyUSB*
```

Normally:

```text
/dev/ttyUSB0
```

Depending on the USB interface, the controller could instead appear as:

```text
/dev/ttyUSB1
```

or:

```text
/dev/ttyACM0
```

---

# Step 2 - Flash ZeDMD

Copy:

```text
zedmd-5.1.7-128x32-flash-only.zip
```

to the Batocera `share` folder.

SSH into Batocera.

Go to:

```bash
cd /userdata
```

Extract:

```bash
unzip -o zedmd-5.1.7-128x32-flash-only.zip
```

Enter the folder:

```bash
cd zedmd-5.1.7-128x32-flash-only
```

Set permissions:

```bash
chmod +x flash-zedmd.sh
```

Flash:

```bash
PORT=/dev/ttyUSB0 ./flash-zedmd.sh
```

When prompted, type:

```text
ZEDMD
```

The script flashes the known-working **ZeDMD 5.1.7 128x32 firmware** to the ESP32.

---

# Step 3 - Configure ZeDMD

After flashing, the LED panels may display the ZeDMD configuration interface.

Set:

```text
Transport: USB
```

Confirm:

```text
Panel width:  128
Panel height: 32
```

Check the RGB order.

The test colors should display correctly:

```text
RED   -> Red
GREEN -> Green
BLUE  -> Blue
```

Adjust RGB order if necessary.

When finished, select:

```text
Exit
```

> [!IMPORTANT]
> Do not leave ZeDMD sitting in its setup interface.
>
> Select **Exit** so the controller can enter normal USB communication mode.

---

# Known-Working ZeDMD Configuration

| Setting | Value |
|---|---|
| Firmware | ZeDMD 5.1.7 |
| Controller | ESP32-WROOM-32U |
| Display | 128x32 |
| Panels | 2 x 64x32 P4 |
| Layout | Horizontal |
| Transport | USB |
| Typical device | `/dev/ttyUSB0` |
| USB package size | 512 |
| Minimum panel refresh | 60 Hz |
| Wi-Fi | Not required |

> [!NOTE]
> Higher refresh values were experimented with later.
>
> **60 Hz is retained here as the known-working configuration.**

---

# Step 4 - Install Batocera Marquee Software

Copy:

```text
batocera-zedmd-marquee-working-combo.zip
```

to the Batocera `share` folder.

SSH into Batocera.

Go to:

```bash
cd /userdata
```

Extract:

```bash
unzip -o batocera-zedmd-marquee-working-combo.zip
```

Enter:

```bash
cd batocera-zedmd-marquee-working-combo
```

Set permissions:

```bash
chmod +x install.sh scripts/*.sh
```

Run:

```bash
./install.sh
```

---

# What the Installer Does

The combined installer reproduces the working **v3.0 base installation** and applies the working **v3.2.1 animation layer**.

It:

1. Removes the older WLED/custom USB marquee hooks.
2. Installs/checks the Pixelcade artwork tooling.
3. Downloads the Pixelcade artwork collection if required.
4. Checks the existing artwork collection for updates.
5. Synchronizes artwork into Batocera's native DMD paths.
6. Enables `dmd_real`.
7. Starts `dmd_real`.
8. Applies the known-working ZeDMD settings.
9. Sets USB packet size to **512 bytes**.
10. Sets minimum panel refresh to **60 Hz**.
11. Installs the v3.2.1 animation scripts.
12. Configures the Dr. Mario screensaver.
13. Configures the Super Mario All-Stars shutdown animation.
14. Configures the random startup animation.
15. Adds the once-per-boot artwork update check.
16. Preserves the native Real DMD architecture for Virtual Pinball.

---

# Step 5 - Reboot

After installation completes:

```bash
reboot
```

> [!IMPORTANT]
> **Do not troubleshoot the marquee before performing this reboot.**
>
> The reboot was necessary during the original working installation.

---

# Step 6 - Test

After Batocera starts again, browse through several systems and games.

Expected system behavior:

```text
SYSTEM SELECTED
      |
      v
SYSTEM MARQUEE
```

Expected game behavior:

```text
GAME SELECTED
      |
      v
GAME MARQUEE
```

---

# Verify dmd_real

Check:

```bash
batocera-services status dmd_real
```

Restart if necessary:

```bash
batocera-services stop dmd_real
batocera-services start dmd_real
```

---

# Verify ZeDMD

`dmd_real` may hold the USB connection.

Stop it temporarily:

```bash
batocera-services stop dmd_real
```

Query ZeDMD:

```bash
zedmd-client -i
```

Expected basics:

```text
Firmware:         5.1.7
CPU:              ESP32
Transport:        USB
Device:           /dev/ttyUSB0
Panel width:      128
Panel height:     32
USB package size: 512
Minimum refresh:  60
```

Restart the service:

```bash
batocera-services start dmd_real
```

---

# Pixelcade Artwork

The project uses Pixelcade's LED-specific artwork collection.

This artwork is particularly useful for a 128x32 LED display because it is designed around low-resolution LED marquees rather than simply displaying full-resolution scraped marquee images.

The initial artwork download can take some time.

Allow the first download to complete.

---

# Artwork Locations

Pixelcade source artwork:

```text
/userdata/system/wled-marquee/pixelcade-art/
```

Batocera system DMD artwork:

```text
/userdata/system/dmd/systems/
```

Batocera game DMD artwork:

```text
/userdata/system/dmd/games/
```

User animations:

```text
/userdata/system/zedmd-marquee/user-animations/
```

Screensaver:

```text
/userdata/system/dmd/screensaver.gif
```

---

# Automatic Artwork Updates

The working v3.0 updater checks the Pixelcade artwork collection **once per boot**.

It waits approximately:

```text
30 seconds
```

after startup before checking.

```text
BATOCERA BOOTS
      |
      v
WAIT ~30 SECONDS
      |
      v
CHECK ARTWORK
      |
      +---------- CURRENT ----------> DONE
      |
      +------ UPDATE AVAILABLE
                    |
                    v
               DOWNLOAD
                    |
                    v
                  DONE
```

The complete artwork library should not be downloaded again on every boot when no update is required.

---

# Manual Artwork Update

Check for an update:

```bash
/userdata/system/zedmd-marquee/artwork-update.sh --check
```

Force an update:

```bash
/userdata/system/zedmd-marquee/artwork-update.sh --force-update
```

Refresh the Batocera artwork links:

```bash
/userdata/system/zedmd-marquee/sync-artwork.sh --updated
```

---

# Game Marquees

When a game is selected:

```text
GAME SELECTED
      |
      v
FIND MATCHING ARTWORK
      |
      v
DISPLAY PNG OR GIF
```

Both static PNG and animated GIF artwork are supported.

---

# Static Artwork During Gameplay

The working configuration avoids replacing valid game artwork with a scrolling text title when a game launches.

Desired behavior:

```text
GAME SELECTED
      |
      v
GAME ARTWORK
      |
      v
LAUNCH GAME
      |
      v
NO NEW MARQUEE COMMAND
      |
      v
GAME ARTWORK REMAINS
```

This is particularly useful with static artwork because the image can remain displayed without continuously retransmitting it.

---

# Startup Animation

After Batocera starts, the v3.2.1 animation layer waits approximately:

```text
6 seconds
```

It chooses **one random local GIF**.

That same animation plays **two times total**.

```text
BATOCERA STARTS
      |
      v
WAIT ~6 SECONDS
      |
      v
CHOOSE RANDOM GIF
      |
      v
PLAY #1
      |
      v
PLAY #2
      |
      v
NORMAL MARQUEE OPERATION
```

A different random GIF is **not** selected between the first and second play.

---

# Dr. Mario Screensaver

The configured screensaver is the Dr. Mario animation.

The setup searches for filenames including:

```text
drnario.gif
drmario.gif
*dr*mario*.gif
*doctor*mario*.gif
```

The selected animation is configured as:

```text
/userdata/system/dmd/screensaver.gif
```

Expected behavior:

```text
BATOCERA IDLE
      |
      v
SCREENSAVER STARTS
      |
      v
DR. MARIO GIF
```

---

# Shutdown Animation

The preferred shutdown animation is:

```text
supermarioallstartheend.gif
```

The installer can also search compatible filename variations.

Expected behavior:

```text
SHUTDOWN SELECTED
       |
       v
SUPER MARIO ALL-STARS
"THE END" GIF
       |
       v
PLAY ONCE
       |
       v
BATOCERA POWERS OFF
```

---

# Re-detect Animation Files

If animations are added or moved:

```bash
/userdata/system/zedmd-marquee/configure-animations.sh
```

This re-scans the available artwork for:

- Dr. Mario
- Super Mario All-Stars ending
- Other available startup animations

---

# Virtual Pinball / Real DMD

This project uses native ZeDMD and Batocera's `dmd_real` architecture.

That allows the same physical 128x32 display to be used as a Real DMD by compatible Virtual Pinball software.

Normal Batocera use:

```text
+-------------------+
| EMULATIONSTATION  |
+---------+---------+
          |
          v
+-------------------+
|     dmd_real      |
+---------+---------+
          |
          v
+-------------------+
|       ZeDMD       |
+---------+---------+
          |
          v
+-------------------+
| 128x32 LED DISPLAY|
+-------------------+
```

Virtual Pinball:

```text
+-------------------+
| VIRTUAL PINBALL   |
| / PINMAME         |
+---------+---------+
          |
          v
+-------------------+
| REAL DMD OUTPUT   |
+---------+---------+
          |
          v
+-------------------+
|       ZeDMD       |
+---------+---------+
          |
          v
+-------------------+
| 128x32 LED DISPLAY|
+-------------------+
```

---

# Diagnostics

After the required reboot:

```bash
/userdata/system/zedmd-marquee/diagnose.sh
```

Animation diagnostics:

```bash
/userdata/system/zedmd-marquee/diagnose-animations.sh
```

---

# Troubleshooting

## ESP32 Not Detected

Check:

```bash
ls -l /dev/ttyUSB*
```

and:

```bash
ls -l /dev/ttyACM*
```

Normally the device is:

```text
/dev/ttyUSB0
```

---

## dmd_real Not Running

Check:

```bash
batocera-services status dmd_real
```

Restart:

```bash
batocera-services stop dmd_real
batocera-services start dmd_real
```

---

## ZeDMD Client Cannot Connect

Stop `dmd_real` first:

```bash
batocera-services stop dmd_real
```

Then:

```bash
zedmd-client -i
```

After testing:

```bash
batocera-services start dmd_real
```

---

## Nothing Works Immediately After Installation

**Reboot first.**

```bash
reboot
```

This was required during the original working installation.

---

## Check Artwork Count

```bash
find /userdata/system/wled-marquee/pixelcade-art \
-type f -iname "*.png" -o -iname "*.gif" | wc -l
```

---

## Check Animated Artwork

```bash
find /userdata/system/wled-marquee/pixelcade-art \
-type f -iname "*.gif" | wc -l
```

---

## Find Dr. Mario

```bash
find /userdata/system/wled-marquee/pixelcade-art \
-type f \
-iname "drnario.gif" \
-o -iname "drmario.gif" \
-o -iname "*dr*mario*.gif" \

```

---

## Find Shutdown Animation

```bash
find /userdata/system/wled-marquee/pixelcade-art \
-type f -iname "supermarioallstartheend.gif"
```

---

# Dependencies

## Required

- Batocera 42+
- ESP32-WROOM-32U
- 2 x 64x32 P4 HUB75 LED panels
- HUB75 cables
- BTF-LIGHTING 5V / 10A / 50W power supply
- Barrel-to-screw-terminal adapter
- USB connection between Batocera and ESP32
- Internet access during initial installation
- `python3`
- `curl` or `wget`
- `unzip`
- `dmd_real`
- `dmd-play`
- `zedmd-client`

## Not Required

The known-working production setup does **not** require:

```text
Git
PlatformIO
WLED
ESP32 Wi-Fi
```

---

# Complete Installation Sequence

```text
+--------------------------------+
|       BUILD HARDWARE           |
|                                |
| 2 x 64x32 P4 panels            |
| ESP32-WROOM-32U                |
| 5V / 10A / 50W PSU             |
+---------------+----------------+
                |
                v
+--------------------------------+
|       FLASH FIRMWARE           |
|                                |
| zedmd-5.1.7-128x32-            |
| flash-only.zip                 |
+---------------+----------------+
                |
                v
+--------------------------------+
|       ZeDMD 5.1.7              |
|                                |
| 128x32                         |
| Transport: USB                 |
| Exit setup                     |
+---------------+----------------+
                |
                v
+--------------------------------+
|    INSTALL BATOCERA PACKAGE    |
|                                |
| v3.0 base + v3.2.1 animations  |
+---------------+----------------+
                |
                v
+--------------------------------+
|           dmd_real             |
|                                |
| Pixelcade artwork              |
| 512-byte USB packets           |
| 60 Hz minimum refresh          |
| Random startup GIF x2          |
| Dr. Mario screensaver          |
| Mario shutdown animation       |
+---------------+----------------+
                |
                v
+================================+
|        REBOOT REQUIRED         |
+================================+
                |
                v
+--------------------------------+
|        TEST MARQUEES           |
+---------------+----------------+
                |
                v
+--------------------------------+
|             DONE               |
+--------------------------------+
```

---

# Production Files

The production setup consists of these two packages:

```text
zedmd-5.1.7-128x32-flash-only.zip

batocera-zedmd-marquee-working-combo.zip
```
