# 📡 LoRa — Off-Grid Mesh Communicator

> **No internet. No WiFi. No towers. Just radio.**

LoRa is a hardware communication system built on ESP32 that lets multiple devices talk to each other over long-range radio (LoRa 433MHz) with zero infrastructure. Group chat, private chat, GPS location sharing, SOS beacon, compass navigation, and Morse code input — all on a 128×64 OLED display the size of your thumb.

Built for our intra-department expo. Assembled on a bedsheet. It worked.

---

## 📸 Hardware

![LoRa Device](screenshots/device.jpg)

> ESP32 + LoRa Ra-02 + NEO-6M GPS + QMC5883L Compass + SH1106 OLED + 2 Push Buttons

---

## 🧠 What It Does

| Feature | Description |
|---|---|
| **Group Chat** | Broadcast messages to all devices in range |
| **Private Chat** | Send encrypted messages to a specific device ID |
| **GPS Location Sharing** | Periodic beacon with real coordinates |
| **Location Tracker** | See where other users are + distance in meters |
| **Directional Arrow** | Points toward another user using compass + GPS bearing |
| **Compass Mode** | Standalone digital compass with heading display |
| **SOS Beacon** | One-click emergency broadcast with GPS coordinates |
| **Morse Code Input** | Type messages using just 2 buttons |
| **Auto Username Discovery** | Devices query each other's names over radio automatically |
| **OLED Power Management** | Auto-dim and power-off after idle to save battery |

---

## 🔧 Hardware Components
![image alt](https://github.com/Karthikmeesala/lora/blob/3ea33ad7400f8a5f3b026926a8037b40c843752d/Screenshot.jpeg)


| Component | Model | Purpose |
|---|---|---|
| Microcontroller | ESP32 | Main processing unit |
| Radio Module | LoRa Ra-02 (SX1278) | 433MHz long-range communication |
| GPS Module | NEO-6M | Location coordinates + time sync |
| Compass | QMC5883L | Magnetic heading for directional arrow |
| Display | SH1106 1.3" OLED 128×64 | UI rendering |
| Input | 2× Push Buttons | Navigation + Morse code input |

---

## 📡 Radio Protocol

All communication happens over LoRa at **433MHz**. Each device gets a random 16-bit ID on boot. Messages follow a plain-text protocol:

```
B:<deviceId>:<lat>:<lon>               → Location beacon (broadcast)
MSG:G:<senderId>:<username>><message>  → Group chat message
MSG:<receiverId>:<senderId>><message>  → Private message
QUERY:<senderId>:<username>:<targetId> → Username discovery request
REPLY:<targetId>:<senderId>:<username> → Username discovery response
SOS:<deviceId>:<lat>:<lon>             → Emergency broadcast
```

- **Sync word:** `0xF3` — prevents crosstalk with other LoRa networks
- **Spreading factor:** 12 — maximum range
- **Tx power:** 20 dBm
- **CRC:** enabled
- **Bandwidth:** 125 kHz

---

## 🗂️ Project Structure

```
LoRa/
├── FINAL_LORAKRAZY.ino       # Main Arduino sketch
├── this_lora.h               # LoRa init, message send/receive, user/chat data structures
├── this_gps.h                # GPS init, location polling, time sync from satellites
├── this_display.h            # OLED UI, screen state machine, compass, Morse decoder
└── this_time.h               # Unix timestamp ↔ date/time conversion (no RTC needed)
```

---

## 🖥️ UI & Navigation

The interface is a **state machine** with 7 screens:

```
SCREEN_MAIN
├── /groupchat      → SCREEN_GROUPCHAT
├── /chat           → SCREEN_CHAT_LIST → SCREEN_CHAT_PERSON
├── /locations      → SCREEN_LOCATION_LIST → SCREEN_LOCATION_PERSON
├── /SOS            → Sends emergency broadcast → returns to SCREEN_MAIN
└── /compass        → SCREEN_COMPASS
```

Navigation uses two buttons. The display auto-dims after **15 seconds** of inactivity and powers off completely after idle — configurable per screen type (chat screens stay on longer).

---

## ⌨️ Morse Code Input

Messages are typed using **2 buttons** — dot and dash. The system decodes standard international Morse code:

- Short press → `.` (dot)
- Long press → `-` (dash)
- Pause between characters → letter boundary
- Special sequence `.-.-. ` → newline / send

Supported: A–Z, with the decoded text appearing live on the bottom bar of the display.

---

## 📍 Location & Navigation

When a device sends a beacon, all other devices store its GPS coordinates. The **location tracker screen** shows:

- Username
- Latitude / Longitude
- Distance in meters (Haversine approximation)
- **Directional arrow** — combines compass heading + GPS bearing to point toward the other person in real space

The arrow updates live as you physically rotate the device.

---

## ⚙️ Pin Configuration

```cpp
// LoRa
#define LORA_SS    5
#define LORA_RST   14
#define LORA_DIO0  26

// GPS (UART)
#define GPS_RX_PIN 15   // Connect to GPS TX
#define GPS_TX_PIN 4    // Connect to GPS RX

// OLED + Compass → I2C (default SDA/SCL pins for ESP32)
// OLED address: 0x3C
// Compass address: auto-detected by QMC5883L library
```

---

## 🚀 Getting Started

### Prerequisites

- Arduino IDE 2.x
- ESP32 board package installed
- Libraries (install via Library Manager):
  - `LoRa` by Sandeep Mistry
  - `TinyGPS++` by Mikal Hart
  - `U8g2` by olikraus
  - `QMC5883LCompass` by mprograms

### Flash Instructions

```
1. Open FINAL_LORAKRAZY.ino in Arduino IDE
2. Select board: ESP32 Dev Module
3. Select the correct COM port
4. Upload
5. Open Serial Monitor at 115200 baud to see debug output
```

### First Boot

On first boot, each device:
1. Initializes LoRa and broadcasts a beacon with its random device ID
2. Starts listening for GPS signal (takes 30–60s outdoors for first fix)
3. Enters the main menu

When two devices are in range, they automatically discover each other's usernames via the `QUERY/REPLY` protocol — no pairing needed.

---

## 🐛 Known Issues

| Issue | Status |
|---|---|
| Username defaults to `unknown:<id>` until QUERY/REPLY completes | Working as intended — resolves within 1–2 seconds |
| GPS first fix takes up to 60s outdoors | Hardware limitation of NEO-6M |
| `myUserName` is hardcoded per device | Change before flashing each unit |
| Message history lost on reboot | No persistent storage — by design for simplicity |

---

## 🔮 Planned Improvements

- [ ] Persistent username stored in EEPROM (no reflash needed)
- [ ] Relay/mesh mode — devices forward messages they receive
- [ ] Battery level indicator on main menu
- [ ] Encrypted private messages
- [ ] Multiple device pairing with acknowledgement (ACK)
- [ ] PCB design to replace breadboard/jumper wire setup

---

## 👥 Team

- Eswar  
- Karthik Rajkumar  
- Mahadev

---

## 💡 Why We Built This

Most communication tools need the internet. This one doesn't. It works in forests, during network outages, at remote campsites, in disaster zones — anywhere two humans are within a few kilometers of each other.

We wanted to see if we could build something that actually solves a real problem using only hardware we could afford and a bedsheet as a workbench.

We could.

---

## 📄 License

MIT License — open source, free to build on.
# lora

