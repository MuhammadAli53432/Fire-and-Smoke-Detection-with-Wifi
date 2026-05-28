# ESP32 Gas & Fire Alert System with Blynk IoT

A smart, connected home safety system using an **ESP32**, **MQ2 Gas Sensor**, and **Flame Sensor** that sends instant push notifications to your smartphone via **Blynk IoT**.

---

## 🌟 Key Features

1. **Dynamic WiFi Provisioning (No Hardcoded Passwords)**: Uses `WiFiManager` to allow connecting the ESP32 to any WiFi network on-the-fly via a smartphone portal.
2. **Real-time Monitoring**: Scans for gas leakage and fire hazards every 2 seconds.
3. **Smart Rate-Limited Alerts**: Prevents notification spamming by incorporating status state-tracking and a 30-second cooldown timer.
4. **Blynk Cloud Integration**: Employs Blynk's IoT infrastructure for remote monitoring and push notifications.

---

## 🌐 Why Use Dynamic WiFi (WiFiManager)?

In standard ESP32 projects, developers hardcode the WiFi Name (SSID) and Password into the code:
```cpp
char ssid[] = "MyHomeWiFi";
char pass[] = "12345678";
```

### The Problems with Hardcoded WiFi:
* **Zero Portability**: If you move the device to another room, office, or router, it loses connection. You must plug it back into your computer and re-flash the entire program.
* **Security Risk**: If you share your code on GitHub, your private WiFi password is exposed.
* **Inconvenience**: If you change your home WiFi password, your device stops working until you rewrite the code.

### The Solution: Dynamic WiFi Manager
Our code integrates `WiFiManager`. Here is how it behaves:
1. **On Boot**: The ESP32 searches for previously saved WiFi credentials in its Non-Volatile Storage (NVS).
2. **Access Point Mode**: If it cannot find any saved credentials (or fails to connect), it hosts its own WiFi hotspot called **`ESP32-Smart-Alert-AP`** (Password: `12345678`).
3. **Captive Portal**: You connect your phone to this network, and a menu appears automatically (or you navigate to `192.168.4.1` in a browser).
4. **Save Credentials**: You scan for local networks, select yours, enter the password, and click **Save**.
5. The ESP32 stores the details, closes the hotspot, connects to your home router, and starts monitoring. **It remembers these details even after powering off!**

---

## 🔌 Hardware Connections & Wiring

| Sensor | ESP32 Pin | Sensor Pin Type | Description |
| :--- | :--- | :--- | :--- |
| **MQ2 Gas Sensor** | `GPIO 32` | Digital Output (`D0` or `DOUT`) | Goes `HIGH` when gas is detected. |
| **Flame Sensor** | `GPIO 35` | Digital Output (`D0` or `OUT`) | Goes `LOW` when fire/flame is detected. |
| **VCC (Both Sensors)** | `3.3V` or `VIN` | Power Input | Powers the sensor modules. |
| **GND (Both Sensors)** | `GND` | Ground | Common reference ground. |

*Note: For the sensors to trigger correctly, adjust the small blue potentiometer dial on each sensor board using a screwdriver until the detection LED turns on at the desired sensitivity.*

---

## 📝 Code Walkthrough & Logic

The source code file can be found here: [esp32_blynk_gas_fire_alert.ino](file:///c:/Users/Muhammad%20Anique/Desktop/blynk%20project%20app/esp32_blynk_gas_fire_alert/esp32_blynk_gas_fire_alert.ino).

### 1. Credentials Setup (Lines 15–17)
```cpp
#define BLYNK_TEMPLATE_ID "TMPL6aAQQet1g"
#define BLYNK_TEMPLATE_NAME "GAS or FLAME ALERT"
#define BLYNK_AUTH_TOKEN "T1CDtagRYFoeGpnebBJ-esg6Mhc2wOkn"
```
These lines must be at the very top. They authenticate your hardware device with your specific Blynk Cloud account.

### 2. Connecting via WiFiManager (Lines 123–140)
Instead of calling `Blynk.begin()`, which locks the ESP32 to a single hardcoded network, we run `WiFiManager`:
```cpp
WiFiManager wm;
if (!wm.autoConnect("ESP32-Smart-Alert-AP", "12345678")) {
  ESP.restart(); // Restart if setup fails or times out
}
```

### 3. Config Blynk with Active Connection (Lines 147–152)
Once WiFiManager secures a connection to the internet, we initialize Blynk using that active connection:
```cpp
Blynk.config(BLYNK_AUTH_TOKEN);
Blynk.connect();
```

### 4. Non-Spamming Alert Logic (Lines 48–94)
Blynk will block accounts that spam notifications. The code implements state flags (`isGasAlertActive`, `isFireAlertActive`) and a cooldown time:
```cpp
const unsigned long ALERT_COOLDOWN_MS = 30000; // 30 seconds
```
* **Event Trigger**: When a sensor goes off, `Blynk.logEvent()` is called, and the state changes to `true`.
* **State Preservation**: The code will not send another notification until the alert is cleared, or until **30 seconds** have passed, preventing server-side rate limits from blocking your device.

---

## ⚙️ Blynk Console Setup

To ensure notifications appear on your smartphone, make sure you configure your Blynk Web Console events:
1. Navigate to **Developer Zone** ➡️ **Templates** ➡️ **GAS or FLAME ALERT** ➡️ **Events & Notifications**.
2. Create/edit two events:
   * **Gas Alert** (Code: `gas_alert`)
   * **Fire Alert** (Code: `fire_alert`)
3. Under the **Notifications** tab of each event:
   * Turn **ON** *Enable notifications*.
   * Turn **ON** *Push notifications to mobile app*.
   * Select **`Device Owner`** in the *Deliver to* dropdown.
4. Under the **Settings** tab of each event:
   * **Crucial**: Change **Event will be sent to user only once per** from `1 hour` to `No limit` or `1 minute` so you can test it repeatedly.
5. Click **Save and Ship** on the template page.
