# ✈️ Flight Monitor - War Thunder Dashboard for Flipper Zero

<h2 align="center">Real-time Aircraft Parameters Monitor for Flipper Zero</h2>

<div align="center">
    <img src="screenshots/1.png" alt="Screenshot 1" width="250">
    <img src="screenshots/2.png" alt="Screenshot 2" width="250">
    <img src="screenshots/4.png" alt="Screenshot 4" width="250">
</div>

---

This is a **comprehensive flight monitoring application** designed for the **Flipper Zero** that interfaces with **War Thunder** flight simulator via **Bluetooth Low Energy (BLE)**. The application provides real-time display of critical flight parameters with an intelligent alarm system for enhanced flight safety.

## ✨ Features Overview

### 🛩️ Flight Parameters Display

The application provides accurate and real-time readings for complete flight telemetry:

* **Altitude (ALT):** Current height above ground in meters (**m**).
* **Speed (SPD):** Indicated Air Speed (IAS) or True Air Speed (TAS) in kilometers per hour (**km/h**).
* **Vertical Speed (V/S):** Rate of climb or descent in meters per second (**m/s**).
* **Throttle (THR):** Engine power setting from 0% to 100%.
* **Flaps (FLP):** Flaps deployment percentage from 0% to 100%.
* **Gear Status:** Landing gear position (UP/DOWN).
* **Pitch & Roll (P/R):** Aircraft orientation angles in degrees.

### 🚨 Intelligent Alarm System

Professional safety monitoring with configurable thresholds:

* **Altitude Warning:** Audible alarm when descending below configured altitude (default: 120m).
* **Gear Warning:** Alert when flying low with gear extended (default: below 150m).
* **Gear Retraction Alert:** Fullscreen warning to retract gear at speed (default: >200m at >150 km/h).
* **Stall Speed Warning:** Protection against dangerously low airspeeds (configurable).
* **Overspeed Warning:** Alert when exceeding safe speed limits (configurable).
* **Engine Failure Detection:** Automatic detection based on power loss + descent (power <100HP + falling + altitude >100m).
* **G-Force Alerts:** Vibration warnings for extreme pitch (>60°) or roll (>70°) at high speeds (>200 km/h).
* **Crash Detection:** Automatic alarm silence when aircraft data stops changing.

### ⚙️ User Interface & Experience

The application is designed for clarity, ease of use, and quick access to critical information:

* **Splash Screen:** 3-second startup logo with airplane graphic and branding.
* **Multiple View Modes:** 
  - Main dashboard with all parameters
  - Throttle-focused view with detailed power display
  - Flaps-focused view with deployment details
  - Orientation view with pitch/roll visualization
* **Settings Menu:** BME680-style menu with:
  - Altitude alarm threshold (5m steps)
  - Gear warning altitude (5m steps)
  - Gear retraction altitude (5m steps)
  - Stall speed threshold (5 km/h steps)
  - Overspeed limit (50 km/h steps)
  - START button for BLE initialization
* **Persistent Configuration:** Settings saved to `/ext/apps_data/flight_monitor/settings.cfg`.
* **Visual Feedback:** Progress bars for throttle and flaps, status messages for connection state.
* **High Refresh Rate:** 100ms data update interval (10 Hz) for smooth operation.

### 📡 Connectivity

* **Bluetooth Low Energy (BLE):** Custom protocol for efficient data transmission.
* **Binary Data Format:** Compact 20-byte packets for minimal latency.
* **Auto-Discovery:** Python server automatically finds and connects to Flipper Zero.
* **Reliable Communication:** Error handling and connection monitoring.

## 🔧 Installation Guide

### Prerequisites

* **Flipper Zero** with f7 firmware
* **War Thunder** game installed on PC
* **Python 3.x** installed on PC

### 1. Compile and Install Flipper App

Navigate to the flipperzero-firmware directory and compile:

```bash
./fbt COMPACT=1 DEBUG=0 launch APPSRC=applications_user/flight_monitor
```

Or manually copy the compiled `.fap` file:

```bash
# After compilation, file is in:
build/f7-firmware-C/.extapps/flight_monitor.fap

# Copy to Flipper:
/ext/apps/Bluetooth/flight_monitor.fap
```

### 2. Set Up Python Server

#### Option A: GUI Server (Recommended)

```bash
cd applications_user/flight_monitor
python flight_server_gui.py
```

The GUI server features:
- Visual interface with status indicators
- IP configuration with persistence
- Auto-start functionality (500ms delay)
- **Auto-dependency installation** (no manual pip install needed!)
- Log output for debugging

#### Option B: Console Server

```bash
cd applications_user/flight_monitor
python flight_server_clean.py
```

The console server is lighter weight and includes:
- Auto-dependency installation
- Clean console output
- Same functionality as GUI version

**Note:** Both servers automatically install missing dependencies (`requests`, `bleak`) on first run!

### 3. Enable War Thunder API

The War Thunder API is enabled by default when the game is running:

1. Launch War Thunder
2. API endpoint is available at: `http://localhost:8111/state`
3. Verify in browser - you should see JSON data
4. Enter a battle to start receiving live flight data

## 🎮 Usage Instructions

### On Flipper Zero:

