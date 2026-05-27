## hs-create-react-module-fields

> Create module fields

# HubSpot Module Fields Guide

This guide helps you create properly structured `fields.tsx` files for HubSpot CMS modules. Follow these patterns and examples to ensure consistency and best practices.

## Field Library Components

Use these pre-built components from `src/components/fieldLibrary/` whenever possible:

```typescript
import {
  ButtonContent,    // Button fields with text, link, and icon options
  ButtonStyle,      // Button styling (primary/secondary/tertiary/accent)
  CardStyle,        // Card styling variants
  HeadingAndText,   // Heading text with level selection
  HeadingStyle,     // Heading style variants
  RichTextContent,  // Rich text editing with different feature sets
  SectionStyle,     // Section styling variants
  LinkStyle,        // Link styling (primary/secondary)
} from '../../fieldLibrary/index.js';
```

## Core Field Patterns

### Basic Module Structure
```typescript
import { ModuleFields, FieldGroup } from '@hubspot/cms-components/fields';

export const fields = (
  <ModuleFields>
    {/* Content fields at top level */}
    <TextField
      name="title"
      label="Title"
      default="Default Title"
    />

    {/* Style fields in STYLE tab */}
    <FieldGroup name="groupStyle" label="Styles" tab="STYLE">
      <SectionStyle sectionStyleDefault="section_variant_1" />
      <HeadingStyle headingStyleDefault="h2" />
    </FieldGroup>
  </ModuleFields>
);
```

### Repeated Content Pattern
```typescript
<RepeatedFieldGroup
  name="items"
  label="Items"
  occurrence={{
    min: 1,
    max: 4,
    default: 2
  }}
  default={[
    { title: 'Item 1' },
    { title: 'Item 2' }
  ]}
>
  <TextField
    name="title"
    label="Item Title"
    default="New Item"
  />
</RepeatedFieldGroup>
```

### Common Field Library Usage

#### HeadingAndText
```typescript
<HeadingAndText
  headingLevelDefault="h2"
  textDefault="Default Heading"
  headingTextLabel="Section Heading" // optional
/>
```

#### ButtonContent
```typescript
<ButtonContent
  textDefault="Learn More"
  linkDefault={{
    open_in_new_tab: true,
    url: { href: '#' }
  }}
  iconPositionDefault="right"
/>
```

#### RichTextContent
```typescript
<RichTextContent
  label="Description"
  featureSet="text" // or "extended"
  richTextDefault="<p>Default content</p>"
/>
```

## Available Field Types

### Basic Fields
```typescript
import {
  TextField,         // Simple text input
  RichTextField,     // Formatted text
  ImageField,        // Image uploads
  LinkField,         // URL/page links
  BooleanField,      // Toggles
  ChoiceField,       // Dropdowns/radio/checkboxes
  ColorField,        // Color selection
  NumberField,       // Numeric input
} from '@hubspot/cms-components/fields';
```

### Specialized Fields
```typescript
import {
  MenuField,          // Menu selection
  IconField,          // Icon selection
  BlogField,          // Blog selection
  LogoField,          // Logo uploads
  TextAlignmentField, // Text alignment
  AlignmentField,     // General alignment
  DateField,          // Date input
} from '@hubspot/cms-components/fields';
```

## Field Visibility

Use for conditional field display:
```typescript
<TextField
  name="conditionalField"
  label="Conditional Field"
  visibility={{
    controlling_field_path: 'someToggle',
    controlling_value_regex: 'true',
    operator: 'EQUAL'
  }}
/>
```

## Best Practices

1. **Structure**
   - Keep content fields at top level
   - Group style fields under `FieldGroup` with `tab="STYLE"`
   - Use descriptive `name` and `label` properties

2. **Defaults**
   - Always set default values
   - Use sensible defaults that work out of the box
   - Set appropriate min/max for repeated fields

3. **Required Fields**
    - Do not require fields.
    - Do not use the 'required' property.

3. **Grouping**
   - Only group related fields
   - Use consistent naming (`group` prefix for groups)
   - Keep style-related fields in STYLE tab

4. **Validation**
   - Set appropriate validation rules
   - Include help text for complex fields
   - Use visibility rules when needed

Remember: The goal is to create intuitive, well-structured fields that provide a good editing experience in the HubSpot CMS.

## Examples

### Menu Fields.tsx

