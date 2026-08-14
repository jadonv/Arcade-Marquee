# Batocera ZeDMD 128×32 P4 LED Marquee

A USB-driven **128×32 P4 HUB75 LED marquee for Batocera**, using an **ESP32-WROOM-32U running ZeDMD**.

The project uses two 64×32 P4 LED panels to create a wide 128×32 arcade marquee capable of displaying:

- System artwork
- Game-specific artwork
- Static PNG marquees
- Animated GIF marquees
- Pixelcade LED artwork
- Custom startup animations
- Custom screensaver animation
- Custom shutdown animation
- Real DMD output for Virtual Pinball

Communication between Batocera and the ESP32 is entirely over **USB**.

No ESP32 Wi-Fi or WLED connection is required.

---

# Hardware

This project was developed and tested around the specific hardware below.

## P4 HUB75 LED Panels

**Quantity: 2**

Panels:

https://amzn.to/4wx3bSe

The software and ZeDMD configuration are specifically set up for:

```text
2 × 64×32 P4 HUB75 panels
1 row × 2 columns

Combined resolution: 128×32
Total pixels:        4096
Pixel pitch:         P4 / 4 mm
```

Physical layout:

```text
┌────────────────┬────────────────┐
│                │                │
│    PANEL 1     │    PANEL 2     │
│     64×32      │     64×32      │
│                │                │
└────────────────┴────────────────┘

          128 × 32 total
```

> [!IMPORTANT]
> This project is configured specifically for **two 64×32 P4 panels**.
>
> Different panel resolutions, scan rates, densities, or panel counts may require changes to the ZeDMD firmware/configuration.

---

## HUB75 Cables

Cables:

https://amzn.to/4fWnVOa

The panels are daisy-chained for **data**:

```text
ESP32
  │
  │ HUB75 ribbon
  ▼
Panel 1 IN
  │
  │ Panel 1 OUT
  ▼
Panel 2 IN
```

The HUB75 ribbon carries display signals.

It is **not the primary power connection for the LED panels**.

---

## ESP32 Controller

Controller:

https://amzn.to/4gdi1XE

Use the **ESP32-WROOM-32U board specified above**.

This is the board used for this build and was selected specifically around the memory requirements and behavior of the HUB75/ZeDMD setup.

The ESP32 connects directly to the Batocera computer over USB:

```text
Batocera PC
     │
     │ USB
     ▼
ESP32-WROOM-32U
     │
     │ HUB75
     ▼
P4 Panels
```

USB provides:

- ESP32 power
- ZeDMD serial communication

The ESP32 does **not** power the LED panels.

---

## 5V LED Panel Power Supply

Power supply:

https://amzn.to/45uAn1z

This project uses the linked **5V / 12A / 60W enclosed switching power supply**.

The supply powers both P4 panels.

```text
Output:          5V DC
Maximum current: 12A
Maximum power:   60W
```

Two 64×32 P4 panels can draw substantial current at high brightness, so the 12A supply provides useful headroom for the two-panel display.

---

# Power Supply Wiring

The linked PSU uses screw terminals for the AC input and 5V DC output.

The labels printed on the actual unit are the authoritative reference.

Typical labels are:

```text
L    = AC Live / Hot
N    = AC Neutral
⏚    = Protective Earth

V-   = 5V DC Negative / Ground
V+   = 5V DC Positive
```

The supply may provide more than one `V+` and `V-` terminal.

Those duplicate output terminals are internally common and make it easier to connect multiple loads.

---

## AC Input

Connect mains power to:

```text
AC LIVE / HOT ─────────► L
AC NEUTRAL ────────────► N
AC GROUND / EARTH ─────► ⏚
```

Diagram:

```text
       AC MAINS
          │
   ┌──────┼──────┐
   │      │      │
   ▼      ▼      ▼
   L      N      ⏚
┌────────────────────┐
│    5V / 12A PSU    │
│                    │
│   V-          V+   │
└────┬───────────┬───┘
     │           │
    GND         +5V
```

