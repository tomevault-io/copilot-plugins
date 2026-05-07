## models-and-state

> Pydantic output and flow state models reference


# Output Models (src/ai_team/models/outputs.py)

- **RequirementsDocument**: project_name, description, target_users, user_stories (UserStory: as_a, i_want, so_that, acceptance_criteria, priority), non_functional_requirements, assumptions, constraints
- **ArchitectureDocument**: system_overview, components, technology_stack, interfaces, data_model, deployment_topology, adrs
- **CodeFile**: path, content, language, file_type (source/test/config/doc), dependencies, size_bytes
- **TestResult / TestRunResult**: total, passed, failed, errors, skipped, coverage_line, coverage_branch, failures, duration_seconds
- **DeploymentConfig**: dockerfile, docker_compose, ci_pipeline, environment_variables, infrastructure
- **ProjectReport**: project_id, project_name, status, files, test_results, summary, duration

# Flow State (src/ai_team/flows/state.py)

**ProjectState**: project_id, project_description, current_phase, requirements, architecture, generated_files, test_results, deployment_config, phase_history, errors, retry_counts, max_retries, started_at, completed_at, metadata

**ProjectPhase** enum: INTAKE, PLANNING, DEVELOPMENT, TESTING, DEPLOYMENT, COMPLETE, ERROR

# GuardrailResult (guardrails/validators.py)

status ("pass" / "fail" / "warn"), message, details, retry_allowed, severity ("info" / "warning" / "critical")

---
> Source: [RickZee/ai-team](https://github.com/RickZee/ai-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
