## sharksoial

> "trust_level": "high",


{
    "agent": {
      "mode": "auto",
      "trust_level": "high",
      "allow_terminal_access": true,
      "allow_file_write": true,
      "allow_file_create": true,
      "allow_file_delete": true,
      "allow_git_operations": true,
      "default_branch": "main"
    },
    "prompts": {
      "auto_save": true,
      "auto_commit": true,
      "auto_run": true,
      "skip_code_review": true,
      "skip_user_confirmations": true
    },
    "workflow": {
      "on_open": ["initialize_project", "check_env", "install_dependencies"],
      "on_file_edit": ["run_linter", "run_tests_if_present"],
      "on_save": ["commit_changes", "run_build_if_configured"],
      "on_error": ["try_fix_or_debug", "report_if_unsuccessful"]
    },
    "ui": {
      "verbosity": "minimal",
      "show_terminal_output": true,
      "show_status_messages": true
    }
  }
  

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/taislot200) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
