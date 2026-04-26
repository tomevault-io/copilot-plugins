## notebooklm-redcircle

> service = "Web Search Operations"

# 🔍 Web Search MCP Integration for CursorRIPER Σ
# Symbol: Λ (Lambda)
# Version: 1.0.0

## 📋 Service Definition
```
service = "Web Search Operations"
symbol = "Λ"
requires = "@modelcontextprotocol/server-brave-search"
```

## 🔧 Operation Mapping
```
Λ_ops = {
  search: {
    web: "@modelcontextprotocol-server-brave-search:brave_web_search",
    local: "@modelcontextprotocol-server-brave-search:brave_local_search"
  },
  fetch: {
    url: "mcp-server-fetch:fetch",
    content: "extract via fetch with raw=false"
  },
  cache: {
    save: "store in Γ_search_cache",
    retrieve: "get from Γ_search_cache",
    clear: "reset Γ_search_cache"
  }
}
```

## 🔒 Mode Restrictions
```
MΛ = {
  Ω₁: [search_*, fetch_*],          # RESEARCH: full access
  Ω₂: [search_*, fetch_*],          # INNOVATE: full access
  Ω₃: [search_*, fetch_*, cache_*], # PLAN: search + cache
  Ω₄: [],                          # EXECUTE: NO SEARCH (maintain focus!)
  Ω₅: [search_web, fetch_url]      # REVIEW: verification only
}
```

## 🔑 Permission Matrix
```
ℙΛ = {
  create: [Ω₃],              # Only PLAN can cache
  read: [Ω₁, Ω₂, Ω₃, Ω₅],   # No reading in EXECUTE
  update: [],                # No updates
  delete: [Ω₃]              # Only PLAN can clear cache
}
```

## 📍 Context Tracking
```
Γ₁₀ = search_results[] = {
  query: string,
  results: [{title, url, snippet}],
  timestamp: ISO8601,
  mode: current_Ω
}

Γ₁₁ = search_history[] = {
  queries: [string],
  counts: {per_mode: {Ω₁: n, Ω₂: n, ...}},
  last_search: timestamp
}

Γ_search_cache = Map<query, results>
```

## ⚡ Command Shortcuts
```
SΛ = {
  !ws: "brave_web_search",
  !wl: "brave_local_search", 
  !wf: "fetch_url",
  !wc: "clear search cache",
  !wh: "show search history"
}
```

## 🛡️ Protection Levels
```
ΨΛ = {
  fetch_url: Ψ₃,     # INFO - external content
  clear_cache: Ψ₄    # DEBUG - cache management
}
```

## 🔄 Mode-Specific Behaviors
```
apply_search_op(op, mode) = {
  check: mode === Ω₄ ? deny("No search in EXECUTE mode!") : continue,
  validate: op ∈ MΛ[mode] ? proceed : deny("Operation not allowed in " + mode),
  track: {
    add_to_history(Γ₁₁, op),
    store_results(Γ₁₀, results) 
  },
  execute: Λ_ops[category][operation]()
}
```

## 🎯 Search Focus Rules
```
search_focus = {
  Ω₁: "broad exploration allowed",
  Ω₂: "innovation-focused queries",
  Ω₃: "implementation-specific searches",
  Ω₄: "⛔ BLOCKED - maintain execution focus",
  Ω₅: "verification searches only"
}
```

## 📊 Search Quotas
```
quotas = {
  per_session: {
    Ω₁: unlimited,
    Ω₂: 50,
    Ω₃: 20,
    Ω₄: 0,
    Ω₅: 10
  },
  rate_limit: "10 searches per minute"
}
```

## 🔌 Feature Detection
```
detect_search() = {
  try: tools.find("@modelcontextprotocol-server-brave-search:*"),
  catch: warn("Brave Search MCP not available. Set BRAVE_SEARCH_API_KEY"),
  fallback: suggest("Get API key at https://brave.com/search/api/")
}
```

## 🔗 Integration with Other Services
```
Λ×Θ = search_and_clone() = {
  1: Λ.search_web(query),
  2: extract_github_urls(results),
  3: Θ.clone_repository(selected_url)
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kousha789123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
