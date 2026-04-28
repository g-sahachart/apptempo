# PRD — Personal Task App (30-Minute MVP)

| Field | Value |
|---|---|
| Document owner | Suthon (DevOps) |
| Status | Draft v3.0 — +status +DnD |
| Date | 2026-04-28 |
| Target release | 30 นาที |

---

## 1. Overview

**Goal.** สร้าง task app ที่ใช้ได้จริงภายใน 30 นาที ด้วย single-file HTML + vanilla JS + localStorage ไม่มี backend ไม่มี build step เปิดในเบราว์เซอร์ใช้ได้เลย

**Why.** ต้องการเครื่องมือเก็บ task ส่วนตัวที่ deploy เป็น static file บน S3/Netlify/GitHub Pages ได้ทันที zero ops

**Out of scope.** auth, multi-device sync, backend, push notification, sharing, recurring task, sub-task, attachment, themes ลึกๆ

---

## 2. Success criteria

- เปิดไฟล์ → ใช้งานได้ทันที (ไม่ต้อง install / build)
- เพิ่ม / เปลี่ยน status / ลบ / ลากจัดเรียง task ได้ภายใน 1 คลิก/keystroke
- รีเฟรชหน้าแล้วข้อมูล + ลำดับยังอยู่ครบ (localStorage)
- โค้ดทั้งหมดอยู่ใน 1 ไฟล์ ≤ 350 บรรทัด
- ใช้งานบนมือถือได้ (responsive + รองรับ touch drag)

---

## 3. MVP feature scope (must-have เท่านั้น)

### 3.1 เพิ่ม task
- Input box ด้านบน + ปุ่ม Add (หรือกด Enter)
- Field: title อย่างเดียว
- task ใหม่ถูกสร้างด้วย status = `todo` และเพิ่มไว้ **บนสุด** ของ list
- บันทึกลง localStorage ทันที

### 3.2 แสดง task list
- แสดงตามลำดับที่ user จัดเรียง (ไม่ auto-sort)
- แต่ละแถวมี: drag handle (≡), status pill, title, ปุ่มลบ (×)
- Status pill มี 3 สี: Todo (เทา) / In Progress (ฟ้า) / Done (เขียว)
- Done → title ขีดฆ่า + จางลง

### 3.3 เปลี่ยน status (3 states)
- คลิกที่ status pill → หมุนวน `todo` → `in_progress` → `done` → `todo`
- หรือเปิด dropdown เลือกตรงๆ ก็ได้
- บันทึก localStorage ทันทีหลังเปลี่ยน

### 3.4 ลบ task
- ปุ่ม × ข้างแถว → confirm() → ลบทันที + บันทึก localStorage

### 3.5 Drag & drop reorder
- ลากแถวด้วย drag handle (≡) เพื่อจัดลำดับใหม่
- รองรับทั้ง mouse และ touch (mobile)
- ขณะลาก: highlight ตำแหน่งที่จะ drop (placeholder)
- ปล่อย → list update + บันทึก `order` ใหม่ลง localStorage
- ใช้ HTML5 Drag & Drop API หรือ pointer events (ไม่พึ่ง library)

### 3.6 Persistence (localStorage)
- Key เดียว: `localStorage["tasks_v1"]`
- รูปแบบ: JSON array เรียงตามลำดับที่แสดงผลตรงๆ (index = order)
- โหลดตอน page load: `JSON.parse(localStorage.getItem("tasks_v1") ?? "[]")`
- Save: ทุกครั้งที่ state เปลี่ยน (add / status change / delete / reorder)
- กัน corrupt: try/catch รอบ parse, ถ้า error → fallback เป็น `[]` + log warning

---

## 4. Tech stack

- HTML5 + CSS (inline `<style>`) + vanilla JS (inline `<script>`)
- ไม่มี framework ไม่มี dependency
- 1 ไฟล์: `index.html`
- Drag & drop: native HTML5 API (`draggable`, `dragstart`, `dragover`, `drop`) + fallback pointer events สำหรับ touch

---

## 5. Data model

```json
{
  "id": "1714300000000",
  "title": "Write PRD",
  "status": "in_progress",
  "createdAt": 1714300000000
}
```

`status` ∈ `"todo" | "in_progress" | "done"` ลำดับใน array = ลำดับที่ user จัดเรียง

---

## 6. Acceptance criteria

| # | Criteria |
|---|---|
| AC-1 | พิมพ์ title แล้วกด Enter → task ใหม่ status = todo อยู่บนสุด, refresh แล้วยังอยู่ |
| AC-2 | คลิก status pill → หมุน todo → in_progress → done → todo, refresh แล้ว state เดิม |
| AC-3 | กด × → confirm → task หาย, refresh แล้วยังหาย |
| AC-4 | ลากแถวสลับลำดับ → ลำดับใหม่คงอยู่หลัง refresh |
| AC-5 | localStorage ว่าง / corrupt → list ว่าง ไม่ throw, console warn ได้ |
| AC-6 | ใช้ได้บน Chrome/Safari/Firefox version ปัจจุบัน + drag บน mobile (touch) ใช้งานได้ |
| AC-7 | หน้าจอกว้าง 360px ยังใช้งานได้ |

---

## 7. Non-goals (ไม่ทำใน MVP)

ไม่มี auth, ไม่มี cloud sync, ไม่มี due date, ไม่มี priority/tag, ไม่มี filter/search, ไม่มี undo, ไม่มี dark mode toggle (ใช้ system preference อย่างเดียว), ไม่มี export/import, ไม่มี multi-list/board view (Kanban columns)

---

## 8. Build plan (30 นาที)

| เวลา | งาน |
|---|---|
| 0–4 นาที | `index.html` skeleton + CSS layout (input, list, status pill colors) |
| 4–12 นาที | JS: state, render(), add / cycle status / delete |
| 12–18 นาที | localStorage load/save + handle corrupt + key versioning |
| 18–25 นาที | Drag & drop (HTML5 API + touch fallback) → save order |
| 25–28 นาที | Mobile responsive + ขัดเกลา UI |
| 28–30 นาที | Smoke test ทั้ง 7 AC + deploy เป็น static file |

---

## 9. Post-30-min backlog (ถ้าใช้แล้วชอบ)

Kanban view (3 columns by status), due date + reminder, tag/filter, undo, sync (Firebase / Supabase), PWA + offline, multi-list, ส่งออก JSON, keyboard shortcuts (j/k navigate, x toggle status)
