                      ┌─┐┌─┐┌─┐╷┌    
                      ├─┘│ │├┬┘├┴┐   
                      ╵  └─┘╵└╴╵ ╵   
                         ┌┐╷       
                         │└┤       
                         ╵ ╵       
                    ┌┐ ┌─╴┌─┐┌┐╷┌─┐
                    ├┴┐├╴ ├─┤│└┤└─┐
                    └─┘└─╴╵ ╵╵ ╵└─┘  
                    
               [ NM-RF-HAT EDITION — ESP32-2432S028R ]

Original project by 0ct0sec CYD port by xom (Xombi3) NM-RF-HAT adaptation: this fork



--[ Contents

1  - What is this

2  - Why this fork exists

3  - Credits

4  - Hardware

5  - What changed in this fork

6  - Installation

     6.1 - Requirements

     6.2 - Library + User_Setup.h

     6.3 - Board settings

     6.4 - API keys

     6.5 - SD card

     6.6 - Flashing

7  - GPS on the NM-RF-HAT

8  - Display orientation

9  - Memory notes (why WARHOG used to crash)

10 - Troubleshooting

11 - Everything else

--[ 1 - What is this

This fork adapts xom's CYD port to run with the NM-RF-HAT companion

board, with the GNSS module connected to the hat's GPS header rather

than wired to the bare CYD UART connector.

Use it only on networks and in areas where you have explicit permission.

Laws on WiFi monitoring, packet injection, and surveillance detection

vary by jurisdiction. This is a research and educational tool.

--[ 2 - Why this fork exists

The upstream CYD port (Xombi3/Porkchop-cyd-Port) expects the GPS module

on GPIO 3 — the wiring you get when you solder a GPS straight to the

CYD's 4-pin UART connector.

The NM-RF-HAT routes its GNSS header differently: GPS data arrives on

GPIO 1, not GPIO 3. Flashing the stock port with a hat-mounted GPS gives

you zero satellites forever, because the firmware is listening on the

wrong pin.

This fork moves the GPS to the pin the hat actually uses, fixes a couple

of CYD/hat-specific issues that surfaced along the way (display config,

orientation, and a memory-fragmentation crash in WARHOG), and documents

all of it so you don't have to rediscover it.

--[ 3 - Credits

Original M5PORKCHOP ............ 0ct0sec

    https://github.com/0ct0sec/M5PORKCHOP

    All core attack modes, the XP/challenge/achievement systems, the

    mood/avatar system, the lore, and the pig himself.

CYD port ....................... xom (Xombi3)

    https://github.com/Xombi3/Porkchop-cyd-Port

    ESP32-2432S028R hardware adaptation, ILI9341/XPT2046 display + touch,

    single-file Arduino port, SD/SFX/WebUI, PORK PATROL, the works.

NM-RF-HAT adaptation ........... this fork

    GPS repinned to the hat's GNSS header (GPIO 1), serial/UART0 pin

    arbitration, CYD User_Setup.h, 180° display+touch orientation,

    8bpp sprite memory fix for WARHOG stability.

Surveillance research .......... deflock.me, flock-you

                                 (colonelpanichacks), WiGLE OUI research

Libraries ...................... TFT_eSPI (Bodmer),

                                 XPT2046_Touchscreen (Paul Stoffregen),

                                 TinyGPS++ (Mikal Hart),

                                 ESP32 Arduino Core (Espressif)

Hat hardware ................... NM-RF-HAT by RockBase

                                 https://github.com/RockBase-iot/NM-RF-HAT

--[ 4 - Hardware

Required:

    ESP32-2432S028R  ("Cheap Yellow Display", the R / resistive version)

    NM-RF-HAT companion board

    GNSS module for the hat's GPS header

Note on CYD variant:

    Get the -R (resistive touch) board, not the -C (capacitive). The

    pin map and touch driver in this firmware are for the R variant.

Note on the hat:

    GPS is UART, so it is independent of the hat's RF DIP switch. The

    switch only matters for the SPI-shared RF modules (CC1101 / nRF24 /

    PN532). For GPS-only use, just don't enable an RF module that would

    fight the bus.

--[ 5 - What changed in this fork

+----------------------------+-------------------------------------------+

| Change                     | Why                                       |

+----------------------------+-------------------------------------------+

| GPS_RX_PIN  3 -> 1         | NM-RF-HAT feeds GPS NMEA into GPIO 1, not |

|                            | GPIO 3. This is THE core fix.             |

| Serial/UART0 pin handling  | GPIO 1 is also UART0 TX. setup() frees it |

|                            | so Serial2 can read GPS on it. (Trade:    |

|                            | USB debug TEXT output goes quiet once GPS |

|                            | claims the pin — see section 7.)          |

| Sprite 16bpp -> 8bpp       | The 320x160 16bpp sprite ate ~102KB in    |

|                            | one block and fragmented the heap until   |

|                            | WARHOG abort()ed. 8bpp halves it (~51KB)  |

|                            | and the crash is gone. (See section 9.)   |

| setRotation(1) -> (3)      | Display + touch flipped 180° for the      |

|                            | NM-RF-HAT mounting. Both calls changed so |

|                            | touch zones aren't mirrored.              |

| CYD User_Setup.h provided  | Stock TFT_eSPI ships an ST7735 config.    |

|                            | The CYD needs ILI9341 on the right pins   |

|                            | or you get a blank/garbled screen.        |

+----------------------------+-------------------------------------------+

--[ 6 - Installation

----[ 6.1 - Requirements

Arduino IDE 2.x

ESP32 Arduino Core by Espressif (3.x)

USB driver for CH340 or CP2102 (board dependent)

MicroSD card formatted FAT32

----[ 6.2 - Library + User_Setup.h

Install via Library Manager:

    TFT_eSPI             by Bodmer

    XPT2046_Touchscreen  by Paul Stoffregen

    TinyGPS++            by Mikal Hart

CRITICAL: copy this repo's User_Setup.h into the TFT_eSPI library

folder, replacing the one that ships with it:

    <Arduino sketchbook>/libraries/TFT_eSPI/User_Setup.h

TFT_eSPI only reads its config from inside its OWN folder. Leaving

User_Setup.h next to the .ino does nothing. The stock file is set up

for an ST7735 display — if you skip this step you get a blank or

garbled screen even though the firmware is booting fine.

----[ 6.3 - Board settings

Board:              ESP32 Dev Module

Upload Speed:       921600

Flash Size:         4MB (32Mb)

Partition Scheme:   Default 4MB with spiffs

PSRAM:              Disabled

----[ 6.4 - API keys

Open PorkChop_CYD.ino, search for "PASTE YOUR API KEYS HERE"

(around line 200), and fill in between the quotes:

    char wigleApiName[48]  = "";   // WiGLE "API Name"  (not your login)

    char wigleApiToken[48] = "";   // WiGLE "API Token"

    char wpasecKey[48]     = "";   // WPA-SEC key (optional, separate site)

WiGLE: wigle.net -> Account -> use the API Name + API Token fields.

Do NOT paste the "Encoded for use" / Authorization string — the

firmware base64-encodes the name:token pair itself. WPA-SEC is a

different service (wpa-sec.stanev.org); leave it "" if unused.

Keys are stored in NVS and survive reflashing.

----[ 6.5 - SD card

FAT32 only (not exFAT/NTFS). Cards >32GB need a third-party FAT32

formatter. The firmware auto-creates these at the card root on first

save, but you can pre-make them:

    /captures    handshake .22000 / PMKID .16800 files

    /wigle       wardriving .csv files

    /logs        session text logs

Insert the card before power-on.

----[ 6.6 - Flashing

1. Put PorkChop_CYD.ino in a folder named PorkChop_CYD (the folder name

   must match the .ino name).

2. Open it in Arduino IDE, select the correct COM port.

3. Upload. First compile is slow (a few minutes) — that's normal.

4. If it stalls on "Connecting....", hold the BOOT button as upload

   starts, release at "Writing at...".

Confirmation: on a good flash the serial monitor prints

    [DISPLAY] Sprite 320x160 8bpp OK

and the pig appears on screen.

--[ 7 - GPS on the NM-RF-HAT

Module:      GNSS module on the NM-RF-HAT GPS header (NMEA, 9600 baud)

ESP32 pin:   GPIO 1   (the hat routes GPS data here, set as GPS_RX_PIN)

THE GPIO 1 QUIRK

    GPIO 1 is also the ESP32's UART0 TX — the same pin the USB serial

    console transmits on. The firmware frees GPIO 1 from UART0 so Serial2

    can read GPS on it. Consequence: once GPS is active, USB debug TEXT

    OUTPUT stops (you'll see boot lines, then the serial monitor fills

    with NMEA-looking garbage — that "garbage" IS the GPS working).

    Serial command INPUT still works on GPIO 3, so set API keys in source

    (section 6.4) rather than typing them blind. To get a clean serial

    log for debugging, flash with the GPS UNPLUGGED.

GETTING A FIX

    GPS does not lock indoors. Go outside with a clear view of the sky,

    open SGT WARHOG, and give it 30-90s (longer on a cold start). The

    status bar moves from searching to showing speed/distance, and each

    logged network gets real lat/lon in the WiGLE CSV.

    If you get zero sats outdoors after several minutes and the module's

    own fix LED is blinking, try changing GPS_BAUD from 9600 to 38400 —

    some GNSS modules ship at 38400.

--[ 8 - Display orientation

This fork sets rotation 3 (a 180° flip from the stock port) for the

NM-RF-HAT mounting, on BOTH the display and the touch layer:

    tft.setRotation(3);     // display

    touch.setRotation(3);   // touch (kept in sync so taps aren't mirrored)

If your mount is the other way up, swap both back to 1. Rotation 1 and 3

are the two landscape orientations; 0 and 2 are portrait. Always change

the display and touch values together or your taps will land in the

wrong zones.

--[ 9 - Memory notes (why WARHOG used to crash)

Symptom: WARHOG ran for a bit, then the board rebooted. Serial showed:

    [HEAP] free=37876 largest=92

    abort() was called ...

Total free heap was fine (~37KB) but the largest CONTIGUOUS block was

92 bytes — the heap was shredded. When a WARHOG scan finished and tried

to allocate a normal buffer, nothing contiguous fit and the board

abort()ed.

Cause: the 320x160 display sprite at 16bpp is one ~102KB contiguous

allocation that never frees, leaving the rest of the heap fragmented.

Fix: drop the sprite to 8bpp (256 colors). That halves it to ~51KB and

returns ~50KB of contiguous space — exactly what WARHOG needs. Visually

near-identical for the pig UI.

    mainSprite.setColorDepth(8);   // was 16

(For the record: releasing the BT controller memory was tried and did

nothing measurable on this board/core combo — the sprite depth was the

real lever. Don't bother re-adding esp_bt_controller_mem_release here.)

--[ 10 - Troubleshooting

Blank or garbled screen

    User_Setup.h not copied into the TFT_eSPI library folder, or the

    stock ST7735 config is still in place. See 6.2.

Screen upside-down or taps mirrored

    Orientation. See section 8 — change BOTH setRotation calls together.

Zero GPS satellites

    1. Are you outdoors with sky view? It will not lock indoors.

    2. Is GPS on GPIO 1 (GPS_RX_PIN = 1)? Required for the NM-RF-HAT.

    3. Still nothing after 5 min with the module's LED blinking?

       Try GPS_BAUD = 38400.

WARHOG (or idle) reboots after a while

    Memory fragmentation. Confirm the sprite is 8bpp (section 9). Check

    the serial line after the sprite init — "largest:" should be well

    above the old ~17KB.

Serial monitor shows only garbage

    Expected when GPS is plugged in — that's NMEA on GPIO 1. Not a fault.

    Flash with GPS unplugged if you need a readable serial log.

WiGLE upload returns 401

    API Name and Token swapped, or the token was truncated past 47 chars

    (the buffers are [48]). Re-check section 6.4.

--[ 11 - Everything else

All the modes, navigation, capture formats, SD layout, WiGLE/WPA-SEC

upload, XP/challenges/achievements, sound, and PORK PATROL behave exactly

as in the upstream CYD port. See the original port's README for the full

mode-by-mode reference:

    https://github.com/Xombi3/Porkchop-cyd-Port

This fork only changes how the hardware is wired and a few CYD/hat

stability fixes — the pig is unchanged.

