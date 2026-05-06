## mcp-servers

> Direct database operations without manual SQL in Supabase dashboard.

# MCP Servers Available

## Supabase MCP
Direct database operations without manual SQL in Supabase dashboard.

### Common Operations
```
# List tables
mcp_supabase_list_tables

# Execute SQL query
mcp_supabase_execute_sql(query: "SELECT * FROM project_summary LIMIT 5")

# Apply migration
mcp_supabase_apply_migration(name: "add_new_column", query: "ALTER TABLE...")

# List migrations
mcp_supabase_list_migrations

# Generate TypeScript types
mcp_supabase_generate_typescript_types

# Get logs for debugging
mcp_supabase_get_logs(service: "postgres" | "auth" | "storage")

# Security/performance advisors
mcp_supabase_get_advisors(type: "security" | "performance")
```

### Edge Functions
```
# List functions
mcp_supabase_list_edge_functions

# Deploy function
mcp_supabase_deploy_edge_function(name: "my-function", files: [...])
```

### When to Use
- Running migrations
- Debugging database issues
- Generating fresh TypeScript types
- Checking for security issues after schema changes

---

## Vercel MCP
Deployment and project management.

### Common Operations
```
# Deploy current project
mcp_vercel_deploy_to_vercel

# List projects
mcp_vercel_list_projects(teamId: "...")

# Get deployment logs
mcp_vercel_get_deployment_build_logs(idOrUrl: "...", teamId: "...")

# Check domain availability
mcp_vercel_check_domain_availability_and_price(names: ["example.com"])
```

### When to Use
- Deploying to production
- Debugging failed deployments
- Checking build logs

---

## Linear MCP
Issue and project tracking.

### Common Operations
```
# List issues
mcp_linear_list_issues(team: "...", assignee: "me")

# Create issue
mcp_linear_create_issue(title: "...", team: "...", description: "...")

# Update issue status
mcp_linear_update_issue(id: "...", state: "done")

# List projects
mcp_linear_list_projects

# Create comment
mcp_linear_create_comment(issueId: "...", body: "...")
```

### When to Use
- Creating tasks from development work
- Updating issue status
- Tracking bugs and features

---

## shadcn MCP
Component registry for UI components.

### Common Operations
```
# Search for components
mcp_shadcn_search_items_in_registries(registries: ["@shadcn"], query: "button")

# View component details
mcp_shadcn_view_items_in_registries(items: ["@shadcn/button"])

# Get usage examples
mcp_shadcn_get_item_examples_from_registries(registries: ["@shadcn"], query: "button-demo")

# Get install command
mcp_shadcn_get_add_command_for_items(items: ["@shadcn/button", "@shadcn/card"])
```

### When to Use
- Adding new UI components
- Finding component examples
- Checking component APIs

---

## Firecrawl MCP
Web scraping and search.

### Common Operations
```
# Scrape a page
mcp_firecrawl_firecrawl_scrape(url: "https://example.com", formats: ["markdown"])

# Search the web
mcp_firecrawl_firecrawl_search(query: "Next.js 15 new features", limit: 5)

# Map a website (discover URLs)
mcp_firecrawl_firecrawl_map(url: "https://docs.example.com")
```

### When to Use
- Researching documentation
- Finding code examples
- Checking competitor features

---

## Browser MCP (cursor-ide-browser)
Browser automation for testing.

### Common Operations
```
# Navigate to URL
mcp_cursor-ide-browser_browser_navigate(url: "http://localhost:3000")

# Take snapshot (for element refs)
mcp_cursor-ide-browser_browser_snapshot

# Click element
mcp_cursor-ide-browser_browser_click(element: "Add Project button", ref: "...")

# Type text
mcp_cursor-ide-browser_browser_type(element: "Address input", ref: "...", text: "123 Main St")

# Take screenshot
mcp_cursor-ide-browser_browser_take_screenshot(name: "dashboard")

# Check console errors
mcp_cursor-ide-browser_browser_console_messages
```

### Testing Flow
1. Navigate to page
2. Snapshot to get element refs
3. Interact with elements
4. Re-snapshot to verify changes
5. Screenshot for visual verification

---

## Puppeteer MCP
Alternative browser automation.

### Common Operations
```
# Navigate
mcp_puppeteer_puppeteer_navigate(url: "http://localhost:3000")

# Screenshot
mcp_puppeteer_puppeteer_screenshot(name: "test")

# Click
mcp_puppeteer_puppeteer_click(selector: ".button-class")

# Fill form
mcp_puppeteer_puppeteer_fill(selector: "input[name='address']", value: "123 Main St")

# Execute JS
mcp_puppeteer_puppeteer_evaluate(script: "document.title")
```

---

## Workflow Examples

### After Schema Changes
1. `mcp_supabase_apply_migration` - Apply the migration
2. `mcp_supabase_generate_typescript_types` - Update types
3. `mcp_supabase_get_advisors(type: "security")` - Check for RLS issues

### Adding New UI Component
1. `mcp_shadcn_search_items_in_registries` - Find component
2. `mcp_shadcn_get_item_examples_from_registries` - See examples
3. `mcp_shadcn_get_add_command_for_items` - Get install command

### Testing Feature
1. `mcp_cursor-ide-browser_browser_navigate` - Go to page
2. `mcp_cursor-ide-browser_browser_snapshot` - Get elements
3. Interact and verify
4. `mcp_cursor-ide-browser_browser_console_messages` - Check for errors

### Deploy to Production
1. `mcp_vercel_deploy_to_vercel` - Deploy
2. `mcp_vercel_get_deployment_build_logs` - Check for errors

---
> Source: [adamj-ops/rehab-budget-pro](https://github.com/adamj-ops/rehab-budget-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
