## talktime-video-app

> This document outlines the core logic and features for the Talk Time application. It is critical that all development adheres strictly to these specifications to ensure data integrity, user privacy, and a seamless, professional user experience.


```markdown
# Talk Time: Application Logic & Feature Specification

## 1. Guiding Principles
This document outlines the core logic and features for the Talk Time application. It is critical that all development adheres strictly to these specifications to ensure data integrity, user privacy, and a seamless, professional user experience.

* **Architecture**: The application must be 100% self-contained in Docker.
* **Methodology**: Follow API-first development practices.
* **Design**: The UI/UX must be modern, sleek, easy to use, and professional, enhance it with apple opacity and glassmorphism but do not overwhelm. Avoid too much gradient colors but use them where necessarry.

---
## 2. User Roles & Privileges
There are three primary user roles: **Admin**, **Volunteer**, and **Student**.

### 2.1. Admin
**Authentication:**
* **Signup**: Admins sign up using their full name, a password, and a 6-digit secret code.
* **Secret Code**: This code is generated once, saved to the database, and must also be stored in a `.env` file for manual verification. The signup form will require this manually-entered secret code for an admin account to be created.
* **Admin URLs**: Admin-related pages must be namespaced under `/talktime/admin/`, for example:
    * `/talktime/admin/signup`
    * `/talktime/admin/login`
    * `/talktime/admin/dashboard`

**Privileges:**
* Admins have 100% superuser privileges over the entire application.
* They can perform all **CRUD** (Create, Read, Update, Delete) operations on any data, including volunteer profiles, student profiles, meeting schedules, and system settings.
* The admin dashboard must provide tools to track new volunteers, view call statistics (successful, missed, canceled), and access full data for all users.

### 2.2. Volunteer
**Privileges:**
Volunteers have standard privileges as currently implemented in the project. Their primary function is to browse the student list and schedule meetings.

**Authentication & Session Management:**
* When a volunteer logs in, the main landing page navigation bar must be dynamically updated. All "Signup" and "Login" buttons should be removed.
* In their place, a profile image should appear with a dropdown menu containing "My Profile" and "Logout" options.
* Any prominent call-to-action buttons on the homepage (e.g., "Let's Get Started") should direct a logged-in volunteer to the student list page.
* If a logged-in volunteer attempts to access a signup or login URL directly, they must be redirected to the main index page.

### 2.3. Student
**Privileges:**
* Students have the most limited privileges.
* Everything on their dashboard is read-only, except for actions explicitly related to joining a scheduled meeting.
* **Call Limits**: Students are limited to a maximum of one call per day.

---
## 3. Core Feature: Scheduling System
The scheduling system is the most critical part of the application and requires careful implementation.

### 3.1. Student List & Availability
**Two-State System**: The list of students available for volunteers to schedule a meeting with must be presented in two distinct states:
* **Available Students**: These are students who can be scheduled for a meeting on the current day or any future day where they do not already have a meeting booked.
* **Scheduled/Unavailable Students**: These are students who have an upcoming meeting scheduled. They cannot be booked for another meeting until their existing one is completed.

**Student Details**: When a volunteer clicks on an available student from the list, they must be taken to a "Student Details" page. This page will display all of the student's data and have only one button: "Schedule Meeting".

### 3.2. Scheduling Rules & Time Zones
**Volunteer Scheduling Window:**
* Volunteers can only schedule meetings within a **3-month time limit** from the current date.
* This functionality must be dynamic. The system should calculate the current date and only allow scheduling up to the end of the third month ahead. For example, if today is January 15th, the volunteer can schedule meetings up to April 30th.
* **Note**: This is a change from the previously mentioned 6-month window. The 3-month limit is the correct specification.

**Time Zone Synchronization (Critical):**
* The scheduler page must feature a 100% functional time zone picker.
* This picker must sync appointments based on the volunteer's local time zone while ensuring the time selected translates to a valid time slot for the student in Kenya (EAT).

**Student Time Constraints:**
* Volunteers can only book time slots that fall between **9:00 AM and 5:00 PM EAT**.
* The time picker must prevent the selection of invalid times and should only offer minute breakpoints of `00`, `15`, `30`, and `45` to simplify the interface.

---
## 4. The Meeting Lifecycle
### 4.1. Notifications
A robust, automated notification system is essential.

**Sequential Notifications**: Once a meeting is scheduled, both the volunteer and the student must receive automated notifications at the following intervals before the meeting:
* 1 hour before.
* 30 minutes before.
* 5 minutes before.

### 4.2. Pre-Meeting Lobby
When there are 5 minutes left until the meeting starts, both the volunteer and the student should be automatically directed to the video conference page. The video call must not start immediately. Users will be in a waiting "lobby" state where:
* They can see themselves in their video panel.
* The other participant's panel will display a story or information about that person (e.g., the volunteer sees a story about the student, and the student sees a story about the volunteer).
* A countdown timer will be prominently displayed, counting down to the exact meeting start time (00:00).

### 4.3. In-Meeting Experience
**Call Initiation**: The video and audio connection will initiate for both users only when the countdown timer hits `00:00`.

**Session Timer**:
* A session timer will remain at `00:00` until the connection is successfully established and both users can see each other.
* Once the connection is successful, the timer will begin counting down from the maximum time allocated for the meeting (as set by an Admin).

**5-Minute Warning**: When the countdown timer reaches 5 minutes remaining, a toast notification must appear on both users' video screens with the message: "5 minutes left. Please wrap up!"

**Call Termination**: When the timer reaches zero, the call must be automatically terminated, and both users should be redirected to their respective dashboards.

### 4.4. Post-Meeting Actions
**Volunteer Rating**: After being redirected to their dashboard, the volunteer must be presented with a pop-up modal to rate the student and the interaction. The rating system must be flexible, allowing the volunteer to submit:
* Only a star rating.
* Only text feedback.
* Both a star rating and text feedback.

**Rescheduling Prompt**: Within the same pop-up, the volunteer should be asked if they would like to have another session with the same student. They should be given an option to schedule a new meeting for a different day right from that prompt.

---
## 5. Volunteer & Student Recognition
### 5.1. Volunteer Abroad Program
For students who also volunteer ("Students from Abroad"), the system must be able to generate a confirmation document. This document will certify their participation in community development through the Talk Time program. It must be designed to look official and contain all necessary information to be considered a valid document by schools and other institutions.

### 5.2. Badges & Gamification
A badge system will be implemented to encourage participation.
* Volunteers who complete 10 or more calls in a month will earn a special badge on their profile.
* More badge tiers and criteria can be implemented later.

### 5.3. Rating System
**Volunteer Rating (Automatic)**: A volunteer's rating will be automatically calculated based on metrics such as:
* Frequency of meetings.
* Number of successfully completed calls.
* Number of canceled meetings.

**Student Rating (Manual)**: A student's rating will be based on the real feedback and star ratings provided by volunteers after each successful meeting.
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nellylemmy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
