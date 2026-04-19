## slick-solutions-mvp

> "language": "typescript",

{
  "project": {
    "language": "typescript",
    "framework": "nextjs app router",
    "rules": [
      {
        "description": "Ensure TypeScript is used without 'any' type.",
        "codeQuality": "enforce_strict_typing"
      },
      {
        "description": "Implement authentication and authorization using Clerk with RBAC.",
        "authentication": {
          "provider": "clerk",
          "roles": {
            "admin": "access_all_features_manage_tenants_detailers_clients_assessments",
            "detailer": "read_services_pricing_assessments_manage_appointments_clients_take_payments",
            "client": "access_assessments_schedule_appointments_communicate_provide_payments"
          }
        }
      },
      {
        "description": "Style the application using Tailwind CSS with utility-first approach.",
        "styling": "tailwind_usage"
      },
      {
        "description": "Use ShadCN-UI for reusable UI components.",
        "ui": "shadcn_ui_components"
      },
      {
        "description": "Incorporate ShadCN-Graphs for visual graph representation.",
        "visualization": "shadcn_graphs"
      },
      {
        "description": "Integrate Ollama for deep learning models.",
        "ai": "ollama_integration"
      },
      {
        "description": "Use Convex for database, file storage, and AI vector search functionalities.",
        "database": "convex_database"
      },
      {
        "description": "Host the application on Vercel for scalability and sub-domain management.",
        "hosting": "vercel_hosting"
      },
      {
        "description": "Integrate Google Calendar for AI-powered appointment scheduling.",
        "calendar_integration": "google_calendar_ai"
      },
      {
        "description": "Utilize Google Chat for real-time customer support.",
        "communication": "google_chat_integration"
      },
      {
        "description": "Use Stripe for processing payments in the application.",
        "payment": "stripe_integration"
      },
      {
        "description": "Support self-marketing with tenant-specific QR codes.",
        "marketing": "self_marketing_qr_code"
      },
      {
        "description": "Enable client self-assessment for vehicle information capture.",
        "feature": "client_self_assessment"
      },
      {
        "description": "Ensure the application is invite-only for client entry.",
        "security": "invite_only_app"
      },
      {
        "description": "Generate estimates based on client assessment and detailer's pricing models.",
        "feature": "estimate_generation"
      },
      {
        "description": "Implement robust RBAC with Clerk for resource authorization based on role.",
        "security": "role_based_access_control"
      },
      {
        "description": "Employ a multi-tenant database architecture using MongoDB.",
        "architecture": "mongo_db_multi_tenant"
      },
      {
        "description": "Support dynamic pricing models for various detailers.",
        "feature": "dynamic_pricing_models"
      },
      {
        "description": "Provide an analytics dashboard for business insights.",
        "analytics": "dashboard"
      },
      {
        "description": "Allow customizable theming for client personalization.",
        "ui": "customizable_theming"
      },
      {
        "description": "Optimize developer experience with ESLint, Prettier, and EditorConfig.",
        "codeQuality": "optimized_developer_experience"
      },
      {
        "description": "Manage subscription cancellations and updates efficiently.",
        "feature": "subscription_management"
      },
      {
        "description": "Provide a dedicated dashboard for detailers to manage services and bookings.",
        "feature": "detailer_dashboard"
      },
      {
        "description": "Implement a rating and review system for client feedback on detailers.",
        "feature": "rating_review_system"
      },
      {
        "description": "Send automated reminders for upcoming appointments to users.",
        "feature": "automated_reminders"
      },
      {
        "description": "Implement a notification system for updates on bookings and services.",
        "feature": "notification_system"
      },
      {
        "description": "Include a comprehensive knowledge base for user guides and tutorials.",
        "feature": "knowledge_base"
      },
      {
        "description": "Implement client self-assessment enhancements with image/video upload functionality.",
        "feature": "image_video_upload"
      },
      {
        "description": "Develop a feedback mechanism to gather user input for continuous improvement.",
        "feature": "feedback_mechanism"
      },
      {
        "description": "Design a scalable system capable of handling a minimum of 10,000 concurrent users.",
        "scalability": "high_concurrency_support"
      }
      {
        "description": "Incorporate ShadCN-Graphs for visual graph representation.",
        "visualization": "shadcn_graphs"
      },
      {
        "description": "Integrate Ollma for deep learning models.",
        "ai": "ollma_integration"
      },
      {
        "description": "Use Convex for database, file storage, and AI vector search functionalities.",
        "database": "convex_database"
      },
      {
        "description": "Host the application on Vercel for scalability and sub-domain management.",
        "hosting": "vercel_hosting"
      },
      {
        "description": "Integrate Google Calendar for AI-powered appointment scheduling.",
        "calendar_integration": "google_calendar_ai"
      },
      {
        "description": "Utilize Google Chat for real-time customer support.",
        "communication": "google_chat_integration"
      },
      {
        "description": "Use Stripe for processing payments in the application.",
        "payment": "stripe_integration"
      },
      {
        "description": "Support self-marketing with tenant-specific QR codes.",
        "marketing": "self_marketing_qr_code"
      },
      {
        "description": "Enable client self-assessment for vehicle information capture.",
        "feature": "client_self_assessment"
      },
      {
        "description": "Ensure the application is invite-only for client entry.",
        "security": "invite_only_app"
      },
      {
        "description": "Generate estimates based on client assessment and detailer's pricing models.",
        "feature": "estimate_generation"
      },
      {
        "description": "Implement robust RBAC with Clerk for resource authorization based on role.",
        "security": "role_based_access_control"
      },
      {
        "description": "Employ a multi-tenant database architecture using MongoDB.",
        "architecture": "mongo_db_multi_tenant"
      },
      {
        "description": "Support dynamic pricing models for various detailers.",
        "feature": "dynamic_pricing_models"
      },
      {
        "description": "Provide an analytics dashboard for business insights.",
        "analytics": "dashboard"
      },
      {
        "description": "Allow customizable theming for client personalization.",
        "ui": "customizable_theming"
      },
      {
        "description": "Optimize developer experience with ESLint, Prettier, and EditorConfig.",
        "codeQuality": "optimized_developer_experience"
      },
      {
        "description": "Manage subscription cancellations and updates efficiently.",
        "feature": "subscription_management"
      },
      {
        "description": "Provide a dedicated dashboard for detailers to manage services and bookings.",
        "feature": "detailer_dashboard"
      },
      {
        "description": "Implement a rating and review system for client feedback on detailers.",
        "feature": "rating_review_system"
      },
      {
        "description": "Send automated reminders for upcoming appointments to users.",
        "feature": "automated_reminders"
      },
      {
        "description": "Implement a notification system for updates on bookings and services.",
        "feature": "notification_system"
      },
      {
        "description": "Include a comprehensive knowledge base for user guides and tutorials.",
        "feature": "knowledge_base"
      };
    ]
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Holding-1-at-a-time) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
