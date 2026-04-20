# Legacy-Display-bridge
# Universal Optical Bridge for Legacy Displays (ESP32-CAM)

![Platform](https://img.shields.io/badge/Platform-ESP32--CAM-green) ![Sensor](https://img.shields.io/badge/Sensor-OV3660_3MP-orange) ![Home Assistant](https://img.shields.io/badge/Integration-Home_Assistant-41BDF5)

## 📌 The Concept: Giving "Dumb" Devices a Voice
Our homes and industries are filled with perfectly functional but "dumb" legacy devices that feature LCD or analog displays but lack Wi-Fi, Bluetooth, or APIs (e.g., Radon detectors, old thermostats, water meters, washing machines).

This project uses an **Aideepen ESP32-CAM-MB** to act as an "Artificial Eye." Instead of spending hundreds of dollars replacing perfectly good standalone devices with "Smart" versions, this $10 hardware bridge visually ports the physical screen into your Home Assistant dashboard.

*Example use-case in this repo: Monitoring an Airthings Corentium Home Radon detector.*

## ⚙️ Hardware Specifications
* **Motherboard:** Aideepen ESP32-CAM-MB (Micro USB)
* **USB-to-Serial Chip:** CH340G (Built-in auto-download circuit)
* **Camera Sensor:** OV3660 (3 Megapixel) with double mode support / NodeMCU compatible.

## 🏗️ Architecture: The "Static Snapshot" Advantage
Most ESP32-CAM tutorials instruct you to deploy a heavy WebServer pushing a continuous MJPEG video stream. **This is an anti-pattern for monitoring static displays.**
1. **Thermal Throttling & Brownouts:** Generating 30 frames per second on an ESP32 causes the chip to overheat, leading to Wi-Fi drops and endless reboot loops.
2. **Resource Waste:** Values on environmental sensors (like Radon, humidity, or water meters) change very slowly. Pushing video for a static screen wastes massive network bandwidth.

**Our Pragmatic Approach:**
This firmware runs a dormant, ultra-lightweight Web Server. When Home Assistant (or your browser) calls the `http://<ESP32_IP>/capture` endpoint, the ESP32 wakes the OV3660 sensor, takes a single, perfectly exposed JPEG snapshot, serves it, and immediately frees the RAM. This guarantees 100% hardware stability over months of uptime.

## 🛠️ Physical Setup & Optical Calibration
1. **Power Supply:** The Aideepen module requires a stable 5V wall charger (minimum 1A, preferably 2A). Do not run it off a weak PC USB port in production.
2. **Focus Adjustment (CRITICAL):** 
   **Buy default, the camera lens is focused to infinity. If placed 5cm away from an LCD screen, the image will be entirely blurry. Move the object further if possible.
   * **Fix:** If not possible., look at the camera lens thread. Carefully scratch off the tiny factory glue dot holding the focus ring. Gently rotate the lens to adjust the focal length for macro photography until the LCD segments are readable.


## 🚀 The Camera only need Software Installation (PlatformIO)
1. Clone this repository and open it in VSCode with the PlatformIO extension.
2. Open `src/main.cpp` and update your Wi-Fi credentials:
   ```cpp
   const char* ssid = "YOUR_WIFI_SSID";
   const char* password = "YOUR_WIFI_PASSWORD";
