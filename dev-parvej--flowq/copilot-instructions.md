## flowq

> - WordPress plugin with proper activation/deactivation hooks

# WordPress Dynamic Survey Plugin - Current Implementation

## What's Currently Implemented

### Core Plugin Structure
- WordPress plugin with proper activation/deactivation hooks
- Database migration system with custom tables
- Internationalization support (text domain: 'flowq')
- Security measures (nonce verification, capability checks)

### Question System
- **Single Question Type Only**: Hardcoded 'single_choice' type in `class-question-manager.php:62`
- Question CRUD operations (create, read, update, delete, duplicate)
- Question fields: title, description, extra_message
- Questions ordered by ID (no custom ordering)

### Answer System
- Multiple answer options per question
- Answer fields: answer_text, answer_value, answer_order
- **Conditional Logic**: Each answer can specify `next_question_id` for branching
- **External Redirects**: Each answer can specify `redirect_url`
- **Skip Question Option**: Questions can be marked as skippable, allowing users to proceed without selecting an answer
- Answer reordering functionality

### Database Schema
Tables created:
- `flowq_surveys`
- `flowq_questions`
- `flowq_answers`
- `flowq_responses`
- `flowq_participants`
- `flowq_sessions`
- `flowq_templates`

### Admin Interface
- Survey management pages
- Question builder with AJAX operations
- Answer options management
- Question duplication
- Analytics and participant tracking
- Shortcode builder
- **Settings Page**: Tabbed navigation interface with multiple tabs
  - **General Tab**: Global form configuration and field visibility
  - **Templates Tab**: Global template selection

#### Survey List (All Surveys) Page
- **Enhanced Survey Cards**: Each card displays comprehensive survey information
  - Form Header (if set)
  - Shortcode (for published surveys)
  - Thank You Page slug (if configured)
  - Question count statistic
  - Total participants, sessions, and responses counts
- **Optimized Database Queries**: Uses LEFT JOIN to fetch question counts in single query (no N+1 problem)
- **2-Column Grid Layout**: Responsive grid showing 2 surveys per row
- **Survey Actions**: Quick access to edit, duplicate, delete, analytics, and question management

#### Add/Edit Survey Page
- **Survey Form Subtitle**: Rich text editor (TinyMCE) for formatted subtitle text
  - Supports bold, italic, underline, links, lists
  - HTML sanitized with `wp_kses_post()`
  - Renders with rich formatting on participant forms
- **Thank You Page Selection**:
  - Dropdown showing all published WordPress pages
  - Pages containing "thank you" or "thankyou" marked with ⭐ star
  - Dynamic "Edit Page" button that updates when selection changes
  - Case-insensitive matching for thank you pages
  - JavaScript-powered page selection and URL generation

### Template System
- **Global Templates**: Template applies to all surveys (not per-survey)
- **Default Templates**: 5 pre-built templates seeded in database
  - Classic - Traditional form-style
  - Modern - Clean, minimalist design
  - Card-based - Elevated card layout
  - Dark Mode - Dark theme with dark backgrounds, white text, and dark input fields
  - Colorful - Vibrant gradient design
- **Template Storage**:
  - Templates table with fields: id, name, description, is_default, preview_image, styles (JSON)
  - Active template stored in options: `flowq_active_template`
  - Preview images: SVG files in `assets/images/templates/`
- **Template Selection UI**: Grid layout with preview cards, active badge, and select buttons
- **Template Handler** (`includes/class-template-handler.php`):
  - Dynamic CSS generation based on template styles
  - Supports customizable colors: primary_color, background_color, text_color
  - Input field theming: input_bg_color, input_border_color, input_text_color
  - Button styles: solid, gradient, elevated
  - Border radius and font family customization
  - Auto-applies to participant forms and question containers
  - Dark theme support with white text on hover for answer options

### Frontend
- Survey display via shortcodes
- AJAX-powered survey navigation
- Session management for participants
- Progress tracking
- Thank you page handling
- **Dynamic Participant Forms**:
  - Two-stage form support (configurable)
  - Single-stage form with all fields combined
  - Privacy policy display with required checkbox
  - Conditional field visibility based on settings
  - Template-aware styling

### Integrations
- REST API endpoints (`class-rest-api.php`)
- Centralized AJAX handling (`class-ajax-handler.php`)

### File Structure
```
flowq/
├── admin/                    # Admin interface
├── includes/                 # Core classes
├── public/                   # Frontend
├── assets/                   # CSS/JS
└── survey-plugin.php        # Main plugin file
```

