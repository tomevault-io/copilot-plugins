## cursor-best-rules-setup

> PROTECTED: "PROTECTED - DO NOT MODIFY",


# 🛡️ Code Protection Syntax

Ψ_syntax = {
  PROTECTED: "PROTECTED - DO NOT MODIFY",
  GUARDED: "GUARDED - ASK BEFORE MODIFYING",
  INFO: "INFO - CONTEXT NOTE",
  DEBUG: "DEBUG - DEBUGGING CODE",
  TEST: "TEST - TESTING CODE",
  CRITICAL: "CRITICAL - BUSINESS LOGIC",
  
  // End markers
  END_PROTECTED: "END-P - PROTECTED REGION END",
  END_GUARDED: "END-G - GUARDED REGION END",
  END_INFO: "END-I - INFO REGION END",
  END_DEBUG: "END-D - DEBUG REGION END",
  END_TEST: "END-T - TEST REGION END",
  END_CRITICAL: "END-C - CRITICAL REGION END"
}

## 💬 Language Comment Formats

Ψ_language_syntax = {
  js: {prefix: "// ", suffix: ""},
  ts: {prefix: "// ", suffix: ""},
  jsx: {prefix: "// ", suffix: ""},
  tsx: {prefix: "// ", suffix: ""},
  py: {prefix: "# ", suffix: ""},
  html: {prefix: "<!-- ", suffix: " -->"},
  php: {prefix: "// ", suffix: ""},
  css: {prefix: "/* ", suffix: " */"},
  scss: {prefix: "/* ", suffix: " */"},
  java: {prefix: "// ", suffix: ""},
  rb: {prefix: "# ", suffix: ""},
  go: {prefix: "// ", suffix: ""},
  rs: {prefix: "// ", suffix: ""},
  c: {prefix: "// ", suffix: ""},
  cpp: {prefix: "// ", suffix: ""},
  cs: {prefix: "// ", suffix: ""},
  swift: {prefix: "// ", suffix: ""},
  kt: {prefix: "// ", suffix: ""},
  dart: {prefix: "// ", suffix: ""},
  md: {prefix: "<!-- ", suffix: " -->"},
  xml: {prefix: "<!-- ", suffix: " -->"},
  sh: {prefix: "# ", suffix: ""},
  bash: {prefix: "# ", suffix: ""},
  sql: {prefix: "-- ", suffix: ""}
}

## ⌨️ Command Shortcuts

Ψ_shorthand = {
  "!cp": apply_protection(PROTECTED),
  "!cg": apply_protection(GUARDED),
  "!ci": apply_protection(INFO),
  "!cd": apply_protection(DEBUG),
  "!ct": apply_protection(TEST),
  "!cc": apply_protection(CRITICAL)
}

apply_protection(type) = {
  detect_language(current_file) ⟶ lang,
  get_comment_syntax(lang) ⟶ {prefix, suffix},
  selection = get_editor_selection(),
  
  // Insert opening marker
  insert_at_selection_start(prefix + Ψ_syntax[type] + suffix),
  
  // Insert end marker
  end_marker = Ψ_syntax["END_" + type],
  insert_at_selection_end(prefix + end_marker + suffix),
  
  // Update protection registry
  Ψ_manage.add(current_file, selection.start_line, selection.end_line, type, "User-added protection")
}

## 🔄 Protection Behaviors

Ψ_behaviors = {
  PROTECTED: {
    Ω₁: acknowledge ∧ document,
    Ω₂: respect_boundaries ∧ alternate_approaches,
    Ω₃: plan_around ∧ never_include,
    Ω₄: refuse_modification ∧ report_attempts,
    Ω₅: verify_untouched ∧ validate
  },
  GUARDED: {
    Ω₁: acknowledge ∧ document,
    Ω₂: consider_changes ∧ document_rationale,
    Ω₃: plan_with_permission ∧ alternatives,
    Ω₄: request_explicit_permission ∧ detail_changes,
    Ω₅: document_changes ∧ justify
  },
  INFO: {
    Ω₁: acknowledge ∧ use_context,
    Ω₂: incorporate_context ∧ respect_intent,
    Ω₃: plan_with_awareness,
    Ω₄: careful_modification ∧ preserve_intent,
    Ω₅: verify_context_preserved
  },
  DEBUG: {
    Ω₁: note_debug_purpose,
    Ω₂: preserve_during_innovation,
    Ω₃: include_in_development_plan,
    Ω₄: maintain_during_dev ∧ consider_cleanup,
    Ω₅: evaluate_necessity
  },
  TEST: {
    Ω₁: document_test_coverage,
    Ω₂: maintain_test_integrity,
    Ω₃: ensure_test_coverage,
    Ω₄: update_with_implementation,
    Ω₅: verify_test_coverage
  },
  CRITICAL: {
    Ω₁: document_thoroughly,
    Ω₂: design_with_extreme_care,
    Ω₃: plan_impact_analysis,
    Ω₄: comprehensive_review ∧ careful_change,
    Ω₅: rigorous_validation
  }
}

## 🔍 Protection Scanner

