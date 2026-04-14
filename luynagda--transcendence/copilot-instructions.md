## transcendence

> "context_initialization": {

{
	"rules": {
		"context_initialization": {
			"description": "Starting point for each interaction",
			"steps": [
				"ALWAYS read `.notes/project_overview.md` and `.notes/task_list.md`",
				"Review relevant component documentation based on task context"
			],
			"project_goals": {
				"main": "Build modern multiplayer Pong game with AI capabilities",
				"technical_focus": [
					"Real-time multiplayer gaming",
					"Clean architecture",
					"State management",
					"Testing coverage",
					"Security"
				]
			}
		},
		"operational_protocol": {
			"description": "How to approach tasks",
			"before_action": [
				"Review related components in the architecture",
				"Check test coverage requirements",
				"Verify state management implications",
				"Consider WebSocket/WebRTC impacts"
			],
			"code_changes": [
				"Maintain clean component architecture",
				"Ensure proper state synchronization",
				"Follow existing patterns for real-time features",
				"Add comprehensive error handling",
				"Include necessary tests"
			],
			"testing_requirements": [
				"Adopt a Test-Driven Development approach",
				"Jest tests for frontend components",
				"Keep test cases updated in related .test.js files"
			]
		},
		"safety_requirements": [
			"NEVER break tests",
			"ALWAYS maintain proper Networking handling",
			"ALWAYS include error boundaries",
			"MAINTAIN clean component architecture"
		],
		"priorities": [
			{
				"source": ".notes/",
				"weight": 1.0
			},
			{
				"source": "transcendence/frontend/",
				"weight": 0.9
			},
			{
				"source": "transcendence/frontend/state/",
				"weight": 0.85
			},
			{
				"source": "transcendence/frontend/UI/",
				"weight": 0.85
			},
			{
				"source": "transcendence/frontend/room/",
				"weight": 0.8
			},
			{
				"source": "transcendence/frontend/pong/",
				"weight": 0.75
			}
		],
		"modes": {
			"base": {
				"description": "For routine component updates",
				"requirements": [
					"Basic error handling",
					"Component documentation",
					"Unit tests",
					"State management consideration"
				]
			},
			"enhanced": {
				"description": "For core game/networking features",
				"requirements": [
					"Comprehensive error handling",
					"WebSocket/WebRTC considerations",
					"State synchronization testing",
					"Performance optimization",
					"Security validation"
				]
			}
		},
		"project_directives": {
			"name": "transcendence",
			"tech_stack": {
				"backend": [
					"Django",
					"Django REST Framework",
					"Channels",
					"PostgreSQL",
					"Daphne",
					"SimpleJWT",
					"django-environ",
					"django-htmx",
					"django-cors-headers",
					"django-truncate",
					"Bugsnag"
				],
				"frontend": [
					"JavaScript",
					"HTMX",
					"Bootstrap 5",
					"WebGL",
					"gl-matrix",
					"regl",
					"Jest",
					"esbuild"
				],
				"deployment": [
					"Docker",
					"Docker Compose"
				]
			},
			"development_principles": [
				"Clean component architecture",
				"Test-driven development",
				"Real-time first",
				"Security by design",
				"Documentation as code"
			]
		},
		"documentation_references": {
			"backend": {
				"core_frameworks": [
					{
						"name": "Django",
						"docs_url": "https://docs.djangoproject.com/",
						"version": "latest"
					},
					{
						"name": "Django Channels",
						"docs_url": "https://channels.readthedocs.io/",
						"version": "latest"
					},
					{
						"name": "Daphne",
						"docs_url": "https://github.com/django/daphne/",
						"version": "latest"
					},
					{
						"name": "Django REST Framework",
						"docs_url": "https://www.django-rest-framework.org/",
						"version": "latest"
					},
					{
						"name": "SimpleJWT",
						"docs_url": "https://django-rest-framework-simplejwt.readthedocs.io/",
						"version": "latest"
					},
					{
						"name": "django-environ",
						"docs_url": "https://django-environ.readthedocs.io/",
						"version": "latest"
					},
					{
						"name": "django-cors-headers",
						"docs_url": "https://github.com/adamchainz/django-cors-headers",
						"version": "latest"
					},
					{
						"name": "Bugsnag",
						"docs_url": "https://docs.bugsnag.com/platforms/python/django/",
						"version": "latest"
					}
				]
			},
			"frontend": {
				"core_technologies": [
					{
						"name": "HTMX",
						"docs_url": "https://htmx.org/docs/"
					},
					{
						"name": "Bootstrap",
						"docs_url": "https://getbootstrap.com/docs/5.0/",
						"version": "5"
					},
					{
						"name": "WebGL",
						"docs_url": "https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/"
					},
					{
						"name": "gl-matrix",
						"docs_url": "https://glmatrix.net/docs/",
						"version": "3.4.3"
					},
					{
						"name": "regl",
						"docs_url": "https://github.com/regl-project/regl/blob/gh-pages/API.md",
						"version": "2.1.1"
					}
				],
				"build_tools": [
					{
						"name": "esbuild",
						"docs_url": "https://esbuild.github.io/",
						"version": "0.24.2"
					}
				],
				"testing": [
					{
						"name": "Jest",
						"docs_url": "https://jestjs.io/docs/",
						"version": "29.7.0"
					}
				]
			}
		},
		"documentation_organization": {
			"description": "Organization of project documentation",
			"core_documentation": {
				"name": "project_overview.md",
				"purpose": "Central reference document",
				"contents": [
					"Project architecture and goals",
					"Technical stack overview",
					"Key features and components",
					"Development principles"
				]
			},
			"task_management": {
				"name": "task_list.md",
				"purpose": "Development tracking",
				"contents": [
					"Current tasks",
					"Implementation status",
					"Test coverage goals",
					"Bug tracking"
				]
			}
		}
	}
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LuyNagda) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
