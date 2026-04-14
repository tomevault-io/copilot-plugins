## upline-front

> use these rules for every chat


# CursorRIPER♦Σ 1.0.2 -(code protect + context)

## 📚 Path & Index Definitions
📂 = "/memory-bank/"
📦 = "/memory-bank/backups/"

𝕋 = [read_files, ask_questions, observe_code, document_findings,
     suggest_ideas, explore_options, evaluate_approaches,
     create_plan, detail_specifications, sequence_steps,
     implement_code, follow_plan, test_implementation,
     validate_output, verify_against_plan, report_deviations]
     
𝕄 = [📂projectbrief.md, 📂systemPatterns.md, 
     📂techContext.md, 📂activeContext.md, 
     📂progress.md, 📂protection.md]

## Ω RIPER Modes

Ω₁ = 🔍R ⟶ +𝕋[0:3] -𝕋[4:15] ⟶ [MODE: RESEARCH]+findings
  ↪ 🔄(/research, /r) ⟶ update(𝕄[2,3])

Ω₂ = 💡I ⟶ +𝕋[4:6] -𝕋[8:15] ⟶ [MODE: INNOVATE]+possibilities
  ↪ 🔄(/innovate, /i) ⟶ update(𝕄[3])

Ω₃ = 📝P ⟶ +𝕋[7:9] -𝕋[10:15] ⟶ [MODE: PLAN]+checklist₁₋ₙ
  ↪ 🔄(/plan, /p) ⟶ update(𝕄[3,4])

Ω₄ = ⚙️E ⟶ +𝕋[10:12] -[improve,create,deviate] ⟶ [MODE: EXECUTE]+progress
  ↪ 🔄(/execute, /e) ⟶ update(𝕄[3,4])

Ω₅ = 🔎RV ⟶ +𝕋[13:15] -[modify,improve] ⟶ [MODE: REVIEW]+{✅|⚠️}
  ↪ 🔄(/review, /rev) ⟶ update(𝕄[3,4])

## 🛡️ Code Protection System

Ψ = [PROTECTED, GUARDED, INFO, DEBUG, TEST, CRITICAL]

Ψ_behavior_summary = {
  Ω₁: identify ∧ document(Ψ),
  Ω₂: respect_boundaries(Ψ) ∧ propose_alternatives,
  Ω₃: plan_around(Ψ) ∧ request_permission(Ψ.GUARDED),
  Ω₄: enforce_protection(Ψ) ∧ follow_guidelines,
  Ω₅: verify_integrity(Ψ) ∧ report_violations
}

## 📎 Context Reference System

Γ = [FILES, FOLDERS, CODE, DOCS, RULES, GIT, NOTEPADS, PINNED]

Γ_symbols = {
  Γ₁: 📄 @Files,
  Γ₂: 📁 @Folders,
  Γ₃: 💻 @Code,
  Γ₄: 📚 @Docs,
  Γ₅: 📏 @Cursor Rules,
  Γ₆: 🔄 @Git,
  Γ₇: 📝 @Notepads,
  Γ₈: 📌 #Files
}

## Mode-Context Mapping

MΓ = {
  Ω₁: [Γ₄, Γ₂, Γ₆],  // RESEARCH: docs, folders, git
  Ω₂: [Γ₃, Γ₄, Γ₇],  // INNOVATE: code, docs, notepads
  Ω₃: [Γ₁, Γ₂, Γ₅],  // PLAN: files, folders, rules
  Ω₄: [Γ₃, Γ₁, Γ₈],  // EXECUTE: code, files, pinned
  Ω₅: [Γ₃, Γ₁, Γ₆]   // REVIEW: code, files, git
}

Γ_behavior = {
  add_context(type, name) = {
    verify_exists(name),
    update_context_list(𝕄[3], type, name),
    set_context_status(name, "active")
  },
  clear_context() = {
    backup_context(),
    reset_context_list(𝕄[3])
  },
  context_for_mode(mode) = {
    mode_contexts = MΓ[mode],
    apply_mode_context(mode_contexts)
  }
}

## Protection-Context Integration

PΓ = {
  Ψ₁ + Γ₃: 🔒💻,  // Protected code
  Ψ₂ + Γ₃: 🛡️💻,  // Guarded code
  Ψ₃ + Γ₃: ℹ️💻,   // Info code
  Ψ₄ + Γ₃: 🐞💻,  // Debug code
  Ψ₅ + Γ₃: 🧪💻,  // Test code
  Ψ₆ + Γ₃: ⚠️💻   // Critical code
}

## Π Project Phases

Π₁ = 🌱UNINITIATED ⟶ framework_installed ∧ ¬project_started
Π₂ = 🚧INITIALIZING ⟶ START_active ∧ setup_ongoing  
Π₃ = 🏗️DEVELOPMENT ⟶ main_development ∧ RIPER_active
Π₄ = 🔧MAINTENANCE ⟶ long_term_support ∧ RIPER_active

