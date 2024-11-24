# 6 ESP32 Object Detection

- สร้าง Account ใน edge impulse

![](../assets/images/edge_impulse_account_1.png)

ติดตั้ง Library EloquentEsp32cam

![](../assets/images/EloquentEsp32cam_install.png)

- ตรวจสอบ Example

![](../assets/images/EloquentEsp32cam_install2.png)

- เลือก ตัวอย่าง **Collect_Images_for_EdgeImpulse.ino**

```c title="Collect_Images_for_EdgeImpulse.ino" linenums="1"
/**
 * Collect images for Edge Impulse image
 * classification / object detection
 *
 * BE SURE TO SET "TOOLS > CORE DEBUG LEVEL = INFO"
 * to turn on debug messages
 */

// if you define WIFI_SSID and WIFI_PASS before importing the library,
// you can call connect() instead of connect(ssid, pass)
//
// If you set HOSTNAME and your router supports mDNS, you can access
// the camera at http://{HOSTNAME}.local

#define WIFI_SSID "TrueGigatexFiber_uS7_2.4G"
#define WIFI_PASS "itbakery@9"
#define HOSTNAME "esp32cam"


#include <eloquent_esp32cam.h>
#include <eloquent_esp32cam/extra/esp32/wifi/sta.h>
#include <eloquent_esp32cam/viz/image_collection.h>

using eloq::camera;
using eloq::wifi;
using eloq::viz::collectionServer;


void setup() {
    delay(3000);
    Serial.begin(115200);
    Serial.println("___IMAGE COLLECTION SERVER___");

    // camera settings
    // replace with your own model!
    // camera.pinout.wroom_s3();
    camera.pinout.aithinker();
    camera.brownout.disable();
    // Edge Impulse models work on square images
    // face resolution is 240x240
    camera.resolution.face();
    camera.quality.high();

    // init camera
    while (!camera.begin().isOk())
        Serial.println(camera.exception.toString());

    // connect to WiFi
    while (!wifi.connect().isOk())
      Serial.println(wifi.exception.toString());

    // init face detection http server
    while (!collectionServer.begin().isOk())
        Serial.println(collectionServer.exception.toString());

    Serial.println("Camera OK");
    Serial.println("WiFi OK");
    Serial.println("Image Collection Server OK");
    Serial.println(collectionServer.address());
}


void loop() {
    // server runs in a separate thread, no need to do anything here
}
```

- compile มี error ให้ทำการแก้ไขไฟล์ใน Library ที่ได้ทำการ Download มา

![](../assets/images/Error_edit_library.png)

![](../assets/images/Error_edit_library1.png)

แล้ว upload อีกครั้ง

![](../assets/images/fix_error.png)

![](../assets/images/collect_imaes_for_edge.png)

- เปิด browser ip ที่แสดง http://192.168.1.184

![](../assets/images/collect_image_web.png)

- กด Start collection เพื่อทำการ บันทึก รูปเพื่อไปใช้สำหรับการ train

![](../assets/images/save_image1.png)

- กด Download รูปภาพ
  ![](../assets/images/save_image_2.png)

- Label รูปภาพ
  ![](../assets/images/save_image_3.png)

- ตัวอย่าง ข้อมูลที่ได้ แต่ละภาพจะมี Label ทุก ไฟล์
  ![](../assets/images/save_image_5.png)

- ตัวอย่าง เก็บภาพกล่อง box
  ![](../assets/images/save_image_6.png)

- ตัวอย่าง เก็บภาพกล่อง Mouse
  ![](../assets/images/save_image_7.png)

![](../assets/images/image_label.png)

## Upload รูปภาพ ไปยัง Edge impulse

- สร้าง project ใน Edge impulse
  ![](../assets/images/edge_impulse_1.png)

- กด สร้าง project
  ![](../assets/images/edge_impulse_2.png)

- ติดตั้งชื่อ project `muict-esp32-cam-object-detect`

![](../assets/images/edge_impulse_3.png)

- เพิ่ม Image เข้าสู่ โปรเจค

![](../assets/images/edge_impulse_4.png)

- กด upload Data

![](../assets/images/edge_impulse_5.png)

- เลือกวิธีการ upload ด้วยการ upload ทั้ง folder

![](../assets/images/edge_impulse_6.png)

- กด upload data

![](../assets/images/edge_impulse_7.png)

![](../assets/images/edge_impulse_8.png)

![](../assets/images/edge_impulse_9.png)

- upload data set ที่เหลือ ทั้งหมด

## Labeling Queue

- หลังจาก upload เรียบร้อย ขั้นตอนต่อไป คือการทำ labeling

![](../assets/images/edge_impulse_10.png)

- ใช้ mouse ลาก คร่อมรูปภาพ และกด save label

![](../assets/images/edge_impulse_11.png)

![](../assets/images/edge_impulse_12.png)

- label ทุกภาพที่อยุ่ที่ Queue จน Queue เป็น 0

## Impulse design > Create impulse

- เลือก Board ปลายทางในการทดสอบ

![](../assets/images/edge_impulse_14.png)

![](../assets/images/edge_impulse_15.png)

- Add process box
  ![](../assets/images/edge_impulse_16.png)

- Add image block
  ![](../assets/images/edge_impulse_17.png)

- Add learning block

![](../assets/images/edge_impulse_18.png)

![](../assets/images/edge_impulse_19.png)

- Save impulse

![](../assets/images/edge_impulse_20.png)

## เปลี่ยนเป็น ภาพ Gray โดยไปที่ เมนู Image

![](../assets/images/edge_impulse_21.png)

- Generate feature

![](../assets/images/edge_impulse_25.png)

## ไปยัง เมนู Object Detection

- กด start training

![](../assets/images/edge_impulse_23.png)

![](../assets/images/edge_impulse_26.png)

## หลังจาทก Training ก็จะทำการ Deploy เลือก Arduino library

![](../assets/images/edge_impulse_27.png)

## Change Target Espressif ESP-EYE

![](../assets/images/edge_impulse_28.png)

![](../assets/images/edge_impulse_29.png)

## Build มาเป็น Zip file

![](../assets/images/edge_impulse_30.png)

## add zip file to Arduino

![](../assets/images/edge_impulse_32.png)

## เลือกไฟล์ ที่ได้ จาก edge impluse

![](../assets/images/edge_impulse_33.png)

## up load สำเร็จ

![](../assets/images/edge_impulse_34.png)

## open file esp32_camera

![](../assets/images/edge_impulse_34.png)

- แก้ ไฟล์

![](../assets/images/edge_impulse_36.png)

- save file esp32_camera_impulse.ino

[Download Code EdgeImpulse.zip](https://drive.google.com/file/d/1EVW4sC0k2A-5q35HiXBrqIAyz6mGf8YV/view?usp=sharing)

![](../assets/images/edge_impulse_37.png)
