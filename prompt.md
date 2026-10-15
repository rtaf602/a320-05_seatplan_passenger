# PROMPT: Passenger Seat Plan A320(05) (ผังที่นั่งผู้โดยสารชั่วคราว)

> ใช้ไฟล์นี้เป็นคำสั่งตั้งต้นให้ AI สร้าง / ดูแลระบบนี้ต่อ (สถานะล่าสุด 15 ต.ค. 2569 · สร้างจากระบบ A340)
> Repo: `rtaf602/a320-05_seatplan_passenger` · หน้าเว็บ: https://rtaf602.github.io/a320-05_seatplan_passenger/
> ระบบพี่น้อง: `rtaf602/a340_seatplan_passenger` (ทุกอย่างเหมือนกัน ยกเว้นผังที่นั่ง) · เว็บช่าง A320(05): `rtaf602/a320-05_seatplan`

---

## 0. กฎที่ต้องทำตามเสมอ

1. ตอบผู้ใช้เป็น **ภาษาไทย** ใช้ภาษาง่ายสำหรับคนที่ไม่ใช่โปรแกรมเมอร์
2. Repo เป็น **public**: ห้ามใส่รหัสผ่าน, รหัส/ลิงก์ไฟล์ Google Sheet, ชื่อจริง, Username, Email, เลขบัตร, วันเกิด ลงในไฟล์ใดของ repo ค่าลับกรอกในส่วน `SETUP` ของหน้า Apps Script แล้วเก็บใน Script Properties เท่านั้น
3. ห้ามแก้ไฟล์ Google Sheet จริงของผู้ใช้ เว้นแต่ผู้ใช้สั่ง
4. ห้ามเก็บหรือบันทึกรหัสผ่าน LOGIN DATABASE แบบอ่านได้ (รหัสภารกิจเก็บแบบอ่านได้ในแท็บ `รายการภารกิจ` ตามที่ผู้ใช้กำหนด)
5. ทุกครั้งที่ส่งงาน ให้บอกให้ชัดว่า **ทดสอบอะไรแล้ว** (เช่น ตัวจำลองในเครื่อง) และ **อะไรยังไม่ได้ทดสอบกับ Google จริง**
6. เมื่อแก้ระบบหลังบ้าน ให้บอกผู้ใช้ทุกครั้งว่าต้องวางโค้ด `apps-script/Code.gs` ใหม่ และ Deploy เป็น **New version** (URL เดิม)
7. แก้หน้าเว็บแล้วให้เปลี่ยนเลข `?v=` ของไฟล์ CSS/JS ใน `index.html` เพื่อล้าง cache

---

## 1. ภาพรวม

เว็บกรอกผังที่นั่งผู้โดยสารเครื่อง A320(05) ของกองทัพอากาศ
- **1 Sector = 1 ภารกิจ = 1 แท็บ** ในไฟล์ Google Sheet **Seat Plan A320 TEMPORARY FOR PASSENGER**
- หน้าเว็บ: GitHub Pages (HTML/CSS/JS ล้วน ไม่มี build)
- หลังบ้าน: Google Apps Script ที่ผูกกับไฟล์ TEMPORARY FOR PASSENGER, Deploy เป็น Web app (ดำเนินการในฐานะ **ฉัน**, ผู้มีสิทธิ์เข้าถึง **ทุกคน**)
- หน้าเว็บส่ง `POST` แบบ `text/plain` (JSON `{action, ...}`) ไปที่ URL `/exec` ใน `config.js`
- ทุกการตรวจรหัสทำที่หลังบ้าน หน้าเว็บได้แค่ผ่าน / ไม่ผ่าน และ token ที่ลงลายมือชื่อ (HMAC-SHA256) เก็บในหน่วยความจำของหน้าเท่านั้น (รีเฟรช = ต้องใส่ใหม่)

### ไฟล์ Google Sheet ที่เกี่ยวข้อง

