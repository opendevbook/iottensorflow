# 4 ESP 32 SP mode

โหมด Station (STA) หรือ SP Mode ใน ESP32 คือโหมดที่ใช้เชื่อมต่อกับเครือข่าย Wi-Fi ที่มีอยู่แล้ว เช่น Wi-Fi Router เพื่อให้ ESP32 สามารถใช้งานอินเทอร์เน็ตหรือเชื่อมต่อกับเซิร์ฟเวอร์ต่าง ๆ ผ่าน Wi-Fi. ในโหมดนี้ ESP32 จะทำหน้าที่เป็น ลูกข่าย (Client) เชื่อมต่อกับ Access Point (AP) ที่มีอยู่.

```c title="esp32_workshop_4.ino"  linenums="1"
#include <WiFi.h>

const char *ssid = "your-SSID";          // ชื่อ SSID ของเครือข่าย Wi-Fi ที่ต้องการเชื่อมต่อ
const char *password = "your-PASSWORD";  // รหัสผ่านของเครือข่าย Wi-Fi

void setup() {
  // เริ่มต้น Serial Monitor
  Serial.begin(115200);
  while (!Serial) {
    // รอจนกว่า Serial จะพร้อม
  }

  // เชื่อมต่อกับ Wi-Fi ในโหมด Station
  WiFi.begin(ssid, password);  // เชื่อมต่อกับ SSID และรหัสผ่าน

  // รอจนกว่า ESP32 จะเชื่อมต่อกับ Wi-Fi
  Serial.println("Connecting to WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);  // รอ 1 วินาที
    Serial.print(".");  // แสดงจุดในระหว่างรอ
  }

  // เมื่อเชื่อมต่อสำเร็จ
  Serial.println("");
  Serial.println("Connected to WiFi");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());  // แสดง IP Address ที่ได้รับจาก Wi-Fi Router
}

void loop() {
  // สามารถใช้การเชื่อมต่อ Wi-Fi ในโค้ดนี้ในการทำงานอื่น ๆ เช่น HTTP Requests, MQTT, etc.
}

```

## Refactor Code

```c title="esp32_workshop_4_1.ino"  linenums="1"
#include <WiFi.h>

const char *ssid = "your-SSID";          // ชื่อ SSID ของเครือข่าย Wi-Fi ที่ต้องการเชื่อมต่อ
const char *password = "your-PASSWORD";  // รหัสผ่านของเครือข่าย Wi-Fi

// ฟังก์ชัน initWiFi สำหรับเชื่อมต่อกับ Wi-Fi
void initWiFi() {
  // เริ่มต้นการเชื่อมต่อ Wi-Fi
  WiFi.begin(ssid, password);

  // รอจนกว่าเชื่อมต่อสำเร็จ
  Serial.println("Connecting to WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);  // รอ 1 วินาที
    Serial.print(".");  // แสดงจุดในระหว่างรอ
  }

  // เมื่อเชื่อมต่อสำเร็จ
  Serial.println("");
  Serial.println("Connected to WiFi");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());  // แสดง IP Address ที่ได้รับจาก Wi-Fi Router
}

void setup() {
  // เริ่มต้น Serial Monitor
  Serial.begin(115200);
  while (!Serial) {
    // รอจนกว่า Serial จะพร้อม
  }

  // เรียกใช้ฟังก์ชัน initWiFi สำหรับเชื่อมต่อ Wi-Fi
  initWiFi();
}

void loop() {
  // ฟังก์ชัน loop สามารถใช้ในการทำงานต่อไป
}
```

การใช้ Wi-Fi event callback ใน ESP32 สามารถช่วยให้โปรแกรมของคุณจัดการสถานะการเชื่อมต่อ Wi-Fi โดยอัตโนมัติ เช่น การเชื่อมต่อใหม่เมื่อการเชื่อมต่อถูกตัดขาดไป. คุณสามารถใช้ WiFi.onEvent() เพื่อกำหนด callback function ที่จะถูกเรียกเมื่อเกิดเหตุการณ์ที่เกี่ยวข้องกับ Wi-Fi, เช่น การเชื่อมต่อสำเร็จหรือการตัดการเชื่อมต่อ.

ในตัวอย่างนี้, เราจะใช้ Wi-Fi event callback เพื่อให้ ESP32 เชื่อมต่อ Wi-Fi ใหม่โดยอัตโนมัติเมื่อการเชื่อมต่อถูกตัดขาด.

โค้ดตัวอย่าง: ใช้ WiFi.onEvent() สำหรับเชื่อมต่อ Wi-Fi ใหม่เมื่อเกิดการตัดการเชื่อมต่อ

