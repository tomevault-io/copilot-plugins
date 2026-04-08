## devq-starter-theme

> - **Theme root:** This directory

# DevQ Starter Theme - Claude Block Generator Instructions

## Theme Architecture

- **Theme root:** This directory
- **Block location:** `blocks/[blockname]/code.php`
- **Block registration:** `functions/blocks.php` — `$basefunctions` array
- **ACF JSON:** `acfjson/group_[blockname]_block.json`
- **Spacing system:** `functions/spacing.php` — centralized responsive spacing
- **Animation helper:** `functions/animations.php` — `devq_aos()` helper for AOS attribute generation
- **Page builder:** `functions/page-builder.php` — programmatic page creation
- **Page presets:** `functions/page-presets.php` — common page layouts
- **Theme settings CSS:** `theme-settings-css.php` — generates CSS variables from ACF options
- **Main stylesheet:** `style.css`
- **Theme updater:** `functions/theme-updater.php` — GitHub release update checker
- **Release workflow:** `.github/workflows/release.yml` — auto-builds release zip on tag push
- **Child theme boilerplate:** `devq-starter-child/` — copy per client site

## Releasing Updates

This theme auto-updates on all sites via a public GitHub repo and `plugin-update-checker`. When a new GitHub release is published, every site running this theme will see the update in WP Admin.

### Release Steps

1. Bump `Version:` in `style.css` (this is the single source of truth)
2. Commit the change
3. Tag and push:
   ```bash
   git tag vX.Y.Z && git push origin master vX.Y.Z
   ```
4. GitHub Actions automatically builds the zip and creates the release — done

### Version Format

Use semver: `MAJOR.MINOR.PATCH`
- **Patch** (1.0.1) — bug fixes, copy changes
- **Minor** (1.1.0) — new blocks, new features
- **Major** (2.0.0) — breaking changes to child themes or block structure

### Do NOT

- Manually create releases on GitHub (the Actions workflow handles this)
- Edit anything in the `plugin-update-checker/` directory (third-party library)

### Child Themes

Per-site customizations (colors, fonts, extra blocks, templates) belong in child themes, not in this repo. Copy `devq-starter-child/` as a starting point for each client.

## Field Naming Convention

All new blocks MUST follow this naming convention. Existing blocks may use legacy names (documented below) but new work should be consistent.

### Standard Field Names

| Purpose | Field Name | Type | Notes |
|---------|-----------|------|-------|
| Main heading | `heading` | text | Never use `title` at block level |
| Above heading | `eyebrow` | text | Small label above heading |
| Below heading | `subheading` | text | Never use `subtitle` or `description` at block level |
| Rich text body | `content` | wysiwyg | Never use `text` or `body` |
| Single button | `button` | link | For blocks with one CTA |
| Primary button | `primary_button` | link | For blocks with two CTAs |
| Secondary button | `secondary_button` | link | For blocks with two CTAs |
| Main image | `image` | image | Generic image field |
| Background image | `background_image` | image | For hero/section backgrounds |
| Background color | `background_color` | color_picker | — |
| Background style | `background` | select | light/dark/primary/secondary |
| Overlay opacity | `overlay_opacity` | number | 0-100 percentage |
| Overlay color | `overlay_color` | color_picker | — |
| Layout toggle | `image_position` | select | left/right |
| Column count | `columns` | select | 2/3/4 |
| Display style | `style` | select | Block-specific options |
| Form embed | `form_shortcode` | text | Gravity Forms shortcode |

### Repeater Item Fields

| Purpose | Field Name | Notes |
|---------|-----------|-------|
| Item heading | `title` | Use `title` inside repeaters (not `heading`) |
| Item body | `description` | Use `description` inside repeaters (not `content`) |
| Item image | `image` | Generic |
| Person photo | `photo` | For team/testimonial repeaters |
| Person name | `name` | — |
| Person role | `role` | Job title / position |
| Person quote | `quote` | For testimonial repeaters |
| Logo image | `logo` | For logo bar / marquee repeaters |
| Item link | `link` | — |
| Icon class | `icon_class` | FontAwesome class string |
| Featured flag | `is_featured` | true_false for highlighted items |
| Star rating | `rating` | number 1-5 |