Ψ_scan = {
  patterns: {
    auth: ["login", "authenticate", "credentials", "password", "token"],
    payment: ["payment", "transaction", "credit", "billing", "invoice"],
    security: ["encrypt", "decrypt", "hash", "salt", "secure"],
    core: ["critical", "essential", "main", "primary", "core"],
    api: ["api", "endpoint", "request", "response", "service"],
    data: ["database", "query", "record", "store", "retrieve"]
  },
  
  detect(file) = {
    lang = detect_language(file),
    code = read_file(file),
    segments = parse(code, lang),
    analysis = []
    
    // Track open markers and match with end markers
    open_markers = []
    
    for segment in segments:
      // Check if this is an end marker
      end_marker_match = match_end_marker(segment)
      if end_marker_match:
        if open_markers.length > 0:
          // Close the most recent matching open marker
          close_marker(open_markers, end_marker_match, analysis)
        continue
        
      // Check if this is an opening marker
      marker_type = match_protection_marker(segment)
      if marker_type:
        open_markers.push({
          type: marker_type,
          line: segment.line_number,
          content: segment
        })
        continue
      
      // Regular code segment - check patterns if not in a marker
      if open_markers.length == 0:
        pattern_matches = match_patterns(segment, Ψ_scan.patterns)
        if pattern_matches:
          analysis.push({
            segment: segment,
            matches: pattern_matches,
            suggested_level: determine_level(pattern_matches)
          })
    
    // Report any unclosed markers
    for marker in open_markers:
      analysis.push({
        segment: marker.content,
        warning: "Unclosed protection marker",
        suggested_action: "Add appropriate end marker"
      })
    
    return analysis
  },
  
  determine_level(matches) = {
    if matches.intersect(["security", "payment", "auth"]).length > 0:
      return "PROTECTED"
    else if matches.intersect(["core", "api"]).length > 0:
      return "CRITICAL"
    else if matches.intersect(["data"]).length > 0:
      return "GUARDED"
    else:
      return "INFO"
  },
  
  match_end_marker(segment) = {
    for type in [PROTECTED, GUARDED, INFO, DEBUG, TEST, CRITICAL]:
      end_marker = "END-" + type.substr(0,1)
      if segment.includes(end_marker):
        return type
    return null
  },
  
  close_marker(open_markers, end_type, analysis) = {
    // Find matching open marker
    matched_idx = -1
    for i = open_markers.length - 1; i >= 0; i--:
      if open_markers[i].type === end_type:
        matched_idx = i
        break
        
    if matched_idx >= 0:
      // Remove the marker from the open list
      marker = open_markers.splice(matched_idx, 1)[0]
      // Process the protection block if needed
      // (no analysis needed for properly marked protection blocks)
    else:
      analysis.push({
        warning: "Unmatched end marker for " + end_type,
        suggested_action: "Add appropriate start marker"
      })
  }
}

## 📊 Protection Management

Ψ_manage = {
  add(file, start_line, end_line, level, rationale) = {
    entry = {
      file: file,
      start_line: start_line,
      end_line: end_line,
      level: level,
      added_date: now(),
      rationale: rationale
    },
    update(σ₆.protected_regions, entry)
  },
  
  approve(file, start_line, end_line, changes) = {
    approval = {
      file: file,
      start_line: start_line,
      end_line: end_line,
      requested_date: now(),
      approved_date: now(),
      changes: changes
    },
    update(σ₆.guarded_approvals, approval)
  },
  
  scan_project() = {
    results = [],
    files = list_project_files(),
    
    for file in files:
      if is_code_file(file):
        scan_result = Ψ_scan.detect(file)
        if scan_result.length > 0:
          results.push({
            file: file,
            findings: scan_result
          })
    
    update(σ₆.scan_history, {
      date: now(),
      files_scanned: files.length,
      protections_found: results.length
    })
    
    return results
  }
}

## 🔄 Protection Commands

Ψ_commands = {
  "/protect-scan": Ψ_manage.scan_project,
  "/protect-status": report_protection_status,
  "/protect-add": add_protection_to_selection,
  "/protect-remove": remove_protection_with_confirmation,
  "/protect-approve": approve_guarded_modification
}

report_protection_status() = {
  regions = read(σ₆.protected_regions),
  summary = summarize(regions),
  return format_report(summary)
}

add_protection_to_selection(level) = {
  selection = get_editor_selection(),
  file = get_current_file(),
  lang = detect_language(file),
  syntax = Ψ_language_syntax[lang],
  
  // Add start marker
  start_comment = syntax.prefix + Ψ_syntax[level] + syntax.suffix,
  insert_at_selection_start(start_comment),
  
  // Add end marker
  end_comment = syntax.prefix + Ψ_syntax["END_" + level] + syntax.suffix,
  insert_at_selection_end(end_comment),
  
  // Register the protected region
  Ψ_manage.add(file, selection.start_line, selection.end_line, level, "User-added protection")
}

remove_protection_with_confirmation(region_id) = {
  region = find_region_by_id(region_id),
  confirm("Are you sure you want to remove protection from this code?"),
  if confirmed:
    // Remove both the start and end markers
    remove_protection_comment(region.file, region.start_line),
    remove_protection_comment(region.file, region.end_line),
    remove_from_registry(region_id)
}

approve_guarded_modification(region_id, changes) = {
  region = find_region_by_id(region_id),
  if region.level != "GUARDED":
    return error("Only GUARDED code can be approved for modification")
  else:
    Ψ_manage.approve(region.file, region.start_line, region.end_line, changes)
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HyunjunJeon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