```c title="esp32_workshop_4_2.ino"  linenums="1"

#include <WiFi.h>

const char *ssid = "TrueGigatexFiber_uS7_2.4G";          // ชื่อ SSID ของเครือข่าย Wi-Fi ที่ต้องการเชื่อมต่อ
const char *password = "itbakery@9";  // รหัสผ่านของเครือข่าย Wi-Fi

// ตั้งค่า DNS
IPAddress primaryDNS(8, 8, 8, 8);  // Google DNS
IPAddress secondaryDNS(8, 8, 4, 4); // Google Secondary DNS (ถ้าต้องการ)

// ฟังก์ชันที่ใช้ในการเชื่อมต่อ Wi-Fi
void initWiFi() {
  // กำหนดค่า IP, Gateway, Subnet Mask, DNS
  WiFi.config(INADDR_NONE, INADDR_NONE, INADDR_NONE, primaryDNS, secondaryDNS);
  WiFi.begin(ssid, password);

  // รอจนกว่าเชื่อมต่อสำเร็จ
  Serial.println("Connecting to WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);  // รอ 1 วินาที
    Serial.print(".");  // แสดงจุดในระหว่างรอ
  }

  // เมื่อเชื่อมต่อสำเร็จ
  Serial.println("");
  Serial.println("Connected to WiFi");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());  // แสดง IP Address ที่ได้รับจาก Wi-Fi Router
}

// ฟังก์ชันที่ใช้จัดการกับ Wi-Fi events
// ฟังก์ชัน callback สำหรับ Wi-Fi events
void WiFiEvent(WiFiEvent_t event) {
  switch (event) {
    case IP_EVENT_STA_GOT_IP:
      Serial.println("Got IP address");
      Serial.print("IP Address: ");
      Serial.println(WiFi.localIP());
      break;
    case WIFI_EVENT_STA_DISCONNECTED:
      Serial.println("Disconnected from Wi-Fi");
      initWiFi();
      break;
    default:
      break;
  }
}

void setup() {
  // เริ่มต้น Serial Monitor
  Serial.begin(115200);
  while (!Serial) {
    // รอจนกว่า Serial จะพร้อม
  }

  // เรียกใช้ Wi-Fi event callback
  WiFi.onEvent(WiFiEvent);

  // เริ่มต้นการเชื่อมต่อ Wi-Fi
  initWiFi();
}

void loop() {
  // สามารถทำงานอื่น ๆ ต่อไปได้ใน loop
}


```

**ความหมายของลูป while:**
ลูป while จะทำงานต่อไปตราบใดที่เงื่อนไขที่กำหนดในลูปเป็น จริง (True). เมื่อเงื่อนไขเป็น เท็จ (False), ลูปจะหยุดทำงาน.

### ตารางความจริงของ `&&` (AND)

| เงื่อนไข 1 (A) | เงื่อนไข 2 (B) | A && B |
| -------------- | -------------- | ------ |
| False          | False          | False  |
| False          | True           | False  |
| True           | False          | False  |
| True           | True           | True   |

```c title="esp32_workshop_4_3.ino"  linenums="1"

#include <WiFi.h>

const char *ssid = "TrueGigatexFiber_uS7_2.4G";          // ชื่อ SSID ของเครือข่าย Wi-Fi ที่ต้องการเชื่อมต่อ
const char *password = "itbakery@9";  // รหัสผ่านของเครือข่าย Wi-Fi

// ตั้งค่า DNS
IPAddress primaryDNS(8, 8, 8, 8);  // Google DNS
IPAddress secondaryDNS(8, 8, 4, 4); // Google Secondary DNS (ถ้าต้องการ)

// ฟังก์ชันที่ใช้ในการเชื่อมต่อ Wi-Fi
void initWiFi() {
  // กำหนดค่า IP, Gateway, Subnet Mask, DNS
  WiFi.config(INADDR_NONE, INADDR_NONE, INADDR_NONE, primaryDNS, secondaryDNS);
  WiFi.begin(ssid, password);

  int try_num = 0;  // ตัวแปรเพื่อเก็บจำนวนครั้งที่พยายามเชื่อมต่อ
  int max_tries = 10;  // กำหนดจำนวนครั้งสูงสุดที่พยายามเชื่อมต่อ

  Serial.println("Connecting to WiFi...");

  // รอจนกว่า Wi-Fi จะเชื่อมต่อ หรือเกินจำนวนครั้งที่กำหนด
  while (WiFi.status() != WL_CONNECTED && try_num < max_tries) {
    delay(1000);  // รอ 1 วินาที
    Serial.print(".");  // แสดงจุดในระหว่างรอ
    try_num++;  // เพิ่มจำนวนครั้ง
  }

  // หากเชื่อมต่อสำเร็จ
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("");
    Serial.println("Connected to WiFi");
    Serial.print("IP Address: ");
    Serial.println(WiFi.localIP());  // แสดง IP Address ที่ได้รับจาก Wi-Fi Router
  } else {
    // ถ้าไม่สามารถเชื่อมต่อได้หลังจากพยายาม 10 ครั้ง
    Serial.println("");
    Serial.println("Failed to connect to Wi-Fi after 10 attempts.");
  }
}

// ฟังก์ชันที่ใช้จัดการกับ Wi-Fi events
// ฟังก์ชัน callback สำหรับ Wi-Fi events
void WiFiEvent(WiFiEvent_t event) {
  switch (event) {
    case IP_EVENT_STA_GOT_IP:
      Serial.println("Got IP address");
      Serial.print("IP Address: ");
      Serial.println(WiFi.localIP());
      break;
    case WIFI_EVENT_STA_DISCONNECTED:
      Serial.println("Disconnected from Wi-Fi");
      initWiFi();
      break;
    default:
      break;
  }
}

void setup() {
  // เริ่มต้น Serial Monitor
  Serial.begin(115200);
  while (!Serial) {
    // รอจนกว่า Serial จะพร้อม
  }

  // เรียกใช้ Wi-Fi event callback
  WiFi.onEvent(WiFiEvent);

  // เริ่มต้นการเชื่อมต่อ Wi-Fi
  initWiFi();
}

void loop() {
  // สามารถทำงานอื่น ๆ ต่อไปได้ใน loop
}

```

## ผลลัพธ์ที่คาดว่าจะได้:

หากเชื่อมต่อสำเร็จ:

```
Connecting to WiFi...
..........
Connected to WiFi
IP Address: 192.168.1.100
```

หากไม่สามารถเชื่อมต่อได้หลังจาก 10 ครั้ง:

```
Connecting to WiFi...
..........
Failed to connect to Wi-Fi after 10 attempts.
```