| ไฟล์ | ใช้ทำอะไร | สิทธิ์ |
|---|---|---|
| Seat Plan A320 TEMPORARY FOR PASSENGER | แท็บ `รายการภารกิจ` + แท็บละภารกิจ (มี Apps Script ของระบบนี้) | อ่าน/เขียน |
| Seat Plan 320(05) (For Admin) | สภาพที่นั่ง จอ Monitor และ NOTE (แท็บ `ผังที่นั่ง` ผังเริ่มแถว 22 สร้างโดย `layout.js` ของเว็บช่าง a320-05_seatplan) — มี Apps Script ของเว็บช่างแยกอยู่ ห้ามวางโค้ดนี้ทับ | อ่านอย่างเดียว |
| LOGIN DATABASE | ตรวจ Username / Password / เลขบัตรข้าราชการ / วันเกิด (แท็บ `DATABASE LOGIN`, หาคอลัมน์จากชื่อหัว) | อ่านอย่างเดียว |
| Passenger Seat Plan A320 User Page | ประวัติผู้ใช้ทั่วไป: แท็บ `หน้าแรก` + แท็บละภารกิจ | เขียน |
| Passenger Seat Plan A320 Admin Page | `HISTORY LOGIN DATA`, `ADMIN ACTIVITY`, `LOCKED USERS`, `LOCKED EMAILS` | เขียน |

---

## 2. ไฟล์ใน repo

| ไฟล์ | หน้าที่ |
|---|---|
| `index.html` | โครงหน้าเว็บทั้งหมด (Gate, รายการภารกิจ, ผังที่นั่ง, หน้าต่าง dialog ต่างๆ) |
| `app.css` | หน้าตา (ห้ามใช้ `color-mix()` และ `box-shadow: inset` เพราะ html2canvas อ่านไม่ได้ ใช้ตัวแปร rgba และ `::before` แทน) |
| `app.js` | ตรรกะหน้าเว็บทั้งหมด |
| `excel.js` | สร้างไฟล์ .xlsx เอง (ผัง A320) (เขียน OOXML + JSZip) `window.SeatExcel.build(model) → Promise<Blob>` |
| `boardingpass.js`, `boardingpass.css`, `emblem.png` | Boarding Pass |
| `config.js` | `window.SEATPLAN_CONFIG = { API_URL: '.../exec' }` เท่านั้น |
| `privacy.html` | นโยบายความเป็นส่วนตัว (ใช้กับ OAuth consent screen) |
| `apps-script/Code.gs` | ระบบหลังบ้านทั้งหมด (ไฟล์เดียว) |
| `README.md` | คู่มือภาษาไทย |

ไลบรารีจาก cdnjs: html2canvas 1.4.1, jsPDF 2.5.1, JSZip 3.10.1 · Google Identity Services (`accounts.google.com/gsi/client`)

---

## 3. ขั้นตอนใช้งานหน้าเว็บ

### 3.1 Gate (ทุกครั้งที่เปิดหน้าเว็บ)
หน้าเรียบง่าย ตัวหนังสือไม่ใหญ่ 2 ขั้น:
1. **"1. Sign in with Google"** (ปุ่ม GIS) — หลังบ้านตรวจ ID token ผ่าน `oauth2.googleapis.com/tokeninfo` และเทียบ `aud` กับ Client ID เก็บอีเมลไว้ลงประวัติ (ถ้ายังไม่ตั้ง Client ID ข้ามขั้นนี้ได้ แต่ประวัติไม่มีอีเมล)
2. **"2. รหัสผ่านเว็บไซต์ในการเข้าใช้งาน"** — ตรวจกับ `ACCESS_PIN` ได้ session token อายุ 12 ชม.

