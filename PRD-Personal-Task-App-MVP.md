# PRD — Personal Task App (30-Minute MVP)

| Field | Value |
|---|---|
| Document owner | Suthon (DevOps) |
| Status | Draft v4.0 — Kanban board |
| Date | 2026-04-28 |
| Target release | 30 นาที |

---

## 1. Overview

**Goal.** สร้าง task app แบบ Kanban board ที่ใช้ได้จริงภายใน 30 นาที ด้วย single-file HTML + vanilla JS + localStorage ไม่มี backend ไม่มี build step เปิดในเบราว์เซอร์ใช้ได้เลย

**Why.** ต้องการเครื่องมือเก็บ task ส่วนตัวที่เห็นภาพรวม flow งาน (Todo → In Progress → Done) deploy เป็น static file บน S3/Netlify/GitHub Pages ได้ทันที zero ops

**Out of scope.** auth, multi-device sync, backend, push notification, sharing, recurring task, sub-task, attachment, themes ลึกๆ

---

## 2. Success criteria

- เปิดไฟล์ → ใช้งานได้ทันที (ไม่ต้อง install / build)
- เพิ่ม / ย้าย status / ลบ / จัดเรียง task ได้ภายใน 1 คลิก/keystroke/drag
- รีเฟรชหน้าแล้วข้อมูล + ลำดับในแต่ละคอลัมน์ยังอยู่ครบ (localStorage)
- โค้ดทั้งหมดอยู่ใน 1 ไฟล์ ≤ 400 บรรทัด
- ใช้งานบนมือถือได้ (responsive — คอลัมน์ stack แนวตั้ง + รองรับ touch drag)

---

## 3. MVP feature scope (must-have เท่านั้น)

### 3.1 เพิ่ม task
- Input box ด้านบน (เหนือ board) + ปุ่ม Add (หรือกด Enter)
- Field: title อย่างเดียว
- task ใหม่ถูกสร้างด้วย status = `todo` และเพิ่มไว้ **บนสุด** ของคอลัมน์ Todo
- บันทึกลง localStorage ทันที

### 3.2 Kanban board (3 columns)
- 3 คอลัมน์เรียงซ้าย → ขวา: **Todo** / **In Progress** / **Done**
- หัวคอลัมน์: ชื่อ status (สีตาม status) + count จำนวน task ในคอลัมน์
- คอลัมน์ว่าง: แสดง placeholder "ว่าง"
- แต่ละ task card มี: drag handle (≡), title, ปุ่มลบ (×)
- task ใน Done → title ขีดฆ่า + จางลง

### 3.3 ย้าย status (drag ระหว่างคอลัมน์)
- ลาก task card จากคอลัมน์หนึ่งไปอีกคอลัมน์ → `status` เปลี่ยนตามคอลัมน์ปลายทาง
- ปล่อยลงพื้นที่ว่างในคอลัมน์ → append ที่ท้ายคอลัมน์
- ปล่อยใกล้ task อื่น → แทรกก่อน/หลัง (ตามตำแหน่ง pointer)
- บันทึก localStorage ทันทีหลัง drop

### 3.4 ลบ task
- ปุ่ม × บน card → confirm() → ลบทันที + บันทึก localStorage

### 3.5 Reorder ภายในคอลัมน์
- ลาก task card ภายในคอลัมน์เดียวกัน → จัดลำดับใหม่
- รองรับทั้ง mouse (HTML5 Drag & Drop API) และ touch (pointer events)
- ขณะลาก: highlight คอลัมน์ปลายทาง + placeholder ตำแหน่งที่จะ drop
- ใช้ native browser API เท่านั้น (ไม่พึ่ง library)

### 3.6 Persistence (localStorage)
- Key เดียว: `localStorage["tasks_v1"]`
- รูปแบบ: JSON array — เรียงตามลำดับ insert/move ตรงๆ (แต่ละคอลัมน์ filter จาก array นี้โดยรักษาลำดับเดิม)
- โหลดตอน page load: `JSON.parse(localStorage.getItem("tasks_v1") ?? "[]")`
- Save: ทุกครั้งที่ state เปลี่ยน (add / move status / delete / reorder)
- กัน corrupt: try/catch รอบ parse + ตรวจ `Array.isArray`, ถ้า fail → fallback `[]` + `console.warn`

