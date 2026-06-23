# Mitsubishi MSZ-HJ25VA — Home Assistant Integration

Modding and integrating Mitsubishi MSZ-HJ25VA HVAC into Home Assistant via ESPHome.

---

## Background

My old MSZ-HJ25VA air conditioner does not support a WiFi module. It looks like all HVACs produced after a certain point do have a port for connecting one.

After searching online I found some interesting information about Mitsubishi ACs in general. All Mitsubishi HVACs use a connector called **CN105** to connect the WiFi module. There is a very nice project on GitHub that enables support for this connector using an ESP device:

- [SwiCago/HeatPump](https://github.com/SwiCago/HeatPump) — Arduino library to control Mitsubishi Heat Pumps via connector CN105.

This library targets Arduino IDE users, which is not (yet) my case. Digging further I found another project that leverages SwiCago and embeds it into ESPHome:

- [geoffdavis/esphome-mitsubishiheatpump](https://github.com/geoffdavis/esphome-mitsubishiheatpump) — ESPHome Climate Component for Mitsubishi Heatpumps using direct serial connection.

Even more promising is a fork that adds a few extra features and is slightly more up to date:

- [echavet/MitsubishiCN105ESPHome](https://github.com/echavet/MitsubishiCN105ESPHome) — ESPHome firmware inspired by GeoffDavis's project, directly integrating the SwiCago library within its codebase.

From a software perspective, it is possible to connect an ESP device and manage the HVAC without buying expensive modules like the **MAC-587IF-E**.

With those projects in mind, I started looking for an option to add the CN105 connector to my MSZ-HJ25VA. I found several approaches based on the IR component and the SmartIR integration for Home Assistant, but using infrared lacks the ability to check the actual status of the heat pump: you can send commands but you cannot know if someone changed the configuration via the remote.

I then found this post on the HA community forum:

- [Mitsubishi AC with Wemos D1 Mini Pro — HA Community](https://community.home-assistant.io/t/mitsubishi-ac-with-wemos-d1-mini-pro/107007/246?page=13)

User *vperas* found that the logic board of the **MSZ-HJ35VA** (same family, just more BTUs!) has the CN105 connector layout printed on the board — it only lacks 2 resistors and the connector itself. This was confirmed in the following issue as well (many comments were deleted by the owner for being off-topic):

- [SwiCago/HeatPump — Issue #13](https://github.com/SwiCago/HeatPump/issues/13)

---

## Requirements

Besides soldering skills (you will need to solder 2 SMD resistors and a connector), the following items are needed:

| Item | Link |
|---|---|
| S05B-PASK-2 connectors | [AliExpress](https://it.aliexpress.com/item/1005008145895889.html) |
| PAP-05V-S connectors *(optional)* | [AliExpress](https://it.aliexpress.com/item/1005007784137238.html) |
| ATOMS3 Lite ESP32S3 M5Stack | [AliExpress](https://it.aliexpress.com/item/1005005177952629.html) |
| 4-pin Jumper Wire HY2.0mm Pitch Grove Cable 20 cm | [Amazon.it](https://www.amazon.it/dp/B0D4PFPSMY) |
| 2× 0603 SMD 1200 Ω resistors | [Amazon.it](https://www.amazon.it/dp/B0DGQ6GKMD) |

The **Atom S3 Lite** was chosen because it is very compact and connects easily with the Grove 4-pin cable. For the best result, replace one connector of the Grove cable with the proper **PAP-05V-S** connector (only 4 wires are needed). Alternatively, trim one side of the connector and clip to let it fit into the **S05B-PASK-2** connector soldered on the board.

> Note: 1200 Ω SMD 0603 resistors are labelled **122** on the top.

---

## Opening the MSZ-HJ25VA Unit

Opening the unit is fairly simple:

1. Remove the 2 small covers on the bottom-left and bottom-right under the vane.
2. Unscrew the 2 Phillips screws.

<img src="https://github.com/user-attachments/assets/7458a613-9210-45fd-a72b-26e8fde128eb" alt="Bottom covers" width="45%">
<img src="https://github.com/user-attachments/assets/131faec3-24f3-4d79-abce-e5b3613fabc1" alt="Screws" width="45%">

3. Slightly pull from the bottom after checking the push/pull plastic lock in the middle of the vane.

<img src="https://github.com/user-attachments/assets/4edb1b67-ed52-4e61-a8e8-47cc91f4f9df" alt="Pull from bottom" width="45%">

4. On the left you will have access to the logic board.

<img src="https://github.com/user-attachments/assets/94275b12-6e80-4315-827e-07e6bd1de4b8" alt="Logic board access" width="45%">

5. Unplug the three connectors and remove the board with its plastic cover — unlock the plastic at the bottom-left. Gently turn the board, open it on the side, pull it up and remove it from the chassis. Take care to gently route all wires clear of the plastic.

---

## The Logic Board

Once removed, the logic board should look like this:

<img src="https://github.com/user-attachments/assets/acda0ef4-6f64-429a-9626-242f961b7dd8" alt="Logic board front" width="45%">
<img src="https://github.com/user-attachments/assets/b663f6fd-93d8-4fb3-bade-142b75e21e14" alt="Logic board back" width="45%">

First, clean the area where the connector and resistors will be placed:

<img src="https://github.com/user-attachments/assets/d0e0b3cc-f719-41e4-8046-34d8eae2daf6" alt="Cleaned area 1" width="45%">
<img src="https://github.com/user-attachments/assets/8f1672e3-6476-4efb-9a9f-018c4e936e10" alt="Cleaned area 2" width="45%">

Remove flux with isopropyl alcohol, then solder the connector and the 2 resistors (1200 Ω, 0603 SMD, labelled **122**):

<img src="https://github.com/user-attachments/assets/274a48a7-52f9-4889-9ba8-d15307db80b1" alt="Soldering 1" width="45%">
<img src="https://github.com/user-attachments/assets/e45d8c9d-b9ae-4f5f-b67d-cc18b85b65a3" alt="Soldering 2" width="45%">

> These are not my best tin welds, but they work!

---

## ATOM S3 Lite with ESPHome

The M5Stack Atom S3 Lite with the 4-pin Grove connector:

<img src="https://github.com/user-attachments/assets/42ae5aa2-9ee2-4901-b587-da40df8a0344" alt="Atom S3 Lite" width="45%">

The cable fits the Atom S3 Lite and can be adapted to fit the **S05B-PASK-2** on the logic board by cutting the side border (near the black cable) and the hook:

<img src="https://github.com/user-attachments/assets/3ab83282-8e8d-403a-a50b-a13560b6ab16" alt="Cable adaptation" width="45%">

After sourcing proper **S05B-PASK-2** connectors, one side of the Grove cable was replaced. Final result:

<img src="https://github.com/user-attachments/assets/1e226339-2232-4ffd-8bf6-4a535ccdcc68" alt="Final assembly" width="45%">

---

## ESPHome Configuration

A sample ESPHome configuration for this device is available here:

- [`msz-hj25-sample.yaml`](msz-hj25-sample.yaml)

**Before flashing**, replace `<macaddress>` and any names in the `substitutions` section. Since I have 2 splits, I use the last 6 characters of the MAC address to generate unique names and IDs.

> Tested with **ESPHome 2026.5.4** — fully working.

A reference config file for **MSZ-AY## models** is also available in the repository.

---

### Exposed Entities in Home Assistant

#### 🌡️ Climate entity

The main entity is a standard HA `climate` object exposed via the `cn105` platform:

| Attribute | Values |
|---|---|
| **Modes** | `OFF`, `COOL`, `HEAT`, `FAN_ONLY`, `DRY` |
| **Fan modes** | `AUTO`, `QUIET`, `LOW`, `MEDIUM`, `HIGH` |
| **Swing modes** | `OFF`, `VERTICAL` |
| **Target temperature range** | 16 °C – 31 °C (1 °C steps) |
| **Current temperature resolution** | 0.1 °C |
| **Update interval** | every 4 s |

#### 📊 Sensors

| Entity | Type | Notes |
|---|---|---|
| **Compressor Frequency** | `sensor` | Frequency (Hz) reported by the heat pump |
| **ISEE Sensor** | `binary_sensor` | iSee motion sensor presence detection |
| **Uptime Connection** | `sensor` | Cumulative uptime of the CN105 serial connection |
| **WiFi Signal dB** | `sensor` | RSSI in dBm — diagnostic, disabled by default |
| **WiFi Signal Percent** | `sensor` | RSSI converted to 0–100 % — diagnostic, disabled by default |
| **Uptime** | `sensor` | Device uptime — diagnostic, disabled by default |

#### 🎛️ Controls

| Entity | Type | Notes |
|---|---|---|
| **Vertical Vane** | `select` | Direct vane position control (independent of swing mode) |
| **Status Led** | `switch` | Enables/disables the ATOM S3 Lite onboard LED; default OFF |
| **Restart** | `button` | Reboots the ESP device — diagnostic, disabled by default |

#### ℹ️ Diagnostic text sensors

| Entity | Notes |
|---|---|
| **ESPHome version** | Firmware version string — disabled by default |
| **IP Address** | Current IP of the ESP — diagnostic, disabled by default |
| **Connected SSID** | Wi-Fi network name — diagnostic, disabled by default |
| **Connected BSSID** | Access point MAC — diagnostic, disabled by default |
| **Mac Wifi Address** | ESP Wi-Fi MAC — diagnostic, disabled by default |

#### 💡 Status LED colour map

The onboard RGB LED of the ATOM S3 Lite reflects the current HVAC mode when the **Status Led** switch is enabled:

| Mode | Colour |
|---|---|
| `OFF` | Off |
| `COOL` | Blue |
| `HEAT` | Red |
| `FAN_ONLY` | White |
| `DRY` | Light blue |
| `AUTO / HEAT_COOL` | Cyan |

---

## Credits

### Eric Chavet — [MitsubishiCN105ESPHome](https://github.com/echavet/MitsubishiCN105ESPHome)

The core of this integration would not exist without the outstanding work of **Eric Chavet** and his project **MitsubishiCN105ESPHome**.

Eric took Geoffrey Davis's pioneering [esphome-mitsubishiheatpump](https://github.com/geoffdavis/esphome-mitsubishiheatpump) component as inspiration and went a significant step further: rather than wrapping the SwiCago library as an external dependency, he directly embedded it into an ESPHome-native codebase. The result is a robust, actively maintained firmware that exposes full climate control — mode, fan speed, vane direction, temperature — as proper ESPHome entities, with clean Home Assistant integration out of the box.

His project (935+ stars, 168+ forks as of mid-2026) has become the go-to solution for anyone looking to connect a Mitsubishi heat pump via CN105 without proprietary cloud modules. The quality of the code, the detailed documentation, and his continued responsiveness to issues and community discussions make it an exceptional open-source contribution.

**Thank you, Eric.**

---

### Acknowledgements

| Project | Author | Role in this build |
|---|---|---|
| [MitsubishiCN105ESPHome](https://github.com/echavet/MitsubishiCN105ESPHome) | Eric Chavet | ESPHome firmware used on the ATOM S3 Lite |
| [esphome-mitsubishiheatpump](https://github.com/geoffdavis/esphome-mitsubishiheatpump) | Geoffrey Davis | Original ESPHome climate component |
| [HeatPump](https://github.com/SwiCago/HeatPump) | SwiCago | Arduino CN105 serial library at the foundation of it all |
