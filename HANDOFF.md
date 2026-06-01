# HANDOFF — เครื่องคำนวณ "ไทยช่วยไทยพลัส" (60/40)

สรุปสำหรับ Claude Code: รับช่วงต่อโปรเจกต์นี้ได้เลย ด้านล่างคือสถานะปัจจุบันทั้งหมด

## 1. โปรเจกต์คืออะไร

เว็บแอป static หน้าเดียว ช่วยคำนวณสิทธิโครงการรัฐ "ไทยช่วยไทยพลัส" แบบ 60/40
(รัฐสมทบ 60% / ผู้รับสิทธิจ่ายเอง 40% ผ่านแอปเป๋าตัง) ภาษาไทยทั้งหมด

- **Stack:** HTML + CSS + vanilla JS ไฟล์เดียว ไม่มี build step ไม่มี dependency รันไทม์
- **ฟอนต์:** Kanit + Sarabun จาก Google Fonts (โหลดผ่าน `<link>`)
- **ไม่ใช้** localStorage / sessionStorage / framework ใด ๆ — state อยู่ใน DOM/ตัวแปร JS ล้วน

## 2. โครงสร้างไฟล์

```
thai-help-thai-plus/
├── index.html      # แอปทั้งหมด (HTML+CSS+JS inline) — ไฟล์เดียวที่ deploy จริง
├── README.md
├── LICENSE         # MIT
├── vercel.json     # cleanUrls=true, static, ไม่มี build
└── .gitignore
```

git init แล้ว มี commit history สะอาด (ดู section 6) branch หลักคือ `main`

## 3. ค่าคงที่ / กติกาโครงการ (สำคัญต่อการคำนวณ)

อยู่ต้น `<script>` ใน index.html:

```js
const DAILY_CAP = 200;            // เพดานยอดใช้สิทธิต่อคน/วัน
const GOV_RATE  = 0.6;            // รัฐช่วย 60%
const YOU_RATE  = 0.4;            // จ่ายเอง 40%
const GOV_DAILY = DAILY_CAP*0.6;  // = 120 รัฐช่วยสูงสุด/วัน
const MONTH_GOV = 1000;           // รัฐช่วยสูงสุด/เดือน/คน
```

กติกาอื่น: รวม 4,000 บาท/คน, ช่วง 1 มิ.ย.–30 ก.ย. 2569, สิทธิไม่ทบเดือนถัดไป

## 4. โครงสร้าง UI (ตามลำดับใน DOM)

1. **Header** — ชื่อโครงการ + badge 60/40
2. **Tabs (sticky)** — สองแท็บ: `จ่ายคนเดียว` / `จ่ายหลายคน` สลับ `.panel.active`
3. **Panel: single** (`#panel-single`) — เครื่องคิดเลขรายคน + แถบสัดส่วน + เตือนเกินเพดาน
4. **Panel: split** (`#panel-split`) — split bill (ดูตรรกะ section 5)
5. **ติดตามสิทธิคงเหลือ** — input ยอดที่รัฐช่วยไปแล้ว → คำนวณสิทธิเหลือ (ส่วนรวม เห็นทั้งสองแท็บ)
6. **เงื่อนไขวงเงิน** — ตารางกติกา (ส่วนรวม)
7. **Support** — การ์ดรับบริจาค (ดู section 7)
8. **Footer** — disclaimer

## 5. ตรรกะ split bill (จุดที่เคยมีบั๊ก ระวังตอนแก้)

หารบิลเท่ากันต่อคนก่อน แล้วค่อยแยกส่วน — **อย่ากลับไปคิดจากเพดาน 200 ตรง ๆ** (บั๊กเดิมคือเอายอดที่จ่ายผ่านแอปเพื่อรับสิทธิมาแสดงเป็นยอดรวมที่ต้องจ่าย ทำให้ส่วนเกินหายไป):