Π_transitions = {
  Π₁→Π₂: 🔄"/start",
  Π₂→Π₃: ✅completion(START_phase),
  Π₃↔Π₄: 🔄user_request
}

## 🏁 START Phase (Π₂)

S₁₋₆ = [requirements, technology, architecture, scaffolding, environment, memory]

START_process = {
  S₀: create_directory(📂),
  S₁: gather(requirements) ⟶ create(𝕄[0]),
  S₂: select(technologies) ⟶ update(𝕄[2]),
  S₃: define(architecture) ⟶ create(𝕄[1]),
  S₄: scaffold(project) ⟶ create(directories),
  S₅: setup(environment) ⟶ update(𝕄[2]),
  S₆: initialize(memory) ⟶ create(𝕄[0:5])
}

## 📑 Memory Templates

Σ_templates = {
  σ₁: """# σ₁: Project Brief\n*v1.0 | Created: {DATE} | Updated: {DATE}*\n*Π: {PHASE} | Ω: {MODE}*\n\n## 🏆 Overview\n[Project description]\n\n## 📋 Requirements\n- [R₁] [Requirement 1]\n...""",
  
  σ₂: """# σ₂: System Patterns\n*v1.0 | Created: {DATE} | Updated: {DATE}*\n*Π: {PHASE} | Ω: {MODE}*\n\n## 🏛️ Architecture Overview\n[Architecture description]\n...""",
  
  σ₃: """# σ₃: Technical Context\n*v1.0 | Created: {DATE} | Updated: {DATE}*\n*Π: {PHASE} | Ω: {MODE}*\n\n## 🛠️ Technology Stack\n- 🖥️ Frontend: [Technologies]\n...""",
  
  σ₄: """# σ₄: Active Context\n*v1.0 | Created: {DATE} | Updated: {DATE}*\n*Π: {PHASE} | Ω: {MODE}*\n\n## 🔮 Current Focus\n[Current focus]\n\n## 📎 Context References\n- 📄 Active Files: []\n- 💻 Active Code: []\n- 📚 Active Docs: []\n- 📁 Active Folders: []\n- 🔄 Git References: []\n- 📏 Active Rules: []\n\n## 📡 Context Status\n- 🟢 Active: []\n- 🟡 Partially Relevant: []\n- 🟣 Essential: []\n- 🔴 Deprecated: []\n...""",
  
  σ₅: """# σ₅: Progress Tracker\n*v1.0 | Created: {DATE} | Updated: {DATE}*\n*Π: {PHASE} | Ω: {MODE}*\n\n## 📈 Project Status\nCompletion: 0%\n...""",
  
  σ₆: """# σ₆: Protection Registry\n*v1.0 | Created: {DATE} | Updated: {DATE}*\n*Π: {PHASE} | Ω: {MODE}*\n\n## 🛡️ Protected Regions\n[Protected code registry]\n\n## 📜 Protection History\n[History and changes]\n\n## ✅ Approvals\n[Modification approvals]""",
  
  symbols: """# 🔣 Symbol Reference Guide\n*v1.0 | Created: {DATE} | Updated: {DATE}*\n\n## 📁 File Symbols\n- 📂 = /memory-bank/\n..."""
}

Φ_memory = {
  create_template(template, params) = template.replace({PLACEHOLDERS}, params),
  initialize() = {
    ensure_directory(📂),
    create_file(𝕄[0], create_template(Σ_templates.σ₁, {DATE: now(), PHASE: current_phase, MODE: current_mode})),
    create_file(𝕄[1], create_template(Σ_templates.σ₂, {DATE: now(), PHASE: current_phase, MODE: current_mode})),
    create_file(𝕄[2], create_template(Σ_templates.σ₃, {DATE: now(), PHASE: current_phase, MODE: current_mode})),
    create_file(𝕄[3], create_template(Σ_templates.σ₄, {DATE: now(), PHASE: current_phase, MODE: current_mode})),
    create_file(𝕄[4], create_template(Σ_templates.σ₅, {DATE: now(), PHASE: current_phase, MODE: current_mode})),
    create_file(𝕄[5], create_template(Σ_templates.σ₆, {DATE: now(), PHASE: current_phase, MODE: current_mode})),
    create_file(📂symbols.md, create_template(Σ_templates.symbols, {DATE: now()}))
  }
}

## 🧰 Memory System

Σ_memory = {
  σ₁ = 📋𝕄[0] ⟶ requirements ∧ scope ∧ criteria,
  σ₂ = 🏛️𝕄[1] ⟶ architecture ∧ components ∧ decisions,
  σ₃ = 💻𝕄[2] ⟶ stack ∧ environment ∧ dependencies,
  σ₄ = 🔮𝕄[3] ⟶ focus ∧ changes ∧ next_steps ∧ context_references,
  σ₅ = 📊𝕄[4] ⟶ status ∧ milestones ∧ issues,
  σ₆ = 🛡️𝕄[5] ⟶ protected_regions ∧ history ∧ approvals
}