### Legacy Field Names (existing blocks)

These fields exist in current blocks and should NOT be renamed (would break client sites):
- FAQ uses `question`/`answer` instead of `title`/`description` — acceptable, domain-specific
- Banner uses `text` instead of `content` — legacy, use `content` for new blocks
- Stats repeater uses `number`/`label`/`prefix`/`suffix` — acceptable, domain-specific
- Pricing repeater uses `price`/`period`/`features` — acceptable, domain-specific

## Creating a New Block

### Step 1: Register the Block

Add the human-readable name to the `$basefunctions` array in `functions/blocks.php`:

```php
$basefunctions = array(
    "Image",
    "Content",
    "Your New Block",  // Add here
);
```

The `devq_filtername()` function processes the name:
- Converts to lowercase
- Removes spaces and hyphens
- Result becomes the folder name and ACF identifier
- Example: `"Your New Block"` → folder `yournewblock`, ACF block `acf/yournewblock`

### Step 2: Create the Block Folder and code.php

Create `blocks/[filteredname]/code.php`. Use this exact template:

```php
<?php
/**
 * [Block Name] Block
 */

// Error handling
if (!function_exists('get_field')) {
    echo 'ACF plugin is not active. This block requires ACF to function properly.';
    return;
}

// ACF Fields - Content Tab
$field1 = get_field('field1');

// Options Tab Fields (ALWAYS include these)
$margin_top = get_field('margin_top') ?: '';
$margin_bottom = get_field('margin_bottom') ?: '';
$margin_top_other = get_field('margin_top_other') ?: 0;
$margin_bottom_other = get_field('margin_bottom_other') ?: 0;
$custom_class = get_field('custom_class');
$custom_id = get_field('custom_id');

// Animation Tab Fields (ALWAYS include these)
$animation_type = get_field('animation_type') ?: 'recommended';
$animation_duration = get_field('animation_duration') ?: 800;
$disable_animation = get_field('disable_animation');

// Generate unique block ID
$unique_block_id = generate_unique_block_id('blockname');

// Build dynamic attributes
$block_classes = 'container-fluid blockname-block';
if ($custom_class) {
    $block_classes .= ' ' . $custom_class;
}

$block_id = $custom_id ? $custom_id : $unique_block_id;

// Build AOS attributes
$is_recommended = ($animation_type === 'recommended');
$aos_attributes = '';
if (!$disable_animation && !$is_recommended) {
    $aos_attributes = 'data-aos="' . esc_attr($animation_type) . '"';
    if ($animation_duration != 800) {
        $aos_attributes .= ' data-aos-duration="' . esc_attr($animation_duration) . '"';
    }
}
// For recommended mode, use devq_aos() helper on individual elements
// $animate = (!$disable_animation && $is_recommended);

// Check required fields
if (!$field1) {
    echo 'Please fill out all required fields for this block.';
    return;
}

?>

<div class="<?php echo esc_attr($block_classes); ?>" <?php echo $block_id ? 'id="' . esc_attr($block_id) . '"' : ''; ?> <?php echo $aos_attributes; ?>>
    <div class="container">
        <!-- Block content here -->
    </div>
</div>

<?php
// Output responsive spacing CSS using unique block ID
output_block_spacing_css($margin_top, $margin_bottom, $margin_top_other, $margin_bottom_other, $block_id);
?>

<style>
.blockname-block {
    /* Base styles */
}

/* Tablet - 1199px and below */
@media (max-width: 1199px) {
    .blockname-block {
        /* Tablet styles */
    }
}

/* Mobile - 767px and below */
@media (max-width: 767px) {
    .blockname-block {
        /* Mobile styles */
    }
}
</style>
```

### Step 3: Create the ACF JSON

Create `acfjson/group_[blockname]_block.json`. Every block MUST have 3 tabs:

1. **Content** — Block-specific fields
2. **Options** — Margin Top, Margin Top Other, Margin Bottom, Margin Bottom Other, Custom Class, Custom ID
3. **Animation** — Animation Type (select), Animation Duration (number), Disable Animation (true/false)