### Key Classes
- `flowq_Question_Manager`: Question CRUD operations
- `flowq_Question_Admin`: Admin AJAX handlers
- `flowq_Frontend`: Frontend display
- `flowq_Manager`: Survey management
- `flowq_Session_Manager`: Session handling
- `flowq_Settings_Admin`: Settings page with tab navigation
- `flowq_DB_Migrator`: Database migrations including templates table
- `flowq_Template_Handler`: Template management and dynamic CSS generation

### General Settings System
Implemented comprehensive global settings system for form configuration:

**Settings Storage:**
- All settings stored as individual WordPress options (not database table)
- 9 global options for form behavior and privacy policies
- Settings accessible via `get_option()` with default values

**Available Settings:**

1. **Two-Stage Form Toggle** (`flowq_two_stage_form`)
   - Default: `1` (enabled)
   - When enabled: Stage 1 (name, email, address, zipcode) → Stage 2 (phone)
   - When disabled: All fields in single form
   - Fully implemented ✅

2. **Allow Multiple Submissions with Same Email** (`flowq_allow_duplicate_emails`)
   - Default: `0` (disabled)
   - Controls whether same email can submit survey multiple times
   - When disabled: Validates email uniqueness per survey before participant creation
   - When enabled: Allows duplicate email submissions
   - Fully implemented ✅

3. **Participant Information Fields**
   - `flowq_field_address` - Default: `1`
   - `flowq_field_zipcode` - Default: `1`
   - `flowq_field_phone` - Default: `1`
   - Name and Email always required (non-configurable)
   - When phone disabled → Two-stage form automatically disabled
   - Fully implemented ✅

4. **Privacy Policy Text** (Rich HTML content)
   - `flowq_privacy_policy` - Single-stage privacy policy
   - `flowq_privacy_policy_stage1` - Stage 1 privacy policy
   - `flowq_privacy_policy_stage2` - Stage 2 privacy policy
   - Default values include sample text with privacy policy links
   - Sanitized with `wp_kses_post()` for safe HTML
   - Displays with required checkbox before form submission
   - Fully implemented ✅

5. **Optional Phone Number Stage** (`flowq_phone_optional`)
   - Default: `0` (disabled)
   - Only visible when two-stage form enabled
   - Allows users to skip phone number in Stage 2 (not yet implemented)

**Frontend Integration:**
- Participant form dynamically shows/hides fields based on settings
- Email duplication validation before participant creation
  - Checks for existing email per survey when duplicate emails disabled
  - Returns error "You have already submitted this survey with this email address" if duplicate found
- Privacy policy displays above submit buttons with template-aware styling
- JavaScript validation for privacy policy checkboxes
- International zip/postal code validation (supports 4-10 characters, Bangladesh 4-digit codes)
- Form adapts between single-stage and two-stage modes automatically
- Button text changes: "Continue" (two-stage) vs "Start Survey" (single-stage)

**Settings UI:**
- General tab in Settings page (`admin/templates/general-settings.php`)
- Interactive field dependencies via JavaScript:
  - Phone number checkbox controls two-stage form availability
  - Two-stage form checkbox toggles privacy policy editor visibility
  - Two-stage form checkbox controls phone optional setting visibility
- WordPress `wp_editor()` for rich text privacy policy editing
- Save handler in `flowq_Settings_Admin::save_general_settings()`

### Recent Updates

#### Survey Management UI Enhancements
- **Enhanced Survey Cards (`admin/templates/all-surveys.php`)**:
  - Added `.survey-card-details` section showing form header, shortcode, and thank you page slug
  - Added question count as first statistic (4 stats total: questions, participants, sessions, responses)
  - Changed grid layout from 3 columns to 2 columns per row (`repeat(2, 1fr)`)
  - Improved visual hierarchy with detail labels and values
- **Database Query Optimization (`includes/class-survey-manager.php`)**:
  - Fixed N+1 query problem by adding `include_question_count` parameter to `get_surveys()`
  - Implemented LEFT JOIN query to fetch question counts in single database call
  - Updated `flowq_Admin::display_all_surveys_page()` to pass `include_question_count => true`
  - Question counts now pre-fetched and included in survey data array
- **Rich Text Subtitle Editor (`admin/templates/add-survey.php`)**:
  - Converted Survey Form Subtitle from plain textarea to `wp_editor()` (TinyMCE)
  - Configured teeny mode with toolbar: bold, italic, underline, link, lists, undo/redo
  - Changed sanitization from `sanitize_textarea_field()` to `wp_kses_post()` for HTML support
  - Updated frontend output in `participant-form.php` from `esc_html()` to `wp_kses_post()`
  - Subtitle now supports rich formatting (bold, links, lists) on participant forms