```typescript
import {
  ModuleFields,
  MenuField,
  FieldGroup,
  ChoiceField,
  NumberField,
  AlignmentField,
  TextField,
} from '@hubspot/cms-components/fields';
import { SizeChoice } from '../../MenuComponent/types.js';
import LinkStyle from '../../fieldLibrary/LinkStyle/index.js';

const sizeOptions: SizeChoice[] = [
  ['none', 'None'],
  ['small', 'Small'],
  ['medium', 'Medium'],
  ['large', 'Large'],
];

export const fields = (
  <ModuleFields>
    <MenuField
      label='Menu'
      name='menu'
      default='default'
    />
    <NumberField
      label='Max menu depth'
      name='maxDepth'
      display='slider'
      min={1}
      max={3}
      helpText='Set the maximum number of menu levels to include. Must always include at least 1 level or else no menu would populate.'
      default={3}
    />
    <TextField
      label='Accessible menu name'
      name='menuName'
      helpText='Set the aria-label on the nav element surrounding the menu. This provides additional context to those using screen readers to better understand the purpose of your menu. <a href="https://www.w3.org/WAI/tutorials/menus/structure/#label-menus">Learn more here</a>'
      default=''
    />
    <FieldGroup
      label='Styles'
      name='styles'
      tab='STYLE'
    >
      <FieldGroup
        label='Menu'
        name='groupMenu'
        display='inline'
      >
        <AlignmentField
          label='Menu alignment'
          name='menuAlignment'
          required={true}
          alignmentDirection='HORIZONTAL'
          default={{ horizontal_align: 'LEFT' }}
        />
        <ChoiceField
          label='Column gap'
          name='menuColumnGap'
          display='select'
          helpText='Changes the amount of space between top level menu items. You can also think as the space between each column in the horizontal menu.'
          choices={sizeOptions}
          default='none'
          required={true}
        />
      </FieldGroup>
      <FieldGroup
        label='Menu items'
        name='groupMenuItems'
        display='inline'
      >
        <ChoiceField
          label='Vertical gap'
          name='menuItemVerticalGap'
          display='select'
          helpText='Changes the amount of space between menu items. Increasing this value will create unclickable space/gaps between each menu item link.'
          choices={sizeOptions}
          required={true}
          default='none'
        />
        <ChoiceField
          label='Padding'
          name='menuItemPadding'
          display='select'
          helpText='Changes the amount of padding surrounding the text of each menu item. Increasing this value will increase the clickable area of each menu item link.'
          choices={sizeOptions}
          required={true}
          default='none'
        />
      </FieldGroup>
      <FieldGroup
        label='Links'
        name='groupLink'
        display='inline'
      >
        <LinkStyle
          linkStyleDefault='primary_links'
        />
      </FieldGroup>
    </FieldGroup>
  </ModuleFields>
);
```

### PricingCard Fields.tsx