### 3.2 รายการภารกิจ
- แท็บ: **ภารกิจที่เปิดอยู่** / **ภารกิจที่เสร็จสิ้นเรียบร้อย** / **ภารกิจทั้งหมด** พร้อมจำนวน และตัวกรองช่วงวันที่บิน (วัน/เดือน/ปี แบบ dropdown) + ปุ่ม "ทุกช่วงเวลา"
- ผู้ใช้ทั่วไปเห็นเฉพาะภารกิจที่เปิดอยู่ แท็บที่เสร็จสิ้น / ทั้งหมด แสดง 🔒 ต้อง Login ด้วย LOGIN DATABASE (`missions.unlock` → view token อายุ 2 ชม. ผูกกับอีเมล) ไม่ต้องใช้รหัสภารกิจ หลังบ้านไม่ส่งภารกิจที่ปิดแล้วให้ถ้าไม่มี view token และเปิดภารกิจที่ปิดแล้วไม่ได้ (`need_view`)
- **+ เปิดภารกิจใหม่**: ฟอร์มมี Sector, เที่ยวบินที่, ชื่อภารกิจ, เส้นทางบิน (IATA), วัน-เดือน-ปีที่บิน, รหัสภารกิจ (≥4 ตัว) + ยืนยัน, ชื่อผู้บันทึก แล้ว **ต่อด้วยช่อง Username, Password, หมายเลขบัตรประจำตัวข้าราชการ, วันเดือนปีเกิด (พ.ศ.) ในฟอร์มเดียวกัน** ปุ่ม "เปิดภารกิจ" (ตรวจกับ LOGIN DATABASE, บันทึก ADMIN ACTIVITY "เปิดภารกิจใหม่")
- เข้าภารกิจต้องใส่ **รหัสภารกิจ** ทุกครั้ง (mission token 12 ชม.)

### 3.3 หน้าผังที่นั่ง (ผู้โดยสาร)
- ข้อมูลภารกิจ: ประเภทเครื่องบิน (A320 ล็อกไว้), Sector, ชื่อภารกิจ, เที่ยวบินที่, เส้นทางบิน (IATA), วันที่บิน, GATE, BOARDING TIME, หมายเหตุ Boarding Pass
- ผังที่นั่ง A320(05) 76 ที่ (กลุ่มซ้าย A B C · ทางเดิน · กลุ่มขวา D E F) เรียงหัว→ท้าย เหมือนไฟล์ For Admin:
  - FRONT CREW ZONE: ห้อง Commander Center `CC1A` (แสดง "CC 1A", แถว `CC` ไม่ถูกกั้น), 1D 1E 1F
  - VVIP OA: 2A, 3D 3F (โต๊ะฝั่งซ้ายแถว 3), 4A
  - VIP OB: 5A 5D 5F, 6A 6D 6F (โต๊ะระหว่างแถว 5-6)
  - VIP OC: 7B 7C 7D 7F, 8B 8C 8D 8F
  - PASSENGER OD: แถว 9-12, 14-18 (ไม่มีแถว 13) A-F
