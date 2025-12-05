# 🐀 IPESTCONTROL

**Smart IoT Pest Detection & Repellent System with Computer Vision**

**IPESTCONTROL** is an advanced, automated pest management system powered by a Raspberry Pi 5 and YOLOv8 AI. It features real-time object detection, dual operating modes, and a secure, isolated power architecture for controlling high-voltage components.

-----

## 🚀 Key Features

  * **🤖 AI-Powered Detection:** Uses **YOLOv8 Nano** running on Raspberry Pi 5 to detect rats in real-time.
  * **📹 Live Monitoring:** Low-latency MJPEG video streaming to a custom Flutter mobile app.
  * **📱 Dual-Interface WiFi:**
      * **Always-On Hotspot:** `IPESTCONTROL` (10.42.0.1) for standalone field use.
      * **Client Mode:** Connects to home WiFi for remote access.
  * **🔋 Smart Power Management:**
      * **System A (Brain):** Powered by UPS HAT (5V) for safe shutdown.
      * **System B (Muscle):** Isolated 12V Li-Ion system for high-power loads (Zapper, IR, Strobes).
  * **🛡️ Safety First:** Galvanically isolated control system using Optocoupler Relays to protect the Pi.

-----

## 📂 Scripts & Architecture

The system runs two main Python services in parallel:

| Script Name | Port | Function |
| :--- | :--- | :--- |
| **`stream_final_1.py`** | **5000** | **The Eyes & Brain.** Handles Camera capture, YOLO AI detection, MJPEG video streaming, and Battery stats reading (UPS). |
| **`pi_wifi_manager.py`** | **80** | **The System Manager.** Handles WiFi Hotspot/Client switching, Saved Networks, and the **Remote Shutdown** command. |

-----

## ⚙️ Operating Modes

### 1\. 🐀 Pest Mode (Rodent Control)

  * **Target:** Rats, Mice.
  * **Action:** AI Detection \> **IR Light (Night Vision)** \> **Strobe Light** + **Ultrasonic Buzzer**.

### 2\. 🦟 Insect Mode (Zapper)

  * **Target:** Mosquitoes, Moths.
  * **Action:** **Blue/UV Light** (Attractant) \> **High-Voltage Zapper** (Elimination).

-----

## 🛠️ Setup Guide

### **1. Hardware Assembly**

1.  **Power Isolation:** Wire the **12V System (Island B)** completely separate from the **5V Pi System (Island A)**.
      * *CRITICAL:* Do NOT connect the grounds of the two batteries together.
2.  **Buck Converter:** Tune the LM2596 to output exactly **5.0V** before connecting the ESP8266.
3.  **Relays:** Remove the yellow `JD-VCC` jumper. Connect `JD-VCC` to the "Dirty" 5V rail (LC Filter output) and `VCC` to the "Clean" 5V rail.

### **2. Software Installation**

1.  **Flash OS:** Raspberry Pi OS (Bookworm) 64-bit.
2.  **Install Dependencies:**
    ```bash
    sudo apt update && sudo apt install -y python3-opencv python3-flask network-manager
    pip3 install ultralytics picamera2 flask pyserial
    ```
3.  **Clone Project:** Place scripts in `/home/yohan/rat_detection/scripts/`.

### **3. Service Configuration**

Create systemd services to run the scripts on boot.

**Camera Service (`/etc/systemd/system/rat-camera.service`):**

```ini
[Service]
ExecStart=/home/yohan/yolo_object/bin/python3 /home/yohan/rat_detection/scripts/stream_final_1.py
Restart=always
User=root
```

**Manager Service (`/etc/systemd/system/wifi-manager.service`):**

```ini
[Service]
ExecStart=/home/yohan/yolo_object/bin/python3 /home/yohan/rat_detection/scripts/pi_wifi_manager.py
Restart=always
User=root
```

**Enable Services:**

```bash
sudo systemctl enable rat-camera
sudo systemctl enable wifi-manager
```

-----

## 💻 Command Cheat Sheet

### **Service Management**

| Action | Command |
| :--- | :--- |
| **Start Manager** | `sudo systemctl start wifi-manager` |
| **Stop Manager** | `sudo systemctl stop wifi-manager` |
| **Restart Manager** | `sudo systemctl restart wifi-manager` |
| **Check Status** | `sudo systemctl status wifi-manager` |

### **Debugging & Processes**

| Action | Command |
| :--- | :--- |
| **Find Camera User** | `sudo fuser -v /dev/media*` |
| **Kill Process** | `sudo kill -9 [PID]` |
| **Check Open Ports** | `sudo netstat -tulpn | grep python` |
| **Get IP Address** | `hostname -I` |
| **Fix WiFi Drops** | `sudo iw dev wlan0 set power_save off` |

-----

## 🔍 Troubleshooting & Logs

### **1. "Pi Not Found" in App**

  * **Cause:** Phone is on a different network or Mobile Data is interfering.
  * **Fix:**
    1.  Connect phone to **IPESTCONTROL** or the **Same Home WiFi**.
    2.  **Turn OFF Mobile Data**.
    3.  Restart the App.

### **2. Camera Offline / Black Screen**

  * **Cause:** Another process (like a zombie script) is holding the camera.
  * **Fix:**
    ```bash
    sudo fuser -k -v /dev/media* # Kills process using camera
    # Then restart your script
    ```

### **3. WiFi Drops / Disconnects**

  * **Cause:** Raspberry Pi Power Saving mode.
  * **Fix:**
    ```bash
    sudo iw dev wlan0 set power_save off
    ```

### **4. Viewing Logs**

If something crashes, check the live logs:

```bash
# View Manager Logs
journalctl -u wifi-manager -f

# View Camera Logs (if set up as service)
journalctl -u rat-camera -f
```

-----

## ✅ Dos and Don'ts

### **DO:**

  * **✅ DO** use the "Shutdown System" button in the app before unplugging the battery to prevent SD card corruption.
  * **✅ DO** wait 30-60 seconds after boot for the WiFi hotspot to appear.
  * **✅ DO** check the 12V battery voltage in the app periodically.

### **DON'T:**

  * **❌ DON'T** connect the Ground (GND) of the 12V battery to the Raspberry Pi. (Risk of noise crash).
  * **❌ DON'T** plug 12V directly into the ESP8266 or Relay `VCC` pin (Instant burnout).
  * **❌ DON'T** touch the Zapper mesh while the system is armed.

-----

## 👥 Developers

  * **Yohan** - Lead Developer (Hardware & Software Integration)

-----

*v1.0.0 - Final Capstone Build*
