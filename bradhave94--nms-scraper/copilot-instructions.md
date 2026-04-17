## nms-scraper

> Wiki scraping and parsing patterns


# Wiki Scraping Patterns

## MediaWiki API Usage

**Base URL**: `https://nomanssky.fandom.com/api.php`

### Category Members API
```python
params = {
    'action': 'query',
    'list': 'categorymembers',
    'cmtitle': f'Category:{category}',
    'cmlimit': 500,
    'format': 'json',
    'formatversion': '2'
}
```

### Page Content API
```python
params = {
    'action': 'query',
    'prop': 'revisions',
    'titles': title,
    'rvprop': 'content',
    'format': 'json',
    'formatversion': '2'
}
```

## Wiki Markup Parsing Patterns

### Infobox Extraction
```python
# Match infobox templates
infobox_pattern = r'\{\{([^}]+infobox[^}]*)\}\}'
# Extract key-value pairs
field_pattern = r'\|\s*([^=\|]+?)\s*=\s*([^|\n]+?)(?=\s*[\|\n])'
```

### Content Section Extraction
```python
# Flexible section parsing (handles spacing variations)
summary_pattern = r'==\s*Summary\s*==\s*(.*?)(?=\s*==|\s*\{\{|\Z)'
game_desc_pattern = r'==\s*Game description\s*==\s*(.*?)(?=\s*==|\s*\{\{|\Z)'
source_pattern = r'==\s*Source\s*==\s*(.*?)(?=\s*==|\s*\{\{|\Z)'
use_pattern = r'==\s*Use\s*==\s*(.*?)(?=\s*==|\s*\{\{|\Z)'
release_pattern = r'==\s*Release history\s*==\s*(.*?)(?=\s*==|\s*\{\{|\Z)'
additional_pattern = r'==\s*Additional information\s*==\s*(.*?)(?=\s*==|\s*\{\{|\Z)'
fishing_pattern = r'==\s*Fishing Bait\s*==\s*(.*?)(?=\s*==|\s*\{\{|\Z)'
progression_pattern = r'==\s*Resource progression\s*==\s*(.*?)(?=\s*==|\s*\{\{|\Z)'
```

### Category Extraction and Cleaning
```python
# Extract categories with pipe handling
category_pattern = r'\[\[Category:([^\]|]+)'
# Clean version suffixes
cleaned_cat = re.sub(r'\s*\([^)]+\)\s*$', '', category)
# Fallback to infobox category if no explicit categories
if not categories and 'category' in infobox:
    categories.append(infobox['category'])
```

### Crafting Recipe Parsing
```python
# {{Craft}} template parsing
craft_pattern = r'\{\{Craft\|([^}]+)\}\}'
# Handle blueprint parameter: skip items with blueprint=yes
if 'blueprint=yes' in material or '=' in material:
    continue
```

### Recipe Template Patterns
- `{{PoC-Refine}}` - Primary refinery recipe template format
  ```
  {{PoC-Refine
  | Input1,qty;Output,qty;Time;Operation%Description
  | Input1,qty;Input2,qty;Output,qty;Time;Operation%Description
  }}
  ```
- `{{Cook}}` - Nutrient processor recipes with multiple variations
  ```
  {{Cook
  | Input1,qty + Input2,qty + Input3,qty → Output,qty ("Operation", Time)
  | Input1,qty + Alt2,qty + Input3,qty → Output,qty ("Operation", Time)
  }}
  ```
- `{{Craft}}` - Basic crafting recipes

## Advanced Filtering System

### Category-Based Filtering
Automatically skip pages with these categories:
- `Category:NPC` - NPCs and merchants
- `Category:Flora` - Plant species
- `Category:Fauna` - Animal species
- `Category:Minerals` - Location-specific minerals
- `Category:Album` - Photo albums and catalogs
- `Category:Mechanics` - Game mechanics/systems
- `Category:Multi-Tool` - Multi-tool pages
- `Category:Cuboid Room` - Room components
- `Category:Container` - Container items
- `Category:Artifact` (conditional) - Only listing/disambiguation pages

### Title-Based Filtering
Skip specific pages by exact title:
```python
skip_titles = [
    'Travel',      # Travel guide page
    'Artifact',    # Artifact listing page
    'Multi-Tool',  # Multi-tool overview page
]
```

### Pattern-Based Filtering
Skip pages with specific content patterns:
- Obsolete items: `{{Version|Obsolete}}`
- Location minerals: `{{Mineral infobox`
- Visual catalogs: "is a visual catalogue"
- Disambiguation: `{{disambig}}`
- Resource divisions: "are one of the divisions"

## Classification Keywords

### Buildings
- "Base Building", "Construction", "Decorative", "Furniture"
- Infobox types: "Building Part", "Decoration"

### Technology
- "Technology", "Upgrade", "Module", "Enhancement"
- Infobox types: "Technology", "Upgrade Module"

### Products
- "Product", "Component", "Manufactured"
- Infobox types: "Product", "Component"

### Raw Materials
- "Element", "Resource", "Raw", "Mineral"
- Infobox types: "Element", "Resource"

### Agricultural Substances
- "Harvested Agricultural Substance", "Flora elements", "Flora"
- Examples: Faecium, Frost Crystal, Fungal Mould, Gamma Root, Solanium, Star Bulb

## Recipe Extraction Process

1. **Refinery Recipes**:
   - Check ALL items in database for `{{PoC-Refine}}` templates
   - Parse format: `Input1,qty;Input2,qty;Output,qty;Time;Operation%Description`
   - Handle multiple recipe variations within a single template

2. **Nutrient Processor Recipes**:
   - Check cooking items for `{{Cook}}` templates
   - Parse format: `Input1,qty + Input2,qty + Input3,qty → Output,qty ("Operation", Time)`
   - Split multiple recipe variations separated by `|`
   - Handle missing ingredients with placeholder IDs (e.g., `missing_wailing_batter`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bradhave94) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