> [!CAUTION]
> The AC input side carries mains voltage.
>
> Use proper insulation, strain relief, grounding, enclosure, and electrical protection appropriate for your installation.
>
> Disconnect mains power before working on the wiring.

---

# Panel Power

Both LED panels are powered directly from the PSU's 5V output.

Each panel has its own red/black power connection.

```text
RED   = +5V
BLACK = V- / GND
```

Connect both panels in parallel:

```text
PSU V+ ─────────────► Panel 1 RED
PSU V+ ─────────────► Panel 2 RED

PSU V- ─────────────► Panel 1 BLACK
PSU V- ─────────────► Panel 2 BLACK
```

If the power supply has multiple output terminals, the cleanest arrangement is:

```text
PSU V+ #1 ──────────► Panel 1 RED
PSU V- #1 ──────────► Panel 1 BLACK

PSU V+ #2 ──────────► Panel 2 RED
PSU V- #2 ──────────► Panel 2 BLACK
```

This gives each panel a direct power connection back to the power supply.

---

## ESP32 Power

The ESP32-WROOM-32U is powered independently through its USB connection to the Batocera computer:

```text
Batocera PC
     │
     │ USB
     ▼
ESP32-WROOM-32U
```

Do **not** power the P4 panels from the ESP32.

```text
ESP32 5V ─────X────► P4 Panel
```

The panel load is far beyond what the ESP32's USB power connection is intended to provide.

---

## ESP32 Ground

No additional ground jumper from the ESP32 to the PSU is required in this build.

The HUB75 ribbon already includes ground connections between the controller and the LED panels.

The working hardware arrangement is:

```text
Batocera PC
     │
     │ USB
     ▼
ESP32
     │
     │ HUB75 ribbon
     ▼
Panel 1
     │
     │ HUB75 ribbon
     ▼
Panel 2
```

while panel power is handled separately:

```text
5V / 12A PSU
   │
   ├──── +5V / V- ───► Panel 1
   │
   └──── +5V / V- ───► Panel 2
```

There is no separate:

```text
ESP32 GND ─────► PSU V-
```

jumper required for the working configuration documented here.

---

# Complete Power Wiring

```text
                        AC MAINS
                            │
                   ┌────────┼────────┐
                   │        │        │
                   ▼        ▼        ▼
                   L        N        ⏚
                ┌──────────────────────┐
                │    5V / 12A PSU      │
                │                      │
                │   V-            V+   │
                └──┬──┬──────────┬──┬─┘
                   │  │          │  │
                   │  │          │  └────► Panel 2 RED
                   │  │          └───────► Panel 1 RED
                   │  │
                   │  └──────────────────► Panel 2 BLACK
                   └─────────────────────► Panel 1 BLACK


Batocera PC
     │
     │ USB
     ▼
ESP32-WROOM-32U
     │
     │ HUB75
     ▼
Panel 1
     │
     │ HUB75
     ▼
Panel 2
```

---

# ESP32 ↔ HUB75 Pinout

Use the following ZeDMD pin mapping:

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

For the **64×32 P4 1/16-scan panels used in this build**, the `E` address line is not required and can remain disconnected.

A typical HUB75 header looks approximately like:

```text
┌─────────────┐
│ R1 │ G1     │
│ B1 │ GND    │
│ R2 │ G2     │
│ B2 │ GND    │
│ A  │ B      │
│ C  │ D      │
│ CLK│ LAT    │
│ OE │ GND    │
└─────────────┘
```

> [!CAUTION]
> Always match the printed **signal labels** on your specific panel.
>
> HUB75 connector arrangements can vary. Do not wire solely by physical connector position.

---

# Hardware Shopping List

| Component | Qty | Link |
|---|---:|---|
| 64×32 P4 HUB75 LED Panel | **2** | https://amzn.to/4wx3bSe |
| HUB75 Cables | As needed | https://amzn.to/4fWnVOa |
| ESP32-WROOM-32U | **1** | https://amzn.to/4gdi1XE |
| 5V / 12A / 60W Power Supply | **1** | https://amzn.to/45uAn1z |