Use `blocks/image/code.php` and `acfjson/group_image_block.json` as the reference implementation.

#### ACF JSON Key Naming Convention

All field keys must be prefixed with the block name to avoid collisions:
- `field_[blockname]_content_tab`
- `field_[blockname]_[fieldname]`
- `field_[blockname]_options_tab`
- `field_[blockname]_margin_top`
- `field_[blockname]_animation_tab`
- `field_[blockname]_animation_type`

#### Options Tab Fields (copy exactly)

| Field | Type | Width | Choices | Conditional |
|-------|------|-------|---------|------------|
| Margin Top | select | 50% | none/small/medium/large/other | — |
| Margin Top Other | number | 50% | append: "px", min: 0 | margin_top == other |
| Margin Bottom | select | 50% | none/small/medium/large/other | — |
| Margin Bottom Other | number | 50% | append: "px", min: 0 | margin_bottom == other |
| Custom Class | text | 50% | prepend: "." | — |
| Custom ID | text | 50% | prepend: "#" | — |

#### Animation Tab Fields (copy exactly)

| Field | Type | Width | Default |
|-------|------|-------|---------|
| Animation Type | select | 50% | recommended |
| Animation Duration | number | 25% | 800, append: "ms", min: 300, max: 3000 |
| Disable Animation | true_false | 25% | 0 |

Animation Type choices: **recommended** (first/default), fade-up, fade-down, fade-left, fade-right, fade-up-right, fade-up-left, fade-down-right, fade-down-left, flip-up, flip-down, flip-left, flip-right, slide-up, slide-down, slide-left, slide-right, zoom-in, zoom-in-up, zoom-in-down, zoom-in-left, zoom-in-right, zoom-out

### Animation System

The animation system has two modes controlled by the Animation Type field:

**Recommended mode** (`$is_recommended = true`): Smart per-element animations tailored to each block type. No AOS on the outer container. Instead, individual elements get `devq_aos()` calls with staggered delays. Use `functions/animations.php` helper:

```php
// Helper: devq_aos($type, $delay, $duration) returns AOS data attributes string
echo devq_aos('fade-up', 100, $animation_duration);
// Output: data-aos="fade-up" data-aos-delay="100"
```

**Manual mode** (any other type): AOS applied to the outer container as a whole. No per-element animation.

#### Recommended Animation Patterns by Block Type

| Category | Blocks | Recommended Behavior |
|----------|--------|---------------------|
| Hero stagger | hero, herovideo, herofullscreen | Per-element fade-up with 100ms stagger |
| Hero slider | heroslider | No AOS (Slick handles transitions) |
| Split layouts | herosplit, textimage, about, contactsplit | Opposing directions based on layout (text fades from its side, image from opposite) |
| Card/grid stagger | cards, team, pricing, stats, featureslist, process, blogposts, testimonials | Header fade-up, items stagger fade-up |
| Timeline | timeline | Header fade-up, items directional (left items fade-right, right items fade-left) |
| Gallery/logobar | gallery, logobar | Header fade-up, items stagger fade-up |
| CTA stagger | cta | Per-element fade-up stagger |
| Banner | banner | fade-down (slides from top) |
| Header+content | video, beforeafter, tabs, comparisontable | Header fade-up, content fade-up with delay |
| Simple | content, faq, map, marquee | fade-up on container |
| Image | image | zoom-in on container |

## CSS Rules

### CSS Variables (set dynamically from Theme Settings)

```css
--primary          /* Primary brand color */
--secondary        /* Secondary brand color */
--tertiary         /* Accent color */
--font1            /* Heading font family */
--font2            /* Body font family */
--heading-weight   /* Heading font weight */
--body-weight      /* Body font weight */
--body-size        /* Body font size */
--button-radius    /* Button border radius */
--button-padding   /* Button padding */
--spacing-small    /* Small spacing value */
--spacing-medium   /* Medium spacing value */
--spacing-large    /* Large spacing value */
--section-padding-top     /* Section top padding */
--section-padding-bottom  /* Section bottom padding */
--transition-default      /* Standard transition: all 0.3s ease-in-out */
--transition-fast         /* Fast transition: all 0.1s ease-in-out */
```

