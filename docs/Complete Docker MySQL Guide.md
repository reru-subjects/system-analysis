# **คู่มือการติดตั้งและใช้งาน MySQL \+ phpMyAdmin บน Docker (Windows 10\) ฉบับสมบูรณ์**

เอกสารนี้รวบรวมขั้นตอนการติดตั้ง การแก้ปัญหาที่พบบ่อย (Restart Loop) และการใช้งานฟีเจอร์ ER Diagram รวมถึงความเข้าใจเรื่อง Database Constraints ไว้ในที่เดียว

## **1\. การติดตั้ง (Installation)**

วิธีที่ง่ายและจัดการได้ดีที่สุดคือการใช้ **Docker Compose** เพื่อรัน MySQL และ phpMyAdmin พร้อมกัน

### **ขั้นตอนการเตรียมไฟล์**

1. สร้างโฟลเดอร์ใหม่ในเครื่องคอมพิวเตอร์ (เช่น mysql-server)  
2. สร้างไฟล์ชื่อ docker-compose.yml ในโฟลเดอร์นั้น  
3. คัดลอกโค้ดด้านล่างไปใส่:

```yml

version: '3.8'

services:  
  # ส่วนของการตั้งค่า MySQL  
  db:  
    image: mysql:latest
    container_name: mysql_container
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD:
      MYSQL_ALLOW_EMPTY_PASSWORD: 'true'
      MYSQL_DATABASE: test
    ports:
      - "3307:3306" # เปิดพอร์ตให้เครื่อง Windows เข้าถึง MySQL ได้โดยตรง
    volumes:
      - ./db_data:/var/lib/mysql

  # ส่วนของการตั้งค่า phpMyAdmin
  phpmyadmin:
    image: phpmyadmin:latest  
    container_name: pma_container  
    restart: always  
    environment:  
      PMA_HOST: db            # บอกให้รู้ว่า MySQL ชื่อ service ว่า 'db' (ตรงกับด้านบน)
      PMA_PORT: 3306
      PMA_USER: root
      PMA_PASSWORD: 
      UPLOAD_LIMIT: 200M       # เพิ่มขนาดไฟล์ upload (เผื่อ import database ใหญ่ๆ) 
    ports:  
      - "8001:80"             # เข้าใช้งานผ่าน browser ที่ port 8001  
    depends_on:  
      - db                    # รอให้ MySQL รันก่อนค่อยรันตัวนี้  
```

### **คำสั่งใช้งาน (Command Line)**

เปิด Command Prompt หรือ PowerShell ในโฟลเดอร์นั้น แล้วใช้คำสั่ง:

* **เริ่มทำงาน:** docker-compose up \-d  
* **หยุดทำงาน:** docker-compose down  
* **ดูสถานะ:** docker-compose ps  
* **ดู Log:** docker logs mysql\_container

**การเข้าใช้งาน:** เปิด Browser ไปที่ http://localhost:8001

## **2\. การแก้ปัญหา MySQL ติดๆ ดับๆ (Restart Loop Troubleshooting)**

หากสั่ง up แล้ว MySQL สถานะเป็น Restarting ตลอดเวลา สาเหตุมักเกิดจาก:

### **สาเหตุที่ 1: Port 3306 ชนกัน (พบบ่อยที่สุดบน Windows)**

* **อาการ:** Log แจ้งว่า Bind on TCP/IP port: Address already in use.  
* **สาเหตุ:** มี MySQL ตัวอื่น (เช่น XAMPP, MySQL Installer) รันอยู่ในเครื่อง Windows  
* **วิธีแก้:**  
  1. ไปที่ Services (services.msc) หา MySQL แล้วกด Stop  
  2. หรือ แก้ไฟล์ docker-compose.yml เปลี่ยน Port เป็น 3307:3306

### **สาเหตุที่ 2: ข้อมูลใน Volume เสียหาย (Corrupted Data)**