```typescript
import { ModuleFields, RepeatedFieldGroup, TextField, FieldGroup, BooleanField, AdvancedVisibility, } from '@hubspot/cms-components/fields';
import { ButtonContent, HeadingAndText } from '../../fieldLibrary/index.js';
import StyleFields from './styleFields.js';

const buttonFieldVisibility: AdvancedVisibility = {
  boolean_operator: 'OR',
  criteria: [{
    controlling_field_path: 'groupPricingCards.groupButton.showButton',
    controlling_value_regex: 'true',
    operator: 'EQUAL',
  }]
}

export const fields = (
  <ModuleFields>
    <RepeatedFieldGroup
      label='Pricing cards'
      name='groupPricingCards'
      occurrence={{
        min: 1,
        max: 4,
        default: 1,
      }}
      default={[
        {
          groupSummary: {
            headingAndTextHeadingLevel: 'h2',
            headingAndTextHeading: 'Starter',
            description: 'Perfect for small businesses or startups looking to kickstart their digital marketing efforts',
            price: '$79',
            timePeriod: '/month',
          },
          groupPlanFeatures: {
            headingAndTextHeadingLevel: 'h3',
            headingAndTextHeading: 'Starter plan includes',
            groupFeatures: [
              {
                feature: 'Social media management',
              },
              {
                feature: 'Email marketing campaigns',
              },
              {
                feature: 'Basic SEO optimization',
              },
              {
                feature: 'Monthly performance reports',
              },
            ],
          },
          groupButton: {
            showButton: true,
            buttonContentText: 'Get started',
            buttonContentLink: {
              open_in_new_tab: true,
            },
            buttonContentShowIcon: false,
            buttonContentIcon: {
              name: 'dollar-sign',
            },
            buttonContentIconPosition: 'right',
          },
        },
        {
          groupSummary: {
            headingAndTextHeadingLevel: 'h2',
            headingAndTextHeading: 'Pro',
            description: 'Ideal for growing businesses that want to expand their online presence and reach a larger audience',
            price: '$129',
            timePeriod: '/month',
          },
          groupPlanFeatures: {
            headingAndTextHeadingLevel: 'h3',
            headingAndTextHeading: 'Pro plan includes',
            groupFeatures: [
              {
                feature: 'Everything in Starter plan',
              },
              {
                feature: 'Advanced social media management',
              },
              {
                feature: 'Email marketing automation',
              },
              {
                feature: 'Basic SEO optimization',
              },
              {
                feature: 'Comprehensive SEO analysis and recommendations',
              },
            ],
          },
          groupButton: {
            showButton: true,
            buttonContentText: 'Get started',
            buttonContentLink: {
              open_in_new_tab: true,
            },
            buttonContentShowIcon: false,
            buttonContentIcon: {
              name: 'dollar-sign',
            },
            buttonContentIconPosition: 'right',
          },
        },
        {
          groupSummary: {
            headingAndTextHeadingLevel: 'h2',
            headingAndTextHeading: 'Enterprise',
            description: 'Tailored for established businesses seeking a holistic digital marketing solution to boost brand visibility and engagement',
            price: '$599',
            timePeriod: '/month',
          },
          groupPlanFeatures: {
            headingAndTextHeadingLevel: 'h3',
            headingAndTextHeading: 'Enterprise plan includes',
            groupFeatures: [
              {
                feature: 'Everything in Pro plan',
              },
              {
                feature: 'Customized social media strategies',
              },
              {
                feature: 'Personalized email marketing campaigns',
              },
              {
                feature: 'Advanced SEO implementation and monitoring',
              },
              {
                feature: 'Bi-weekly performance reports and strategy consultations',
              },
            ],
          },
          groupButton: {
            showButton: true,
            buttonContentText: 'Contact sales',
            buttonContentLink: {
              open_in_new_tab: true,
            },
            buttonContentShowIcon: false,
            buttonContentIcon: {
              name: 'dollar-sign',
            },
            buttonContentIconPosition: 'right',
          },
        },
      ]}
    >
      <FieldGroup
        label='Pricing summary'
        name='groupSummary'
        display='inline'
      >
        <HeadingAndText
          headingTextLabel='Pricing plan title'
          headingLevelDefault='h2'
          textDefault='Starter'
        />
        <TextField
          label='Description'
          name='description'
          default='Perfect for small businesses or startups looking to kickstart their digital marketing efforts'
        />
        <TextField
          label='Price'
          name='price'
          required={true}
          default='$79'
        />
        <TextField
          label='Time period'
          name='timePeriod'
          required={true}
          default='/month'
        />
      </FieldGroup>
      <FieldGroup
        label='Pricing plan features'
        name='groupPlanFeatures'
        display='inline'
      >
        <HeadingAndText
          headingTextLabel='Features title'
          headingLevelDefault='h3'
          textDefault='Starter plan includes'
        />
        <RepeatedFieldGroup
          label='Features'
          name='groupFeatures'
          occurrence={{
            min: 1,
            max: 20,
            default: 5,
          }}
          default={[
            {
              feature: 'Social media management',
            },
            {
              feature: 'Email marketing campaigns',
            },
            {
              feature: 'Basic SEO optimization',
            },
            {
              feature: 'Monthly performance reports',
            },
          ]}
        >
          <TextField
            label='Feature'
            name='feature'
            default='Social media management'
          />
        </RepeatedFieldGroup>
      </FieldGroup>
      <FieldGroup
        label='Button'
        name='groupButton'
        display='inline'
      >
        <BooleanField
          label='Show button'
          name='showButton'
          display='toggle'
          default={true}
        />
        <ButtonContent
          textDefault='Get started'
          linkDefault={{
            open_in_new_tab: true,
          }}
          textVisibility={buttonFieldVisibility}
          linkVisibility={buttonFieldVisibility}
        />
      </FieldGroup>
    </RepeatedFieldGroup>
    <StyleFields />
  </ModuleFields>
);

```

## Important Rules for Field Library and Repeater Groups

### Field Library Component Usage
1. **CRITICAL**: Before using any field library component, you MUST read its implementation first to understand its field names
2. **Field Names Must Match**: When referencing fields from a field library component (e.g., in default values), the property names MUST exactly match the internal field names of the component
3. **Example of Proper Usage**:
```typescript
// First, read ButtonContent component to know its field names are:
// - buttonContentText
// - buttonContentLink
// - buttonContentIcon
// - buttonContentIconPosition

// Then use those exact names in your implementation:
<ButtonContent
  textDefault="Learn More"      // Sets default for buttonContentText
  linkDefault={{               // Sets default for buttonContentLink
    open_in_new_tab: true,
    url: { href: '#' }
  }}
  iconPositionDefault="right"  // Sets default for buttonContentIconPosition
/>
```

### Repeater Group Default Values
1. **CRITICAL**: In repeater group defaults, object keys MUST match the exact field names defined within the group
2. **NO Arbitrary Keys**: Never use arbitrary key-value pairs in default objects
3. **Example of Proper Usage**:
```typescript
<RepeatedFieldGroup
  name="items"
  default={[
    {
      // These keys MUST match the field names defined in the group
      title: "Item 1",    // Matches <TextField name="title" />
      description: "...", // Matches <TextField name="description" />
    }
  ]}
>
  <TextField name="title" />
  <TextField name="description" />
</RepeatedFieldGroup>
```

4. **Example of INCORRECT Usage**:
```typescript
// ❌ WRONG - arbitrary keys that don't match field names
<RepeatedFieldGroup
  name="items"
  default={[
    {
      itemTitle: "Item 1",      // Wrong - should be "title"
      itemDescription: "..."    // Wrong - should be "description"
    }
  ]}
>
  <TextField name="title" />
  <TextField name="description" />
</RepeatedFieldGroup>
```

---
> Source: [HubSpot/cms-elevate-theme-public](https://github.com/HubSpot/cms-elevate-theme-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
