## personal-website

> "name": "Page Titles",

{
  "rules": [
    {
      "name": "Page Titles",
      "description": "Every page must have a proper title in the format 'Page Title | Shreyas Prakash' (except home page which is just 'Shreyas Prakash')",
      "patterns": {
        "src/pages/**/*.astro": {
          "title": "<BaseLayout title=\"{pageName} | Shreyas Prakash\">"
        },
        "src/pages/index.astro": {
          "title": "<BaseLayout title=\"Shreyas Prakash\">"
        },
        "src/layouts/**/*.astro": {
          "postTitle": "<BaseLayout title={`${post.data.title} | Shreyas Prakash`}>"
        }
      }
    },
    {
      "name": "Newsletter Components",
      "description": "Rules for newsletter form components and their usage",
      "patterns": {
        "src/components/NewsletterForm.astro": {
          "baseForm": "Basic newsletter form without headers, used on homepage",
          "containerRules": "Must use max-w-2xl mx-auto px-4 for consistent container width",
          "structure": [
            "form element with flex layout",
            "email input with dark mode support",
            "submit button with hover states",
            "convertkit script inclusion"
          ]
        },
        "src/components/NewsletterFormWithHeaders.astro": {
          "usage": "Only used at the end of blog posts, not on homepage",
          "headers": {
            "title": "Invite me to your inbox?",
            "subtitle": "2-3 ideas every month across design and tech"
          },
          "alignment": "Center aligned text",
          "spacing": "mt-12 mb-24 for consistent vertical rhythm",
          "imports": "Must import NewsletterForm component"
        }
      },
      "containerWidth": {
        "rule": "All newsletter components must stay within max-w-3xl container",
        "padding": "Must include px-4 for consistent side spacing"
      }
    },
    {
      "name": "Post Tag Display",
      "description": "Rules for displaying post tags",
      "patterns": {
        "src/pages/**/*.astro": {
          "tagDisplay": {
            "maxTags": 1,
            "excludeTags": ["rough-notes"],
            "description": "Show only one tag per post, excluding 'rough-notes' tag"
          }
        }
      }
    }
  ],
  "titleFormats": {
    "home": "Shreyas",
    "post": "{postTitle} | Shreyas",
    "tag": "{tagName} | Shreyas",
    "page": "{pageName} | Shreyas"
  }
} 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shreyas-makes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
