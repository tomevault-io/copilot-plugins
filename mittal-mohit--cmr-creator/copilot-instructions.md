## cmr-creator

> for searching JIRA

# JIRA Search Best Practices Rule

## **Core Search Strategy**

When searching JIRA tickets, follow this proven approach:

### **1. Start with Exact Service Name Searches (Most Reliable)**
```sql
project = "Change Management" AND summary ~ "inventory-location-service"
project = "Change Management" AND summary ~ "scm-task-manager"
key = CMR-137573
```

### **2. Use Project-Specific Searches with Absolute Date Ranges (Most Reliable)**
```sql
project = "Change Management" AND created >= "2025-01-01"
project = "Change Management" AND created >= "2025-04-01"
project = "Change Management" AND created >= "2025-06-01"
```

### **3. Search for Technical Terms (More Reliable than Service Names)**
```sql
project = "Change Management" AND (description ~ "P0" OR description ~ "metrics")
project = "Change Management" AND (description ~ "monitoring" OR description ~ "grafana")
project = "Change Management" AND (description ~ "deployment" OR description ~ "canary")
```

### **4. Use Exact Ticket Keys (When Known)**
```sql
key = CMR-137573
```

### **5. Service Name Variations (Less Reliable)**
```sql
project = "Change Management" AND summary ~ "inventory" AND summary ~ "location"
project = "Change Management" AND description ~ "inventory" AND description ~ "location"
```

## **Search Limitations Discovered**

### **❌ What Doesn't Work:**
- **Relative date syntax**: `created >= -6m`, `created >= -3m`, `created >= -1m` (JIRA limitation)
- Using wrong project name: `project = CMR` (should be `project = "Change Management"`)
- Hyphenated service names: `"inventory-location-service"` (in some contexts)
- Case-sensitive searches
- Special characters in service names
- Direct service name searches with hyphens (in some JQL contexts)

### **✅ What Works Reliably:**
- **Correct project name**: `project = "Change Management"`
- **Absolute date ranges**: `created >= "2025-01-01"` (instead of relative dates)
- **Exact service names in summary**: `summary ~ "inventory-location-service"`
- **Exact ticket keys**: `key = CMR-137573`
- **Service names in square brackets**: `[service-name]` pattern
- Project-based searches with correct project name
- Technical terms in descriptions
- Individual service name components

## **Updated Recommended Search Sequence**

1. **Exact Service Name with Correct Project**: `project = "Change Management" AND summary ~ "service-name"`
2. **Exact Ticket Key**: `key = CMR-137573` (if known)
3. **Project + Absolute Date**: `project = "Change Management" AND created >= "2025-01-01"`
4. **Technical Terms**: `project = "Change Management" AND (description ~ "P0" OR description ~ "metrics")`
5. **Service Variations**: `project = "Change Management" AND summary ~ "service-component"`

## **When Searching for Service-Specific Tickets**

1. **Start with exact service name in summary** (most reliable)
2. **Use exact ticket key if known**
3. **Use project-based search with absolute date ranges**
4. **Look for technical terms in descriptions**
5. **Search for individual service name components**
6. **Check assignees or labels for patterns**

## **Result Analysis**

- Always check total count
- If 0 results, try broader time periods using absolute dates
- If results found, examine for patterns
- Report which searches worked vs failed
- Use successful patterns for future searches
- **Always try exact service name first**

## **Example Successful Search**

**Target**: Find JIRA tickets for "inventory-location-service" with P0 metrics

**Working Approach**:
1. `project = "Change Management" AND summary ~ "inventory-location-service"` → **Found CMR-137573**
2. `key = CMR-137573` → **Found CMR-137573**
3. `project = "Change Management" AND created >= "2025-01-01"` → Found multiple tickets
4. Examine results for service-specific content

**Failed Approaches**:
- `project = CMR AND summary ~ "inventory-location-service"` → 0 results (wrong project name)
- `project = "Change Management" AND summary ~ "inventory-location-service" AND created >= -6m` → 0 results (relative date syntax doesn't work)
- `description ~ "inventory-location-service"` → 0 results
- `summary ~ "inventory"` → 0 results (too broad)

## **Key Learning**

**The most reliable search method is using the correct project name "Change Management" with the exact service name in the summary field and absolute date ranges**:

```sql
project = "Change Management" AND summary ~ "service-name" AND created >= "2025-01-01"
```

**Examples:**
- For ILS: `project = "Change Management" AND summary ~ "inventory-location-service" AND created >= "2025-01-01"`
- For SCM Task Manager: `project = "Change Management" AND summary ~ "scm-task-manager" AND created >= "2025-01-01"`

**Recommended Absolute Date Ranges:**
```sql
-- Past 6 months
project = "Change Management" AND summary ~ "service-name" AND created >= "2025-01-01"

-- Past 3 months
project = "Change Management" AND summary ~ "service-name" AND created >= "2025-04-01"

-- Past 1 month
project = "Change Management" AND summary ~ "service-name" AND created >= "2025-06-01"

-- Past 2 weeks
project = "Change Management" AND summary ~ "service-name" AND created >= "2025-07-15"
```

**Critical Note**: JIRA's relative date syntax (`-6m`, `-3m`, `-1m`) is **unreliable** and often returns 0 results even when tickets exist. Always use **absolute dates** for reliable time constraints.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mittal-mohit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