- Monitor 21 จอ (Wall 21.5" / Wall 27" / OVHD 12") แสดงเป็นแถบ "Monitor" / "OVHD Monitor" เหนือกลุ่มที่นั่ง รหัสเดียวกับเว็บช่าง: `MON-F1/F2` (หน้า CC 1A / หน้า 1D-1F), `MON-A1..A3` (หน้า 2A, หน้า 3D-3F, หลัง 4A), `MON-B1..B4` (หน้า 5A, หน้า 5D-5F, หลัง 6A, หลัง 6D-6F), `MON-C1/C2` (หน้า 7B-7C, 7D-7F), `MON-D1/D2` (หน้า 9A-9C, 9D-9F), `MON-D10L/R`, `D12`, `D15`, `D17` (OVHD)
- ภารกิจใหม่ตั้ง CREW ZONE หัวเครื่อง = แถว 1 ให้อัตโนมัติ (`DEFAULT_CREW_FRONT`)
- สภาพที่นั่งอ่านสดจาก For Admin: `ปกติ` / `ขัดข้อง-ใช้ได้` / `ใช้ไม่ได้` และจอ `จอปกติ` / `จอเสื่อม` / `จอเสีย` + NOTE (cache 20 วินาที) หาตำแหน่งจากชื่อในช่อง (ชื่อ / สภาพ / NOTE เรียงลงมา) แบบเดียวกับ `buildMap_` ของเว็บช่าง จอชื่อซ้ำจับคู่ตามลำดับซ้าย→ขวา บน→ล่าง ที่นั่งขัดข้องแสดงเป็น **แถบสีเล็กใต้ที่นั่ง** ทั้งบนจอ, PREVIEW, PDF
- แตะที่นั่ง → กรอก ยศ/ชื่อ/สกุล ภาษาไทย และ ยศ/ชื่อ/สกุล ภาษาอังกฤษ (ตัวพิมพ์ใหญ่) ที่นั่งแสดงชื่อย่อ เช่น `มานะ พ.` / `Mana P.`
- ช่อง **ชื่อผู้บันทึก** ต้องกรอกทุกครั้งก่อน SAVE
- ปุ่ม: **CLEAR** (ล้างเฉพาะรายชื่อ) · **SAVE** · **PREVIEW** · **EXPORT TO PDF** · **EXPORT TO EXCEL** · **PRINT BOARDING PASS** (พิมพ์เฉพาะผู้โดยสาร ไม่รวมลูกเรือ)
- กันบันทึกทับ: ถ้ามีคนอื่น SAVE ไปก่อน (เลขเวอร์ชันไม่ตรง) ขึ้นหน้าต่าง "มีผู้ SAVE ข้อมูลไปก่อน" → "โหลดข้อมูลล่าสุด" แล้ว SAVE ใหม่ · ใช้ LockService กันบันทึกพร้อมกัน
- ออกจากหน้าโดยยังไม่ SAVE ขึ้นเตือน
- ประวัติการบันทึกของภารกิจ (พับได้) — ผู้ใช้ทั่วไป **ไม่เห็นแถวที่เกี่ยวกับรายชื่อลูกเรือ** (ยกเว้น "กั้นที่นั่งลูกเรือ") แอดมินเห็นทั้งหมด

### 3.4 การจัดการภารกิจ (ต้อง Login LOGIN DATABASE ทุกครั้ง)
- **เปลี่ยนรหัสภารกิจ**
- **ปิดภารกิจ**: ต้องใส่ **รหัสภารกิจ** + Login
- **เปิดภารกิจอีกครั้ง**: ต้องใส่รหัสภารกิจ + Login
- ภารกิจที่ปิดแล้วดูได้อย่างเดียว

### 3.5 ADMIN (ปุ่มล่างสุด ต้อง Login, admin token 2 ชม.)
- กั้น **CREW ZONE**: "จากหัวเครื่อง ถึงแถว" และ "จากท้ายเครื่อง ถึงแถว" (ห้อง CC ไม่ถูกกั้น)
- กรอกรายชื่อลูกเรือ, PREVIEW / PDF `(CREW)` / EXCEL / Boarding Pass ลูกเรือ, LOGOUT
- ไม่มีการจำกัดตาม Role (ผู้ใช้ยืนยันว่าไม่ต้องทำเพิ่ม)

### 3.6 Export
- **PDF**: html2canvas + jsPDF, A4 แนวตั้ง แบ่งหน้าตามบล็อก ชื่อไฟล์เวลาไทย เช่น `261013_10.44 TH-LT_RTAF201_Seat Plan A320(05) (Pax).pdf` / `(CREW)`
- **EXCEL** (หน้าผู้โดยสาร, ADMIN และในหน้า PREVIEW): ถามก่อนว่า **"มีรายชื่อที่ SAVE แล้ว"** หรือ **"ฟอร์มเปล่า (ไว้กรอกเอง)"** ได้ .xlsx แก้ไขได้ หน้าตาและการแบ่งหน้า A4 เหมือน PDF (ตาราง 10 คอลัมน์คงที่: A ขอบ · B เลขแถว · C-E ที่นั่ง A-C · F ทางเดิน · G-I ที่นั่ง D-F · J ขอบ, scale คงที่ไม่ใช้ fit-to-page, ตัดหน้าด้วย rowBreaks แบบเดียวกับ PDF) ฟอร์มเปล่ามีชื่อไฟล์ `(Pax Blank Form)` (ห้ามมีตัวไทยในชื่อไฟล์ดาวน์โหลด)
- **Boarding Pass**: ขนาด 203 × 82.5 มม. A4 ได้ 3 ใบ/แผ่น จัดกึ่งกลาง (ขอบบน-ล่าง 22.75 มม., ข้าง 3.5 มม., ห่าง 2 มม., รอยตัดอยู่ในขอบกระดาษ) พิมพ์ที่ Scale 100%, Margins None, เปิด Background graphics มีตารางรหัส IATA → ชื่อเมือง

---

## 4. ระบบหลังบ้าน (`apps-script/Code.gs`)

### 4.1 SETUP / Script Properties
`SETUP = { accessPin, googleClientId, adminSeatSheet, loginDatabase, userLogSheet, adminLogSheet }` กรอกในหน้า Apps Script แล้วรัน `setup()` ครั้งเดียว → เก็บใน Script Properties (`ACCESS_PIN`, `GOOGLE_CLIENT_ID`, `ADMIN_SEAT_SHEET_ID`, `LOGIN_DB_ID`, `USER_LOG_ID`, `ADMIN_LOG_ID`, `SECRET` สุ่มเอง) แล้วลบค่าใน SETUP ได้ · `setup()` สร้างแท็บประวัติ / แท็บล็อกให้เอง

### 4.2 Actions
`config`, `enter`, `missions`, `missions.unlock`, `create`, `open`, `load`, `save`, `log`, `mpw.change`, `mission.close`, `mission.reopen`, `admin.login`, `admin.logout`, `admin.save` · `doGet` คืน `{"ok":true,"app":"Seat Plan A320(05) Passenger API"}`

Token: `s` session 12 ชม. · `m` mission 12 ชม. · `a` admin 2 ชม. · `v` view 2 ชม. (ผูกอีเมล)

### 4.3 แท็บ `รายการภารกิจ`
หัวคอลัมน์: ID, ชื่อแท็บ, ชื่อภารกิจ, เที่ยวบินที่, เส้นทางบิน, วันที่บิน, ผู้โดยสาร, สร้างเมื่อ, แก้ไขล่าสุด, ผู้บันทึกล่าสุด, Sector, สถานะ, รหัสภารกิจ, ปิด / เปิดล่าสุด, แท็บประวัติ (User Page)

### 4.4 แท็บภารกิจ
- A3:B11 ข้อมูลภารกิจ (B3 = A320) (ประเภทเครื่องบิน, Sector, ชื่อภารกิจ, เที่ยวบินที่, เส้นทางบิน, วันที่บิน, GATE, BOARDING TIME, หมายเหตุ Boarding Pass)
- B13:B14 CREW ZONE หัว/ท้าย · แถว 15 บันทึกล่าสุด · แถว 16 เวอร์ชัน
- แถว 18 หัวตาราง, แถว 19 ลงไป: ที่นั่ง, โซน, ประเภท (ผู้โดยสาร/ลูกเรือ), ยศ (ไทย), ชื่อ (ไทย), สกุล (ไทย), RANK (EN), FIRST NAME (EN), LAST NAME (EN), ผู้บันทึก, เวลาบันทึก
- คอลัมน์ M ลงไป (เริ่มแถว 3): ประวัติ — เวลา, Email, ผู้บันทึก, ประเภท, ที่นั่ง / ช่อง, ค่าเดิม, ค่าใหม่

### 4.5 LOGIN DATABASE
หาคอลัมน์จากชื่อหัว: ยศ, ชื่อ-สกุล, ตำแหน่งงาน, ID Card ขรก., ID Card ปชช., BIRTH DATE, Role, Username, Salt, hash, วันสิ้นสุดการใช้งาน · ตรวจรหัสแบบ **PBKDF2-HMAC-SHA256** (แบบเดียวกับระบบ eadmin) + เลขบัตรข้าราชการ + วันเกิด + ยังไม่หมดอายุ · ข้อความผิดพลาดกลางๆ ไม่บอกว่าช่องไหนผิด

### 4.6 ไฟล์ประวัติ
- **User Page** แท็บ `หน้าแรก`: Timestamp, Email, ผู้บันทึก, Action, Detail, **ภารกิจที่เปิดอยู่ ณ ขณะนั้น** (คอลัมน์ F: จำนวน + รายชื่อภารกิจที่เปิดอยู่) · แท็บละภารกิจ: Timestamp, Email, ผู้บันทึก, Action, Detail
- **Admin Page**:
  - `HISTORY LOGIN DATA`: Timestamp, Email, Username, Event, ใช้สำหรับ, ภารกิจ, รายละเอียด
  - `ADMIN ACTIVITY`: Timestamp, Email, Username, ภารกิจ, Action, Detail
  - `LOCKED USERS` / `LOCKED EMAILS`: Username|Email, สถานะ, จำนวนครั้งที่ผิด, พักถึงเวลา, รอบ, อัปเดตล่าสุด, วิธีปลดบล็อก
- เวลาทั้งหมดเป็น Asia/Bangkok

### 4.7 กติกาล็อก (เหมือนกันทุกรหัส)
**ผิดติดกัน 5 ครั้ง → พัก 60 นาที (PAUSED) → พ้นเวลาได้ลองอีกรอบ (LAST_TRY) → ผิดอีก 5 ครั้ง → BLOCKED** จนแอดมินแก้ช่องสถานะเป็น `ACTIVE` (ระบบรีเซ็ตตัวนับให้)

| ใส่ผิดตอน | นับที่ |
|---|---|
| รหัสผ่านเว็บไซต์ | Email (`LOCKED EMAILS`) |
| รหัสภารกิจตอนเข้าภารกิจ | Email |
| Login LOGIN DATABASE (เปิดภารกิจใหม่, ดูภารกิจที่ปิด, เปลี่ยนรหัส, ปิด/เปิดภารกิจ, ADMIN) รวมรหัสภารกิจตอนปิด/เปิด | ทั้ง Username (`LOCKED USERS`) และ Email |

- Email ที่ถูกพัก/บล็อก เข้าเว็บไม่ได้ · Username ที่ถูกพัก/บล็อก Login ไม่ได้จากทุกอีเมล
- ใส่ถูกในรอบ LAST_TRY **ไม่ล้าง** สถานะ LAST_TRY (ตั้งใจไว้ ผู้ใช้ยืนยันให้คงไว้)

---

## 5. การติดตั้ง / อัปเดต

1. เปิดไฟล์ **Seat Plan A320 TEMPORARY FOR PASSENGER** → ส่วนขยาย › Apps Script
2. วางโค้ด `apps-script/Code.gs` ทับทั้งหมด → Ctrl+S (ติดตั้งครั้งแรก: กรอก SETUP แล้วรัน `setup`)
3. ทำให้ใช้งานได้ › จัดการการทำให้ใช้งานได้ › ✏️ › Version: **New version** › Deploy (URL เดิม)
4. หน้าเว็บ: push ขึ้น GitHub แล้วกด Ctrl+F5
5. Sign in with Google: Google Cloud Console → OAuth consent screen (External, ใช้ `privacy.html`) → OAuth client ID แบบ Web application → Authorized JavaScript origins `https://rtaf602.github.io` → ใส่ Client ID ใน `SETUP.googleClientId` แล้วรัน `setup`
6. ถ้า Apps Script ขึ้น "An unknown error has occurred" ให้ใช้หน้าต่างไม่ระบุตัวตนที่ Login บัญชีเดียว

---

## 6. การทดสอบ

- ตัวจำลอง Apps Script ในเครื่อง (จำลอง SpreadsheetApp / PropertiesService / CacheService / LockService / UrlFetchApp) ทดสอบทุก action และกติกาล็อก
- Playwright (Chromium) ทดสอบหน้าเว็บ: Gate, เปิดภารกิจใหม่, เข้า/บันทึก, ปิดภารกิจ, แท็บที่ต้อง Login, PDF, Excel (ตรวจด้วย openpyxl และเรนเดอร์ด้วย LibreOffice)
- สิ่งที่ต้องให้ผู้ใช้ทดสอบเองบน Google จริงทุกครั้ง: Deploy ใหม่แล้ว `/exec` ตอบ ok, Login, การล็อก, ไฟล์ Excel ใน Microsoft Excel
