## crm

> Provides a personalized overview of work in progress.

# Sales Representative User Journey

This document outlines the full journey of a **Sales Representative** using the Doctor CRM application. It describes each screen and interaction in detail to guide the frontend and backend implementation teams.

---

## 🧑‍💼 User Role: Sales Representative

### Primary Responsibilities:

* Manage and convert assigned doctor leads
* Log and track interactions with leads
* Use AI tools to aid in lead conversion
* Stay on top of personal tasks

---

## 🔐 1. Login Page

### Screen Elements:

* **Form Fields:**

  * `Username` (text input)
  * `Password` (password input)
* **Buttons:**

  * `Login` (submit button)
* **Validation:**

  * Show error if login fails

### Behavior:

* On successful login, redirect to Sales Rep Dashboard
* No "Forgot Password" or MFA in MVP

---

## 🏠 2. Dashboard (Sales Rep View)

### Purpose:

Provides a personalized overview of work in progress.

### Sections:

#### A. "My Tasks" List

* **Columns:** Task Type, Lead Name, Due Date/Time, Priority
* **Actions:** Click to view full task detail (modal or redirect to lead)

#### B. "Leads by Status" Chart

* **Pie Chart** showing number of assigned leads grouped by lead status
* **Interactive:** Clicking a slice filters the leads table

#### C. "Daily Activities" Chart

* **Bar chart** showing number of calls/emails/meetings logged each day (past 7 days)

### Nav Items:

* Dashboard
* Leads
* Reports (limited)
* Profile (logout)

---

## ➕ 3. Create a New Lead

### Access:

* Navigation bar → "Leads" → `+ New Lead` button

### Form Fields:

All fields optional (but validation should still apply):

* Doctor Name
* Clinic Name
* Specialty
* Primary Contact Number (allow Indian formats)
* Email
* City
* Source of Lead (dropdown)
* Initial Notes
* Years of Experience
* Clinic Type (enum)
* Preferred Communication Channel (enum)
* Estimated Patient Volume
* Existing EMR/App Usage (boolean)
* Specific Pain Points
* Referral Source

### Behavior:

* On submit, lead is created and assigned to the Sales Rep
* Redirect to Lead Profile View

---

## 📋 4. View Assigned Leads

### Access:

* Navigation bar → Leads

### Table Columns:

* Doctor Name
* Clinic Name
* Specialty
* City
* Lead Status
* Last Activity Date
* Assigned Rep (read-only: self)

### Functionality:

* Search by name/clinic
* Sort by status/date
* Filter by status

### Row Actions:

* Click row → navigate to Lead Profile View

---

## 🧾 5. Lead Profile View

### Layout:

Split into scrollable sections:

#### A. Contact Info Panel

* Static info: doctor, clinic, phone, email, city, etc.

#### B. Lead Status

* **Dropdown** with allowed statuses (based on transition rules)
* Show **status level** (1 to 6)
* If marking as "Lost", show "Reason for Loss" dropdown (mandatory)

#### C. Activity Log Tab

* Reverse chronological log of all interactions
* Each entry shows: Type, Date/Time, Outcome, Summary, Full Notes

#### D. Notes Section

* Free-form field to add general lead notes (not activity-specific)

#### E. AI Panels

* **Deal Coach Advice Panel**: Button to trigger LLM call, displays response
* **Customer Persona Panel**: Button to generate persona using past activities
* **Objection Handler Panel**: Text input to paste objection + display 3 AI-suggested responses

#### F. Task View

* List of tasks for the current lead assigned to the user
* Options to create new task (see below)

---

## 🗣️ 6. Log Activity

### Trigger:

* Button: `+ Log Activity`

### Form Fields:

* Activity Type: Call, Email, WhatsApp, In-person
* Date/Time: datetime picker (required)
* Duration: optional for Calls
* Outcome: dropdown (e.g., Successful, Unsuccessful)
* Summary: short text (required)
* Full Notes: optional multiline

### Behavior:

* Appears in activity log
* Updates "Last Activity" date in lead table

---

## ✅ 7. Lead Status Update

### Options:

* Dropdown list of allowed statuses
* Transitions must follow:

  * Forward-only by level (e.g., L1 → L2)
  * Within-level status changes allowed
  * Only allowed to mark as "Lost" from specific prior states

### UI Feedback:

* If invalid transition: disable option or show validation error
* Reason for Loss required if status is "Lost"

---

## 📅 8. Task Management

### Access:

* From dashboard “My Tasks”
* From lead profile (linked tasks)

### Create Task Fields:

* Task Type: dropdown
* Due Date: date picker
* Due Time: time picker
* Description: required multiline
* Priority: High/Medium/Low
* Assigned To: auto-filled as current user

### List View:

* Tasks sorted by due date
* Show overdue indicators (e.g., red color)
* Status: no completion toggle in MVP

---

## 🧠 9. AI Tool Interactions

### On Lead Profile:

#### A. Deal Coach AI

* Button: “Get AI Coach Advice”
* Displays recommendations like "Schedule a demo", "Follow-up via WhatsApp"

#### B. Customer Persona Builder

* Button: “Generate Customer Persona”
* Output fields: Communication Style, Tech Adoption, Pain Points, etc.
* Fallback message for sparse data

#### C. Objection Handler

* Textbox to paste objection
* Response: 3 bullet points with AI-suggested responses

---

## 🚪 10. Logout

### Access:

* From profile dropdown menu → `Logout`

### Behavior:

* Ends session
* Redirects to login screen

---

## ✅ Permissions Recap

| Feature            | Sales Rep Access      |
| ------------------ | --------------------- |
| Create/Edit Leads  | ✅ Yes (assigned only) |
| Change Lead Status | ✅ Yes (assigned only) |
| Log Activities     | ✅ Yes (assigned only) |
| Use AI Tools       | ✅ Yes (assigned only) |
| Create/View Tasks  | ✅ Yes (self only)     |
| View Reports       | ✅ Limited (self)      |
| Assign Leads/Tasks | ❌ No                  |
| Manage Users       | ❌ No                  |
| Upload CSV Leads   | ❌ No                  |

---

## 💡 Summary

This user journey ensures that Sales Reps:

* Have a focused interface showing only their leads
* Can log and track all meaningful interactions
* Can use AI to improve conversion outcomes
* Stay organized with tasks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alhassan069) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