```js
perBill   = B / N;                       // ยอดบิลต่อคน
perApp    = min(perBill, DAILY_CAP);     // ส่วนที่ได้สิทธิ (<=200)
perExcess = max(0, perBill - DAILY_CAP); // ส่วนเกิน ไม่ได้สิทธิ
perGov    = perApp * 0.6;                // รัฐช่วย/คน (<=120)
perSelf   = perApp * 0.4 + perExcess;    // จ่ายเอง/คน
```

**กรอบความเข้าใจ (สำคัญ):** ผู้ใช้ยืนยันว่าส่วนเกิน "ไม่ต้องควักเงินสด" — เติมเงินเข้ากระเป๋าตังแล้วจ่ายผ่านแอป แอปหักให้อัตโนมัติ ดังนั้น **ห้ามใช้คำว่า "เงินสด/ควัก"** ทั้งแอป ใช้ "จ่ายเอง (ผ่านเป๋าตัง)" แทน

การ์ดรายคน: ตัวเลขเด่นสุด = **ยอดของเขา** (perBill) ใต้เส้นประมีสองบรรทัด *แยกบรรทัดชัด* — `จ่ายเอง` (สีส้ม) และ `รัฐช่วย` (สีเขียว) ขนาดเท่ากัน (`.pbd span{display:block}`)

ตรวจความถูกต้องได้เสมอด้วย: `govTotal + selfTotal === B`

## 6. git log (ล่าสุดอยู่บน)

```
Swap order: tracker above conditions
Switch single vs split payment into sticky tabs (no scrolling)
Add USDC-on-Base support section (0xTheSleeper) with reveal button + QR
Add split bill ... / fixes
Initial commit: ไทยช่วยไทยพลัส 60/40 calculator
```

## 7. Support section (crypto)

- ชื่อที่แสดง: **0xTheSleeper**
- เครือข่าย: **USDC on Base**
- ที่อยู่: `0x871FFe85d1CC11916f4C6d2B81a321C22a3f4415`
- พฤติกรรม: เริ่มต้นซ่อนที่อยู่ ปุ่มเดียว ("แสดง QR + ที่อยู่") กดแล้วเผย QR + ที่อยู่ + ปุ่มคัดลอก
- **QR เป็น SVG ฝัง inline** (สร้างจาก python `qrcode`, encode ที่อยู่เปล่า ๆ) ไม่พึ่ง service ภายนอก ถ้าจะเปลี่ยนที่อยู่ ต้อง regenerate QR ใหม่ (เนื้อ path ใน `#qr-path`)

## 8. งานที่เหลือ — deploy (ผู้ใช้มีแค่สมาร์ทโฟน)

ยังไม่ได้ขึ้น GitHub/Vercel **ไม่มี GitHub/Vercel connector ที่ผูกบัญชีผู้ใช้** จึงทำให้อัตโนมัติไม่ได้จากฝั่งแชต ขั้นตอนที่ตั้งใจไว้:

1. สร้าง public repo บน GitHub (CLI: `gh repo create thai-help-thai-plus --public --source=. --push` หรืออัปไฟล์ผ่านเว็บ)
2. Import เข้า Vercel → Framework Preset = **Other** (static ล้วน) → Deploy
3. ทุก push เข้า `main` → auto-redeploy

ถ้า Claude Code มีสิทธิ์เข้า GitHub/Vercel ของผู้ใช้ ก็ทำขั้น 1–2 ให้อัตโนมัติได้เลย

## 9. ข้อควรระวังตอนแก้ต่อ

- แก้ที่ `index.html` ที่เดียว (ในเครื่องผู้ใช้ไฟล์พรีวิวชื่อ `thai-chuay-thai-calculator.html` เป็นสำเนาเดียวกัน)
- หลังแก้ ตรวจว่า `.panel` เปิด/ปิด `<div>` ครบคู่ (มี 2 panel)
- เป็น static — ห้ามใส่ browser storage; ถ้าจะเก็บประวัติการใช้ข้ามวัน ต้องคุยเรื่อง backend/ทางเลือกก่อน
- ข้อความทั้งหมดเป็นภาษาไทย โทนสุภาพ ลงท้าย "ครับ"
