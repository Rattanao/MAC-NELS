# Check EDI MAC-NELS

เครื่องมือตรวจสอบเอกสารขาเข้าเรือก่อนจัดทำใบขนสินค้าขาเข้า (Thailand ocean import) —
เทียบ **MANIFEST** (รายงานจากสายเรือ, .xls/.xlsx) กับ **ENTER** (ฟอร์ม ATTACHED SHEETS
ของ MAC-NELS SHIPPING (THAILAND) LTD., .pdf) ทีละ B/L ย่อย แล้วสร้างรายงาน `EDI.xlsx` แบบไฮไลต์สีบอกจุดที่ต้องแก้ก่อนยื่นใบขน

> **ข้อจำกัด:** หน้าเว็บรองรับเฉพาะ **ฟอร์ม ATTACHED SHEETS ของ MAC-NELS**
> ถ้าเจอฟอร์มของเจ้าอื่นหน้าเว็บจะแจ้งว่าไม่ใช่ฟอร์ม MAC-NELS · ไฟล์ CNTRS ไม่นำมาเทียบ

## Check EDI MAC-NELS บนเว็บ (ให้เพื่อนใช้ได้ ไม่ต้องติดตั้ง)

https://rattanao.github.io/MAC-NELS/

1. ลากไฟล์ **MANIFEST (.xls/.xlsx)** และ **ENTER ของ MAC-NELS (.pdf ได้หลายไฟล์)** มาวางพร้อมกัน
   (ตั้งชื่ออะไรก็ได้ — แยกประเภทจากนามสกุล; ไฟล์ที่ชื่อมี `CNTRS` จะถูกข้าม)
2. ใส่ SHED NO. ที่แจ้ง (ถ้ามี) แล้วกด **ตรวจสอบ**
3. ดูผลบนหน้าเว็บ (ตารางสีเหมือน EDI.xlsx, ค้นหา/กรอง "แสดงเฉพาะที่ไม่ตรง") หรือกด **ดาวน์โหลด EDI.xlsx**

ไฟล์ถูกประมวลผลในเบราว์เซอร์ของผู้ใช้เท่านั้น ไม่มีการอัปโหลดไปที่ใด

โค้ดหน้าเว็บอยู่ที่ `docs/` (`index.html` + `core.js` — พอร์ตจาก `build_edi.py` layout H เฉพาะฟอร์ม MAC-NELS,
ฐานเดียวกับ [Check EDI Penanshin](https://rattanao.github.io/Penanshin/) และ
[Check EDI Quality](https://rattanao.github.io/check-edi-Quality/))

## ฟอร์ม ENTER ที่รองรับ

หัวตาราง `MARKS NO.` / `PACKAGES` / `DESCRIPTION OF GOODS` แล้วต่อบล็อก:

```
<แถวสินค้า: marks | 4 BOXES | description>
CONSIGNEE [n] : <ชื่อผู้รับ>
CTRN NO       : <เลขตู้>/<ขนาด>         G.W.:  477.000 KGS.
B/L NO        : <เลข B/L>               MEAS:  1.814 CBMS.
```

ยอดรวมท้ายเอกสาร `TOTAL PKGS / TOTAL G.W. / TOTAL MEAS` ใช้เทียบกับ MANIFEST

## ตรวจอะไรบ้าง

- CONSIGNEE, CONTAINER NO., STATUS (CY / LCL / LCL-CFS), TOTAL PACKAGE, PACKAGING (`PLTS` = PALLET)
- GROSS WEIGHT, MEASUREMENT
- MARKS, DESCRIPTION — เทียบทีละคู่ ไม่นับช่องว่าง/วรรคตอน แต่คำที่เกิน/ขาดต้องเตือน (สีแดง)
- REEFER TEMP, DG (CLASS/UN), TRANSIT/TRANSHIPMENT — พบฝั่งใดก็เตือนเสมอ
- SHED NO., ประเทศปลายทาง, CARGO MOVEMENT, TAX ID เทียบ NOTIFY — **ถูกต้องตามกฎ = แสดง `-`**
  (แสดงค่าจริงเฉพาะตอนไม่ตรง/ต้องตรวจ)
- ข้อความสีแดงที่ลูกค้าพิมพ์ทับ PDF (FreeText) — ใช้กับ B/L ที่อยู่ใต้ข้อความ, `ทุก BL` = ใช้กับทุก B/L
- ยอดรวมทั้งฉบับเทียบกับ MANIFEST GRAND TOTAL และ B/L ที่มีใน ENTER แต่ตกหล่นจาก MANIFEST (วิกฤต)

สี: แดง ⚠ = ไม่ตรง · ส้ม = ต้องให้คนยืนยัน · เหลือง = ไม่มี ENTER เทียบ · เขียว = ผ่าน (คอลัมน์สรุป)

## ใช้งานแบบโปรแกรม Python (ในเครื่อง)

`build_edi.py` อ่าน ENTER ได้หลายฟอร์ม (MAC-NELS, Penanshin, CMS / THE M CORP, AGN, Quality, Cargoport ฯลฯ)

```bash
pip install -r requirements.txt
python build_edi.py --manifest MANIFEST.xls --enter ENTER.pdf --outdir output --shed 0141
```

ถ้า MANIFEST เป็นของทั้งลำเรือ แต่ ENTER เป็นของลูกค้ารายเดียว ให้เพิ่ม `--only-enter`
(รายงานเฉพาะ B/L ที่อยู่ใน ENTER และไม่เทียบ GRAND TOTAL ทั้งลำ)

## วิธีเปิดเว็บให้เพื่อนใช้ (GitHub Pages)

1. ขึ้น GitHub: https://github.com/Rattanao/MAC-NELS → **Settings → Pages**
2. **Build and deployment → Source:** *Deploy from a branch*
3. **Branch:** `main` และโฟลเดอร์ `/ (root)` → กด **Save**
4. รอ ~1 นาที ลิงก์จะขึ้นที่หัวหน้า Pages: `https://rattanao.github.io/MAC-NELS/`
   (หน้าแรกของ repo จะเด้งไป `docs/` เอง)
5. ส่งลิงก์ให้เพื่อนได้เลย — แก้โค้ดแล้ว `git push` หน้าเว็บอัปเดตเองภายในไม่กี่นาที

## หมายเหตุเรื่องข้อมูลลูกค้า

`input/`, `output/` และไฟล์ `.xls/.xlsx/.pdf` ถูก git-ignore ไว้ — repo นี้เก็บเฉพาะโปรแกรม
ไม่เก็บเอกสารจริงของลูกค้า

---
Project by Ms. Rattana Mudlae — Position Import DOC.
