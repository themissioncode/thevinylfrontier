# The Vinyl Frontier – API, Function, and Component Documentation

This document describes the externally visible components, functions, and operating interfaces that make up **The Vinyl Frontier**, an NFC-enabled jukebox built around Raspberry Pi hardware and the Phoniebox software stack. Although this repository contains project notes rather than executable source code, the system exposes a clear set of behaviors that integrators, event technologists, and contributors can rely on. The goal of this document is to make those behaviors explicit, provide usage instructions, and share practical examples so you can extend or reproduce the build with confidence.

## Table of Contents

1. [System Overview](#system-overview)
2. [Component Inventory](#component-inventory)
3. [Operational Data Flow](#operational-data-flow)
4. [Public Interfaces & Functions](#public-interfaces--functions)
5. [Usage Recipes & Examples](#usage-recipes--examples)
6. [Testing & Verification](#testing--verification)
7. [Extension Points & Customization](#extension-points--customization)
8. [Troubleshooting Reference](#troubleshooting-reference)
9. [Glossary](#glossary)

---

## System Overview

- **Purpose**: Deliver a tactile, screen-free music experience by mapping NFC-tagged coasters to curated audio sources.
- **Form Factor**: Raspberry Pi 4 + PN532 NFC reader housed inside a wooden case with LiPo battery power and Bluetooth audio out.
- **Core Software**: Raspberry Pi OS running [Phoniebox](http://phoniebox.de/index-en.html) for web-based media management, NFC card binding, and playback control.
- **Key Behaviors**:
  - Detect NFC tags via the PN532 HAT.
  - Resolve each tag into a Phoniebox “card” (action) such as an MP3 playlist, live stream, or Spotify link.
  - Initiate playback through the Pi’s headphone jack (fed into a Bluetooth transmitter) or any other configured output.

---

## Component Inventory

### Hardware Components

| Component | Purpose | Configuration Notes |
| --- | --- | --- |
| Raspberry Pi 4 (other Pi models compatible) | Hosts the OS, NFC services, Phoniebox web server, and audio output. | Requires stable Wi-Fi. Use standoffs to align the HAT stack below the case lid. |
| PN532 NFC HAT | Reads NFC tags when coasters are placed on the lid. | Install per Waveshare instructions and enable via I2C or SPI. |
| LiPo Battery HAT | Provides portable power. | Dimension battery capacity for session length; ensure safe charging circuitry. |
| Bluetooth Headphone Adapter | Converts Pi’s analog or USB audio to wireless output. | Plug into Pi audio jack; pair with speakers/headphones after boot. |
| 38 mm NFC Tags | Encoded identifiers for each coaster. Larger tags improve read range through the case lid. | Program NTAG213-compatible stickers for reliability. |
| Printed Coasters / Physical Tokens | User-facing interface that displays album art or instructions. | Affix NFC tag centrally for consistent read alignment. |
| Case / Enclosure | Houses the stack while allowing NFC reads through the lid. | Non-metallic materials (wood, acrylic) preserve NFC range. |

### Software Components

| Component | Description | Configuration |
| --- | --- | --- |
| Raspberry Pi OS (Lite or Desktop) | Base OS managing services and networking. | Keep packages updated; enable SSH for remote access. |
| Phoniebox (Stretch installer) | Provides media library, NFC card bindings, and playback controls accessible via browser. | Install following [official guide](https://github.com/MiczFlor/RPi-Jukebox-RFID/wiki/INSTALL-stretch). |
| PN532 Driver Stack | Kernel modules & utilities that expose the NFC reader to Phoniebox. | Follow Waveshare PN532 guide for installation and diagnostic scripts. |
| Bluetooth Audio Service | Handles outbound audio over Bluetooth transmitter. | Optional; analog output works by default for bench testing. |

---

## Operational Data Flow

1. **Boot & Services**: Pi powers on, launches network, PN532 daemon, and Phoniebox web server.
2. **NFC Detection**: PN532 polls for tags; on detection it publishes the UID to Phoniebox.
3. **Card Resolution**: Phoniebox matches the UID with a configured “card” (playlist, stream, folder, or script).
4. **Playback Execution**: Playback service launches the corresponding media asset and routes audio to the configured output.
5. **Remote Control**: Users can monitor status, change volume, or trigger tracks via the Phoniebox web UI from any device on the LAN.

---

## Public Interfaces & Functions

> **Note**: The project reuses Phoniebox’s internal APIs. The functions below describe the stable, user-facing entry points you can rely on when operating or extending the system.

### 1. Boot Controller

| Function | Description | Inputs | Outputs | Failure Modes |
| --- | --- | --- | --- | --- |
| `boot.initialize()` | Powers hardware, mounts filesystems, launches Phoniebox services. | Power switch, charged LiPo battery or PSU. | Running web UI accessible at `http://<pi-address>/`. | Boot loop due to low battery; service not running if SD corrupt. |
| `boot.healthCheck()` | Verifies that PN532, Wi-Fi, and audio services report ready. | Optional SSH session. | Status report (OK / warnings). | Missing driver modules, misconfigured country code for Wi-Fi. |

**Usage**: After assembly, toggle power and wait ~60 s. Confirm LED indicators on PN532 and browse to the Pi’s IP from another device.

### 2. NFC Tag Service

| Function | Description | Inputs | Outputs | Notes |
| --- | --- | --- | --- | --- |
| `nfc.scan()` | Reads UID when a tag is within range. | Physical tag placed above reader. | UID string (e.g., `04A2247BCB5D80`). | Requires PN532 Hat spacing to avoid shielding. |
| `nfc.diagnose()` | Runs Waveshare PN532 tests to validate wiring. | SSH terminal. | Pass/fail logs. | Useful before sealing enclosure. |

**Example Diagnostic Command**

```bash
python3 pn532_read.py
```

### 3. Tag Binding Service

| Function | Description | Parameters | Output | Example |
| --- | --- | --- | --- | --- |
| `tag.bind(tagId, actionType, actionTarget)` | Associates a UID with a Phoniebox action (folder, stream, Spotify URI). | `tagId`: UID; `actionType`: `folder`, `webradio`, `spotify`, `script`; `actionTarget`: path or URL. | Binding stored in Phoniebox DB. | Bind via Phoniebox web UI → `Card/ID` tab. |
| `tag.list()` | Displays all configured bindings. | None | Table of tag → action mappings. | Useful for audits and sharing coaster packs. |
| `tag.unbind(tagId)` | Removes an association. | `tagId` | Tag returns to unassigned state. | Use when reprinting coasters. |

### 4. Media Library Service

| Function | Description | Parameters | Output |
| --- | --- | --- | --- |
| `media.import(source, destinationFolder)` | Uploads MP3/WAV files via the `Folders/Files` UI or SCP. | `source`: local path; `destinationFolder`: target playlist folder. | New folder visible in UI and selectable for tag binding. |
| `media.list(filterPattern?)` | Lists available folders/playlists. | Optional text filter. | Filtered listing for quick binding. |
| `media.testPlay(folder)` | Plays a folder without NFC trigger (UI button). | `folder`: folder name. | Immediate playback for QA. |

### 5. Playback Control Service

| Function | Description | Parameters | Output |
| --- | --- | --- | --- |
| `playback.start(actionTarget, outputDevice)` | Launches playback for the folder or stream. | `actionTarget`: e.g., `/home/pi/RPi-Jukebox-RFID/shared/audio/Album`; `outputDevice`: `analog`, `hdmi`, `bluetooth`. | Audio routed to speakers or headphones. |
| `playback.pause()` | Pauses current track. | — | Playback halted. |
| `playback.resume()` | Resumes paused media. | — | Audio resumes. |
| `playback.stop()` | Stops playback and frees device. | — | Idle state. |
| `playback.setVolume(level)` | Sets system volume (0–100). | `level`: integer. | Adjusted volume. |

All functions accessible via the Phoniebox browser UI or command-line scripts documented upstream.

### 6. Remote Control & Networking

| Function | Description | Parameters | Output |
| --- | --- | --- | --- |
| `remote.openDashboard()` | Opens the Phoniebox interface from another device. | `piAddress`: IPv4 on LAN. | Browser UI with navigation tabs. |
| `remote.getIp()` | Retrieves Pi IP (e.g., via router admin or `hostname -I`). | — | IP string. |
| `network.configureWifi(ssid, passphrase, country)` | Sets Wi-Fi credentials via `raspi-config` or editing `wpa_supplicant.conf`. | Credentials, country code (important for NFC compliance). | Persistent network connectivity. |

### 7. Power & Mobility Service

| Function | Description | Parameters | Output |
| --- | --- | --- | --- |
| `power.monitor()` | Reads LiPo HAT indicators to ensure sufficient voltage. | — | LED / voltage reading. |
| `power.charge()` | Recharges LiPo safely before events. | USB-C or micro-USB input on HAT. | Full battery (indicated by LEDs). |
| `power.shutdown()` | Initiates safe shutdown to prevent SD corruption. | `sudo shutdown now`. | Pi powers off before disconnecting battery. |

---

## Usage Recipes & Examples

### Example 1: Play a Local MP3 Folder
1. **Import**: Zip the album locally and upload via Phoniebox `Folders/Files`.
2. **Verify**: Use `media.testPlay()` from the UI to ensure audio plays through headphones.
3. **Bind**: Go to `Card/ID`, scan the desired coaster, and bind the folder.
4. **Deploy**: Place the coaster on the case lid; playback starts automatically via Bluetooth adapter.

### Example 2: Link a Web Radio Stream
1. **Gather Stream URL** (e.g., `https://example.com/stream.mp3`).
2. **Create Web Radio Entry** in Phoniebox and save.
3. **Bind Tag** with `actionType=webradio`.
4. **Test** by tapping the coaster; ensure the stream buffers without interruption on Wi-Fi.

### Example 3: Troubleshoot Spotify Integration (Experimental)
1. Install the Spotify-enabled Phoniebox variant (v3.0+ as recommended in README).
2. Authenticate with Spotify developer credentials.
3. Use `tag.bind(tagId, spotify, spotify:playlist:xxx)` to connect playlists.
4. If playback fails, fall back to MP3/WAV assets while monitoring the upstream issue tracker.

### Example 4: Bench Testing Before Bluetooth Deployment
1. Connect wired headphones to the Pi audio jack.
2. Run `playback.start(<folder>, analog)` from the UI.
3. Confirm audio quality before chaining in the Bluetooth transmitter.

---

## Testing & Verification

- **Hardware Smoke Test**: Run PN532 diagnostic, verify battery voltage, and ensure Bluetooth transmitter powers on.
- **Software Regression**:
  - Confirm Phoniebox service status: `sudo systemctl status phoniebox`.
  - Validate tag database: `/home/pi/RPi-Jukebox-RFID/settings/cards.json`.
  - Test web UI availability across Wi-Fi.
- **User Journey Simulation**: Have someone unfamiliar with the system select a coaster and play music; capture any UX friction or latency.

---

## Extension Points & Customization

- **Enclosure Variations**: Reuse vintage Hi-Fi shells or jukebox frames; ensure NFC reader remains close to interaction surface.
- **Alternate Outputs**: Replace Bluetooth adapter with powered speakers or multi-room systems (e.g., Snapcast).
- **Automation Hooks**: Utilize Phoniebox’s MQTT or scripting hooks to trigger lighting effects when certain tags are scanned.
- **Content Packs**: Generate curated coaster sets (e.g., party playlists, educational podcasts) and publish the `tag.list()` results so others can import them.

---

## Troubleshooting Reference

| Symptom | Likely Cause | Fix |
| --- | --- | --- |
| Tag not recognized | NFC reader too far from lid, interference, or unregistered UID. | Re-seat PN532 with taller standoffs, use 38 mm tags, confirm binding exists. |
| Audio plays through Pi speaker only | Bluetooth adapter not paired or powered. | Pair device after boot; revert to analog output for diagnostics. |
| Web UI unreachable | Pi offline or IP changed. | Check router DHCP table, attach HDMI temporarily, or use `nmap` to discover host. |
| Playback stutters | Wi-Fi congestion or low battery. | Move closer to router, charge battery, or switch to wired Ethernet during setup. |

---

## Glossary

- **Phoniebox**: Open-source jukebox system combining RFID/NFC triggers with a web-managed media library.
- **Card/ID**: Phoniebox UI section where NFC UIDs are linked to actions.
- **Action**: The playable object bound to a tag (folder, web radio, Spotify playlist, or script).
- **UID**: Unique identifier read from an NFC tag.
- **PN532**: NFC controller chip used on the Raspberry Pi hat.
- **LiPo**: Lithium Polymer battery providing portable power.

---

### References

- Project README (`README.md`) for build motivation, parts list, and installation considerations.
- [Phoniebox Wiki](https://github.com/MiczFlor/RPi-Jukebox-RFID/wiki) for upstream API scripts and advanced configuration.

