# 2 ESP32 Scan Wifi

```c title="esp32-workshop-2.ino"  linenums="1"
#include <WiFi.h>

void setup() {
  // เริ่มต้น Serial Monitor
  Serial.begin(115200);
  while (!Serial) {
    // รอจนกว่า Serial จะพร้อม
  }

  // เชื่อมต่อกับ Wi-Fi (สามารถเว้นไว้ถ้าไม่ต้องการเชื่อมต่อ)
  WiFi.begin("your-SSID", "your-PASSWORD"); // ใช้ชื่อ SSID และรหัสผ่านของเครือข่าย Wi-Fi

  // รอจนกว่าเชื่อมต่อได้
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }

  // แสดง IP ที่ได้รับเมื่อเชื่อมต่อ
  Serial.println("Connected to WiFi");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  // เริ่มการสแกน Wi-Fi
  Serial.println("Scanning for Wi-Fi networks...");

  // พิมพ์หัวตาราง
  Serial.println("-----------------------------------------------------------");
  Serial.println("| No | SSID                | RSSI    | MAC Address       |");
  Serial.println("-----------------------------------------------------------");

  int networksFound = WiFi.scanNetworks();
  if (networksFound == 0) {
    Serial.println("| No networks found                                      |");
  } else {
    for (int i = 0; i < networksFound; i++) {
      String ssid = WiFi.SSID(i);  // ชื่อของเครือข่าย Wi-Fi
      int rssi = WiFi.RSSI(i);     // สัญญาณ RSSI (Strength)
      String macAddress = WiFi.BSSIDstr(i);  // MAC Address ของเครือข่าย

      // พิมพ์ข้อมูลแต่ละเครือข่ายในตาราง
      Serial.print("| ");
      Serial.print(i + 1);                    // หมายเลขเครือข่าย
      Serial.print("  | ");
      Serial.print(ssid);                     // ชื่อของเครือข่าย
      Serial.print("  | ");
      Serial.print(rssi);                     // RSSI (Signal Strength)
      Serial.print("    | ");
      Serial.print(macAddress);               // MAC Address
      Serial.println(" |");
    }
  }

  // จบการสแกน Wi-Fi
  Serial.println("-----------------------------------------------------------");
  Serial.println("Scan complete.");

}

void loop() {
  // ไม่ต้องทำอะไรใน loop
}


```
