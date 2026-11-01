# Report_ใบงานการทดลองที่ 13.1
### การสร้าง Embedded MQTT Broker และ Client ภายใน Kestrel .NET 8 ด้วยไลบรารี MQTTnet

---


#### สังเกตผลลัพธ์
* หน้าต่าง Kestrel จะพิมพ์ Log สีฟ้าว่าดักจับข้อความได้ถูกต้อง
* หน้าต่าง `mosquitto_sub` จะได้รับข้อความที่ถูกส่งต่อผ่าน Broker สำเร็จทันที

<img width="1316" height="410" alt="image" src="https://github.com/user-attachments/assets/f61e7d7f-2683-4004-bcc5-f68850cd55e8" />


<img width="1311" height="290" alt="image" src="https://github.com/user-attachments/assets/26b82729-9c8b-4709-bff7-84e89111f0c2" />


<img width="1317" height="342" alt="image" src="https://github.com/user-attachments/assets/ec4cb931-76c6-44e4-bf37-6b0da7d52e77" />


---

## 4. คำถามท้ายการทดลอง (Review Questions)
1. การรัน Embedded MQTT Broker ภายใน Kestrel มีข้อได้เปรียบกว่าการติดตั้งซอฟต์แวร์ Mosquitto แยกต่างหากอย่างไรในแง่ของการกระจายงาน (Deployment) และการประมวลผลข้อมูลในหน่วยความจำ?

   **ตอบ**
   - **ด้าน Deployment** รวบทั้ง Web API และ MQTT ไว้ในแอปเดียว (Single Executable) ทำให้ติดตั้งและดูแลรักษาง่าย ไม่ต้องลงซอฟต์แวร์ Broker เพิ่มเติม
   - **ด้านการประมวลผล** ส่งผ่านข้อมูลระหว่างฝั่ง MQTT และ Web/Database ในหน่วยความจำ (In-Memory) ได้ทันที ช่วยลด Network Overhead ที่เกิดจากการสื่อสารข้ามโปรแกรม
      
2. ฟังก์ชัน `InterceptingPublishAsync` ใน MQTTnet สามารถนำไปประยุกต์ใช้ในการกรองข้อมูลที่ผิดปกติ (Edge Validation) ก่อนส่งต่อให้ Client อื่นได้อย่างไร?

   **ตอบ** ฟังก์ชันนี้ใช้ดักจับข้อความทั้งหมดก่อนถูกกระจาย (Route) ไปยังผู้รับ เราจึงแทรกโค้ด C# ตรวจสอบค่าเซนเซอร์หรือแก้ไขโครงสร้างข้อมูลได้ หากพบข้อมูลผิดปกติก็สามารถสั่งทิ้ง (Drop) ข้อความนั้นได้ทันที ป้องกันข้อมูลขยะส่งไปถึงปลายทาง
