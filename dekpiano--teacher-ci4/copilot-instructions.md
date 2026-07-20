## teacher-ci4

> This document provides context and rules for the Gemini CLI to ensure that UX/UI design and other tasks are performed consistently and in accordance with project conventions.

# Gemini CLI Context and Rules

This document provides context and rules for the Gemini CLI to ensure that UX/UI design and other tasks are performed consistently and in accordance with project conventions.

## 1. Database Schema

The application uses a MySQL database named `skjacth_academic`. The following tables have been identified:

### `tb_register`

This table stores student registration information for subjects.

| Column | Type | Description |
|---|---|---|
| `StudentID` | varchar(17) | Student ID (17 characters) |
| `SubjectID` | int(10) | Subject ID |
| `Score100` | text | Score (out of 100) |
| `Grade` | varchar(3) | Grade |
| `RegisterYear` | varchar(6) | Semester/Academic Year |
| `RegisterClass` | varchar(10) | Class |
| `TeacherID` | varchar(20) | Teacher ID |
| `StudyTime` | text | Study Time |
| `Grade_Type` | varchar(20) | Grade Type |
| `RepeatStatus` | varchar(10) | Repeat Status |
| `RepeatYear` | varchar(6) | Repeat Year |
| `RepeatTeacher` | varchar(15) | Repeat Teacher |
| `RepeatConfirm` | varchar(20) | Repeat Confirm |
| `Grade_UpdateTime` | datetime | Grade Update Time |

### `tb_send_plan`

This table stores information about submitted lesson plans.

| Column | Type | Description |
|---|---|---|
| `seplan_ID` | int(5) | Plan ID |
| `seplan_namesubject` | varchar(50) | Subject Name |
| `seplan_coursecode` | varchar(10) | Course Code |
| `seplan_gradelevel` | varchar(2) | Grade Level |
| `seplan_typesubject` | varchar(10) | Subject Type |
| `seplan_typeplan` | varchar(30) | Plan Type |
| `seplan_sendcomment` | text | Send Comment |
| `seplan_createdate` | datetime | Create Date |
| `seplan_usersend` | varchar(20) | User Send |
| `seplan_learning` | varchar(15) | Learning |
| `seplan_inspector1` | varchar(60) | Inspector 1 |
| `seplan_comment1` | text | Comment 1 |
| `seplan_inspector2` | varchar(60) | Inspector 2 |
| `seplan_comment2` | text | Comment 2 |
| `seplan_status1` | varchar(30) | Status 1 |
| `seplan_status2` | varchar(30) | Status 2 |
| `seplan_year` | varchar(4) | Year |
| `seplan_term` | varchar(1) | Term |
| `seplan_file` | text | File |
| `seplan_checkdate1` | datetime | Check Date 1 |
| `seplan_checkdate2` | datetime | Check Date 2 |

### Other Tables

Based on the controllers and models, the following tables are also likely to exist:

- `tb_assessment`
- `tb_curriculum`
- `tb_homeroom`
- `tb_login`
- `tb_students`
- `tb_subjects`
- `tb_personnel`

## 2. Project Conventions

### Naming Conventions

- **Controllers:** PascalCase, with the suffix `Controller` (e.g., `CurriculumController.php`).
- **Models:** PascalCase, with the suffix `Model` (e.g., `CurriculumModel.php`).
- **Views:** Directories are lowercase. View files are lowercase with underscores (e.g., `welcome_message.php`).

### File Structure

The project follows the CodeIgniter 4 framework structure.

- `app/`: Contains the application's core code (Controllers, Models, Views, etc.).
- `public/`: The web server's document root.
- `writable/`: Directory for storing cache, logs, and other writable data.
- `tests/`: Contains the application's tests.
- `vendor/`: Composer dependencies.

## 3. User Roles

Based on the controllers and views, the application appears to have the following user roles:

- **Teacher:**  Can manage curriculum, homeroom, and assessments.
- **Admin/Principal:** Can manage teachers, students, and other system settings.
- **Student:** Can view their grades and other information.

## 4. UX/UI Design Rules

- **Template:** The application uses the **Sneat Bootstrap 5 HTML Admin Template**. All new UI components should be consistent with the Sneat style.
- **Responsiveness:** The UI should be responsive and work on all screen sizes.
- **Clarity:** The UI should be clear and easy to understand.
- **Consistency:** The UI should be consistent across the entire application.

## 5. CODEIGNITER 4 (CI4) INTEGRATION RULES

### Controllers

- **Base Controller:** All controllers must extend the `BaseController`.
- **Data Passing:** Pass data to views using the `$this->response->setJSON()` method for JSON responses or by passing an array to the `view()` helper.
- **Security:** Use the built-in security features of CodeIgniter, such as `esc()` for escaping output and CSRF protection.

### Models

- **Database Connection:** Use the `$this->db->table()` method to interact with the database.
- **Query Builder:** Use the Query Builder class for all database queries.
- **Data Validation:** Use the Validation class to validate all incoming data.

### Views

- **Template:** Use the **Sneat** template for all views. The main layout file is `app/Views/teacher/layout/main.php`.
- **Data Display:** Use the `esc()` function to escape all data that is displayed in the view.
- **Partials:** Use partials for reusable view components, such as headers, footers, and sidebars.

### Routes

- **RESTful Routes:** Use RESTful routes for all API endpoints.
- **Named Routes:** Use named routes for all web routes.
- **Route Groups:** Use route groups to organize related routes.

## 6. Sneat Bootstrap 5 HTML Admin Template - Design Principles

- **Bootstrap 5 Foundation:** Built on the latest Bootstrap 5 framework, ensuring a mobile-first, responsive, and modern component-based structure.
- **Modern & Clean Design:** Provides a clean, modern, and intuitive user interface that enhances user experience.
- **Responsiveness:** The template is fully responsive, adapting to various screen resolutions from mobile devices to large desktops.
- **Developer-Friendly:** The code is well-structured, making it easy to customize and extend. It includes a logical file and folder structure.
- **Customization:** Highly customizable using SASS variables for easy theming and style adjustments.
- **Rich Components:** Comes with a rich set of pre-built components, cards, forms, tables, and UI elements that can be readily used.
- **Iconography:** Primarily uses **Boxicons** for a consistent and rich set of icons. The project also integrates **Bootstrap Icons**.
- **Cross-Browser Compatibility:** Ensures consistent performance and appearance across modern web browsers.

---
> Source: [dekpiano/teacher_ci4](https://github.com/dekpiano/teacher_ci4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
