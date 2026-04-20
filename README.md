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

   
## 🚀 Software Installation (PlatformIO)
This project is built using PlatformIO. If you do not have PlatformIO installed, it is essentially an extension for Visual Studio Code (VSCode). 
👉 **[Read the Official PlatformIO Installation Guide Here](https://docs.platformio.org/en/latest/integration/ide/vscode.html)**

Once PlatformIO is installed and running:
1. Create a new project in PlatformIO and select the `AI Thinker ESP32-CAM` board.
2. Replace the automatically generated `platformio.ini` file with the **`platformio.ini`** provided in this repository (This is CRITICAL as it enables the PSRAM required for the camera to function).
3. Replace the `src/main.cpp` file with the **`main.cpp`** provided in this repository.
4. Open `main.cpp` and insert your Wi-Fi credentials:
   ```cpp
   const char* ssid = "YOUR_WIFI_SSID";
   const char* password = "YOUR_WIFI_PASSWORD";
   ```
3. Connect your Aideepen ESP32-CAM-MB to your PC and click **Upload**.

## ⚠️ Troubleshooting: The "It Won't Upload/Boot" Guide
The ESP32-CAM is notorious for being frustrating to flash. If you hit a wall, you are not alone. Here are the most common issues and their exact fixes:

### 1. Compilation Error: `fatal error: WiFi.h: No such file or directory`
* **The Cause:** Your PlatformIO configuration is trying to use the native C Espressif IoT framework (`framework = espidf`), but the code is written for Arduino C++.
* **The Fix:** Ensure your `platformio.ini` has `framework = arduino`. (If you use the file provided in this repository, this is already fixed for you).

### 2. Upload Error: `Failed to connect to ESP32: Timed out waiting for packet header`
* **The Cause:** The ESP32 is not entering "Download Mode". 
* **The Fix (Aideepen MB):** The Aideepen motherboard usually handles this automatically. If it fails, hold the `IO0` button, press the `RST` button once, then release `IO0`, and try uploading again.
* **The Fix (Generic FTDI):** If you are not using the Aideepen motherboard, you MUST physically connect the `GPIO 0` pin to the `GND` pin with a jumper wire before applying power to flash the board.

### 3. Linux/Ubuntu Error: `[Errno 5] Input/output error` or Port Disconnects
* **The Cause:** On modern Linux distributions (like Ubuntu 22.04/24.04), a background service called `brltty` (a Braille display reader) mistakenly identifies the CH340G USB chip as a Braille terminal and hijacks/disconnects the port.
* **The Fix:** Open your Linux terminal and aggressively remove the service:
  ```bash
  sudo apt remove brltty
  sudo udevadm control --reload-rules

## 🏠 Home Assistant Integration (Phase 1)
To integrate this into a Home Assistant dashboard without using a heavy camera integration, use an HTML card with a simple Javascript timer. This silently forces a refresh every 15 minutes, fetching the latest snapshot.

```html
<img id="legacy_device_cam" src="http://<YOUR_ESP32_IP>/capture">
<script>
  setInterval(function() {
    var cam = document.getElementById("legacy_device_cam");
    cam.src = "http://<YOUR_ESP32_IP>/capture?t=" + new Date().getTime();
  }, 900000); // 15 minutes
</script>
```

***
   
