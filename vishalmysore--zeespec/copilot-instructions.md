## zeespec

> I have defined the system requirements across several dimensions. Please use this information to architect the database schema, design the API, and plan the implementation as requested.

# SYSTEM SPECIFICATION OVERVIEW

I have defined the system requirements across several dimensions. Please use this information to architect the database schema, design the API, and plan the implementation as requested.

## DIMENSION: WHAT

# WHAT — What is this system about?
Describe what your system is and what it manages. Write it like you're explaining it to a friend.
Think about: what are the main "things" in your system? What information do you track about each one?

## Your Description
[Replace this with your own description]

## Example
I want a library management system where users can check out and return books.
The main things in this system are:

- **Books** — each book has a title, an author, and a unique ID. A book can either be available or checked out.
- **Members** — people who are registered with the library. They have a name and a membership number.
- **Loans** — when a member checks out a book, that creates a loan. It tracks which member has which book, when they took it, and when it's due back.

Each book can only be with one person at a time. A member can borrow multiple books at once.


---

## DIMENSION: HOW

# HOW — How should this system be built?
Describe the technology choices, architecture, and any specific preferences you have.
Think about: what language or framework? What kind of database? Any tools or platforms you want to use?
You don't need to be technical — just describe it in plain English and the AI will figure out the details.

## Your Description
[Replace this with your own description]

## Example
The backend should be built with Spring Boot. It should expose a REST API that the frontend can talk to.
The frontend should be built in Node.js using React. It should be a simple web app that library staff can use from a browser.
For the database, I want to use PostgreSQL.
I'd like the project split into two folders — one for the backend and one for the frontend. Each should have its own setup instructions so a developer can get either one running independently.
I don't need anything fancy for deployment right now — just make sure it runs locally with a simple startup command.


---

## DIMENSION: WHO

# WHO — Who uses this system?
Describe the different types of people or systems that interact with your system.
Think about: who are the users? Do different users have different levels of access?
What can each type of person do — and what are they not allowed to do?

## Your Description
[Replace this with your own description]

## Example
There are two types of users:
- **Library Staff** — these are the people who work at the library. They can add new books, register new members, check books out on behalf of a member, and mark a book as returned. They can also see which books are currently checked out and by whom.
- **Members** — regular library members can log in to see which books they currently have checked out and when each one is due back. They cannot check books out themselves — that has to be done by staff.
- **Nobody** should be able to delete a book or a member record — only staff can mark a book as unavailable if it's lost or damaged.


---

## DIMENSION: WHEN

# WHEN — When does it do things automatically?
Describe things that happen without a user clicking a button.
Think about: schedule tasks, automatic notifications, or triggers based on other actions.

## Your Description
[Replace this with your own description]

## Example
- When a book is one day overdue, the system should automatically send an email to the member.
- Every Monday morning, a report of all overdue books should be emailed to the library manager.
- When a new book is added, send a notification to members who have expressed interest in that author.


---

## DIMENSION: WHERE

# WHERE — Where does it live?
Describe how people access it and what environments it runs in.
Think about: is it a web app? A mobile app? An internal tool? Where should it be hosted?

## Your Description
[Replace this with your own description]

## Example
- This is an internal web application for library staff and a public web portal for members.
- It should be optimized for desktop use first.
- The system should run on AWS using Docker containers.
- Staff should access it via a VPN-protected URL.


---

## DIMENSION: WHY

# WHY — Why does it exist?
Describe the goals, rules, and constraints that govern the system.
Think about: what problem are you solving? What are the most important business rules?

## Your Description
[Replace this with your own description]

## Example
- The goal is to reduce the time staff spend manually tracking book loans and sending reminders.
- **Rule**: A member cannot check out more than 5 books at a time.
- **Rule**: A member with more than $10 in late fees is blocked from checking out new books.
- **Goal**: The system should be able to handle up to 10,000 members and 50,000 books.


---

## DIMENSION: UPSTREAM

# UPSTREAM — What does it receive?
Describe the information or data coming into your system from other systems.
Think about: external APIs, manual data imports, or shared databases.

## Your Description
[Replace this with your own description]

## Example
- Book details are imported from the National Library API using the ISBN.
- User authentication is handled by the company's existing Google Workspace (SSO).
- Member address updates are synced from the City Records database nightly.


---

## DIMENSION: DOWNSTREAM

# DOWNSTREAM — What does it send?
Describe the information or data your system sends to other systems.
Think about: emails, reports, webhooks, or external data feeds.

## Your Description
[Replace this with your own description]

## Example
- Email notifications are sent via SendGrid.
- Daily financial summaries are pushed to the accounting system's API.
- Late fee payment records are sent to the payment gateway (Stripe).


---


# INSTRUCTIONS FOR THE AI AGENT
1. Analyze the dimensions provided above.
2. Identify any ambiguities or missing information and ask clarifying questions if necessary.
3. Generate a technical specification including database models, API contracts, and core logic patterns.
4. If requested, proceed to generate the implementation candidates.

---
> Source: [vishalmysore/zeespec](https://github.com/vishalmysore/zeespec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
