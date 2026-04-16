## sop-maker

> "sessionDuration": "7d",

{
  "auth": {
    "providers": {
      "firebase": {
        "primary": true,
        "sessionDuration": "7d",
        "roleMapping": {
          "admin": ["full_access"],
          "editor": ["create", "edit", "view"],
          "viewer": ["view"]
        }
      },
      "supabase": {
        "jwtIntegration": true,
        "rolePolicy": "inherit_from_firebase"
      }
    },
    "routes": {
      "public": ["/", "/login", "/signup", "/api/public/*"],
      "protected": ["/dashboard/*", "/sops/*", "/api/protected/*"],
      "admin": ["/admin/*", "/api/admin/*"]
    }
  },
  "database": {
    "supabase": {
      "schemas": {
        "public": {
          "tables": {
            "sop_templates": {
              "rls": true,
              "policies": {
                "select": "authenticated",
                "insert": "admin_or_editor",
                "update": "admin_or_editor",
                "delete": "admin"
              }
            },
            "user_sops": {
              "rls": true,
              "policies": {
                "select": "authenticated_or_owner",
                "insert": "authenticated",
                "update": "owner_or_editor",
                "delete": "owner_or_admin"
              }
            },
            "sop_steps": {
              "rls": true,
              "foreignKeys": ["sop_id"],
              "policies": {
                "select": "authenticated_or_owner",
                "insert": "authenticated",
                "update": "owner_or_editor",
                "delete": "owner_or_editor"
              }
            },
            "sop_media": {
              "rls": true,
              "foreignKeys": ["step_id"],
              "policies": {
                "select": "authenticated_or_owner",
                "insert": "authenticated",
                "update": "owner_or_editor",
                "delete": "owner_or_editor"
              }
            }
          }
        }
      }
    }
  },
  "api": {
    "rate_limits": {
      "public": "100/hour",
      "authenticated": "1000/hour",
      "admin": "5000/hour"
    },
    "cors": {
      "origins": ["https://yourdomain.com", "http://localhost:3000"],
      "methods": ["GET", "POST", "PUT", "DELETE"],
      "credentials": true
    }
  },
  "storage": {
    "firebase": {
      "buckets": {
        "sop-media": {
          "maxFileSize": "10MB",
          "allowedTypes": ["image/jpeg", "image/png", "image/gif", "video/mp4", "application/pdf"],
          "access": "authenticated"
        },
        "user-exports": {
          "maxFileSize": "50MB",
          "allowedTypes": ["application/pdf", "application/vnd.openxmlformats-officedocument.wordprocessingml.document"],
          "access": "owner"
        }
      }
    }
  },
  "ai": {
    "features": {
      "sop_generation": {
        "access": "authenticated",
        "rate_limit": "50/day",
        "max_tokens": 4000
      },
      "step_suggestions": {
        "access": "authenticated",
        "rate_limit": "200/day",
        "max_tokens": 1000
      }
    }
  },
  "export": {
    "formats": ["pdf", "docx", "html"],
    "branding": {
      "allowed": true,
      "position": "footer",
      "customizable": "premium_only"
    },
    "access_control": {
      "watermark": "free_tier",
      "password_protection": "premium_only",
      "expiry_links": "premium_only"
    }
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/durellwilson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
