# TF03 LiDAR Integration with Pixhawk (2.4.8 & CubePilot)

This repository provides a complete guide to integrating the **Benewake TF03 LiDAR rangefinder** with the **Pixhawk 2.4.8** or **CubePilot Cube+** flight controller using ArduCopter 4.6.x firmware.

Designed for object detection, terrain following, and low-altitude control.

---

## 📌 Hardware Used

- **LiDAR**: Benewake TF03 (100m version)
- **Flight Controller**: Pixhawk 2.4.8 / CubePilot Cube+
- **Firmware**: ArduCopter v4.6.1
- **Connection Type**: UART (Serial)
- **Operating Mode**: Manual RC, not Auto

---

## 🔌 Wiring

### TF03 Pinout

| TF03 Wire | Signal     | Connects To Pixhawk |
|-----------|------------|---------------------|
| Red       | +5V        | VCC                 |
| Black     | GND        | GND                 |
| Yellow    | TX         | RX of SERIAL4       |
| Green     | RX         | TX of SERIAL4       |

> **Note**: TF03 uses UART (not I2C or CAN) in this configuration. Confirm JST-to-DF13 wiring with a multimeter.

---

## 🔧 Serial Port Setup

| Parameter        | Value     | Purpose |
|------------------|-----------|---------|
| `SERIAL4_PROTOCOL` | `9`       | Lidar type |
| `SERIAL4_BAUD`     | `115200` | TF03 default baud |

---

## 📡 Rangefinder Setup

| Parameter         | Value       | Description |
|-------------------|-------------|-------------|
| `RNGFND1_TYPE`     | `27`        | Benewake Serial |
| `RNGFND1_MIN_CM`   | `30`        | Minimum range (cm) |
| `RNGFND1_MAX_CM`   | `500`       | Maximum usable range |
| `RNGFND1_ORIENT`   | `25`        | Downward (25) or Forward (0) |
| `RNGFND1_SCALING`  | `1.0`       | Keep default |
| `RNGFND1_ADDR`     | `0`         | Not used for serial |

---

## 📐 EKF Settings (Altitude Source)

| Parameter             | Value | Description |
|-----------------------|-------|-------------|
| `EK3_SRC1_POSZ`       | `1`   | Use rangefinder |
| `EK3_RNG_USE_HGT`     | `3`   | Use RF below 3m AGL |
| `EK3_HGT_MODE`        | `0`   | Keep Baro as main source |

> Set `RNGFND1_MAX_CM ≥ EK3_RNG_USE_HGT * 100`. Otherwise, EKF will ignore RF readings.

---

## 🎮 RC Controlled Avoidance (Manual Mode)

To **enable obstacle avoidance manually**:

1. Set `RC6_OPTION = 38` or `40`  
   - `38` → Enable ADAB (Avoidance Dynamic Adjusted Braking)  
   - `40` → Enable Obstacle Avoidance in Manual Mode
2. Set switch on RC to Channel 6
3. Test by toggling switch during flight; avoidance activates dynamically

---

## ✈️ Prevent Drone from Descending Below a Fixed Height (Hover Floor)

To **prevent descending below X meters**, e.g. **1m** for test:

```lua
-- Lua script (scripts/range_limit_logic.lua)
if rangefinder:distance_cm() < 100 then
  -- Prevent descent
end
```

### 🔁 Alternatively, Use Parameter-Only Setup

- `EK3_RNG_USE_HGT = 1`
- `RNGFND1_MAX_CM = 150`
- Use **AltHold** mode

---

### 🛠️ Troubleshooting

#### Common Problems

| **Symptom**               | **Likely Cause**                   | **Fix**                                                            |
|---------------------------|------------------------------------|---------------------------------------------------------------------|
| Only detecting 0.5–1m     | Wrong `MAX_CM` or wrong serial config | Match TF03 with correct `SERIALx_BAUD` and `PROTOCOL = 9`          |
| No readings               | Wrong wiring (TX/RX flipped)       | Use logic analyzer or re-check cable pinout                        |
| TF03 not detected         | Baud mismatch or wrong port        | Use Benewake Viewer to verify TF03 UART settings and baud rate     |