* **อาการ:** Log แจ้ง Error เกี่ยวกับ InnoDB หรือ Permission denied  
* **สาเหตุ:** การปิด Docker ไม่สมบูรณ์ หรือย้ายเครื่อง  
* **วิธีแก้:**  
  1. สั่ง docker-compose down  
  2. **ลบโฟลเดอร์ db\_data** ที่อยู่ในโฟลเดอร์งานทิ้ง  
  3. สั่ง docker-compose up \-d ใหม่ (ข้อมูลเก่าจะหาย และเริ่มสร้างใหม่)

### **สาเหตุที่ 3: แรมไม่พอ (OOM Kill)**

* **อาการ:** Container ดับเองโดยไม่มี Error ชัดเจน (Exited 137\)  
* **วิธีแก้:** ไปที่ Docker Desktop \> Settings \> Resources แล้วเพิ่ม Memory ให้เป็นอย่างน้อย 2GB หรือ 4GB

## **3\. การสร้าง ER-Diagram ใน phpMyAdmin**

### **ขั้นตอนการเข้าใช้งาน**

1. เลือก **Database** ที่ต้องการทางเมนูซ้ายมือ (คลิกที่ชื่อฐานข้อมูลเลย)  
2. มองหาเมนูแถบบนชื่อ **"Designer"** (ถ้าไม่เจอให้กด More \> Designer)

### **การใช้งานเครื่องมือ**

* **จัดเรียง:** ลากวางตารางได้ตามใจชอบ  
* **บันทึก:** กดปุ่ม "Save page" ทางเมนูซ้ายเพื่อจำตำแหน่ง  
* **สร้างความสัมพันธ์ (เส้นโยง):**  
  1. คลิกปุ่ม **"Create relation"** ทางซ้าย  
  2. คลิกเลือก **Primary Key** (คีย์หลัก) ของตารางแม่  
  3. คลิกเลือก **Foreign Key** (คีย์นอก) ของตารางลูก  
  4. กด OK (เลือก ON DELETE/UPDATE ตามต้องการ)

## **4\. ความเข้าใจเรื่อง ON DELETE และ ON UPDATE**

การตั้งค่านี้คือ "กฎ" ที่ใช้จัดการข้อมูลลูก (Child) เมื่อข้อมูลแม่ (Parent) ถูกเปลี่ยนแปลง

### **ตัวอย่าง: แม่ (ลูกค้า) \-\> ลูก (ใบสั่งซื้อ)**

| **Action** | **ความหมาย** | **เหมาะกับสถานการณ์ไหน?** |

| **RESTRICT** (Default) | **"ห้ามทำ"** ถ้าจะลบลูกค้าที่มีใบสั่งซื้อ ระบบจะ Error ไม่ยอมให้ลบ | **ข้อมูลสำคัญ:** เช่น ใบเสร็จ, ข้อมูลการเงิน ที่ห้ามหายเด็ดขาด |

| **CASCADE** | **"ทำตามกัน"** ลบลูกค้า \-\> ใบสั่งซื้อหายหมด แก้รหัสลูกค้า \-\> รหัสในใบสั่งซื้อเปลี่ยนตาม | **ข้อมูลส่วนประกอบ:** เช่น ลบโพสต์แล้วคอมเมนต์หาย, หรือใช้กับการแก้ไข ID (Update) |

| **SET NULL** | **"ปล่อยเกาะ"** ลบลูกค้า \-\> ใบสั่งซื้อยังอยู่ แต่ช่องรหัสลูกค้ากลายเป็นว่าง (NULL) | **ข้อมูลประวัติ:** ต้องการเก็บประวัติไว้ดู แม้เจ้าของข้อมูลจะไม่มีตัวตนแล้ว |

| **NO ACTION** | ทำงานคล้าย RESTRICT ใน MySQL มาตรฐาน | ไม่ค่อยนิยมใช้ (ใช้ RESTRICT ชัดเจนกว่า) |

### **สูตรแนะนำ (Best Practice)**

สำหรับการเชื่อมโยงตารางทั่วไป แนะนำให้ตั้งค่าดังนี้:

* **ON DELETE:** เลือก **RESTRICT** (เพื่อป้องกันอุบัติเหตุข้อมูลหาย)  
* **ON UPDATE:** เลือก **CASCADE** (เพื่อให้แก้ ID ได้ แล้วข้อมูลไหลตามกันไปไม่พัง)
