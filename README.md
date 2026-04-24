# 📸 ESP32-CAM Smart Attendance System

![Platform](https://img.shields.io/badge/Platform-ESP32--CAM-E7352C?style=for-the-badge&logo=espressif&logoColor=white)
![Protocol](https://img.shields.io/badge/Protocol-WhatsApp%20API-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)
![Display](https://img.shields.io/badge/Display-OLED%20I2C-blue?style=for-the-badge)
![IoT](https://img.shields.io/badge/Category-IoT%20%7C%20Embedded-orange?style=for-the-badge)
[![License](https://img.shields.io/badge/License-Open%20Source-green?style=for-the-badge)](LICENSE)

An automated, IoT-powered attendance system built with the **ESP32-CAM**. It captures a student's image, records an accurate timestamp via NTP, and instantly sends the attendance proof — **name, IN/OUT status, time, location, and photo** — directly to a WhatsApp number using the CircuitDigest Cloud API.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Components Required](#components-required)
- [Circuit Diagram & Wiring](#circuit-diagram--wiring)
- [How It Works](#how-it-works)
- [Source Code Explanation](#source-code-explanation)
- [Getting Started](#getting-started)
- [Troubleshooting](#troubleshooting)
- [Applications](#applications)
- [Future Enhancements](#future-enhancements)
- [FAQ](#faq)
- [References](#references)

---

## 🌐 Overview

Traditional manual attendance systems are repetitive, error-prone, and time-consuming. This project replaces them with a **smart, automated solution** that:

- 📷 Captures an image of the student as visual proof
- 🕐 Fetches real-time timestamps from an NTP server
- 📲 Sends attendance records instantly to WhatsApp
- 🔘 Uses a rotary encoder for simple, hands-free user interaction
- 📟 Displays everything on an OLED screen

> ✅ No server required — runs fully on the ESP32-CAM with cloud API integration.

---

## 🛠️ Components Required

| S.No | Component | Purpose |
|------|-----------|---------|
| 1 | **ESP32-CAM** | Microcontroller + image capture |
| 2 | **Rotary Encoder** | Navigate and select menu options |
| 3 | **OLED Display (0.96")** | Display student names and system status |
| 4 | **Perf/Puff Board** | Easy component interconnection |
| 5 | **USB Power Supply (5V)** | Power the system |
| 6 | **Jumper Wires** | Circuit connections |

---

## 🔌 Circuit Diagram & Wiring

The system connects the **ESP32-CAM**, **OLED display (I2C)**, and **Rotary Encoder** as follows:

| Component | Pin | ESP32-CAM Pin |
|-----------|-----|---------------|
| OLED SDA | SDA | GPIO 14 |
| OLED SCL | SCL | GPIO 15 |
| Rotary CLK | CLK | GPIO 13 |
| Rotary DT | DT | GPIO 12 |
| Rotary SW | SW | GPIO 2 |
| VCC (OLED + Encoder) | 3.3V | 3.3V |
| GND | GND | GND |

> 📌 Refer to the full circuit diagram in the [project article](https://circuitdigest.com/microcontroller-projects/attendance-system-using-esp32-cam-development-board).

---

## ⚙️ How It Works

```
Power ON → Connect WiFi → Sync NTP Time → Show Student List on OLED
     ↓
Student rotates encoder → Scrolls through names
     ↓
Student clicks encoder → Selects name
     ↓
Rotate to choose IN / OUT → Click to confirm
     ↓
Countdown: 3... 2... 1... → Flash LED ON → Camera Captures Image
     ↓
Data packaged: Name + IN/OUT + Timestamp + Location + Image
     ↓
HTTP POST → CircuitDigest Cloud API → WhatsApp Message Delivered ✅
```

**WhatsApp message includes:**
- 📷 Captured image of the student
- 👤 Student name
- 🚪 Entry / Exit status (IN or OUT)
- 🕐 Exact date and time
- 📍 Predefined location

---

## 💻 Source Code Explanation

### 1. WiFi & API Credentials
```cpp
const char *ssid    = "YOURSSID";
const char *pwd     = "YOURWIFIPASSWORD";
const char *apiKey  = "YOURAPIKEY";
const char *phone   = "YOURNUMBER";
```
> Replace these with your actual credentials before uploading.

---

### 2. Camera Initialization
```cpp
cfg.pixel_format = PIXFORMAT_JPEG;
cfg.frame_size   = FRAMESIZE_QVGA;
cfg.jpeg_quality = 12;
cfg.fb_count     = 1;

if (esp_camera_init(&cfg) != ESP_OK) {
    ESP.restart();
}
```
> Configures the camera for JPEG output at QVGA resolution. Auto-restarts if initialization fails.

---

### 3. Rotary Encoder Reading
```cpp
int clkState = digitalRead(CLK);
if (clkState == LOW && lastCLK == HIGH) {
    lastCLK = clkState;
    return (digitalRead(DT) != clkState) ? 1 : -1;
}
```
> Returns `+1` for clockwise rotation (scroll down) and `-1` for counter-clockwise (scroll up).

---

### 4. Student List
```cpp
String users[]  = {"Vedha", "Arun", "Priya", "Kiran"};
int totalUsers  = 4;
int selectedIndex = 0;
```
> Add or remove names here to customize the student list.

---

### 5. Sending Data to WhatsApp
```cpp
client.println("POST /api/v1/whatsapp/send-with-image HTTP/1.1");
client.println("Host: www.circuitdigest.cloud");
client.print(body);
client.write(imgBuf, imgLen);
```
> Sends attendance data and the captured image to the cloud API as a multipart HTTP POST request.

---

## 🚀 Getting Started

### Prerequisites
- Arduino IDE with **ESP32 board support** installed
- Required Libraries:
  - `esp_camera.h`
  - `WiFi.h`
  - `Wire.h`
  - `Adafruit_GFX.h`
  - `Adafruit_SSD1306.h`
  - `HTTPClient.h` or `WiFiClientSecure.h`
- A CircuitDigest Cloud API Key → [Get it here](https://circuitdigest.com/microcontroller-projects/send-whatsapp-messages-using-arduino-circuitdigest-cloud)

### Steps
1. **Clone the repository:**
   ```bash
   git clone https://github.com/Circuit-Digest/Smart-Attendance-System.git
   cd Smart-Attendance-System
   ```

2. **Open the `.ino` file** in Arduino IDE.

3. **Update credentials** in the code:
   ```cpp
   const char *ssid   = "YOUR_WIFI_SSID";
   const char *pwd    = "YOUR_WIFI_PASSWORD";
   const char *apiKey = "YOUR_API_KEY";
   const char *phone  = "YOUR_PHONE_WITH_COUNTRY_CODE";
   ```

4. **Add your student names** to the `users[]` array.

5. **Select Board:** `AI Thinker ESP32-CAM`

6. **Upload** and open Serial Monitor at `115200 baud` to verify.

---

## 🔧 Troubleshooting

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| **WiFi not connecting** | Wrong credentials or out of range | Re-check SSID/password; move closer to router |
| **Camera init failed** | Wrong pin config or low power | Verify pin definitions; use stable 5V supply |
| **OLED not displaying** | Wrong I2C address or bad wiring | Try `0x3C` or `0x3D`; check SDA/SCL connections |
| **Image not sent to WhatsApp** | Invalid API key or network issue | Verify API key; check internet connection |
| **Rotary encoder skipping** | Noise or loose connections | Add debounce delay in code; check wiring |

---

## 📦 Applications

- 🏢 **Offices & Companies** — Employee attendance with real-time WhatsApp updates
- 🎓 **Schools & Coaching Centers** — Student attendance with digital records
- 🎪 **Events & Conferences** — Quick check-in with image verification
- 🏠 **Hostels & Secure Areas** — Entry/exit monitoring with photo proof
- 📚 **Libraries & Study Centers** — Track student visits digitally
- 📝 **Examination Halls** — Prevent malpractice with image-based attendance

---

## 🔮 Future Enhancements

- 🤖 **Face Recognition** — Auto-identify students without manual selection
- ☁️ **Cloud Database** — Store all attendance records online (Firebase / Google Sheets)
- 📱 **Mobile App** — Real-time notifications and remote monitoring
- 📍 **GPS Location Tagging** — Record exact location along with attendance
- 🔑 **Fingerprint / RFID Support** — Multi-factor authentication for better security

---

## ❓ FAQ

**Q1. Can this system work without the internet?**
> No. Internet is required for NTP time sync and sending WhatsApp messages.

**Q2. How does the system get the correct time?**
> It fetches real-time data from an **NTP (Network Time Protocol)** server automatically after connecting to WiFi.

**Q3. What is the rotary encoder used for?**
> It lets users scroll through the student list and select IN/OUT options without a touchscreen or keyboard.

**Q4. How is attendance sent to the user?**
> Via an HTTP POST request to the **CircuitDigest Cloud API**, which forwards it as a WhatsApp message with the captured image.

**Q5. What happens if WiFi is lost?**
> The system attempts to reconnect automatically and may restart if reconnection fails.

**Q6. Can I add more students?**
> Yes! Simply add names to the `users[]` array in the code and update `totalUsers`.

**Q7. Is the system secure?**
> Basic API-key authentication is provided. You can enhance security by adding HTTPS, encrypted storage, or user PINs.

---

## 📚 References

- 📄 [Project Article – CircuitDigest](https://circuitdigest.com/microcontroller-projects/attendance-system-using-esp32-cam-development-board)
- 💬 [Send WhatsApp from ESP32 – Tutorial](https://circuitdigest.com/microcontroller-projects/send-whatsapp-messages-using-arduino-circuitdigest-cloud)
- 🔔 [Motion Detection + WhatsApp Alert with ESP32](https://circuitdigest.com/microcontroller-projects/real-time-motion-detection-with-xiao-esp32-whatsapp-alert-system)

---

## 📄 License

This project is **open-source**. Feel free to use, modify, and distribute it for educational and non-commercial purposes.

---

*Developed with ❤️ by the [Circuit Digest](https://circuitdigest.com) Team.*