- **Thank You Page Dropdown Selection (`admin/templates/add-survey.php`)**:
  - Converted text input to dropdown using `get_pages()` to show all published pages
  - Pages containing "thank you" or "thankyou" (case-insensitive) marked with ⭐ star
  - Added dynamic "Edit Page" button with JavaScript handler
  - Button shows/hides and updates URL based on dropdown selection
  - Improved UX with visual highlighting of recommended pages

#### Technical Implementation Details

**Modified Files:**
1. **`admin/templates/all-surveys.php`**:
   - Added new `.survey-card-details` section with form header, shortcode, and thank you page display
   - Updated statistics section to include question count as first metric
   - Changed CSS grid from `repeat(auto-fill, minmax(400px, 1fr))` to `repeat(2, 1fr)`
   - Updated to use pre-fetched `question_count` from survey data array

2. **`includes/class-survey-manager.php`**:
   - Modified `get_surveys()` method signature to accept `include_question_count` parameter
   - Implemented LEFT JOIN query: `SELECT s.*, COUNT(q.id) as question_count FROM surveys s LEFT JOIN questions q ON s.id = q.survey_id GROUP BY s.id`
   - Updated `create_survey()` and `update_survey()` to use `wp_kses_post()` for form_subtitle
   - Added GROUP BY clause for correct aggregation

3. **`admin/class-admin.php`**:
   - Updated `display_all_surveys_page()` to pass `include_question_count => true` parameter
   - Changed form_subtitle sanitization from `sanitize_textarea_field()` to `wp_kses_post()`

4. **`admin/templates/add-survey.php`**:
   - Replaced form_subtitle textarea with `wp_editor()` call
   - Configured editor with teeny mode, media buttons disabled, limited toolbar
   - Converted thank_you_page_slug text input to `<select>` dropdown
   - Added `get_pages()` call to fetch all published WordPress pages
   - Implemented star marking logic using `stripos()` for case-insensitive matching
   - Added JavaScript handler for dynamic Edit Page button visibility and URL updates
   - Created pages array with ID and post_name mapping for JavaScript

5. **`public/templates/participant-form.php`**:
   - Changed form_subtitle output wrapper from `<p>` to `<div>` with class `survey-form-subtitle`
   - Replaced `esc_html()` with `wp_kses_post()` for subtitle rendering

**Key Code Patterns:**
- **N+1 Prevention**: Always use JOIN queries when fetching related counts in loops
- **Rich Text Sanitization**: Use `wp_kses_post()` for HTML content, not `sanitize_textarea_field()`
- **Page Selection**: Use `get_pages()` with `post_status => 'publish'` for WordPress page lists
- **Dynamic UI**: Use jQuery to handle dropdown changes and update related elements (buttons, URLs)
- **Case-Insensitive Matching**: Use `stripos()` instead of `strpos()` for flexible text matching

#### Previous Updates
- **Email Duplication Validation**: Implemented frontend validation to prevent duplicate email submissions
  - Added `email_exists_for_survey()` method in Participant Manager
  - Validates email uniqueness before participant creation when setting is disabled
  - Returns user-friendly error message for duplicate submissions
- **General Settings System**: Implemented comprehensive form configuration with 5 setting groups
  - Removed Two-Page Survey Form setting (not in current version scope)
  - Completed email duplication validation (Setting #2)
- Added General tab to Settings page with field visibility controls
- Created privacy policy system with template-aware rendering
- Implemented dynamic participant form with conditional field display
- Added international zip code validation (4-10 characters, supports Bangladesh)
- Privacy policy checkboxes with JavaScript validation
- Template handler extended to style privacy policy sections
- Two-stage/single-stage form mode switching based on settings
- Added Settings page with tab navigation (`admin/class-settings-admin.php`)
- Implemented template system with database table and seeded 5 default templates
- Created SVG preview images for all templates
- Built template selection UI with flexbox grid layout
- Created `flowq_Template_Handler` class for centralized template management
- Integrated dynamic CSS generation into participant forms and question containers
- Removed box shadows and borders from containers for cleaner design
- Enhanced Dark Mode template with:
  - Dark container backgrounds (#2d3748)
  - Dark input fields (#1a202c) with proper borders and white text
  - White text on hover for answer options
  - Dark themed skip buttons with hover effects

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dev-parvej)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dev-parvej)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
