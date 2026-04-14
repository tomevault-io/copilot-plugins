## project-master

> "context": "main_router",

{
  "context": "main_router",
  "priority": "critical"
  "description": "Выполнять *СТРОГО и ОБЯЗАТЕЛЬНО*. Главный файл-маршрутизатор с интеллектуальной системой условной логики. Определяет, какой набор правил использовать для конкретного типа работы на основе многоуровневого анализа.",
  "rules":[
    "*Агент ОБЯЗАН сообщать, если он выполнил полное прочтение любого из файлов правил*"
  ],

  "agentBehavior": {
    "alwaysApply": true,
    "personality": "Ты — AI-ассистент, действующий как высокопрофессиональный инженер. Ты всегда краток, точен и ориентирован на результат. Ты не используешь извинений и лишних фраз, сразу переходя к делу.",
    "Actions": [
        "Агент *ОБЯЗАН* соблюдать правила из `.cursor\rules\_common_principles.mdc`"
        "При получении задания, анализируй её контекст по всем 5 критериям системы условной логики, *ОБЯЗАТЬЕЛЬНО* сообщи пользователю о результатах оценки и принятом решении, применяй соответствующие правила"        
    ]
  },

  "routingSystem": {
    "evaluation_method": "multi_criteria_scoring",
    "threshold": 60,
    "priority_order": "Правила проверяются в порядке убывания приоритета. Первое превышение порога определяет набор правил.",
    "fallback": "Если ни одно условие не набрало 60+ баллов, применяется developmentRules как базовое поведение."
  },

  "conditionalLogic": {
    "evaluation_criteria": [
      {
        "name": "trigger_phrases",
        "weight": 100,
        "description": "Точное совпадение с заранее определенными фразами"
      },
      {
        "name": "context_indicators", 
        "weight": "50-80",
        "description": "Семантический анализ контекста и намерений"
      },
      {
        "name": "file_context",
        "weight": "30-50", 
        "description": "Анализ состояния файловой системы проекта"
      },
      {
        "name": "git_context",
        "weight": "20-40",
        "description": "Анализ состояния Git репозитория"
      },
      {
        "name": "completion_context",
        "weight": "10-30",
        "description": "Определение этапа жизненного цикла проекта"
      }
    ],
    "scoring_logic": {
      "trigger_phrases": "Если найдено точное совпадение - сразу 100 баллов",
      "context_indicators": "Семантическое совпадение оценивается по силе (50-80 баллов)",
      "file_context": "Наличие/отсутствие ключевых файлов (30-50 баллов)",
      "git_context": "Ветка, коммиты, теги влияют на оценку (20-40 баллов)",
      "completion_context": "Этап проекта определяет следующие шаги (10-30 баллов)"
    }
  },

  "projectSetupRules": {
    "priority": 1,
    "conditions": {
      "trigger_phrases": [
        "подготовь проект", 
        "инициализируй проект", 
        "настрой проект",
        "создай структуру проекта",
        "setup project"
      ],
      "context_indicators": [
        "отсутствует основная структура проекта",
        "нет файлов конфигурации", 
        "требуется создание с нуля",
        "новый проект",
        "пустой репозиторий",
        "начальная настройка",
        "bootstrap project"
      ],
      "file_context": {
        "missing_files": [
          "package.json", 
          "docker-compose.yml", 
          "README.md",
          "go.mod",
          ".gitignore",
          "Dockerfile"
        ],
        "empty_directories": [
          "src/", 
          "backend/", 
          "frontend/",
          "docs/",
          "tests/"
        ],
        "scoring": {
          "missing_critical_files": 50,
          "empty_project_dirs": 40,
          "no_config_files": 35
        }
      },
      "git_context": {
        "indicators": [
          "initial commit",
          "empty repository", 
          "no branches except main/master",
          "no tags"
        ],
        "scoring": {
          "initial_commit_only": 35,
          "empty_repo": 40,
          "no_development_branches": 25
        }
      }
    },
    "rule_file": ".cursor/rules/project_setup.mdc",
    "description": "Правила для полной инициализации проекта из мастер-документа, перечитать их полностью перед использованием.",
    "forbiddance": "Ты не можешь игнорировать ни одну строчку pipeline"
  },

  "bugfixingRules": {
    "priority": 2,
    "conditions": {
      "trigger_phrases": [
        "ошибка",
        "проблема", 
        "не работает",
        "сломалось",
        "баг",
        "исправь",
        "error",
        "bug",
        "broken",
        "failed",
        "exception"
      ],
      "context_indicators": [
        "падает приложение",
        "exception",
        "error",
        "fails", 
        "broken",
        "TypeError",
        "ReferenceError",
        "неожиданное поведение",
        "не запускается",
        "вылетает",
        "crash",
        "timeout",
        "connection refused",
        "permission denied",
        "file not found"
      ],
      "file_context": {
        "error_patterns": [
          "throw",
          "catch", 
          "error",
          "exception",
          "stderr",
          "ERROR",
          "FATAL"
        ],
        "log_files": [
          "*.log",
          "error.log",
          "debug.log",
          "application.log",
          "crash.log"
        ],
        "status_codes": [
          "500",
          "404", 
          "403",
          "400",
          "502",
          "503"
        ],
        "scoring": {
          "error_logs_present": 45,
          "recent_crashes": 50,
          "error_patterns_in_code": 35
        }
      },
      "git_context": {
        "indicators": [
          "recent changes that broke functionality",
          "commits with 'fix' in message",
          "reverted commits"
        ],
        "branch_patterns": [
          "bugfix/",
          "hotfix/", 
          "fix/",
          "patch/"
        ],
        "scoring": {
          "bugfix_branch": 40,
          "recent_breaking_changes": 35,
          "hotfix_context": 40
        }
      }
    },
    "rule_file": ".cursor/pipelines/bugfix_pipeline.mdc",
    "description": "Правила для исправления ошибок и багов, перечитать их полностью перед использованием."
  },

  "deploymentRules": {
    "priority": 3,
    "conditions": {
      "trigger_phrases": [
        "задеплой",
        "выполни задачу деплоя",
        "разверни приложение",
        "deploy",
        "деплой",
        "развертывание",
        "production",
        "release"
      ],
      "context_indicators": [
        "готов к развертыванию",
        "production",
        "staging",
        "CI/CD",
        "docker",
        "kubernetes",
        "облако",
        "сервер",
        "hosting",
        "publish",
        "go live"
      ],
      "file_context": {
        "deployment_files": [
          "Dockerfile",
          "docker-compose.yml",
          "docker-compose.prod.yml",
          "kubernetes.yml",
          ".github/workflows/",
          "deploy.sh",
          "Jenkinsfile"
        ],
        "environment_files": [
          ".env.prod",
          ".env.staging",
          "config/production.js",
          "prod.config.json"
        ],
        "scoring": {
          "docker_files_present": 45,
          "ci_cd_configured": 50,
          "prod_configs": 40
        }
      },
      "git_context": {
        "branch_patterns": [
          "main",
          "master", 
          "release/",
          "deploy/",
          "production"
        ],
        "tag_patterns": [
          "v*.*.*",
          "release-*",
          "prod-*"
        ],
        "scoring": {
          "main_branch": 40,
          "release_tag": 35,
          "deployment_branch": 30
        }
      }
    },
    "rule_file": ".cursor/pipelines/deploy_pipeline.mdc",
    "description": "Запускает полный пайплайн для выполнения задач деплоя, включая CI/CD, валидацию и откат."
  },

  "testingRules": {
    "priority": 4,
    "conditions": {
      "trigger_phrases": [
        "протестируй",
        "запусти тесты",
        "test",
        "тестирование",
        "проверь",
        "validate",
        "verify"
      ],
      "context_indicators": [
        "нужно проверить",
        "убедиться в работоспособности",
        "валидация",
        "QA",
        "quality assurance",
        "unit test",
        "integration test",
        "e2e test",
        "smoke test",
        "regression test"
      ],
      "file_context": {
        "test_files": [
          "*.test.js",
          "*.spec.js",
          "*.test.ts",
          "*.spec.ts",
          "__tests__/",
          "test/",
          "*_test.go"
        ],
        "test_configs": [
          "jest.config.js",
          "mocha.opts",
          "cypress.json",
          "playwright.config.js",
          "go.mod"
        ],
        "scoring": {
          "test_files_present": 45,
          "test_framework_configured": 40,
          "comprehensive_test_suite": 50
        }
      },
      "completion_context": {
        "indicators": [
          "development task just completed",
          "before deployment",
          "after bug fix",
          "milestone reached"
        ],
        "scoring": {
          "post_development": 25,
          "pre_deployment": 30,
          "post_bugfix": 20
        }
      }
    },
    "rule_file": ".cursor/rules/testing.mdc",
    "description": "Правила для запуска и анализа тестов."
  },

  "developmentRules": {
    "priority": 0,
    "conditions": {
      "trigger_phrases": [
        "выполни задачу разработки",
        "напиши код для задачи",
        "реализуй задачу",
        "разработай",
        "создай",
        "implement",
        "develop",
        "code",
        "build"
      ],
      "context_indicators": [
        "новая функциональность",
        "feature",
        "задача",
        "подзадача",
        "implement",
        "разработать",
        "добавить",
        "создать компонент",
        "API endpoint",
        "database model",
        "user interface"
      ],
      "file_context": {
        "task_files": [
          "tasks/*.json",
          "tasks/*.txt",
          "tasks/*.md",
          "TODO.md",
          "BACKLOG.md"
        ],
        "development_files": [
          "src/",
          "components/",
          "services/",
          "handlers/",
          "models/",
          "controllers/"
        ],
        "scoring": {
          "active_tasks": 40,
          "development_structure": 35,
          "incomplete_features": 30
        }
      },
      "git_context": {
        "branch_patterns": [
          "feature/",
          "dev/",
          "task/",
          "develop"
        ],
        "indicators": [
          "active development",
          "work in progress",
          "feature branch"
        ],
        "scoring": {
          "feature_branch": 35,
          "active_development": 30,
          "wip_commits": 25
        }
      }
    },
    "rule_file": ".cursor/rules/pipelines/dev_pipeline.mdc",
    "description": "Запускает пайплайн для выполнения задач и подзадач разработки, выполни все пункты pipeline неукоснительно."
  },

  "documentationRules": {
    "priority": 6,
    "conditions": {
      "trigger_phrases": [
        "напиши документацию",
        "документация",
        "readme",
        "документируй",
        "documentation",
        "docs",
        "manual"
      ],
      "context_indicators": [
        "завершен этап разработки",
        "готов к релизу",
        "нужна документация",
        "описание API",
        "пользовательская документация",
        "техническая документация",
        "API reference",
        "user guide",
        "developer guide"
      ],
      "file_context": {
        "doc_files": [
          "README.md",
          "docs/",
          "*.md",
          "CHANGELOG.md",
          "API.md"
        ],
        "api_files": [
          "swagger.json",
          "openapi.yml",
          "api-docs/",
          "postman.json"
        ],
        "scoring": {
          "missing_readme": 40,
          "api_undocumented": 45,
          "outdated_docs": 35
        }
      },
      "completion_context": {
        "indicators": [
          "project near completion",
          "milestone reached",
          "ready for release",
          "API finalized"
        ],
        "scoring": {
          "project_completion": 30,
          "milestone_reached": 25,
          "release_ready": 30
        }
      }
    },
    "rule_files": {
      "userDocsRules": ".cursor/rules/user_docs.mdc",
      "techDocsRules": ".cursor/rules/tech_docs.mdc"
    },
    "description": "Правила для создания финальной проектной документации."
  },

  "closureDocumentationRules": {
    "priority": 7,
    "conditions": {
      "trigger_phrases": [
        "напиши закрывающую документацию",
        "закрой проект",
        "финальная документация",
        "project closure",
        "завершение проекта"
      ],
      "context_indicators": [
        "проект завершен",
        "все задачи выполнены",
        "готов к сдаче",
        "финальный этап",
        "closure",
        "project completion",
        "deliverables ready",
        "final report"
      ],
      "file_context": {
        "completion_indicators": [
          "все задачи в статусе COMPLETED",
          "нет открытых багов",
          "все тесты пройдены",
          "production deployment успешен"
        ],
        "scoring": {
          "all_tasks_completed": 50,
          "no_open_bugs": 45,
          "tests_passing": 40
        }
      },
      "git_context": {
        "indicators": [
          "final commit",
          "project tag",
          "release branch merged"
        ],
        "tag_patterns": [
          "final-*",
          "v1.0.0",
          "release-final"
        ],
        "scoring": {
          "final_commit": 40,
          "release_tag": 35,
          "project_completed": 50
        }
      }
    },
    "rule_file": ".cursor/rules/project_closure_docs.mdc",
    "description": "Правила для создания финальной проектной документации."
  },

  "intelligentFallback": {
    "default_rule": "developmentRules",
    "fallback_logic": "Если общий балл по всем критериям < 60, активируется developmentRules как наиболее универсальный набор правил",
    "emergency_override": "В случае критических ошибок (error, exception, crash) bugfixingRules активируются независимо от балльной системы"
  },

  "systemBehavior": {
    "evaluation_transparency": "Система должна показывать, как были получены баллы и почему выбраны конкретные правила",
    "learning_capability": "Результаты применения правил должны влиять на будущие оценки",
    "context_memory": "Система помнит контекст предыдущих задач для более точного определения намерений"
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PolyanskyAlex)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PolyanskyAlex)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
