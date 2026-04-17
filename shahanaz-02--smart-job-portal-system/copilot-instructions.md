## smart-job-portal-system

> This project is built **strictly** according to the "Master Prompt" documentation. Every endpoint, file path, and test case maps 1:1 to the plan.

# Smart Job Portal System - Project Master Context

This project is built **strictly** according to the "Master Prompt" documentation. Every endpoint, file path, and test case maps 1:1 to the plan.

## Project Information
- **App Name:** jobportalsystem
- **Base Package:** `com.edutech.jobportalsystem`
- **Spring Boot Version:** 3.2.0
- **Java Version:** 17
- **Database:** MySQL (jobportal_db)

## Core Functional Requirements
1. **User Registration & Profile Management:** Secure accounts and role-based profiles.
2. **Job Management:** Recruiters can CRUD job postings.
3. **Job Search & Filtering:** Job seekers filter by skills, location, etc.
4. **Application Management:** Applicants apply; recruiters manage status.
5. **Resume Management:** Binary upload (LONGBLOB) for resumes.
6. **Role-Based Authentication:** ADMIN, RECRUITER, JOB_SEEKER.
7. **JWT Authorization:** Stateless secure API communication.

## Detailed Backend File Map (Strict - within /backend)
- `backend/src/main/java/com/edutech/jobportalsystem/config/SecurityConfig.java`
- `backend/src/main/java/com/edutech/jobportalsystem/controller/AdminController.java`
- `backend/src/main/java/com/edutech/jobportalsystem/controller/RecruiterController.java`
- `backend/src/main/java/com/edutech/jobportalsystem/controller/JobSeekerController.java`
- `backend/src/main/java/com/edutech/jobportalsystem/controller/AuthController.java`
- `backend/src/main/java/com/edutech/jobportalsystem/entity/User.java`
- `backend/src/main/java/com/edutech/jobportalsystem/entity/Job.java`
- `backend/src/main/java/com/edutech/jobportalsystem/entity/Application.java`
- `backend/src/main/java/com/edutech/jobportalsystem/entity/Resume.java`
- `backend/src/main/java/com/edutech/jobportalsystem/jwt/JwtRequestFilter.java`
- `backend/src/main/java/com/edutech/jobportalsystem/jwt/JwtUtil.java`
- `backend/src/main/java/com/edutech/jobportalsystem/repository/UserRepository.java`
- `backend/src/main/java/com/edutech/jobportalsystem/repository/JobRepository.java`
- `backend/src/main/java/com/edutech/jobportalsystem/repository/ApplicationRepository.java`
- `backend/src/main/java/com/edutech/jobportalsystem/repository/ResumeRepository.java`
- `backend/src/main/java/com/edutech/jobportalsystem/service/UserService.java`
- `backend/src/main/java/com/edutech/jobportalsystem/service/JobService.java`
- `backend/src/main/java/com/edutech/jobportalsystem/service/ApplicationService.java`
- `backend/src/main/java/com/edutech/jobportalsystem/service/ResumeService.java`

## Detailed Frontend File Map (Completed ✅)
- `frontend/src/app/login/login.component.[ts|html]` - Professional Login UI
- `frontend/src/app/registration/registration.component.[ts|html]` - User Registration
- `frontend/src/app/job-list/job-list.component.[ts|html]` - Job Search & Admin Dashboard
- `frontend/src/app/post-job/post-job.component.[ts|html]` - Recruiter Job Management (CRUD)
- `frontend/src/app/applications/applications.component.[ts|html]` - Status Management
- `frontend/src/app/resume/resume.component.[ts|html]` - Resume Upload
- `frontend/src/services/http.service.ts` - Centralized API Service
- `frontend/src/services/auth.service.ts` - JWT Session Management

## Technical Constraints (Strict)
- **JWT Secret:** `smartjobportalsecretkey2024`
- **JWT Expiration:** `86400000` (24 hours)
- **Multipart Limit:** `5MB`
- **Hibernate Dialect:** `org.hibernate.dialect.MySQL8Dialect`
- **Auth Authorities:** Roles must be prefixed with `ROLE_` (e.g., `ROLE_RECRUITER`).

## Frontend Enhancements Added
- **Global CSS:** Professional UI with consistent spacing, typography, and interactive feedback.
- **Admin Features:** Integrated User and Job management within `job-list`.
- **Recruiter CRUD:** Full Create, Read, Update, and Delete capabilities for job postings.
- **Role-Based UI:** Automatic navigation and component adjustment based on user role.
- **Responsive Layout:** Mobile-friendly container-based design.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shahanaz-02) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