### Class Naming

- All CSS classes MUST be unique to the block: `.blockname-block`, `.blockname-content`, etc.
- Outer wrapper: `container-fluid blockname-block`
- Inner wrapper: `container`
- NEVER set font-size on h1-h6, p, ul, ol, li (globally controlled)

### Breakpoints

- **Tablet:** `@media (max-width: 1199px)`
- **Mobile:** `@media (max-width: 767px)`
- Only use these two breakpoints. Do NOT use 991px.

### Buttons

Use the `.btn` class system for all buttons. Do NOT use `.btn-inline`.

| Class | Usage |
|-------|-------|
| `btn` | Primary button (solid primary bg, white text) |
| `btn btn-secondary` | Secondary color variant |
| `btn btn-tertiary` | Tertiary color variant |
| `btn btn-white` | White bg, primary text (for dark backgrounds) |
| `btn btn-outline` | Transparent bg, primary border/text |
| `btn btn-outline-secondary` | Transparent bg, secondary border |
| `btn btn-outline-white` | Transparent bg, white border/text (for dark backgrounds) |

## Escaping Rules

- Text content: `esc_html()`
- URLs: `esc_url()`
- HTML attributes: `esc_attr()`
- Rich text / WYSIWYG: `wp_kses_post()`
- CSS in style tags: `wp_strip_all_tags()`

## Available JS Libraries

Always loaded:
- **jQuery** (WordPress bundled)
- **AOS** — Scroll animations (initialized in footer)
- **Mobile Menu** — Custom vanilla JS (`assets/js/mobile-menu.js`)

Conditionally loaded (auto-detected from page content):
- **Slick** — Carousel/slider (loaded when Hero Slider or Testimonials blocks present)
- **BeefUp** — Accordion (loaded when FAQ block present)
- **Magnific Popup** — Lightbox/modal (loaded when Gallery block present)

Available but commented out in `functions/scripts.php` (uncomment when needed):
- **jQuery Validate** — Form validation

## Repeater Field Pattern

```php
<?php if (have_rows('items')) :
    while (have_rows('items')) : the_row();
        $title = get_sub_field('title');
        $content = get_sub_field('content');
        ?>
        <div class="blockname-item">
            <h3><?php echo esc_html($title); ?></h3>
            <?php echo wp_kses_post($content); ?>
        </div>
        <?php
    endwhile;
endif; ?>
```

## Testing Checklist

After creating a block:
1. All 3 tabs appear (Content, Options, Animation)
2. Margin options work (None/Small/Medium/Large/Other)
3. Custom class and ID applied correctly
4. AOS animation works and can be disabled
5. Responsive design correct at 1199px and 767px
6. No console JS errors
7. No PHP errors
8. All output properly escaped

## Block Category

All blocks register under the `devq` category. The block slug is `acf/[filteredname]`.

## Spacing System

The spacing system is centralized in `functions/spacing.php`:
- `generate_unique_block_id($type)` — Creates a unique ID for each block instance
- `output_block_spacing_css($top, $bottom, $top_other, $bottom_other, $id)` — Outputs responsive `<style>` tag
- Desktop values: Small (20px), Medium (40px), Large (80px) — configurable in Theme Settings
- Mobile values: Small (15px), Medium (25px), Large (40px) — auto-applied at 767px

## Available Blocks (30 total)

### Heroes
| Block | Slug | Description |
|-------|------|-------------|
| Hero | `acf/hero` | Full-width hero with background image/gradient/solid, overlay, CTAs. Layout tab: alignment, width, height, vertical position, scroll indicator |
| Hero Split | `acf/herosplit` | Two-column hero: text + image, position toggle |
| Hero Video | `acf/herovideo` | Background video hero (YouTube/upload) with overlay, fallback image for mobile, Layout tab |
| Hero Slider | `acf/heroslider` | Multi-slide hero carousel using Slick, per-slide bg/overlay/content, Layout tab with slider options |
| Hero Fullscreen | `acf/herofullscreen` | Always 100vh hero with animated scroll indicator, Layout tab |

