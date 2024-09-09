

---

### ESP BLE Provisioning Using ESP32
#### Embedded Software Intern Assignment - Nineti GmbH

---

### Introduction
This project involves implementing ESP BLE provisioning using the ESP-IDF framework. The objective is to enable Wi-Fi configuration on an ESP32 microcontroller via Bluetooth Low Energy (BLE). A mobile device, using the ESP BLE Provisioning app, sends Wi-Fi credentials to the ESP32, allowing it to connect to the Wi-Fi network.

---

### Hardware and Software Setup

**Hardware Used:**
- **ESP32 Microcontroller**: Used for BLE provisioning and Wi-Fi connection.
- **Smartphone**: For BLE connection using the ESP BLE Provisioning app.

**Software Used:**
- **ESP-IDF (Espressif IoT Development Framework)**: For writing and deploying the firmware.
- **GitHub**: For version control and project submission.
- **ESP BLE Provisioning App**: For configuring Wi-Fi credentials via BLE.

---

### Implementation Steps

1. **Setting Up ESP32 for BLE Provisioning**
   - Initialize Non-Volatile Storage (NVS) to store Wi-Fi credentials.
   - Configure the ESP32 as a BLE server to allow connections via BLE.
   - Use the ESP BLE Provisioning app to send Wi-Fi credentials (SSID and password) to the ESP32.

   ```cpp
   #include "WiFiProv.h"
   #include "WiFi.h"

   // Define proof of possession (PIN) and service details
   const char* proofOfPossession = "abcd1234"; // PIN provided by the device, entered by user in the app
   const char* deviceName = "PROV_123"; // Device name, typically starting with "Prov_"
   const char* devicePassword = NULL; // Password for SofAP method (NULL means no password required)
   bool clearProvisionedData = true; // Automatically clear previously provisioned data when true

   // Event handler function for WiFi and provisioning events
   void handleSystemProvisioningEvent(arduino_event_t *event) {
       switch (event->event_id) {
       case ARDUINO_EVENT_WIFI_STA_GOT_IP:
           Serial.print("\nIP Address acquired: ");
           Serial.println(IPAddress(event->event_info.got_ip.ip_info.ip.addr));
           break;
       case ARDUINO_EVENT_WIFI_STA_DISCONNECTED:
           Serial.println("\nDisconnected. Reconnecting to AP...");
           break;
       case ARDUINO_EVENT_PROV_START:
           Serial.println("\nProvisioning initiated. Enter your Wi-Fi credentials using the smartphone app.");
           break;
       case ARDUINO_EVENT_PROV_CRED_RECV: {
           Serial.println("\nWi-Fi credentials received:");
           Serial.print("\tSSID: ");
           Serial.println((const char*) event->event_info.prov_cred_recv.ssid);
           Serial.print("\tPassword: ");
           Serial.println((const char*) event->event_info.prov_cred_recv.password);
           break;
       }
       case ARDUINO_EVENT_PROV_CRED_FAIL: {
           Serial.println("\nProvisioning failed! Please reset to factory settings and try again.");
           if (event->event_info.prov_fail_reason == WIFI_PROV_STA_AUTH_ERROR)
               Serial.println("\nIncorrect Wi-Fi AP password.");
           else
               Serial.println("\nWi-Fi AP not found. Consider adding \"nvs_flash_erase()\" before beginProvision().");
           break;
       }
       case ARDUINO_EVENT_PROV_CRED_SUCCESS:
           Serial.println("\nProvisioning successful.");
           break;
       case ARDUINO_EVENT_PROV_END:
           Serial.println("\nProvisioning process ended.");
           break;
       default:
           break;
       }
   }

   void setup() {
       Serial.begin(115200);
       WiFi.onEvent(handleSystemProvisioningEvent);

       Serial.println("Starting BLE provisioning...");
       // UUID for BLE provisioning
       uint8_t serviceUUID[16] = {0xb4, 0xdf, 0x5a, 0x1c, 0x3f, 0x6b, 0xf4, 0xbf,
                                   0xea, 0x4a, 0x82, 0x03, 0x04, 0x90, 0x1a, 0x02 };
       WiFiProv.beginProvision(WIFI_PROV_SCHEME_BLE, WIFI_PROV_SCHEME_HANDLER_FREE_BTDM, WIFI_PROV_SECURITY_1, proofOfPossession, deviceName, devicePassword, serviceUUID, clearProvisionedData);
       log_d("Generated BLE QR Code");
       WiFiProv.printQR(deviceName, proofOfPossession, "ble");
   }

   void loop() {
       // Main loop is intentionally left empty
   }
   ```

2. **Provisioning via ESP BLE Provisioning App**
   - The ESP32 broadcasts its BLE service, and the smartphone connects using the ESP BLE Provisioning app.
   - Provide Wi-Fi credentials (SSID and password) via the app.
   - The ESP32 attempts to connect to the Wi-Fi network after receiving the credentials.

3. **Wi-Fi Connection**
   - After receiving the credentials, the ESP32 connects to the network and prints the connection status:

   ```
   [WiFi] Connected IP address: 192.168.1.100
   ```

---

### Challenges and Troubleshooting
- **BLE Connection Timeout**: Initially, the BLE connection timed out due to incorrect app permissions. Adjusting the app’s permissions resolved the issue.
- **Incorrect Wi-Fi Credentials**: The ESP32 retries connection when credentials are incorrect. Added feedback to the smartphone for connection failures.

---

### Expected Results
- The ESP32 should successfully connect to Wi-Fi after BLE provisioning.
- Screenshots can demonstrate expected outputs.
- ### Screenshots



**Wi-Fi Connection Status:**

![Connection Status](images/Screenshot (31).png)


---

### Conclusion
This project demonstrates successful BLE-based Wi-Fi provisioning using the ESP32. It allows for easy wireless configuration of the ESP32 without hardcoding Wi-Fi credentials, making it flexible and user-friendly.

---

### References
- [ESP-IDF BLE Provisioning Example](https://github.com/espressif/esp-idf/tree/master/examples/provisioning)
- [ESP32 Wi-Fi Provisioning Tutorial by Rui Santos & Sara Santos](https://RandomNerdTutorials.com/esp32-wi-fi-provisioning-ble-arduino/)
- [Additional Documentation](https://github.com/espressif/arduino-esp32/tree/master/libraries/WiFiProv/examples/WiFiProv)

---


