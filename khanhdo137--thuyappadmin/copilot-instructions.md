## thuyappadmin

> This is an admin dashboard for "Thú Y Bình Dương" - a veterinary clinic management system in Vietnam. The application manages various aspects of veterinary practice operations.

# Veterinary Clinic Domain Guide

## Business Context
This is an admin dashboard for "Thú Y Bình Dương" - a veterinary clinic management system in Vietnam. The application manages various aspects of veterinary practice operations.

## Core Business Entities

### 1. Appointments (Lịch khám)
- **Page**: [src/pages/AppointmentsPage.js](mdc:src/pages/AppointmentsPage.js)
- **Route**: `/admin/appointments`
- **Purpose**: Manage pet appointment scheduling
- **Key Fields**: Date/time, pet, owner, doctor, service, status
- **Vietnamese Terms**: Lịch khám, Cuộc hẹn, Đặt lịch

### 2. Customers (Khách hàng)  
- **Page**: [src/pages/CustomersPage.js](mdc:src/pages/CustomersPage.js)
- **Route**: `/admin/customers`
- **Purpose**: Manage pet owners and customer information
- **Key Fields**: Name, phone, email, address, registration date
- **Vietnamese Terms**: Khách hàng, Chủ thú cưng

### 3. Doctors (Bác sĩ)
- **Page**: [src/pages/DoctorsPage.js](mdc:src/pages/DoctorsPage.js)
- **Route**: `/admin/doctors`
- **Purpose**: Manage veterinarian staff information
- **Key Fields**: Name, specialization, license number, contact info
- **Vietnamese Terms**: Bác sĩ thú y, Chuyên khoa

### 4. Pets (Thú cưng)
- **Page**: [src/pages/PetsPage.js](mdc:src/pages/PetsPage.js)
- **Route**: `/admin/pets`
- **Purpose**: Manage pet records and medical history
- **Key Fields**: Name, species, breed, age, owner, medical history
- **Vietnamese Terms**: Thú cưng, Vật nuôi, Hồ sơ bệnh án

### 5. Services (Dịch vụ)
- **Page**: [src/pages/ServicesPage.js](mdc:src/pages/ServicesPage.js)
- **Route**: `/admin/services`
- **Purpose**: Manage clinic services and pricing
- **Key Fields**: Service name, description, price, duration
- **Vietnamese Terms**: Dịch vụ, Khám bệnh, Phẫu thuật, Tiêm chủng

### 6. News (Tin tức)
- **Page**: [src/pages/NewsPage.js](mdc:src/pages/NewsPage.js)
- **Route**: `/admin/news`
- **Purpose**: Manage clinic announcements and articles
- **Key Fields**: Title, content, publish date, category
- **Vietnamese Terms**: Tin tức, Thông báo, Bài viết

### 7. Users (Tài khoản)
- **Page**: [src/pages/UsersPage.js](mdc:src/pages/UsersPage.js)
- **Route**: `/admin/users`
- **Purpose**: Manage system user accounts and permissions
- **Key Fields**: Username, email, role, permissions, status
- **Vietnamese Terms**: Tài khoản, Người dùng, Quyền hạn

## Common Veterinary Terminology

### Vietnamese UI Text
- **Actions**: Thêm mới, Chỉnh sửa, Xóa, Xem chi tiết
- **Status**: Hoạt động, Tạm dừng, Đã hủy, Hoàn thành
- **Search**: Tìm kiếm, Lọc, Sắp xếp
- **Time**: Ngày, Giờ, Tuần, Tháng, Năm
- **Medical**: Khám bệnh, Chẩn đoán, Điều trị, Đơn thuốc

### Data Relationships
```
Customer (1) → (N) Pets
Pet (1) → (N) Appointments
Doctor (1) → (N) Appointments
Service (1) → (N) Appointments
User (1) → (N) Various entities (audit trail)
```

## Business Rules to Consider

### Appointment Management
- Validate appointment time slots
- Prevent double-booking of doctors
- Check pet ownership
- Require service selection
- Track appointment status workflow

### Customer Management
- Link multiple pets to one customer
- Maintain contact information history
- Track customer loyalty/visit frequency

### Medical Records
- Maintain comprehensive pet health history
- Link appointments to medical records
- Track vaccination schedules
- Store treatment outcomes

## Icon Usage
Consistent Material-UI icons used in navigation from [src/layout/DashboardLayout.js](mdc:src/layout/DashboardLayout.js):
- `<Dashboard />` - Dashboard
- `<Schedule />` - Appointments
- `<People />` - Customers
- `<MedicalServices />` - Doctors
- `<Pets />` - Pets
- `<LocalHospital />` - Services
- `<Article />` - News
- `<AccountCircle />` - Users

## Branding
- **Clinic Name**: "Thú Y Bình Dương"
- **Logo**: [public/logo-home.png](mdc:public/logo-home.png)
- **Primary Color**: Blue (#1976d2) - Medical/trustworthy
- **Visual Style**: Professional, clean, accessible

## Data Formats
Follow Vietnamese date/time and number formats:
- **Date**: DD/MM/YYYY
- **Time**: 24-hour format (HH:mm)
- **Phone**: Vietnamese mobile format
- **Currency**: VND (Vietnamese Dong)

## Accessibility Considerations
- Support Vietnamese language input
- Consider users with varying technical skills
- Mobile-responsive for field use
- Clear visual hierarchy for medical data

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khanhdo137) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
