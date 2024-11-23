# 5 การเริ่มต้นและการนำโปรเจกต์ไปใช้งาน ESP32 และ Tensorflow.js

ขั้นตอนแรกในการใช้งาน ESP32-CAM ร่วมกับ TensorFlow.js คือการระบุวัตถุที่ประกอบเป็นหน้าเว็บที่ใช้แสดงผลสรุปของโปรเจกต์นี้ เพื่อใช้งานไลบรารี TensorFlow JavaScript เราจำเป็นต้องทำตามขั้นตอนดังนี้:

1. นำเข้า TensorFlow JavaScript libraries
2. โหลดโมเดล ในโปรเจกต์นี้จะใช้โมเดล COCO-SSD ที่ผ่านการฝึกสอนแล้ว
3. สร้างป้ายกำกับ สำหรับวัตถุที่ได้รับการประมวลผล ซึ่งจะถูกแสดงผลบนวิดีโออินพุต โดยการวาดสี่เหลี่ยมรอบวัตถุที่ตรวจพบผ่านโมเดล COCO-SSD

ด้วยขั้นตอนเหล่านี้ เราสามารถใช้ ESP32-CAM เพื่อแสดงผลการตรวจจับวัตถุในวิดีโอผ่าน TensorFlow.js ได้

ในส่วนนี้ของโค้ดแอ็กชัน เราจะนำเข้า TensorFlow.js เพื่อวิเคราะห์ภาพที่ได้รับในเบราว์เซอร์ ในบรรทัดถัดไป เราจะนำเข้าโมเดล COCO-SSD ร่วมกับ TensorFlow.js ด้วยการใช้คำสั่งดังนี้:

```c title="esp32-tensorflowjs.ino"
#include <WiFi.h>
#include <esp_camera.h>
#include <WiFiClient.h>
#include <WiFiAP.h>
#include <Arduino.h>

const char* ssid = "TrueGigatexFiber_uS7_2.4G";
const char* password = "itbakery@9";

// ESP32-CAM Pin Definitions (adjust as necessary for your specific model)
#define PWDN_GPIO_NUM    32
#define RESET_GPIO_NUM   -1
#define XCLK_GPIO_NUM    0
#define SIOD_GPIO_NUM    26
#define SIOC_GPIO_NUM    27
#define Y9_GPIO_NUM      35
#define Y8_GPIO_NUM      34
#define Y7_GPIO_NUM      39
#define Y6_GPIO_NUM      36
#define Y5_GPIO_NUM      21
#define Y4_GPIO_NUM      19
#define Y3_GPIO_NUM      18
#define Y2_GPIO_NUM      5
#define VSYNC_GPIO_NUM   25
#define HREF_GPIO_NUM    23
#define PCLK_GPIO_NUM    22

WiFiServer server(80);

void setup() {
  // Start the serial communication for debugging
  Serial.begin(115200);
  Serial.println();

  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("Connected to WiFi");
  Serial.println(WiFi.localIP());
  // Start the camera
  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;
  config.frame_size = FRAMESIZE_VGA;
  config.jpeg_quality = 12;
  config.fb_count = 2;

  // Initialize the camera
  if (esp_camera_init(&config) != ESP_OK) {
    Serial.println("Camera init failed");
    return;
  }

  // Start the server
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (client) {
    String request = client.readStringUntil('\r');
    client.flush();

    // Serve HTML page
    if (request.indexOf("GET / ") != -1) {
      sendHTML(client);
    }
    // Capture and send the image for object detection
    else if (request.indexOf("GET /capture") != -1) {
      captureImage(client);
    }
    client.stop();
  }
}

void sendHTML(WiFiClient& client) {
  String html = "<html><body>"
                "<h1>ESP32-CAM with TensorFlow.js</h1>"
                "<script src=\"https://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js\"></script>"
                "<script src=\"https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@4.0.0/dist/tf.min.js\"></script>"
                "<script src=\"https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd@3.0.0\"></script>"
                "<script>"
                "let img = new Image();"
                "img.src = '/capture';"
                "img.onload = function() {"
                "  document.body.appendChild(img);"
                "  cocoSsd.load().then(model => {"
                "    model.detect(img).then(predictions => {"
                "      predictions.forEach(prediction => {"
                "        let div = document.createElement('div');"
                "        div.innerHTML = prediction.class + ': ' + prediction.score.toFixed(2);"
                "        document.body.appendChild(div);"
                "      });"
                "    });"
                "  });"
                "};"
                "</script>"
                "</body></html>";

  client.print(html);
}

void captureImage(WiFiClient& client) {
  // Capture the image from the ESP32-CAM
  camera_fb_t *fb = esp_camera_fb_get();
  if (!fb) {
    Serial.println("Camera capture failed");
    client.println("HTTP/1.1 500 Internal Server Error");
    return;
  }

  // Send HTTP header
  client.println("HTTP/1.1 200 OK");
  client.println("Content-Type: image/jpeg");
  client.println("Connection: close");
  client.println();

  // Send image data
  client.write(fb->buf, fb->len);

  // Return the frame buffer to be used again
  esp_camera_fb_return(fb);
}

```