Σ_update(mode) = {
  Ω₁: σ₃ += technical_details, σ₄ = current_focus, set_context(MΓ[Ω₁]),
  Ω₂: σ₄ += potential_approaches, σ₂ += design_decisions, set_context(MΓ[Ω₂]),
  Ω₃: σ₄ += planned_changes, σ₅ += expected_outcomes, set_context(MΓ[Ω₃]),
  Ω₄: σ₅ += implementation_progress, σ₄ += step_completion, set_context(MΓ[Ω₄]),
  Ω₅: σ₅ += review_findings, σ₄ += review_status, set_context(MΓ[Ω₅])
}

## Σ_context System

Σ_context = {
  active_references: [],
  status_map: {},
  add_reference(type, name, status = "active") = {
    active_references.push({type, name, added: now()}),
    status_map[name] = status,
    update_file(𝕄[3], format_context_section())
  },
  remove_reference(name) = {
    active_references = active_references.filter(ref => ref.name !== name),
    delete status_map[name],
    update_file(𝕄[3], format_context_section())
  },
  clear_references() = {
    backup_context_refs(),
    active_references = [],
    status_map = {},
    update_file(𝕄[3], format_empty_context())
  },
  set_status(name, status) = {
    status_map[name] = status,
    update_file(𝕄[3], format_context_section())
  },
  context_for_mode(mode) = {
    backup_context_refs(),
    clear_references(),
    for context_type in MΓ[mode] {
      add_reference(context_type, "auto:" + mode, "essential")
    }
  },
  format_context_section() = generate_context_markdown()
}

## Σ_backup System

Σ_backup = {
  backup_format = "YYYY-MM-DD_HH-MM-SS",
  create_backup() = copy_files(𝕄, 📦 + timestamp(backup_format)),
  backup_context() = {
    ctx_backup = {refs: Σ_context.active_references, status: Σ_context.status_map}
    write_json(📦 + "context_" + timestamp(backup_format) + ".json", ctx_backup)
  }
}

## 🔍 Context Commands

Φ_context_commands = {
  !af(file) = Σ_context.add_reference(Γ₁, file),             // Add file reference
  !ad(folder) = Σ_context.add_reference(Γ₂, folder),         // Add folder reference
  !ac(code) = Σ_context.add_reference(Γ₃, code),             // Add code reference
  !adoc(doc) = Σ_context.add_reference(Γ₄, doc),             // Add documentation reference
  !ar(rule) = Σ_context.add_reference(Γ₅, rule),             // Add rule reference
  !ag(git) = Σ_context.add_reference(Γ₆, git),               // Add git reference
  !an(notepad) = Σ_context.add_reference(Γ₇, notepad),       // Add notepad reference
  !pf(file) = Σ_context.add_reference(Γ₈, file),             // Pin file to context
  !cs(ref, status) = Σ_context.set_status(ref, status),      // Set context status
  !cr(ref) = Σ_context.remove_reference(ref),                // Remove context reference
  !cc = Σ_context.clear_references(),                        // Clear all context references
  !cm = Σ_context.context_for_mode(current_mode)             // Set context for current mode
}

## ⚠️ Safety Protocols

Δ₁ = destructive_op(x) ⟶ warn ∧ confirm ∧ Σ_backup.create_backup()
Δ₂ = phase_transition(Πₐ→Πᵦ) ⟶ verify ∧ Σ_backup.create_backup() ∧ update
Δ₃ = reinit_attempt ∧ ¬Π₁ ⟶ warn ∧ confirm("CONFIRM RE-INITIALIZATION") ∧ Σ_backup.create_backup()
Δ₄ = error(x) ⟶ report("Framework issue: " + x) ∧ suggest_recovery(x)
Δ₅ = context_change() ⟶ Σ_backup.backup_context() ∧ update_context_references()

## 📂 File System Operations

Φ_file = {
  ensure_directory(path) = path_exists(path) ? noop : create_directory(path),
  init() = ensure_directory(📂) ∧ ensure_directory(📦),
  check_files() = ∀file ∈ 𝕄, check_exists(file)
}

## 📊 Context Operations

Φ_context = {
  expand(Γₙ) = get_full_context(Γₙ),  // Expand context reference
  filter(Γₙ, criteria) = filter_context_by(Γₙ, criteria),  // Filter context
  persist(Γₙ, 📂) = save_context_to_memory(Γₙ, 📂),  // Save context
  retrieve(Γₙ, 📂) = load_context_from_memory(Γₙ, 📂),  // Load context
  rank(Γₙ, relevance) = prioritize_context(Γₙ, relevance)  // Prioritize context
}

## 🔗 Extended Cross-References

χ_refs = {
  standard: "[↗️σ₁:R₁]",  // Standard cross-reference
  with_context: "[↗️σ₁:R₁|Γ₃]",  // Cross-reference with context
  context_only: "[Γ₃:ClassA]",  // Context reference
  protection_context: "[Ψ₁+Γ₃:validate()]"  // Protection with context
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iakev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/iakev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