### Content
| Block | Slug | Description |
|-------|------|-------------|
| Text Image | `acf/textimage` | Two-column text + image with position toggle |
| Content | `acf/content` | Rich text content with width options (narrow/default/wide) |
| About | `acf/about` | Company intro with image, text, optional stat boxes |
| Blog Posts | `acf/blogposts` | Dynamic post grid via WP_Query, category filter, 2/3/4 columns |
| Tabs | `acf/tabs` | Tabbed content (horizontal/vertical), accordion on mobile |

### Cards & Grids
| Block | Slug | Description |
|-------|------|-------------|
| Cards | `acf/cards` | Icon/image cards in 2/3/4 columns with hover lift |
| Team | `acf/team` | Team member cards with photo, role, social links |
| Pricing | `acf/pricing` | Pricing tier cards with featured highlight |
| Comparison Table | `acf/comparisontable` | Feature comparison matrix with highlighted columns |

### Social Proof
| Block | Slug | Description |
|-------|------|-------------|
| Testimonials | `acf/testimonials` | Quote cards with carousel or grid mode |
| Logo Bar | `acf/logobar` | Client/partner logos, grayscale option |
| Stats | `acf/stats` | Animated counter numbers with light/dark/primary bg |
| Marquee | `acf/marquee` | Infinite scrolling text or logos, CSS animation |

### Media
| Block | Slug | Description |
|-------|------|-------------|
| Image | `acf/image` | Full/contained image with mobile swap option |
| Gallery | `acf/gallery` | Filterable image grid with lightbox |
| Video | `acf/video` | YouTube/Vimeo embed with thumbnail play button |
| Map | `acf/map` | Map iframe embed, contained or full-width |
| Before/After | `acf/beforeafter` | Draggable image comparison slider |

### Conversion
| Block | Slug | Description |
|-------|------|-------------|
| Banner | `acf/banner` | Thin announcement/promo strip, dismissible |
| CTA | `acf/cta` | Full-width accent section with heading + button |
| Contact Split | `acf/contactsplit` | Two-column: contact info + form embed |

### Lists
| Block | Slug | Description |
|-------|------|-------------|
| FAQ | `acf/faq` | Accordion FAQ using BeefUp |
| Process | `acf/process` | Numbered steps with icons |
| Features List | `acf/featureslist` | Two-column icon + text feature items |
| Timeline | `acf/timeline` | Alternating vertical timeline for milestones |

## Child Theme Blocks

Child themes can override existing blocks and register new ones without modifying the parent theme.

### Override Precedence

| Asset | Resolution | Mechanism |
|-------|-----------|-----------|
| `code.php` | Child theme first | ACF's `locate_template()` fallback |
| `style.css` | Child theme first | `register_acf_block_types()` checks `get_stylesheet_directory()` before `get_template_directory()` |
| `script.js` | Child theme first | Same as style.css |
| ACF JSON | Child theme first | `acf/settings/load_json` in `functions/acf.php` |

### Adding a New Block from a Child Theme

1. Add the block name via the `devq_blocks` filter in the child theme's `functions.php`:
   ```php
   add_filter('devq_blocks', function ($blocks) {
       $blocks[] = 'My Custom Block';
       return $blocks;
   });
   ```
2. Create `blocks/mycustomblock/code.php` in the child theme
3. Create `acfjson/group_mycustomblock_block.json` in the child theme
4. Optionally add `blocks/mycustomblock/style.css` and/or `script.js`

The block is automatically allowed in the editor — `devq_allowed_block_types()` calls `devq_get_blocks()`, which applies the filter.

### Overriding an Existing Block

- **Template:** Copy the parent's `blocks/[name]/code.php` to the same path in the child theme. No filter needed.
- **Styles:** Copy `blocks/[name]/style.css` to the child theme. The child version replaces (not supplements) the parent's.
- **Scripts:** Copy `blocks/[name]/script.js` to the child theme. Same behavior as styles.