1. Power on Flipper Zero
2. Navigate to: **Apps → Bluetooth → Flight Monitor**
3. Application shows splash screen for 3 seconds
4. Settings menu appears - configure thresholds or press **OK** on **START**
5. Status shows **"Waiting for data..."** or **"No flight data"**

### On PC:

1. Ensure War Thunder is running
2. Launch Python server (GUI or console version)
3. Server will auto-install dependencies if needed
4. Server finds Flipper and starts transmitting data
5. Watch status indicators for connection confirmation

### In-Game:

1. Enter any battle (Air Battles recommended)
2. Get into aircraft cockpit
3. Flight data appears on Flipper display!
4. Switch views with directional buttons
5. Alarms activate based on configured thresholds

## 📊 Data Flow Diagram

```
┌──────────────┐         ┌───────────────┐         ┌──────────────┐
│  War Thunder │ ◄─HTTP─►│ Python Server │ ◄─BLE──►│ Flipper Zero │
│   localhost  │  JSON   │  (PC)         │ Binary  │  (Display)   │
│   :8111      │         │               │         │              │
└──────────────┘         └───────────────┘         └──────────────┘
     100ms                   Processing                100ms
    Polling                  + Encoding              Refresh Rate
```

1. Game exports telemetry via HTTP (JSON format)
2. Python server polls every 100ms
3. Data parsed and encoded to binary (20 bytes)
4. Transmitted via BLE to Flipper Zero
5. Flipper decodes and displays with visual elements
6. Alarm system evaluates thresholds continuously

## 🛠️ Technical Specifications

### Binary Protocol Format

```c
// 20-byte packet structure
struct __attribute__((packed)) FlightData {
    int16_t  altitude;        // Altitude in meters (-32768 to 32767)
    uint16_t speed;           // Speed in km/h (0 to 65535)
    int8_t   vertical_speed;  // Vertical speed in m/s (-128 to 127)
    int16_t  throttle;        // Throttle 0-100 (scaled)
    int8_t   pitch;           // Pitch angle in degrees
    int8_t   roll;            // Roll angle in degrees
    uint8_t  gear;            // Gear status: 0=UP, 1=DOWN
    uint8_t  flaps;           // Flaps percentage 0-100
    int16_t  power;           // Engine power in HP
    int16_t  fuel;            // Fuel in kg
};
```

### BLE Characteristics

* **Service UUID:** `8fe5b3d5-2e7f-4a98-2a48-7acc60fe0000`
* **RX Characteristic UUID:** `19ed82ae-ed21-4c9d-4145-228e62fe0000`
* **Device Name Pattern:** `"Flight XXXX"` (XXXX = last 4 MAC digits XOR 0x0003)

### Configuration File Structure

Binary file stored at `/ext/apps_data/flight_monitor/settings.cfg`:

```c
typedef struct {
    uint16_t altitude_alarm;      // Default: 120 meters
    uint16_t gear_warning_alt;    // Default: 150 meters
    uint16_t gear_retract_alt;    // Default: 200 meters
    uint16_t stall_speed;         // Default: 100 km/h
    uint16_t overspeed;           // Default: 700 km/h
} FlightConfig;
```

### Performance Metrics

* **Refresh Rate:** 100ms (10 Hz)
* **BLE Latency:** <50ms typical
* **Data Packet Size:** 20 bytes
* **Configuration Size:** 10 bytes
* **Memory Footprint:** ~2 KB stack size

## 📁 Project Structure

```
flight_monitor/
├── application.fam          # Flipper application manifest
├── flight_monitor.h         # Main header file
├── flight_monitor.c         # Main application logic
├── icon.png                 # 10x10 airplane sprite
├── manifest.yml             # App catalog manifest
├── CHANGELOG.md             # Version history
├── README.md                # This file
├── helpers/                 # Helper modules
├── views/                   # View rendering code
├── screenshots/             # Application screenshots
│   ├── 1.png               # Main dashboard
│   ├── 2.png               # Settings menu
│   └── 3.png               # Alerts in action
├── flight_server_gui.py     # Python server with GUI
├── flight_server_clean.py   # Python server (console)
└── test_flight.py           # Testing utilities
```

## 🐛 Troubleshooting

### "Cannot connect to game"
- Verify War Thunder is running
- Check API in browser: http://localhost:8111/state
- If not working, restart the game

### "Flipper Zero not found"
- Ensure Flight Monitor app is running on Flipper
- Check that PC Bluetooth is enabled
- Try restarting the app on Flipper

### "Connection Lost"
- Distance between PC and Flipper too large
- Move devices closer together
- Restart Python server

### Data not updating
- Make sure you're in flight (not hangar)
- Restart Python server
- Check logs in terminal

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📜 License

This project is distributed under the MIT License.

## 👨‍💻 Author

**Dr.Mosfet**

Created for the War Thunder and Flipper Zero communities with ❤️

## 🙏 Acknowledgments

* Flipper Zero development team for the amazing platform
* War Thunder for the flight simulation experience
* BME680 app developers for menu inspiration
* PC Monitor app by Olejka for BLE foundation
* Community testers and contributors

## 📞 Support

For issues, questions, or feature requests:
- Open an issue on GitHub
- Check existing documentation
- Review CHANGELOG.md for version history

---

**Version:** 1.0.0  
**Last Updated:** April 6, 2026  
**Status:** Stable Release

**Note:** Using this during actual flight is strongly discouraged! 😄