---

# Software Architecture

```text
Batocera
   │
   ├── EmulationStation
   ├── Pixelcade artwork
   ├── dmd_real
   ├── dmd-play
   ├── startup animations
   ├── screensaver
   └── shutdown animation
   │
   │ USB
   ▼
ZeDMD 5.1.7
   │
   ▼
ESP32-WROOM-32U
   │
   │ HUB75
   ▼
┌───────────────────────────────┐
│    P4 64×32  │   P4 64×32    │
└───────────────────────────────┘

          128 × 32
```

---

# Installation Files

The production installation uses **two ZIP files**.

## 1. Firmware Flash Package

```text
zedmd-5.1.7-128x32-flash-only.zip
```

This package handles only the ESP32 firmware.

It downloads and flashes the known-working:

```text
ZeDMD 5.1.7
128×32 firmware
```

Firmware flashing is intentionally kept separate from the Batocera installation.

---

## 2. Batocera Marquee Package

```text
batocera-zedmd-marquee-working-combo.zip
```

This package combines the **last known-working scripts** from:

```text
batocera-zedmd-marquee-v3.0
+
batocera-zedmd-marquee-v3.2.1
```

It handles:

- Pixelcade artwork tooling/download
- Pixelcade artwork update checks
- Native Batocera DMD artwork paths
- `dmd_real`
- ZeDMD USB configuration
- System marquees
- Game marquees
- Static artwork
- Animated artwork
- Startup animation
- Screensaver animation
- Shutdown animation
- Virtual Pinball Real DMD compatibility

No experimental custom-boot firmware or later v4.x code is included.

---

# Installation Overview

Use the packages in this order:

```text
FLASH PACKAGE
      │
      ▼
ZeDMD 5.1.7
128×32
      │
      ▼
Set Transport = USB
      │
      ▼
EXIT ZeDMD setup
      │
      ▼
BATOCERA COMBO PACKAGE
      │
      ├── v3.0 base
      └── v3.2.1 animations
      │
      ▼
REBOOT BATOCERA
      │
      ▼
TEST
```

> [!IMPORTANT]
> The reboot after the Batocera installation is required.
>
> During the original working setup, the ESP32 was visible over USB but the marquee did not begin operating correctly until Batocera had been completely restarted.

---

# Step 1 — Connect the ESP32

Connect the ESP32 to the Batocera PC using USB.

Check detection:

```bash
ls -l /dev/ttyUSB*
```

Normally the ESP32 appears as:

```text
/dev/ttyUSB0
```

Depending on the USB interface, it could instead appear as:

```text
/dev/ttyUSB1
/dev/ttyACM0
```

---

# Step 2 — Flash ZeDMD

Copy:

```text
zedmd-5.1.7-128x32-flash-only.zip
```

into the Batocera `share` folder.

SSH into Batocera.

Go to `/userdata`:

```bash
cd /userdata
```

Extract:

```bash
unzip -o zedmd-5.1.7-128x32-flash-only.zip
```

Enter the directory:

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

The script downloads the official ZeDMD 5.1.7 128×32 firmware and flashes it to the ESP32.

---

# Step 3 — Configure ZeDMD

After flashing, the LED panels may display the ZeDMD configuration screen.

Set:

```text
Transport: USB
```

Confirm:

```text
Panel resolution: 128×32
```

Check the RGB order.

The test colors should display correctly:

```text
RED   → Red
GREEN → Green
BLUE  → Blue
```

Adjust RGB order if required.

When finished, navigate to:

```text
Exit
```

and exit the ZeDMD setup screen.

> [!IMPORTANT]
> Do not leave ZeDMD sitting in its configuration menu.
>
> Exit setup so it can enter normal USB host communication mode.

---

# Known-Working ZeDMD Configuration

The known-working configuration for this project is:

| Setting | Value |
|---|---:|
| Firmware | ZeDMD 5.1.7 |
| Controller | ESP32-WROOM-32U |
| Display | 128×32 |
| Panels | 2 × 64×32 P4 |
| Layout | 1 × 2 |
| Transport | USB |
| Typical serial device | `/dev/ttyUSB0` |
| USB package size | 512 bytes |
| Minimum panel refresh | 60 Hz |
| Wi-Fi | Not required |

> [!NOTE]
> Higher refresh values were experimented with later.
>
> **60 Hz is documented here because it belongs to the known-working production configuration.**

---

# Step 4 — Install Batocera Marquee Software

Copy:

```text
batocera-zedmd-marquee-working-combo.zip
```

into the Batocera `share` folder.

SSH into Batocera:

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

The combined installer reproduces the working **v3.0 base installation** and then applies the working **v3.2.1 animation layer**.

It:

1. Removes the older WLED/custom USB marquee hooks.
2. Installs/checks the Pixelcade artwork tooling.
3. Downloads the Pixelcade artwork collection if needed.
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

# Step 5 — Reboot

After the installer completes:

```bash
reboot
```

> [!IMPORTANT]
> **Do not troubleshoot the marquee before performing this reboot.**
>
> During the original installation, `/dev/ttyUSB0` was present but the display did not begin working correctly until Batocera was fully restarted.

---

# Step 6 — Test

After Batocera comes back up, browse through several systems and games.

Expected:

```text
System selected
      │
      ▼
System marquee
```

and:

```text
Game selected
      │
      ▼
Game marquee
```

---

# Verify `dmd_real`

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

Because `dmd_real` may hold the USB connection, stop it first:

```bash
batocera-services stop dmd_real
```

Query ZeDMD:

```bash
zedmd-client -i
```

Expected basics:

```text
Firmware: 5.1.7
CPU: ESP32
Transport: USB
Device: /dev/ttyUSB0
Panel width: 128
Panel height: 32
USB package size: 512
Minimum refresh: 60
```

Restart `dmd_real`:

```bash
batocera-services start dmd_real
```

---

# Pixelcade Artwork

The project uses Pixelcade's LED-specific artwork collection.

This is preferable to ordinary scraped marquee images because the artwork is designed around low-resolution LED displays.

The initial artwork download can be large.

That is normal.

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

Expected behavior:

```text
Batocera boot
     │
     ▼
Wait ~30 sec
     │
     ▼
Check Pixelcade artwork
     │
     ├── Current
     │      │
     │      ▼
     │     Done
     │
     └── Update available
            │
            ▼
       Download update
            │
            ▼
           Done
```

The entire artwork library should not be downloaded again on every boot.

---

# Manual Artwork Update

Check:

```bash
/userdata/system/zedmd-marquee/artwork-update.sh --check
```

Force an update:

```bash
/userdata/system/zedmd-marquee/artwork-update.sh --force-update
```

Refresh Batocera artwork links:

```bash
/userdata/system/zedmd-marquee/sync-artwork.sh --updated
```

---

# Game Marquees

When a game is selected:

```text
Game selected
      │
      ▼
Find matching Pixelcade/Batocera artwork
      │
      ▼
Display PNG/GIF
```

Both static PNG and animated GIF artwork are supported.

---

# Static Artwork During Gameplay

The working configuration intentionally avoids replacing a valid game marquee when the game launches.

Previous behavior:

```text
Game selected
      │
      ▼
Game artwork
      │
      ▼
Launch game
      │
      ▼
Scrolling game title
```

Desired behavior:

```text
Game selected
      │
      ▼
Game artwork
      │
      ▼
Launch game
      │
      ▼
Send nothing new
      │
      ▼
Existing artwork remains
```

This is especially useful for static artwork because the image can remain on the panels without constant communication.

---

# Startup Animation

After Batocera starts, the v3.2.1 animation layer waits approximately:

```text
6 seconds
```

It then chooses **one random local GIF**.

The same GIF plays **twice total**:

```text
Choose random GIF
       │
       ▼
     Play #1
       │
       ▼
     Play #2
       │
       ▼
Normal marquee operation
```

A new random GIF is not chosen for the second play.

---

# Dr. Mario Screensaver

The configured screensaver is the Dr. Mario animation.

The setup searches for:

```text
drnario.gif
drmario.gif
*dr*mario*.gif
*doctor*mario*.gif
```

The matching file is configured as:

```text
/userdata/system/dmd/screensaver.gif
```

Expected behavior:

```text
Batocera idle
      │
      ▼
Screensaver begins
      │
      ▼
Dr. Mario animation
```

---

# Shutdown Animation

The preferred shutdown animation is:

```text
supermarioallstartheend.gif
```

The installer can also search compatible filename variations.

Expected shutdown:

```text
Shutdown selected
        │
        ▼
Super Mario All-Stars
"The End"
        │
        ▼
Play once
        │
        ▼
Batocera powers off
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
- other available animations

---

# Virtual Pinball / Real DMD

The project deliberately uses native ZeDMD and Batocera's `dmd_real` architecture.

This means the same physical display can be used as a Real DMD by compatible Virtual Pinball software.

```text
EmulationStation
      │
      ▼
dmd_real
      │
      ▼
ZeDMD
      │
      ▼
128×32 P4 display
```

and:

```text
Virtual Pinball / PinMAME
           │
           ▼
       Real DMD output
           │
           ▼
         ZeDMD
           │
           ▼
    same 128×32 display
```

---

# Diagnostics

After the required reboot, run:

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

Normally:

```text
/dev/ttyUSB0
```

---

## `dmd_real` Not Running

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

First make sure `dmd_real` is stopped:

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

This was necessary during the original working installation.

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
- 2 × 64×32 P4 HUB75 panels
- HUB75 ribbon cables
- 5V / 12A / 60W linked power supply
- USB connection between Batocera and ESP32
- Internet access during initial installation
- `python3`
- `curl` or `wget`
- `unzip`
- `dmd_real`
- `dmd-play`
- `zedmd-client`

## Not Required

The working production setup does **not** require:

```text
Git
PlatformIO
WLED
ESP32 Wi-Fi
```

---

# Complete Installation Sequence

```text
┌─────────────────────────────────────┐
│ BUILD HARDWARE                      │
│                                     │
│ 2 × 64×32 P4 panels                 │
│ ESP32-WROOM-32U                     │
│ 5V / 12A PSU                        │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│ FLASH PACKAGE                       │
│                                     │
│ zedmd-5.1.7-128x32-flash-only.zip   │
└──────────────────┬──────────────────┘
                   │
                   ▼
            ZeDMD 5.1.7
                   │
                   ▼
             Transport USB
                   │
                   ▼
          EXIT ZeDMD setup
                   │
                   ▼
┌─────────────────────────────────────┐
│ BATOCERA PACKAGE                    │
│                                     │
│ batocera-zedmd-marquee-working-     │
│ combo.zip                           │
│                                     │
│ v3.0 + v3.2.1 working scripts       │
└──────────────────┬──────────────────┘
                   │
                   ▼
             dmd_real
                   │
                   ├── Pixelcade artwork
                   ├── 512-byte USB
                   ├── 60 Hz refresh
                   ├── startup GIF ×2
                   ├── Dr. Mario
                   └── Mario shutdown
                   │
                   ▼
          ┌─────────────────┐
          │ REBOOT REQUIRED │
          └────────┬────────┘
                   │
                   ▼
           TEST MARQUEES
                   │
                   ▼
                 DONE
```

---

# Production Files

The production setup consists of these two files:

```text
zedmd-5.1.7-128x32-flash-only.zip

batocera-zedmd-marquee-working-combo.zip
```

These represent the **known-working installation path**.

Later experiments involving:

```text
Custom ZeDMD boot firmware
Custom boot GIF firmware
Git-based firmware builders
PlatformIO
Windows compilation
v4.x consolidated packages
```

are **not part of this production setup**.
