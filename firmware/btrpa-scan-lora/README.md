# btrpa-scan-lora

**Distributed BLE Detection Mesh Network for Time-Critical Search & Rescue Operations**

Extends [btrpa-scan](https://github.com/TrustedSec/btrpa-scan) BLE detection capabilities onto LoRa mesh networks, enabling coordinated multi-node search operations with real-time detection sharing across multiple field teams.

## 🎯 Use Case

**Active search for missing person with known Bluetooth device MAC address:**

- Pacemaker MAC obtained from phone app disconnect event
- Apple Watch/smartphone MAC from previous pairing records
- Medical device with known BLE identifier
- Any Bluetooth-enabled device with exact MAC address

**Why LoRa Mesh?**

- **Long Range**: 2-10km depending on terrain (no cell towers needed)
- **No Infrastructure**: Works in remote areas without WiFi/cellular
- **Team Coordination**: All field teams see each other's detections instantly
- **Command Center**: Real-time detection mapping at homebase
- **Battery Efficient**: 24+ hours on standard USB power bank

---

## 🚀 Quick Start

### Option 1: Web Flasher (Recommended for Non-Technical Users)

1. **Start the web flasher:**
   ```bash
   cd web-flasher
   python3 -m http.server 8000
   ```
   Open `http://localhost:8000` in **Chrome** or **Edge** browser

2. **Configure your target:**
   - Enter the missing person's device MAC address
   - Set unique Node ID (NODE-001, NODE-002, etc.)
   - Click "Download config.h"
   - Place downloaded file in `include/config.h`

3. **Rebuild and flash:**
   ```bash
   pio run --target upload
   ```
   Or use the "Connect & Flash" button in the web interface

4. **Deploy:**
   - Device starts scanning immediately on power-up
   - OLED shows "SCANNING..." with statistics
   - TRUE HIT alert displays when target detected
   - All detections transmitted via LoRa to other nodes

### Option 2: Command Line (For Developers)

```bash
# 1. Edit configuration
nano include/config.h

# 2. Build and flash
pio run --target upload

# 3. Monitor serial output
python3 monitor.py
```

---

## 📦 Hardware Requirements

### Per Field Node
- **Heltec WiFi LoRa 32 V3** ($25-30 on Amazon/AliExpress)
  - ESP32-S3 microcontroller
  - SX1262 LoRa radio (915 MHz US / 868 MHz EU)
  - 128x64 OLED display built-in
  - USB-C for power and programming

- **Power:** USB power bank (10,000mAh = 24+ hours runtime)
- **Optional:** GPS module (NEO-6M, connects to GPIO 37/38)

### Homebase Station
- One Heltec device (same as field nodes)
- Laptop/computer with Python 3
- USB cable

**Supported Hardware:**
- ✅ Heltec WiFi LoRa 32 V3 (tested)
- ✅ Heltec WiFi LoRa 32 V2 (supported)

---

## ✨ Features

### ✅ Implemented (v1.0)

**BLE Detection:**
- TRUE HIT: Exact MAC address matching
- POSSIBLE HIT: Medical device prefix database
- Active scanning for ~50m detection range
- 500ms scan interval for fast detection

**LoRa Mesh Network:**
- Node-to-node communication (2-10km range)
- Automatic message forwarding
- TRUE HIT priority transmission
- Position beacon broadcasting (25m movement threshold)
- Homebase integration

**OLED Display:**
- Real-time "SCANNING..." status with animated dots
- TRUE HIT alert (3 seconds, large text)
- POSSIBLE HIT alert (2 seconds)
- Statistics: scan count, hit ratios
- Non-blocking display (no watchdog issues)

**GPS Integration:**
- Automatic GPS module detection
- Position tagging on all detections
- Movement-based beaconing
- Works without GPS (N/A for coordinates)

**Homebase Receiver:**
- Python command center software
- Real-time LoRa message monitoring
- CSV logging with timestamps
- Google Maps link generation
- Node status tracking

**Web Flasher:**
- Browser-based firmware deployment
- Drag-and-drop MAC configuration
- No command line required
- ESP Web Tools integration

---

## 🏗️ System Architecture

```
Field Team A          Field Team B          Field Team C
┌──────────┐         ┌──────────┐         ┌──────────┐
│ NODE-001 │         │ NODE-002 │         │ NODE-003 │
│ BLE Scan │◄──LoRa──┤ BLE Scan │◄──LoRa──┤ BLE Scan │
│ + OLED   │  Mesh   │ + OLED   │  Mesh   │ + OLED   │
│ + GPS    │         │ + GPS    │         │ + GPS    │
└────┬─────┘         └────┬─────┘         └────┬─────┘
     │                    │                    │
     │         LoRa Mesh Network               │
     │         (2-10km range)                  │
     └────────────────┬───────────────────────┘
                      │
                      ▼
               ┌──────────────┐
               │  HOMEBASE    │
               │  • LoRa RX   │
               │  • Python    │
               │  • CSV Log   │
               │  • GPS Map   │
               └──────────────┘
```

**Detection Flow:**
1. Field Node scans for BLE devices (500ms interval)
2. TRUE HIT detected → OLED displays alert + LoRa transmission
3. All nodes receive LoRa message via mesh forwarding
4. Homebase logs detection with GPS coordinates and Google Maps link
5. Command center dispatches nearest team to location

---

## ⚙️ Configuration

### Target MAC Addresses

Edit `include/config.h` to add target MAC addresses:

```cpp
const char* TARGET_MACS[] = {
    "70:b3:d5:b3:4a:bc",  // Medtronic pacemaker (from phone disconnect)
    "98:fe:e1:be:61:4f",  // Apple Watch (from pairing history)
    "28:34:ff:74:aa:99",  // Test device
};
```

**Important:** Format must be lowercase, colon-separated.

### Node ID

Each device needs a unique identifier:

```cpp
#define NODE_ID_CONFIG "NODE-001"  // Change for each device
```

Use naming: NODE-001, NODE-002, NODE-003, etc.

### LoRa Frequency

**CRITICAL:** Set correct frequency for your region:

```cpp
#define LORA_FREQUENCY 915  // US: 915 MHz, EU: 868 MHz, Asia: 923 MHz
```

All nodes must use the same frequency. Violating RF regulations may result in fines.

### Medical Device Prefixes (Optional)

Add known medical device MAC prefixes for POSSIBLE HIT alerts:

```cpp
const MedicalDevicePrefixConfig MEDICAL_DEVICE_PREFIXES[] = {
    {"70:b3:d5:b3:4", "Pacemaker/ICD/CRT", "Medtronic"},
    // Add more as discovered
};
```

---

## 📱 Field Operation

### Pre-Deployment Checklist

- [ ] All nodes flashed with identical firmware
- [ ] Each node has unique NODE_ID configured
- [ ] Target MAC address(es) added to config.h
- [ ] All nodes set to same LoRa frequency
- [ ] Power banks charged (10,000mAh recommended)
- [ ] Homebase receiver running on command laptop
- [ ] GPS modules connected (if using position tracking)
- [ ] Antennas properly attached to all nodes

### Operating Instructions

**Power On:**
1. Connect USB power bank to node
2. Wait 5 seconds for boot sequence
3. Display shows "Initializing..."
4. Wait for "LoRa: Mesh ready" message
5. OLED displays "SCANNING..." with animated dots

**During Search:**
- Carry node while searching assigned area
- Screen updates every 500ms showing scan count
- Node transmits position beacon every 25m of movement
- Battery indicator shows remaining capacity

**When TRUE HIT Occurs:**
- OLED displays **"TRUE HIT!"** in large text
- MAC address and signal strength (RSSI) shown
- GPS coordinates displayed (if module present)
- Alert automatically transmitted to all nodes via LoRa
- Screen holds alert for 3 seconds, then returns to scanning

**Team Coordination:**
- All nodes receive mesh alerts from other teams
- Homebase displays all detections with locations
- Command center coordinates response
- Converge on detection location

### Battery Life

- **Continuous BLE Scanning:** 24-36 hours on 10,000mAh
- **With GPS:** 18-24 hours
- **Power Saving Tip:** Disable GPS if position tracking not needed

---

## 🏠 Homebase Receiver Setup

The homebase receiver provides command center visibility into all field detections.

### Installation

```bash
cd homebase

# Install Python dependencies
pip3 install pyserial

# Run homebase receiver (auto-detect port)
python3 homebase_receiver.py

# Or specify port manually
python3 homebase_receiver.py -p /dev/ttyUSB0  # Linux/Mac
python3 homebase_receiver.py -p COM3           # Windows
```

### Console Output

Real-time detection display:

```
======================================================================
🚨 TRUE HIT DETECTED! 🚨
======================================================================
⏰ Time: 2026-02-16 17:30:45
📍 Node: NODE-002
📱 MAC: 28:34:ff:74:aa:99
📡 RSSI: -66 dBm
🗺️  GPS: 37.774929, -122.419418
   https://maps.google.com/?q=37.774929,-122.419418
----------------------------------------------------------------------
```

### CSV Logging

All detections automatically saved to:
```
logs/homebase_session_YYYYMMDD_HHMMSS.csv
```

Import into Excel/Google Sheets for analysis or batch import GPS coordinates into mapping software.

### Network Status

Homebase displays periodic status every 60 seconds:
- Number of active field nodes
- Total detections received
- Last seen timestamp for each node

See `homebase/README.md` for advanced features.

---

## 🛠️ Development & Building

### Prerequisites

```bash
# Install PlatformIO (macOS)
brew install platformio

# Install PlatformIO (Linux/Windows)
pip3 install platformio

# Install Python dependencies for homebase
pip3 install pyserial
```

### Building Firmware

```bash
cd firmware/btrpa-scan-lora

# Edit configuration
nano include/config.h

# Build for Heltec V3
pio run -e heltec_wifi_lora_32_V3

# Build and flash
pio run --target upload

# Monitor serial output
python3 monitor.py
```

### Project Structure

```
firmware/btrpa-scan-lora/
├── include/
│   └── config.h              # User configuration
├── src/
│   └── main.cpp              # Main firmware (LoRa + BLE + display)
├── platformio.ini            # Build configuration
├── web-flasher/
│   ├── index.html            # Browser-based flasher UI
│   ├── manifest.json         # ESP Web Tools configuration
│   ├── firmware.bin          # Compiled firmware binary
│   ├── bootloader.bin        # ESP32-S3 bootloader
│   └── partitions.bin        # Partition table
├── homebase/
│   ├── homebase_receiver.py  # Command center software
│   ├── logs/                 # Detection logs directory
│   └── README.md
├── monitor.py                # Serial monitor utility
└── README.md                 # This file
```

---

## 📊 Serial Output Examples

### TRUE HIT Detection

```
🚨 ========== TRUE HIT ==========
Node: NODE-001
Target MAC: 28:34:ff:74:aa:99
RSSI: -66 dBm
GPS: N/A
Time: 5416 ms
================================
📡 Sending TRUE HIT via LoRa mesh...
LoRa: Sending message (type 1, 100 bytes)
LoRa: Message sent successfully
```

### LoRa Message Received

```
📩 LoRa message received:
  From: NODE-002
  Type: 1
  🚨 TRUE HIT ALERT from mesh!
  MAC: 28:34:ff:74:aa:99
  RSSI: -72 dBm
  GPS: 37.774929, -122.419418
```

### Statistics (every 30 seconds)

```
--- Statistics ---
Uptime: 120 seconds
Total scans: 18
TRUE HITs: 1
POSSIBLE HITs: 0
GPS: No fix
------------------
```

---

## 🚨 Troubleshooting

### Build Errors

**Error: `PSRAM ID read error`**
- Fixed in platformio.ini (PSRAM flag removed)
- Heltec V3 doesn't have PSRAM - this is expected

**Error: `No serial output`**
- Ensure USB cable supports data (not charge-only)
- Use `/dev/cu.usbserial-0001` not `/dev/tty.usbserial-0001`
- Hardware UART enabled in platformio.ini (CDC on boot = 0)

**Multiple definition errors**
- Ensure only one .cpp file in src/ directory
- Clean build: `pio run --target clean && pio run`

### Runtime Issues

**Watchdog Timer Reboots**
- Fixed with non-blocking LoRa operations
- Display delays reduced to prevent blocking
- Should not occur in v1.0

**OLED Display Blank**
- Vext power automatically enabled on GPIO 36
- I2C pins: SDA=17, SCL=18, RST=21
- Check antenna connection (blocks I2C if loose)

**No LoRa Messages Received**
- Verify all nodes use same frequency (915 MHz US)
- Check antenna properly connected
- LoRa range: 2km urban, 10km+ rural
- Ensure both devices show "LoRa: Mesh ready" at boot

**BLE Not Detecting Devices**
- MAC format: `aa:bb:cc:dd:ee:ff` (lowercase, colon-separated)
- Apple devices randomize MACs - use recently observed MAC
- Detection range: ~50m for most devices
- Check device is broadcasting (Bluetooth enabled)

**GPS Not Detected**
- GPS module is optional
- Node continues without GPS (coordinates show "N/A")
- Check module connection to GPIO 37 (RX) and 38 (TX)
- Initial fix can take 30-60 seconds outdoors

---

## 📈 Performance Metrics

### BLE Scanning
- **Scan Interval:** 500ms
- **Scan Window:** 450ms
- **Detection Range:** ~50m (active scan)
- **Battery Impact:** ~100mA continuous

### LoRa Mesh
- **Frequency:** 915 MHz (US)
- **Bandwidth:** 125 kHz
- **Spreading Factor:** SF10 (optimized for range)
- **TX Power:** 20 dBm
- **Range:** 2km urban, 10km+ rural, 20km+ line-of-sight
- **Message Size:** 100 bytes per detection
- **Latency:** <1 second for TRUE HIT transmission

### System Capacity
- **Nodes per Network:** 50+ (mesh auto-routing)
- **Detections per Hour:** 1000+ (practical limit)
- **Concurrent TRUE HITs:** Real-time forwarding

### Flash/RAM Usage
- **Flash:** 591,117 bytes (17.7% of 3.3MB)
- **RAM:** 32,148 bytes (9.8% of 320KB)
- **Available for Extensions:** Plenty of room

---

## 🛣️ Roadmap

### ✅ v1.0 (Current - Feb 2026)
- [x] BLE detection with exact MAC matching
- [x] LoRa mesh communication (RadioLib)
- [x] OLED display with non-blocking alerts
- [x] GPS position tagging and beaconing
- [x] Homebase receiver with CSV logging
- [x] Web flasher for easy deployment

### 🚧 v1.1 (Planned - Q2 2026)
- [ ] OTA firmware updates over LoRa
- [ ] Web-based configuration (no rebuild required)
- [ ] SMS/email alerts from homebase
- [ ] Battery level monitoring and low-power mode
- [ ] Improved mesh routing algorithm

### 🔮 v2.0 (Future)
- [ ] Live web dashboard with real-time map
- [ ] Historical heatmap of search coverage
- [ ] Integration with CAD/dispatch systems
- [ ] Mobile app for field commanders
- [ ] Weatherproof enclosures and mounting options
- [ ] Solar charging support

---

## 🔐 Security & Privacy

### Data Handling
- **Local Storage Only:** No cloud uploads
- **Temporary Logs:** Delete after operation completion
- **MAC Filtering:** Only configured targets logged
- **Offline Operation:** No internet connectivity required

### Legal Considerations
- **Authorized Use Only:** SAR operations and law enforcement
- **BLE Scanning:** Passive listening, no active device interaction
- **LoRa Regulations:** Must comply with regional RF transmission laws
- **Privacy:** Minimize retention of detected device data post-operation

---

## 📝 License

Based on [btrpa-scan](https://github.com/TrustedSec/btrpa-scan) by David Kennedy (@HackingDave)

---

## ⚠️ Disclaimer

This tool is designed for **authorized search and rescue operations** and **law enforcement** use only.

**Intended Use:**
- Time-critical missing person searches
- Authorized SAR operations
- Law enforcement investigations with proper warrants
- Emergency response scenarios

**NOT Intended For:**
- Unauthorized tracking or surveillance
- Commercial use
- Personal/hobby projects without proper context
- Stalking or harassment

Users are responsible for:
- Obtaining proper authorization before deployment
- Complying with local RF transmission regulations (LoRa frequency)
- Following privacy and data handling laws
- Proper disposal of logs after operation

---

## 🙏 Acknowledgments

- **David Kennedy (@HackingDave)** - Original btrpa-scan concept
- **TrustedSec** - Security research community
- **Heltec Automation** - ESP32 LoRa hardware
- **RadioLib** - Excellent LoRa communication library
- **SAR Community** - Field testing and invaluable feedback

---

## 📞 Support

**For Active SAR Operations:**
- Open GitHub issue with `[URGENT]` prefix
- Include serial output logs
- Describe field conditions and issue

**General Questions:**
- GitHub Discussions
- See troubleshooting section above
- Check `web-flasher/README.md` and `homebase/README.md`

---

**Built for those who search when seconds matter. 🔍**
