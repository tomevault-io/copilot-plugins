## school-management-app

> เอกสารนี้สรุปภาพรวมโครงสร้างการทำงาน สถาปัตยกรรม และโมดูลต่างๆ ของโปรเจกต์

# 🏫 โครงสร้างระบบ School Management Application

เอกสารนี้สรุปภาพรวมโครงสร้างการทำงาน สถาปัตยกรรม และโมดูลต่างๆ ของโปรเจกต์

## 🛠 Tech Stack (เทคโนโลยีที่ใช้)
- **Frontend Framework**: React (ผ่าน Vite)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Animations**: Framer Motion
- **Routing**: React Router DOM
- **Backend/Database**: Firebase (Firestore)
- **Icons**: React Icons (`react-icons/hi2`)

## 📂 โครงสร้างโฟลเดอร์หลัก (Project Structure)
โปรเจกต์นี้ใช้การจัดโครงสร้างแบบ Feature-based เพื่อให้โค้ดเป็นระเบียบและดูแลรักษาง่าย

- `src/components/` - คอมโพเนนต์ที่ใช้ร่วมกันในหลายๆ หน้า (เช่น Layouts, Button, Modal)
- `src/features/` - จัดเก็บโค้ดแยกตามโมดูลการทำงานหลัก
  - `home/` - หน้า Dashboard และ Grid Menu หลัก
  - `teachers/` - ระบบจัดการครู (TeacherManager, TeacherCard, etc.)
  - *(และโมดูลอื่นๆ ตามระบบ)*
- `src/hooks/` - Custom Hooks สำหรับจัดการ State และ Business Logic (เช่น `useAuth`, `useRolePermissions`, `useTeacherManager`)
- `src/types/` - Type Definitions (Interface) สำหรับ TypeScript

## 🧩 โมดูลและฟีเจอร์การทำงาน (Core Modules)
จากเมนูหลักในระบบ (`ALL_MENUS`) ระบบประกอบด้วยฟีเจอร์หลักดังนี้:

1. **ระบบจัดการบุคลากรและผู้ใช้ (People Management)**
   - จัดการผู้ใช้ (User Directory & Roles)
   - จัดการครู (Teacher Management)
   - จัดการนักเรียน (Student Records) รวมถึงการนำเข้าข้อมูลแบบ CSV (Bulk Import)
2. **ระบบการจัดการพื้นฐาน (Core Management)**
   - ระบบจัดการห้องเรียน (Classroom Management)
   - กำหนดสิทธิ์การใช้งาน (Role Permission Manager)
   - การตั้งค่าระบบและปีการศึกษา (Settings)
3. **ระบบจัดการการศึกษา (Academic Modules)**
   - หลักสูตร (Curriculum)
   - ตารางสอน (Class Schedule)
   - ปฏิทินการศึกษา (Academic Calendar)
   - แผนการสอน (Syllabus)
4. **ระบบการเรียนการสอน (Teaching & Assessment)**
   - จัดการการสอน (Teaching: เช็คชื่อ ภาระงาน สอบ)
   - การเข้าเรียน (Attendance Records)
   - ผลการเรียน (Grades: คะแนนและเกรด)
5. **ระบบสอบ (Examination)**
   - ห้องสอบออนไลน์ (Online Exam & Assessment)
   - คลังข้อสอบ (Question Bank & Repository)
6. **อื่นๆ**: ระบบประกาศข่าวสาร (Announcements)

## 🔐 ระบบสิทธิ์การเข้าถึง (RBAC - Role-Based Access Control)
ระบบใช้กลไกการกรองสิทธิ์ผ่าน `useAuth` และ `useRolePermissions` โดยอ้างอิงจาก `featureKey` ของแต่ละเมนู เพื่อแสดงผลหน้าต่างและฟังก์ชันที่ผู้ใช้คนนั้นๆ ได้รับอนุญาตเท่านั้น

## 🎨 UI/UX Design & Interactions
- **Glassmorphism**: การออกแบบ UI เน้นความโปร่งแสงและดูทันสมัย
- **Fluid Animations**: ใช้ Framer Motion ในการทำ Transition ระหว่างหน้า และ Animation ของการ์ดเมนู
- **Gesture/Touch**: รองรับการ Drag/Swipe เพื่อเปลี่ยนหน้า (Pagination) ได้อย่างลื่นไหล

---
> Source: [methaedu/school-management-app](https://github.com/methaedu/school-management-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