## คำอธิบายโค้ด:

1. การตั้งค่า Wi-Fi: ESP32-CAM เชื่อมต่อกับเครือข่าย Wi-Fi โดยใช้ข้อมูลประจำตัวที่ระบุใน ssid และ password
2. การเริ่มต้นกล้อง: กล้องจะถูกเริ่มต้นด้วยฟังก์ชัน esp_camera_init() โดยมีการกำหนดพินและการตั้งค่าที่เหมาะสม (เช่น FRAMESIZE_VGA)
3. การเสิร์ฟหน้า HTML: ESP32-CAM จะเสิร์ฟหน้าเว็บที่ประกอบไปด้วยสคริปต์ TensorFlow.js และโมเดล coco-ssd จาก CDN เมื่อหน้าเว็บถูกโหลดขึ้นมา จะมีการแสดงภาพที่จับจาก ESP32-CAM และรันโมเดล TensorFlow.js เพื่อทำการตรวจจับวัตถุในภาพ
4. การจับภาพ: path /capture จะจับภาพจาก ESP32-CAM และส่งภาพไปยังเบราว์เซอร์ในรูปแบบ JPEG จากนั้นภาพนี้จะถูกประมวลผลโดย TensorFlow.js สำหรับการตรวจจับวัตถุ

## HTML และ JavaScript (ฝังใน ESP32)

หน้า HTML ที่ฝังใน ESP32-CAM ประกอบไปด้วย:

1. TensorFlow.js: ทำการโหลด TensorFlow.js และโมเดล COCO-SSD จาก CDN
2. การแสดงภาพ: ทำการร้องขอภาพจาก ESP32-CAM path /capture และแสดงภาพบนหน้าเว็บ
3. การตรวจจับวัตถุ: เมื่อภาพถูกโหลดแล้ว สคริปต์จะรันโมเดลตรวจจับวัตถุบนภาพและแสดงผลลัพธ์ของวัตถุที่ตรวจพบ พร้อมกับคะแนนความเชื่อมั่น

**หมายเหตุ:**

- TensorFlow.js บน ESP32: TensorFlow.js จะทำงานในเบราว์เซอร์ ไม่ใช่ในตัว ESP32 เอง โดยที่ ESP32-CAM จะรับผิดชอบแค่การจับภาพและส่งภาพไปยังเบราว์เซอร์ การตรวจจับวัตถุจะเกิดขึ้นในเบราว์เซอร์โดยใช้โมเดล TensorFlow.js
- ข้อพิจารณาด้านประสิทธิภาพ: การตรวจจับวัตถุจะเกิดขึ้นในเบราว์เซอร์ ดังนั้น ESP32-CAM จะมีหน้าที่แค่จับภาพและสตรีมภาพไปยังเบราว์เซอร์ ควรตรวจสอบให้แน่ใจว่า ESP32-CAM เชื่อมต่อกับเครือข่ายที่มีแบนด์วิดธ์เพียงพอสำหรับการจับภาพแบบเรียลไทม์