### Removing a Parent Block

```php
add_filter('devq_blocks', function ($blocks) {
    return array_diff($blocks, array('Marquee', 'Timeline'));
});
```

## New Site Setup

When asked to set up a new client site, follow this two-phase sequence:

### Phase 1 — Bootstrap (before theme is active)

1. **Read config:** `~/.devq/config.json` — contains plugin zip paths, license keys, and default page list.

2. **Install plugins:**
   ```bash
   wp plugin install "{acf_zip_path}" --activate
   wp plugin install "{gf_zip_path}" --activate
   wp plugin install wordpress-seo --activate
   ```

3. **Activate licenses:**
   ```bash
   # ACF Pro
   wp eval 'update_option("acf_pro_license", "{acf_license_key}");'

   # Gravity Forms
   wp eval 'GFFormsModel::save_key("{gf_license_key}");'
   ```

4. **Bootstrap theme + child theme:**
   ```bash
   wp eval-file "wp-content/themes/devq-starter/scripts/bootstrap.php" "Client Name"
   ```
   This copies `devq-starter-child/` → `clientname-child/`, replaces "Client Name" in `style.css`, and activates the child theme.

### Phase 2 — Content scaffold (requires active theme)

5. **Scaffold content:**
   ```bash
   wp eval-file "wp-content/themes/devq-starter/scripts/setup-site.php"
   ```
   This creates pages from presets, sets Home as front page, builds the Primary Menu, deletes default content (Sample Page, Hello world!), and sets permalinks to `/%postname%/`.

   To use custom pages: `wp eval-file "...setup-site.php" home about services contact landing`

### Verification

6. **Verify everything worked:**
   ```bash
   wp theme status
   wp plugin list
   wp post list --post_type=page --fields=ID,post_title,post_status
   wp option get page_on_front
   wp menu list
   ```

### Important Notes

- `~/.devq/config.json` is machine-level config (secrets stay out of git)
- `scripts/bootstrap.php` is standalone — no theme dependency
- `scripts/setup-site.php` requires the theme to be active (uses `devq_create_page()`)
- Available presets: `home`, `about`, `contact`, `services`, `landing`

## Programmatic Page Creation

### Creating a Page with Blocks

Use `devq_create_page()` to create pages programmatically:

```php
$post_id = devq_create_page(array(
    'title' => 'About Us',
    'slug' => 'about',
    'status' => 'publish',
    'blocks' => array(
        array(
            'name' => 'Hero',
            'fields' => array(
                'heading' => 'About Our Company',
                'subheading' => 'Learn more about what we do',
                'overlay_opacity' => 60,
            ),
        ),
        array(
            'name' => 'Content',
            'fields' => array(
                'content' => '<p>Company description here.</p>',
            ),
        ),
    ),
));
```

### Using Presets

Presets define common page layouts with empty placeholder fields:

```php
// Get all presets
$presets = devq_get_page_presets();

// Create a page from a preset
$post_id = devq_create_page(array(
    'title' => 'Home',
    'status' => 'publish',
    'blocks' => $presets['home'],
));
```

Available presets: `home`, `about`, `contact`, `services`, `landing`

### WP-CLI Commands

```bash
# Create a single page with preset
wp devq create-page --title="About" --preset=about --status=draft

# Bulk create from JSON file
wp devq bulk-create --file=pages.json

# List available presets
wp devq list-presets

# List registered blocks
wp devq list-blocks
```

### REST API (fallback when WP-CLI unavailable)

```bash
curl -X POST http://localhost/wp-json/devq/v1/create-page \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{"title":"About","preset":"about","status":"draft"}'
```

### Bulk Page Creation Workflow

1. Create pages from presets: `wp devq create-page --title="Home" --preset=home --status=publish`
2. Or run the setup script: `wp eval-file "wp-content/themes/devq-starter/scripts/setup-site.php"`
3. Edit pages in WP admin to fill in real content
4. See the "New Site Setup" section above for the full orchestration workflow

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jyager31)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jyager31)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
