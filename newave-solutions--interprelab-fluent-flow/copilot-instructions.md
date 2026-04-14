## interprelab-fluent-flow

> This document provides the core business, product, and technical architecture knowledge for the InterpreLab project. Use this as the single source of truth when generating code, content, or strategic recommendations.

# InterpreLab Project Knowledge Base for Gemini

This document provides the core business, product, and technical architecture knowledge for the InterpreLab project. Use this as the single source of truth when generating code, content, or strategic recommendations.

## 1. Business & Product Knowledge

### Core Identity & Mission

- **Project Name:** **InterpreLab**

- **Elevator Pitch:** InterpreLab is a cutting-edge, AI-driven training and real-time assistance platform for medical interpreters, focused on human skill optimization and bridging critical communication gaps in healthcare.

- **Founder Profile:** The founder is a seasoned expert in the medical interpreting field and an early adopter/developer of AI-driven, agentic technologies. The focus is on continuous innovation of backend AI services.

### Products & Services

#### InterpreBot 🤖 (AI Training & Assessment)

- **Function:** Provides realistic, interactive linguistic assessments to evaluate and hone interpreters' core skills with deep grammatical and contextual analysis.

- **Key Features:**

  - Deep analysis of linguistic accuracy, terminology, and **grammatical correctness (tense, syntax)**.
  - Generates a detailed performance dashboard pinpointing "weak areas of opportunity".
  - Acts as an AI Mentor to guide users through **Customized Learning Paths** based on their assessment results.

#### InterpreCoach 🎧 (Real-Time AI Assistance Browser Extension)

- **Function:** A multi-modal, real-time AI assistant that integrates discreetly into existing interpreter platforms via a browser extension.

- **Key Features:**

  - **Advanced Terminology Management:** Provides real-time translations with additional context, **relevant images**, and for medications, lists the **generic name, brand name, and aliases**.
  - **Acoustic & Voice Training:** Utilizes backend speech regulator agents to analyze and assist with voice softness, deepness, pitch, and speed.
  - **Key Insights & Summarization:** Actively listens and summarizes critical points of the conversation (medication instructions, reason for encounter) into concise bullet points on the user's display.
  - **Predictive Assistance:** Infers conversational context to proactively prepare relevant vocabulary and resources for faster insights.

#### Certification-Ready Training Courses 🎓

- **Offering:** 40 to 60-hour Healthcare Medical Interpreter Training Courses.

- **Accreditation:** Approved by **NBCMI** and **CCHI** as prerequisite courses for their written certification exams.

#### Interpreter Community & Resources (InterpreLinks) 🤝

- **Function:** A dedicated social web application and professional network for interpreters.

- **Features:** Community forums, job boards, and a curated library of resources (videos, mock scenarios, dictionaries, legal references).

### Target Audience

- **Primary:** Individual medical interpreters (certified professionals, students, and those preparing for certification).

- **Secondary:** Language Service Companies (LSCs) for training and quality assurance.

- **Tertiary:** Healthcare institutions.

### Brand Voice & Tone

- **Professional, Authoritative, Innovative, Empowering, Supportive, Precise, Transparent.**

---

## 2. Technical & Architectural Directives

### Frontend Code Generation

- **Tech Stack:** **React** with **Next.js** (App Router). Code must be in **TypeScript**.

- **Styling:** **Tailwind CSS** exclusively. Use utility-first classes.

- **Architecture:** Modular, component-based structure (e.g., `HeroSection.tsx`, `FeaturesSection.tsx`). Components must be reusable.

- **Accessibility:** Must adhere to **WCAG 2.1 AA** standards (semantic HTML, alt tags, keyboard navigation).

### Backend & Google Cloud Integration

- **Philosophy:** **Serverless-first architecture.**

- **Core Services:**

  - **Authentication:** **Firebase Authentication**.
  - **Database:** **Cloud Firestore** (NoSQL best practices).
  - **Backend Logic:** **Cloud Functions (2nd Gen)** using Python or Node.js.
  - **AI/ML APIs:** Integrate with **Speech-to-Text API** and **Natural Language API**.
  - **File Storage:** **Google Cloud Storage** (use signed URLs for security).

- **API Design:** **RESTful principles**. Route all Cloud Functions through **API Gateway**.

- **Security:** **NEVER** hardcode secrets. Use **Secret Manager** or environment variables.

### Deployment & DevOps (CI/CD)

- **Hosting:**

  - **Frontend:** **Firebase Hosting**.
  - **Backend:** **Cloud Functions** or **Cloud Run** (if containerized).

- **CI/CD:** Use **Google Cloud Build**. Generate a `cloudbuild.yaml` for automating builds and deployments.

- **Containerization:** For complex services, provide a `Dockerfile` for deployment to **Cloud Run**.

- **Domain Guidance:** When asked, provide steps to connect the `interprelab.com` domain to Firebase Hosting by updating DNS records (A and TXT) at the domain registrar.

### Content & SEO Directives

- **Target Persona:** "The Aspiring/Certified Medical Interpreter".

  - **Pain Points:** Career stagnation, certification pressure, lack of modern tools, complex terminology.
  - **Goals:** Skill improvement, career advancement, higher accuracy, better patient outcomes.

- **Primary Keywords:** "medical interpreter training", "AI for interpreters", "NBCMI prerequisite course", "CCHI approved training", "interpreter skills assessment", "InterpreCoach", "InterpreBot".

- **Content Structure:** Use clear headings (H1, H2, H3). Use lists for features. Always include a strong Call to Action (CTA) like "Start Your Free Assessment" or "Enroll Now".

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/newave-solutions)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/newave-solutions)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
