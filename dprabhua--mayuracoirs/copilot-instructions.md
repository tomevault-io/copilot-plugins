## mayuracoirs

> "required_meta_tags": [

{
    "version": "1.0",
    "rules": {
        "seo": {
            "required_meta_tags": [
                "viewport",
                "description",
                "keywords",
                "author",
                "robots"
            ],
            "required_schema_markup": [
                "Organization",
                "WebPage",
                "BreadcrumbList"
            ],
            "required_alt_tags": true,
            "required_canonical_urls": true,
            "required_social_meta": [
                "og:title",
                "og:description",
                "og:image",
                "twitter:card",
                "twitter:title",
                "twitter:description",
                "twitter:image"
            ]
        },
        "security": {
            "required_headers": [
                "Content-Security-Policy",
                "X-Frame-Options",
                "X-Content-Type-Options",
                "Referrer-Policy",
                "Permissions-Policy"
            ],
            "forbidden_patterns": [
                "api_key",
                "password",
                "secret",
                "token"
            ],
            "required_ssl": true,
            "required_csrf_protection": true,
            "required_sanitization": true
        },
        "performance": {
            "max_image_size_kb": 200,
            "required_lazy_loading": true,
            "required_resource_hints": [
                "preconnect",
                "preload",
                "prefetch"
            ],
            "max_css_size_kb": 50,
            "max_js_size_kb": 100,
            "required_compression": true
        },
        "accessibility": {
            "required_aria_labels": true,
            "required_alt_text": true,
            "required_skip_links": true,
            "required_color_contrast": true,
            "required_keyboard_navigation": true
        },
        "code_quality": {
            "required_semantic_html": true,
            "required_responsive_design": true,
            "required_mobile_first": true,
            "required_clean_urls": true,
            "required_error_handling": true
        },
        "content": {
            "required_meta_description_length": {
                "min": 120,
                "max": 160
            },
            "required_title_length": {
                "min": 30,
                "max": 60
            },
            "required_heading_structure": true,
            "required_image_optimization": true
        },
        "analytics": {
            "required_privacy_policy": true,
            "required_cookie_consent": true,
            "required_analytics_implementation": true,
            "required_event_tracking": true
        },
        "maintenance": {
            "required_sitemap": true,
            "required_robots_txt": true,
            "required_error_pages": true,
            "required_backup_strategy": true
        },
        "enforcement": {
            "theme": {
                "required_colors": {
                    "primary": "#2e7d32",
                    "secondary": "#1b5e20",
                    "accent": "#4caf50",
                    "background": "#ffffff",
                    "text": "#333333"
                },
                "required_fonts": {
                    "primary": "Poppins",
                    "weights": [300, 400, 500, 600, 700]
                },
                "required_spacing": {
                    "container_padding": "1rem",
                    "section_margin": "2rem",
                    "element_gap": "1rem"
                }
            },
            "layout": {
                "required_structure": [
                    "header",
                    "main",
                    "footer"
                ],
                "required_components": [
                    "navbar",
                    "page-header",
                    "container",
                    "footer-grid"
                ],
                "required_responsive_breakpoints": {
                    "mobile": "320px",
                    "tablet": "768px",
                    "desktop": "1024px"
                }
            },
            "components": {
                "required_classes": {
                    "container": "container",
                    "section": "section",
                    "button": "btn",
                    "card": "card",
                    "form": "form"
                },
                "required_attributes": {
                    "images": ["alt", "width", "height", "loading"],
                    "links": ["href", "aria-label"],
                    "buttons": ["type", "aria-label"]
                }
            },
            "patterns": {
                "required_naming": {
                    "files": "kebab-case",
                    "classes": "kebab-case",
                    "ids": "camelCase"
                },
                "required_organization": {
                    "css": {
                        "base": "styles.css",
                        "components": "components/",
                        "utilities": "utilities/"
                    },
                    "js": {
                        "main": "script.js",
                        "modules": "src/js/"
                    },
                    "images": {
                        "root": "images/",
                        "formats": ["webp", "png", "jpg"]
                    }
                }
            },
            "documentation": {
                "required_comments": {
                    "sections": true,
                    "components": true,
                    "functions": true
                },
                "required_readme": {
                    "setup": true,
                    "structure": true,
                    "components": true
                }
            }
        }
    },
    "file_patterns": {
        "html": {
            "required_doctype": true,
            "required_lang_attribute": true,
            "required_charset": true,
            "required_viewport": true
        },
        "css": {
            "required_vendor_prefixes": true,
            "required_media_queries": true,
            "required_responsive_units": true
        },
        "js": {
            "required_strict_mode": true,
            "required_error_handling": true,
            "required_async_defer": true
        },
        "images": {
            "required_webp_format": true,
            "required_alt_text": true,
            "required_dimensions": true
        }
    },
    "validation": {
        "html": {
            "required_w3c_validation": true,
            "required_schema_validation": true
        },
        "css": {
            "required_w3c_validation": true
        },
        "js": {
            "required_linting": true,
            "required_minification": true
        }
    }
} 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dprabhua) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