---

## 4. Tech stack

- HTML5 + CSS (inline `<style>`, CSS Grid สำหรับ layout คอลัมน์) + vanilla JS (inline `<script>`)
- ไม่มี framework ไม่มี dependency
- 1 ไฟล์: `index.html`
- Drag & drop: native HTML5 API (`draggable`, `dragstart`, `dragover`, `drop`) สำหรับ mouse + Pointer Events (`pointerdown/move/up`) บน drag handle สำหรับ touch

---

## 5. Data model

```json
{
  "id": "1714300000000abcd",
  "title": "Write PRD",
  "status": "in_progress",
  "createdAt": 1714300000000
}
```

`status` ∈ `"todo" | "in_progress" | "done"`
ลำดับใน array = ลำดับที่ user จัดเรียง (filter ทีละ status เพื่อ render ในแต่ละคอลัมน์ โดยรักษา relative order)

---

## 6. Acceptance criteria

| # | Criteria |
|---|---|
| AC-1 | พิมพ์ title แล้วกด Enter → task ใหม่ status = todo อยู่บนสุดของคอลัมน์ Todo, refresh แล้วยังอยู่ |
| AC-2 | ลาก task จากคอลัมน์ Todo → In Progress → status เปลี่ยน, refresh แล้ว state เดิม + อยู่ในคอลัมน์ใหม่ |
| AC-3 | กด × → confirm → task หาย, refresh แล้วยังหาย |
| AC-4 | ลากสลับลำดับภายในคอลัมน์เดียวกัน → ลำดับใหม่คงอยู่หลัง refresh |
| AC-5 | ลากไปทิ้งในพื้นที่ว่างของคอลัมน์ → task ไปอยู่ท้ายคอลัมน์นั้น |
| AC-6 | localStorage ว่าง / corrupt → board ว่างทุกคอลัมน์ ไม่ throw, console.warn ได้ |
| AC-7 | ใช้ได้บน Chrome/Safari/Firefox version ปัจจุบัน + touch drag บน mobile ใช้งานได้ |
| AC-8 | หน้าจอกว้าง ≤ 720px → คอลัมน์ stack แนวตั้ง, ที่ 360px ยังใช้งานได้ |

---

## 7. Non-goals (ไม่ทำใน MVP)

ไม่มี auth, ไม่มี cloud sync, ไม่มี due date, ไม่มี priority/tag, ไม่มี filter/search, ไม่มี undo, ไม่มี dark mode toggle (ใช้ system preference อย่างเดียว), ไม่มี export/import, ไม่มี multi-board, ไม่มี edit title (ลบแล้วสร้างใหม่), ไม่มี custom status (fix 3 คอลัมน์)

---

## 8. Build plan (30 นาที)

| เวลา | งาน |
|---|---|
| 0–4 นาที | `index.html` skeleton + CSS Grid 3 คอลัมน์ + status colors |
| 4–10 นาที | JS: state, render() filter ตาม status, add / delete |
| 10–16 นาที | localStorage load/save + handle corrupt + key versioning |
| 16–25 นาที | Drag & drop: ภายในคอลัมน์ + ข้ามคอลัมน์ (HTML5 + touch fallback) → save |
| 25–28 นาที | Mobile responsive (stack คอลัมน์ ≤ 720px) + ขัดเกลา UI |
| 28–30 นาที | Smoke test ทั้ง 8 AC + deploy เป็น static file |

---

## 9. Post-30-min backlog (ถ้าใช้แล้วชอบ)

Edit task title (double-click), due date + reminder, tag/filter, undo, sync (Firebase / Supabase), PWA + offline, custom columns/statuses, multi-board, ส่งออก JSON, keyboard shortcuts (j/k navigate, 1/2/3 ย้าย status), WIP limit ต่อคอลัมน์, archive Done อัตโนมัติ
