# Report_ใบงานการทดลองที่ 13.2
### การพัฒนา Hardware Serial Bridge ดักฟังข้อมูล USB-UART จาก ESP32 แปลงเข้าสู่ MQTT Topics และ REST API

---

### กิจกรรมการทดสอบสะพานเชื่อมต่อ

> [!WARNING] **ข้อพึงระวังเรื่องพอร์ตสื่อสารอนุกรม (Serial Port Exclusive Lock):**
> 1. ตรวจสอบหมายเลข COM Port ของบอร์ด ESP32 ผ่าน Device Manager แล้วแก้ไขตัวแปร `_portName` ในไฟล์ `Services.cs` ให้ตรงกับเครื่องของคุณ (เช่น `COM24`)
> 2. **ต้องปิด `idf monitor`, PuTTY หรือ Serial Monitor อื่นๆ ก่อนรัน Kestrel เสมอ** เพราะบน Windows พอร์ต COM จะถูกล็อกใช้งานแบบ Exclusive Access (เปิดได้เพียง 1 โปรแกรมในเวลาเดียวกัน) หากมีโปรแกรมอื่นเปิดพอร์ตค้างไว้ C# จะเปิดพอร์ตไม่สำเร็จและส่งคำสั่งไม่ได้ (`sentToSerial: false`)

1. เสียบบอร์ด ESP32 เข้ากับเครื่อง และสั่งรัน Kestrel:
```powershell
dotnet run
```
**ผลลัพธ์จากการทดลอง**

<img width="965" height="386" alt="image" src="https://github.com/user-attachments/assets/1abbded7-3cb2-4155-9617-28fafb66073f" />

---
2. ทดสอบเรียกดูค่าเซนเซอร์ผ่านเบราว์เซอร์หรือ cURL:
```powershell
curl http://localhost:5000/api/telemetry/latest
```
**ผลลัพธ์จากการทดลอง**

<img width="968" height="571" alt="image" src="https://github.com/user-attachments/assets/34424219-4b28-4936-a4df-63c042be628f" />

---
3. เปิด `mosquitto_sub`:
```powershell
mosquitto_sub -h 127.0.0.1 -p 1883 -t "iot/esp32/telemetry" -i "sub_client" -v
```
*สังเกตว่าข้อมูลที่อ่านจากสาย USB Serial ถูกส่งต่อเข้าหัวข้อ MQTT อัตโนมัติ!*

**ผลลัพธ์จากการทดลอง**
```
PS D:\1-2569\IOT\Week-13-Kestrel-Edge-Gateway-and-Protocol-Integration> mosquitto_sub -h 127.0.0.1 -p 1883 -t "iot/esp32/telemetry" -i "sub_client" -v
iot/esp32/telemetry {"voltage":0.48,"raw":599}
iot/esp32/telemetry {"voltage":0.34,"raw":417}
iot/esp32/telemetry {"voltage":0.10,"raw":119}
iot/esp32/telemetry {"voltage":0.05,"raw":64}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
iot/esp32/telemetry {"voltage":0.09,"raw":113}
iot/esp32/telemetry {"voltage":0.56,"raw":701}
iot/esp32/telemetry {"voltage":1.81,"raw":2243}
iot/esp32/telemetry {"voltage":2.02,"raw":2512}
iot/esp32/telemetry {"voltage":2.56,"raw":3171}
iot/esp32/telemetry {"voltage":2.66,"raw":3298}
iot/esp32/telemetry {"voltage":2.89,"raw":3582}
iot/esp32/telemetry {"voltage":2.79,"raw":3467}
iot/esp32/telemetry {"voltage":1.88,"raw":2327}
iot/esp32/telemetry {"voltage":1.40,"raw":1742}
iot/esp32/telemetry {"voltage":0.35,"raw":435}
iot/esp32/telemetry {"voltage":0.21,"raw":259}
iot/esp32/telemetry {"voltage":0.07,"raw":86}
iot/esp32/telemetry {"voltage":0.03,"raw":33}
iot/esp32/telemetry {"voltage":0.00,"raw":0}
```
---
4. ทดสอบสั่งเปิด-ปิดไฟผ่าน REST API:
```powershell
curl -X POST http://localhost:5000/api/led/on
```
*สังเกตว่าคำสั่ง `LED:ON` ถูกส่งผ่านสาย USB เข้าไปควบคุมบอร์ดทันที!*

**ผลลัพธ์จากการทดลอง**
<img width="1477" height="1108" alt="S__6029315_0" src="https://github.com/user-attachments/assets/32ab6755-45f8-46c7-a15c-278f93fe2f76" />

<img width="1477" height="1108" alt="S__6029316_0" src="https://github.com/user-attachments/assets/a8fe282d-d3d9-4d7e-aca9-068cb7a8d7cd" />

---

## 4. คำถามท้ายการทดลอง
1. สถาปัตยกรรมแบบ Serial-to-MQTT Bridge นี้ช่วยให้อุปกรณ์รุ่นเก่า (Legacy Hardware) ที่ไม่มี Wi-Fi สามารถเข้าร่วมระบบนิเวศ IoT สมัยใหม่ได้อย่างไร?

   **ตอบ** อุปกรณ์รุ่นเก่า มักสื่อสารผ่านพอร์ต Serial ธรรมดา สถาปัตยกรรม Bridge จะช่วยรับข้อมูลดิบจากสาย Serial มาแปลงรูปแบบ (เช่น JSON) และเชื่อมต่อเข้ากับเครือข่าย MQTT หรือ REST API ทำให้ระบบคลาวด์และแอปพลิเคชันสมัยใหม่สามารถดึงข้อมูลเหล่านั้นไปใช้งานได้เสมือนเป็นอุปกรณ์ IoT ปกติ
   
2. เพราะเหตุใดเราจึงต้องใช้ `BackgroundService` ในการอ่านข้อมูล Serial แทนที่จะเปิดพอร์ตอ่านตรงๆ ภายในฟังก์ชันของ REST API?

   **ตอบ**
   - พอร์ต Serial บนระบบปฏิบัติการจะถูกล็อกสิทธิ์การเข้าถึงแบบจำเพาะ (Exclusive Lock) หากมี API Request เข้ามาพร้อมกันหลายตัวจะทำให้เกิด Error พอร์ตชนกัน
   - ข้อมูล Serial เป็นการสตรีมมิ่งต่อเนื่อง การให้ API ไปรออ่าน (Blocking) จะทำให้ฝั่ง Client เกิด Timeout การใช้ BackgroundService เพื่อเปิดพอร์ตค้างไว้เบื้องหลังเธรดเดียว และอัปเดตค่าลงหน่วยความจำ (Cache) จะทำให้ REST API สามารถดึงค่าล่าสุดไปตอบกลับได้อย่างรวดเร็วและปลอดภัย
