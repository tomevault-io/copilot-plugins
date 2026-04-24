## singularity-labs-site

> UI Components API reference and guidelines


# UI Components

- Reusable UI components are defined in the "packages/ui" package named "@kit/ui".
- By exporting the component from the "exports" field, we can import it using the "@kit/ui/{component-name}" format.

## Styling
- Styling is done using Tailwind CSS. We use the "cn" function from the "@kit/ui/utils" package to generate class names.
- Avoid fixes classes such as "bg-gray-500". Instead, use Shadcn classes such as "bg-background", "text-secondary-foreground", "text-muted-foreground", etc.

Makerkit leverages two sets of UI components:
1. **Shadcn UI Components**: Base components from the Shadcn UI library
2. **Makerkit-specific Components**: Custom components built on top of Shadcn UI

## Importing Components

```tsx
// Import Shadcn UI components
import { Button } from '@kit/ui/button';
import { Card } from '@kit/ui/card';

// Import Makerkit-specific components
import { If } from '@kit/ui/if';
import { Trans } from '@kit/ui/trans';
import { ProfileAvatar } from '@kit/ui/profile-avatar';
```

## Core Shadcn UI Components

| Component | Description | Import Path |
|-----------|-------------|-------------|
| `Accordion` | Expandable/collapsible content sections | `@kit/ui/accordion` |
| `AlertDialog` | Modal dialog for important actions | `@kit/ui/alert-dialog` |
| `Alert` | Status/notification messages | `@kit/ui/alert` |
| `Avatar` | User profile images with fallback | `@kit/ui/avatar` |
| `Badge` | Small status indicators | `@kit/ui/badge` |
| `Breadcrumb` | Navigation path indicators | `@kit/ui/breadcrumb` |
| `Button` | Clickable action elements | `@kit/ui/button` |
| `Calendar` | Date picker and date display | `@kit/ui/calendar` |
| `Card` | Container for grouped content | `@kit/ui/card` |
| `Checkbox` | Selection input | `@kit/ui/checkbox` |
| `Command` | Command palette interface | `@kit/ui/command` |
| `DataTable` | Table with enhanced functionality | `@kit/ui/data-table` |
| `Dialog` | Modal window for focused interactions | `@kit/ui/dialog` |
| `DropdownMenu` | Menu triggered by a button | `@kit/ui/dropdown-menu` |
| `Form` | Form components with validation | `@kit/ui/form` |
| `Input` | Text input field | `@kit/ui/input` |
| `Label` | Text label for form elements | `@kit/ui/label` |
| `NavigationMenu` | Hierarchical navigation component | `@kit/ui/navigation-menu` |
| `Popover` | Floating content triggered by interaction | `@kit/ui/popover` |
| `RadioGroup` | Radio button selection group | `@kit/ui/radio-group` |
| `ScrollArea` | Customizable scrollable area | `@kit/ui/scroll-area` |
| `Select` | Dropdown selection menu | `@kit/ui/select` |
| `Separator` | Visual divider between content | `@kit/ui/separator` |
| `Sheet` | Sliding panel from screen edge | `@kit/ui/sheet` |
| `Sidebar` | Advanced sidebar navigation | `@kit/ui/shadcn-sidebar` |
| `Skeleton` | Loading placeholder | `@kit/ui/skeleton` |
| `Switch` | Toggle control | `@kit/ui/switch` |
| `Table` | Data display in rows and columns | `@kit/ui/table` |
| `Tabs` | Tab-based navigation | `@kit/ui/tabs` |
| `Textarea` | Multi-line text input | `@kit/ui/textarea` |
| `Tooltip` | Contextual information on hover | `@kit/ui/tooltip` |

## Makerkit-specific Components

| Component | Description | Import Path |
|-----------|-------------|-------------|
| `If` | Conditional rendering component | `@kit/ui/if` |
| `Trans` | Internationalization text component | `@kit/ui/trans` |
| `Sidebar` (Makerkit) | Navigation sidebar with context | `@kit/ui/sidebar` |
| `Page` | Page layout with navigation | `@kit/ui/page` |
| `GlobalLoader` | Full-page loading indicator | `@kit/ui/global-loader` |
| `ImageUploader` | Image upload component | `@kit/ui/image-uploader` |
| `ProfileAvatar` | User avatar with fallback | `@kit/ui/profile-avatar` |
| `DataTable` (Enhanced) | Extended data table with pagination | `@kit/ui/enhanced-data-table` |
| `Stepper` | Multi-step process indicator | `@kit/ui/stepper` |
| `CookieBanner` | GDPR-compliant cookie notice | `@kit/ui/cookie-banner` |
| `CardButton` | Card-styled button | `@kit/ui/card-button` |
| `MultiStepForm` | Form with multiple steps | `@kit/ui/multi-step-form` |
| `EmptyState` | Empty data placeholder | `@kit/ui/empty-state` |
| `AppBreadcrumbs` | Application path breadcrumbs | `@kit/ui/app-breadcrumbs` |
| `VersionUpdater` | App version update notifier | `@kit/ui/version-updater` |

## Marketing Components

Import all marketing components with:
```tsx
import {
  Hero,
  HeroTitle,
  GradientText,
  // etc.
} from '@kit/ui/marketing';
```

Key marketing components:
- `Hero`, `SecondaryHero` - Hero sections
- `FeatureCard`, `FeatureGrid` - Feature showcases
- `Footer`, `Header` - Page structure
- `NewsletterSignup` - Email collection
- `ComingSoon` - Coming soon page template

## Utility Functions

```tsx
import { cn } from '@kit/ui/utils';
```

- `cn`: Utility for conditional class name joining using Tailwind

## Common Patterns

### Conditional Rendering with If
```tsx
<If condition={isLoading} fallback={<Content />}>
  <Spinner />
</If>
```

### Using Translations
```tsx
<Trans i18nKey="common:welcomeMessage" />
```

### Page Layouts
```tsx
<Page style="sidebar">
  <PageNavigation>
    <SidebarNavigation config={navigationConfig} />
  </PageNavigation>
  <PageBody>
    {/* Page content */}
  </PageBody>
</Page>
```

### Form with Validation
```tsx
<Form {...form}>
  <FormField
    name="email"
    control={form.control}
    render={({ field }) => (
      <FormItem>
        <FormLabel>Email</FormLabel>
        <FormControl>
          <Input {...field} />
        </FormControl>
        <FormMessage />
      </FormItem>
    )}
  />
  <Button type="submit">Submit</Button>
</Form>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drumnation) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
