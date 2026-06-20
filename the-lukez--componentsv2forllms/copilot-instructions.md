## componentsv2forllms

> Provides complete Discord Message Components V2 reference. Auto-invoke when

---
name: discord-components-v2
description: >
  Provides complete Discord Message Components V2 reference. Auto-invoke when
  the user is working with Discord bot components, message components, buttons,
  selects, modals, action rows, sections, containers, or Components V2 payloads.
---

# Discord: Message Components V2 (for LLMs)

Since 2025, message components got a lot more powerful, allowing for more interactive and dynamic user experiences.

## IS_COMPONENTS_V2 Flag

To use the new layout and content components (Section, Text Display, Thumbnail, Media Gallery, File, Separator, Container), send the message flag `1 << 15` (`32768`) as `flags` in your message payload. Once set on a message, this flag cannot be removed.

When `IS_COMPONENTS_V2` is active:

- `content` and `embeds` fields are disabled — use Text Display and Container instead
- Attachments must be exposed through components
- `poll` and `stickers` are disabled
- Messages allow up to **40 total components**

## Overview

- **Layout Components** — For organizing and structuring content (Action Row, Section, Container, Label)
- **Content Components** — For displaying static text, images, and files (Text Display, Thumbnail, Media Gallery, File)
- **Interactive Components** — For user interactions (Button, Select Menus, Text Input, File Upload, Radio Group, Checkbox Group, Checkbox)

### Component Types

| Type | Name                                      | Description                                                    | Style       | Usage          |
| ---- | ----------------------------------------- | -------------------------------------------------------------- | ----------- | -------------- |
| 1    | [Action Row](#action-row)                 | Container to display a row of interactive components           | Layout      | Message        |
| 2    | [Button](#button)                         | Button object                                                  | Interactive | Message        |
| 3    | [String Select](#string-select)           | Select menu for picking from defined text options              | Interactive | Message, Modal |
| 4    | [Text Input](#text-input)                 | Text input object                                              | Interactive | Modal          |
| 5    | [User Select](#user-select)               | Select menu for users                                          | Interactive | Message, Modal |
| 6    | [Role Select](#role-select)               | Select menu for roles                                          | Interactive | Message, Modal |
| 7    | [Mentionable Select](#mentionable-select) | Select menu for mentionables (users _and_ roles)               | Interactive | Message, Modal |
| 8    | [Channel Select](#channel-select)         | Select menu for channels                                       | Interactive | Message, Modal |
| 9    | [Section](#section)                       | Container to display text alongside an accessory component     | Layout      | Message        |
| 10   | [Text Display](#text-display)             | Markdown text                                                  | Content     | Message, Modal |
| 11   | [Thumbnail](#thumbnail)                   | Small image that can be used as an accessory                   | Content     | Message        |
| 12   | [Media Gallery](#media-gallery)           | Display images and other media                                 | Content     | Message        |
| 13   | [File](#file)                             | Displays an attached file                                      | Content     | Message        |
| 14   | [Separator](#separator)                   | Component to add vertical padding between other components     | Layout      | Message        |
| 17   | [Container](#container)                   | Container that visually groups a set of components             | Layout      | Message        |
| 18   | [Label](#label)                           | Container associating a label and description with a component | Layout      | Modal          |
| 19   | [File Upload](#file-upload)               | Component for uploading files                                  | Interactive | Modal          |
| 21   | [Radio Group](#radio-group)               | Single-choice set of options                                   | Interactive | Modal          |
| 22   | [Checkbox Group](#checkbox-group)         | Multi-selectable group of checkboxes                           | Interactive | Modal          |
| 23   | [Checkbox](#checkbox)                     | Single checkbox for yes/no choice                              | Interactive | Modal          |

## Anatomy of a Component

All components share these base fields:

| Field | Type    | Description                                                                                                          |
| ----- | ------- | -------------------------------------------------------------------------------------------------------------------- |
| type  | integer | The type of the component                                                                                            |
| id?   | integer | Optional 32-bit identifier; auto-generated sequentially if omitted. `0` is treated as empty and replaced by the API. |

### Custom ID

Interactive components (buttons, selects, text inputs, etc.) must have a `custom_id`. It is returned in the interaction payload when a user interacts with the component.

| Field     | Type   | Description                                                        |
| --------- | ------ | ------------------------------------------------------------------ |
| custom_id | string | Developer-defined identifier; 1–100 characters, unique per message |

---

## Action Row

A top-level layout component containing interactive components.

Can contain **one** of:

- Up to 5 [Buttons](#button)
- A single select component (String, User, Role, Mentionable, or Channel Select)

> In modals, [Label](#label) is recommended over Action Row. Action Row with Text Inputs in modals is deprecated.

#### Action Row Structure

| Field      | Type    | Description                                   |
| ---------- | ------- | --------------------------------------------- |
| type       | integer | `1` for action row                            |
| id?        | integer | Optional identifier                           |
| components | array   | Up to 5 Buttons, or a single select component |

#### Example

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 1,
      "components": [
        { "type": 2, "custom_id": "click_yes", "label": "Accept", "style": 1 },
        {
          "type": 2,
          "label": "Learn More",
          "style": 5,
          "url": "https://example.com/"
        },
        { "type": 2, "custom_id": "click_no", "label": "Decline", "style": 4 }
      ]
    }
  ]
}
```

---

## Button

An interactive component. Must be inside an [Action Row](#action-row) or a [Section](#section)'s `accessory` field.

#### Button Structure

| Field     | Type          | Description                                                  |
| --------- | ------------- | ------------------------------------------------------------ |
| type      | integer       | `2` for a button                                             |
| id?       | integer       | Optional identifier                                          |
| style     | integer       | A [button style](#button-styles)                             |
| label?    | string        | Button text; max 80 characters                               |
| emoji?    | partial emoji | `name`, `id`, and `animated`                                 |
| custom_id | string        | Required for non-link, non-premium buttons; 1–100 characters |
| sku_id?   | snowflake     | Required for premium buttons                                 |
| url?      | string        | Required for link buttons; max 512 characters                |
| disabled? | boolean       | Defaults to `false`                                          |

#### Button Styles

| Name      | Value | Action                                                                  | Required Field |
| --------- | ----- | ----------------------------------------------------------------------- | -------------- |
| Primary   | 1     | Most important or recommended action                                    | `custom_id`    |
| Secondary | 2     | Alternative or supporting actions                                       | `custom_id`    |
| Success   | 3     | Positive confirmation or completion                                     | `custom_id`    |
| Danger    | 4     | Irreversible consequences                                               | `custom_id`    |
| Link      | 5     | Navigates to a URL; no interaction sent                                 | `url`          |
| Premium   | 6     | Purchase; no interaction sent; auto-shows shop icon, SKU name and price | `sku_id`       |

Rules:

- Non-link, non-premium buttons: must have `custom_id`; cannot have `url` or `sku_id`
- Link buttons: must have `url`; cannot have `custom_id`
- Premium buttons: must have `sku_id`; cannot have `custom_id`, `label`, `url`, or `emoji`

#### Example

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 1,
      "components": [
        { "type": 2, "custom_id": "click_yes", "label": "Accept", "style": 1 }
      ]
    }
  ]
}
```

---

## String Select

Allows users to select one or more from defined options. Must be inside an [Action Row](#action-row) in messages, or a [Label](#label) in modals.

#### String Select Structure

| Field         | Type    | Description                                                                          |
| ------------- | ------- | ------------------------------------------------------------------------------------ |
| type          | integer | `3` for string select                                                                |
| id?           | integer | Optional identifier                                                                  |
| custom_id     | string  | 1–100 characters                                                                     |
| options       | array   | Up to 25 [select options](#string-select-option-structure)                           |
| placeholder?  | string  | Placeholder text; max 150 characters                                                 |
| min_values?   | integer | Min items to choose; min 0, max 25; defaults to 1. Must be ≥1 if `required` is true. |
| max_values?   | integer | Max items to choose; max 25; defaults to 1                                           |
| required?\*   | boolean | Modal only — whether the field is required (defaults to `true`)                      |
| disabled?\*\* | boolean | Message only — whether disabled (defaults to `false`)                                |

\* `required` is ignored in messages. \*\* `disabled` causes an error in modals.

#### String Select Option Structure

| Field        | Type          | Description                                |
| ------------ | ------------- | ------------------------------------------ |
| label        | string        | User-facing name; max 100 characters       |
| value        | string        | Dev-defined value; max 100 characters      |
| description? | string        | Additional description; max 100 characters |
| emoji?       | partial emoji | `id`, `name`, and `animated`               |
| default?     | boolean       | Shows option as selected by default        |

The number of options must be within the range of `min_values` and `max_values`.

#### Interaction Response Structure

| Field            | Type             | Description                         |
| ---------------- | ---------------- | ----------------------------------- |
| type\*           | integer          | `3`                                 |
| component_type\* | integer          | `3`                                 |
| id               | integer          | Unique identifier for the component |
| custom_id        | string           | Developer-defined identifier        |
| values           | array of strings | The values of the selected options  |

\* `component_type` returned in message interactions; `type` in modal interactions.

#### Example

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 1,
      "id": 1,
      "components": [
        {
          "type": 3,
          "id": 2,
          "custom_id": "favorite_bug",
          "placeholder": "Favorite bug?",
          "options": [
            {
              "label": "Ant",
              "value": "ant",
              "description": "(best option)",
              "emoji": { "name": "🐜" }
            },
            {
              "label": "Butterfly",
              "value": "butterfly",
              "emoji": { "name": "🦋" }
            },
            {
              "label": "Caterpillar",
              "value": "caterpillar",
              "emoji": { "name": "🐛" }
            }
          ]
        }
      ]
    }
  ]
}
```

---

## Text Input

Allows users to enter free-form text. Modal only. Must be inside a [Label](#label).

> The `label` field directly on Text Input is deprecated — use the `label` and `description` fields on the [Label](#label) component instead.

#### Text Input Structure

| Field        | Type    | Description                                             |
| ------------ | ------- | ------------------------------------------------------- |
| type         | integer | `4` for text input                                      |
| id?          | integer | Optional identifier                                     |
| custom_id    | string  | 1–100 characters                                        |
| style        | integer | `1` = Short (single-line), `2` = Paragraph (multi-line) |
| min_length?  | integer | Minimum input length; min 0, max 4000                   |
| max_length?  | integer | Maximum input length; min 1, max 4000                   |
| required?    | boolean | Defaults to `true`                                      |
| value?       | string  | Pre-filled value; max 4000 characters                   |
| placeholder? | string  | Placeholder text if empty; max 100 characters           |

#### Interaction Response Structure

| Field     | Type    | Description                  |
| --------- | ------- | ---------------------------- |
| type      | integer | `4`                          |
| id        | integer | Unique identifier            |
| custom_id | string  | Developer-defined identifier |
| value     | string  | The user's input text        |

#### Example

```json
{
  "type": 9,
  "data": {
    "custom_id": "game_feedback_modal",
    "title": "Game Feedback",
    "components": [
      {
        "type": 18,
        "label": "What did you find interesting about the game?",
        "description": "Please give us as much detail as possible!",
        "component": {
          "type": 4,
          "custom_id": "game_feedback",
          "style": 2,
          "min_length": 100,
          "max_length": 4000,
          "placeholder": "Write your feedback here...",
          "required": true
        }
      }
    ]
  }
}
```

---

## User Select

Allows users to select one or more server members. Must be inside an [Action Row](#action-row) in messages, or a [Label](#label) in modals.

#### User Select Structure

| Field           | Type    | Description                                                                          |
| --------------- | ------- | ------------------------------------------------------------------------------------ |
| type            | integer | `5` for user select                                                                  |
| id?             | integer | Optional identifier                                                                  |
| custom_id       | string  | 1–100 characters                                                                     |
| placeholder?    | string  | Max 150 characters                                                                   |
| default_values? | array   | Array of [default value objects](#select-default-value-structure)                    |
| min_values?     | integer | Min items to choose; min 0, max 25; defaults to 1. Must be ≥1 if `required` is true. |
| max_values?     | integer | Max items to choose; max 25; defaults to 1                                           |
| required?\*     | boolean | Modal only — defaults to `true`                                                      |
| disabled?\*\*   | boolean | Message only — defaults to `false`                                                   |

\* `required` is ignored in messages. \*\* `disabled` causes an error in modals.

#### Select Default Value Structure

| Field | Type      | Description                        |
| ----- | --------- | ---------------------------------- |
| id    | snowflake | ID of a user, role, or channel     |
| type  | string    | `"user"`, `"role"`, or `"channel"` |

#### Interaction Response Structure

| Field            | Type                 | Description                   |
| ---------------- | -------------------- | ----------------------------- |
| type\*           | integer              | `5`                           |
| component_type\* | integer              | `5`                           |
| id               | integer              | Unique identifier             |
| custom_id        | string               | Developer-defined identifier  |
| resolved         | resolved data object | Resolved user/member entities |
| values           | array of snowflakes  | IDs of selected users         |

\* `component_type` in message interactions; `type` in modal interactions.

#### Examples

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 1,
      "components": [
        {
          "type": 5,
          "custom_id": "user_select",
          "placeholder": "Select a user"
        }
      ]
    }
  ]
}
```

With default values:

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 1,
      "components": [
        {
          "type": 5,
          "custom_id": "user_select",
          "placeholder": "Select a user",
          "default_values": [{ "id": "123456789012345678", "type": "user" }]
        }
      ]
    }
  ]
}
```

---

## Role Select

Same as [User Select](#user-select) but for guild roles. Type `6`. Default values must use type `"role"`.

Fields and rules are identical to User Select — `required` is modal-only, `disabled` is message-only.

#### Interaction Response Structure

Same as User Select but type `6`; `values` contains role snowflakes; `resolved` contains `roles`.

#### Example

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 1,
      "components": [
        {
          "type": 6,
          "custom_id": "role_select",
          "placeholder": "Select a role"
        }
      ]
    }
  ]
}
```

---

## Mentionable Select

Combines user and role selection. Type `7`. Default values can use type `"user"` or `"role"`.

Fields and rules are identical to User Select — `required` is modal-only, `disabled` is message-only.

#### Interaction Response Structure

Same structure as User Select but type `7`; `values` contains user and/or role snowflakes; `resolved` may contain `users`, `members`, and `roles`.

#### Example

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 1,
      "components": [
        {
          "type": 7,
          "custom_id": "who_to_ping",
          "placeholder": "Who?",
          "default_values": [
            { "id": "123456789012345678", "type": "user" },
            { "id": "987654321098765432", "type": "role" }
          ]
        }
      ]
    }
  ]
}
```

---

## Channel Select

Allows users to select one or more channels. Type `8`. Can be filtered by channel type. Must be inside an [Action Row](#action-row) in messages, or a [Label](#label) in modals.

#### Channel Select Structure

| Field           | Type    | Description                                                                          |
| --------------- | ------- | ------------------------------------------------------------------------------------ |
| type            | integer | `8` for channel select                                                               |
| id?             | integer | Optional identifier                                                                  |
| custom_id       | string  | 1–100 characters                                                                     |
| channel_types?  | array   | List of [channel types](#channel-types) to include                                   |
| placeholder?    | string  | Max 150 characters                                                                   |
| default_values? | array   | Array of [default value objects](#select-default-value-structure) (type `"channel"`) |
| min_values?     | integer | Min items to choose; min 0, max 25; defaults to 1. Must be ≥1 if `required` is true. |
| max_values?     | integer | Max items to choose; max 25; defaults to 1                                           |
| required?\*     | boolean | Modal only — defaults to `true`                                                      |
| disabled?\*\*   | boolean | Message only — defaults to `false`                                                   |

\* `required` is ignored in messages. \*\* `disabled` causes an error in modals.

#### Channel Types

| Name                | Value | Description                                          |
| ------------------- | ----- | ---------------------------------------------------- |
| GUILD_TEXT          | 0     | Text channel within a server                         |
| DM                  | 1     | Direct message between users                         |
| GUILD_VOICE         | 2     | Voice channel within a server                        |
| GROUP_DM            | 3     | Direct message between multiple users                |
| GUILD_CATEGORY      | 4     | Organizational category containing up to 50 channels |
| GUILD_ANNOUNCEMENT  | 5     | Announcement channel (formerly news)                 |
| ANNOUNCEMENT_THREAD | 10    | Thread within a GUILD_ANNOUNCEMENT channel           |
| PUBLIC_THREAD       | 11    | Thread within a GUILD_TEXT or GUILD_FORUM channel    |
| PRIVATE_THREAD      | 12    | Private thread within a GUILD_TEXT channel           |
| GUILD_STAGE_VOICE   | 13    | Stage channel for events with an audience            |
| GUILD_DIRECTORY     | 14    | Hub directory channel                                |
| GUILD_FORUM         | 15    | Channel that can only contain threads                |
| GUILD_MEDIA         | 16    | Media channel that can only contain threads          |

#### Interaction Response Structure

Same structure as User Select but type `8`; `values` contains channel snowflakes; `resolved` contains `channels`.

#### Example

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 1,
      "components": [
        {
          "type": 8,
          "custom_id": "notification_channel",
          "channel_types": [0],
          "placeholder": "Which text channel?"
        }
      ]
    }
  ]
}
```

---

## Section

A top-level layout component (message only, requires IS_COMPONENTS_V2) that associates content with an accessory component displayed alongside it.

#### Section Structure

| Field      | Type    | Description                                    |
| ---------- | ------- | ---------------------------------------------- |
| type       | integer | `9` for section                                |
| id?        | integer | Optional identifier                            |
| components | array   | 1–3 [Text Display](#text-display) components   |
| accessory  | object  | A [Button](#button) or [Thumbnail](#thumbnail) |

#### Example

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 9,
      "components": [
        {
          "type": 10,
          "content": "The game is out now! Check it out on our website."
        }
      ],
      "accessory": {
        "type": 11,
        "media": { "url": "https://example.com/gamepreview.webp" }
      }
    }
  ]
}
```

---

## Text Display

A content component that renders markdown text. Behavior mirrors the `content` field of a message. Pingable mentions respect `message.allowed_mentions`. Requires IS_COMPONENTS_V2 in messages; usable directly as a top-level modal component.

#### Text Display Structure

| Field   | Type    | Description              |
| ------- | ------- | ------------------------ |
| type    | integer | `10` for text display    |
| id?     | integer | Optional identifier      |
| content | string  | Markdown text to display |

#### Interaction Response Structure

| Field | Type    | Description       |
| ----- | ------- | ----------------- |
| type  | integer | `10`              |
| id    | integer | Unique identifier |

#### Example

```json
{
  "flags": 32768,
  "components": [
    { "type": 10, "content": "# Real Game v7.3" },
    {
      "type": 10,
      "content": "Hope you're excited, the update is finally here!\n- Fixed treasure chest bug\n- Improved server stability"
    },
    { "type": 10, "content": "-# Small print goes here..." }
  ]
}
```

---

## Thumbnail

A content component that displays a small image. Only usable as the `accessory` of a [Section](#section). Requires IS_COMPONENTS_V2. Supports images (including GIF and WEBP); videos not supported.

#### Thumbnail Structure

| Field        | Type                | Description                                               |
| ------------ | ------------------- | --------------------------------------------------------- |
| type         | integer             | `11` for thumbnail                                        |
| id?          | integer             | Optional identifier                                       |
| media        | unfurled media item | An [unfurled media item](#unfurled-media-item) with a URL |
| description? | string              | Alt text; max 1024 characters                             |
| spoiler?     | boolean             | Whether to blur the image. Defaults to `false`            |

See [Section](#section) for a usage example.

---

## Media Gallery

A top-level content component (message only, requires IS_COMPONENTS_V2) displaying 1–10 media items in a gallery layout.

#### Media Gallery Structure

| Field | Type    | Description                                               |
| ----- | ------- | --------------------------------------------------------- |
| type  | integer | `12` for media gallery                                    |
| id?   | integer | Optional identifier                                       |
| items | array   | 1–10 [media gallery items](#media-gallery-item-structure) |

#### Media Gallery Item Structure

| Field        | Type                | Description                                    |
| ------------ | ------------------- | ---------------------------------------------- |
| media        | unfurled media item | An [unfurled media item](#unfurled-media-item) |
| description? | string              | Alt text; max 1024 characters                  |
| spoiler?     | boolean             | Whether to blur the item. Defaults to `false`  |

#### Example

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 10,
      "content": "Live webcam shots as of 18-04-2025 at 12:00 UTC"
    },
    {
      "type": 12,
      "items": [
        {
          "media": { "url": "https://example.com/webcam1.webp" },
          "description": "Aerial view of industrial complex."
        },
        {
          "media": { "url": "https://example.com/webcam2.webp" },
          "description": "Aerial view of old buildings."
        },
        {
          "media": { "url": "https://example.com/webcam3.webp" },
          "description": "Street view of downtown."
        }
      ]
    }
  ]
}
```

---

## File

A top-level content component (message only, requires IS_COMPONENTS_V2) that displays one uploaded attachment. Reference files using the `attachment://` protocol. Send payload as `multipart/form-data`.

#### File Structure

| Field    | Type                | Description                                                     |
| -------- | ------------------- | --------------------------------------------------------------- |
| type     | integer             | `13` for file                                                   |
| id?      | integer             | Optional identifier                                             |
| file     | unfurled media item | Only supports `attachment://<filename>` references              |
| spoiler? | boolean             | Whether to blur the file. Defaults to `false`                   |
| name?    | string              | Read-only; provided by the API in responses                     |
| size?    | integer             | Read-only; file size in bytes; provided by the API in responses |

#### Example

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 10,
      "content": "# New game version released for testing!\nGrab the game here:"
    },
    { "type": 13, "file": { "url": "attachment://game.zip" } },
    { "type": 10, "content": "Latest manual artwork here:" },
    { "type": 13, "file": { "url": "attachment://manual.pdf" } }
  ]
}
```

---

## Separator

A top-level layout component (message only, requires IS_COMPONENTS_V2) that adds vertical padding and an optional visual divider.

#### Separator Structure

| Field    | Type    | Description                                               |
| -------- | ------- | --------------------------------------------------------- |
| type     | integer | `14` for separator                                        |
| id?      | integer | Optional identifier                                       |
| divider? | boolean | Whether to show a visual divider line. Defaults to `true` |
| spacing? | integer | Padding size: `1` = small, `2` = large. Defaults to `1`   |

#### Example

```json
{
  "flags": 32768,
  "components": [
    { "type": 10, "content": "It's dangerous to go alone!" },
    { "type": 14, "divider": true, "spacing": 1 },
    { "type": 10, "content": "Take this." }
  ]
}
```

---

## Container

A top-level layout component (message only, requires IS_COMPONENTS_V2) that visually groups components with an optional accent color bar.

#### Container Structure

| Field         | Type    | Description                                                     |
| ------------- | ------- | --------------------------------------------------------------- |
| type          | integer | `17` for container                                              |
| id?           | integer | Optional identifier                                             |
| components    | array   | Child components (see below)                                    |
| accent_color? | integer | RGB accent color from `0x000000` to `0xFFFFFF`; `null` for none |
| spoiler?      | boolean | Whether to blur the container. Defaults to `false`              |

#### Container Child Components

- [Action Row](#action-row)
- [Text Display](#text-display)
- [Section](#section)
- [Media Gallery](#media-gallery)
- [Separator](#separator)
- [File](#file)

#### Example

```json
{
  "flags": 32768,
  "components": [
    {
      "type": 17,
      "accent_color": 703487,
      "components": [
        { "type": 10, "content": "# You have encountered a wild coyote!" },
        {
          "type": 12,
          "items": [{ "media": { "url": "https://example.com/coyote.webp" } }]
        },
        { "type": 10, "content": "What would you like to do?" },
        {
          "type": 1,
          "components": [
            {
              "type": 2,
              "custom_id": "pet_coyote",
              "label": "Pet it!",
              "style": 1
            },
            {
              "type": 2,
              "custom_id": "feed_coyote",
              "label": "Attempt to feed it",
              "style": 2
            },
            {
              "type": 2,
              "custom_id": "run_away",
              "label": "Run away!",
              "style": 4
            }
          ]
        }
      ]
    }
  ]
}
```

---

## Label

A top-level layout component for modals that wraps a single interactive component with a label text and optional description.

> The `description` may display above or below the `component` depending on platform.

#### Label Structure

| Field        | Type    | Description                                        |
| ------------ | ------- | -------------------------------------------------- |
| type         | integer | `18` for label                                     |
| id?          | integer | Optional identifier                                |
| label        | string  | Label text; max 45 characters                      |
| description? | string  | Optional description; max 100 characters           |
| component    | object  | The wrapped component (see child components below) |

#### Label Child Components

- [Text Input](#text-input)
- [String Select](#string-select)
- [User Select](#user-select)
- [Role Select](#role-select)
- [Mentionable Select](#mentionable-select)
- [Channel Select](#channel-select)
- [File Upload](#file-upload)
- [Radio Group](#radio-group)
- [Checkbox Group](#checkbox-group)
- [Checkbox](#checkbox)

#### Interaction Response Structure

| Field     | Type    | Description                                     |
| --------- | ------- | ----------------------------------------------- |
| type      | integer | `18`                                            |
| id        | integer | Unique identifier                               |
| component | object  | The interaction response of the child component |

#### Example

```json
{
  "type": 9,
  "data": {
    "custom_id": "game_feedback_modal",
    "title": "Game Feedback",
    "components": [
      {
        "type": 18,
        "label": "What did you find interesting about the game?",
        "description": "Please give us as much detail as possible!",
        "component": {
          "type": 4,
          "custom_id": "game_feedback",
          "style": 2,
          "min_length": 100,
          "max_length": 4000,
          "placeholder": "Write your feedback here...",
          "required": true
        }
      }
    ]
  }
}
```

---

## File Upload

Allows users to upload files in modals. Must be inside a [Label](#label).

#### File Upload Structure

| Field       | Type    | Description                                                                          |
| ----------- | ------- | ------------------------------------------------------------------------------------ |
| type        | integer | `19` for file upload                                                                 |
| id?         | integer | Optional identifier                                                                  |
| custom_id   | string  | 1–100 characters                                                                     |
| min_values? | integer | Min files to upload; min 0, max 10; defaults to 1. Must be ≥1 if `required` is true. |
| max_values? | integer | Max files to upload; max 10; defaults to 1                                           |
| required?   | boolean | Whether upload is required to submit modal; defaults to `true`                       |

Max file size is determined by the user's upload limit in that channel.

#### Interaction Response Structure

| Field     | Type                | Description                                           |
| --------- | ------------------- | ----------------------------------------------------- |
| type      | integer             | `19`                                                  |
| id        | integer             | Unique identifier                                     |
| custom_id | string              | Developer-defined identifier                          |
| values    | array of snowflakes | IDs of uploaded files found in `resolved.attachments` |

#### Example

```json
{
  "type": 9,
  "data": {
    "custom_id": "bug_submit_modal",
    "title": "Bug Submission",
    "components": [
      {
        "type": 18,
        "label": "File Upload",
        "description": "Upload a screenshot showing the bug.",
        "component": {
          "type": 19,
          "custom_id": "file_upload",
          "min_values": 1,
          "max_values": 10,
          "required": true
        }
      }
    ]
  }
}
```

---

## Radio Group

A modal-only interactive component for selecting exactly one option from a list. Must be inside a [Label](#label).

#### Radio Group Structure

| Field     | Type    | Description                                                   |
| --------- | ------- | ------------------------------------------------------------- |
| type      | integer | `21` for radio group                                          |
| id?       | integer | Optional identifier                                           |
| custom_id | string  | 1–100 characters                                              |
| options   | array   | 2–10 [radio group options](#radio-group-option-structure)     |
| required? | boolean | Whether a selection is required to submit; defaults to `true` |

#### Radio Group Option Structure

| Field        | Type    | Description                              |
| ------------ | ------- | ---------------------------------------- |
| value        | string  | Dev-defined value; max 100 characters    |
| label        | string  | User-facing label; max 100 characters    |
| description? | string  | Optional description; max 100 characters |
| default?     | boolean | Shows option as selected by default      |

#### Interaction Response Structure

| Field     | Type    | Description                                             |
| --------- | ------- | ------------------------------------------------------- |
| type      | integer | `21`                                                    |
| id        | integer | Unique identifier                                       |
| custom_id | string  | Developer-defined identifier                            |
| value     | ?string | The selected option's value, or `null` if none selected |

#### Example

```json
{
  "type": 9,
  "data": {
    "custom_id": "class_selection_modal",
    "title": "Class Selection",
    "components": [
      {
        "type": 18,
        "label": "Choose your class",
        "description": "Your class determines your style of play.",
        "component": {
          "type": 21,
          "custom_id": "class_radio",
          "options": [
            {
              "value": "warrior",
              "label": "Warrior",
              "description": "Strong and brave"
            },
            {
              "value": "rogue",
              "label": "Rogue",
              "description": "Weak and squishy"
            },
            { "value": "wizard", "label": "Wizard", "description": "Nerd" },
            {
              "value": "bard",
              "label": "Bard",
              "description": "Annoys everyone"
            }
          ]
        }
      }
    ]
  }
}
```

---

## Checkbox Group

A modal-only interactive component for selecting one or many options via checkboxes. Must be inside a [Label](#label).

#### Checkbox Group Structure

| Field       | Type    | Description                                                                          |
| ----------- | ------- | ------------------------------------------------------------------------------------ |
| type        | integer | `22` for checkbox group                                                              |
| id?         | integer | Optional identifier                                                                  |
| custom_id   | string  | 1–100 characters                                                                     |
| options     | array   | 1–10 [checkbox group options](#checkbox-group-option-structure)                      |
| min_values? | integer | Min items to select; min 0, max 10; defaults to 1. Must be ≥1 if `required` is true. |
| max_values? | integer | Max items to select; min 1, max 10; defaults to number of options                    |
| required?   | boolean | Whether selecting within the group is required; defaults to `true`                   |

#### Checkbox Group Option Structure

| Field        | Type    | Description                              |
| ------------ | ------- | ---------------------------------------- |
| value        | string  | Dev-defined value; max 100 characters    |
| label        | string  | User-facing label; max 100 characters    |
| description? | string  | Optional description; max 100 characters |
| default?     | boolean | Shows option as checked by default       |

#### Interaction Response Structure

| Field     | Type             | Description                                                   |
| --------- | ---------------- | ------------------------------------------------------------- |
| type      | integer          | `22`                                                          |
| id        | integer          | Unique identifier                                             |
| custom_id | string           | Developer-defined identifier                                  |
| values    | array of strings | Values of selected options; empty array `[]` if none selected |

#### Example

```json
{
  "type": 9,
  "data": {
    "custom_id": "day_selection_modal",
    "title": "Study Days",
    "components": [
      {
        "type": 18,
        "label": "Which days are you free?",
        "description": "Choose all the days you're able to meet up.",
        "component": {
          "type": 22,
          "custom_id": "event_checkbox",
          "options": [
            { "value": "march-4", "label": "March 4th" },
            { "value": "march-5", "label": "March 5th" },
            {
              "value": "march-7",
              "label": "March 7th",
              "description": "This is a Saturday"
            },
            { "value": "march-9", "label": "March 9th" },
            { "value": "march-10", "label": "March 10th" }
          ]
        }
      }
    ]
  }
}
```

---

## Checkbox

A modal-only single checkbox for yes/no style questions. Must be inside a [Label](#label).

> Checkboxes cannot be set as `required`. To enforce a required checkbox, use a [Checkbox Group](#checkbox-group) with a single option and `required: true`.

#### Checkbox Structure

| Field     | Type    | Description                |
| --------- | ------- | -------------------------- |
| type      | integer | `23` for checkbox          |
| id?       | integer | Optional identifier        |
| custom_id | string  | 1–100 characters           |
| default?  | boolean | Whether checked by default |

#### Interaction Response Structure

| Field     | Type    | Description                             |
| --------- | ------- | --------------------------------------- |
| type      | integer | `23`                                    |
| id        | integer | Unique identifier                       |
| custom_id | string  | Developer-defined identifier            |
| value     | boolean | `true` if checked, `false` if unchecked |

#### Example

```json
{
  "type": 9,
  "data": {
    "custom_id": "secret_note_modal",
    "title": "Secret Note",
    "components": [
      {
        "type": 18,
        "label": "Do you like me?",
        "description": "😳",
        "component": {
          "type": 23,
          "custom_id": "like_checkbox"
        }
      }
    ]
  }
}
```

---

## Unfurled Media Item

A piece of media referenced by URL, used in Thumbnail, Media Gallery, and File components. Only the `url` field is settable by developers — all other fields are read-only and returned by the API.

#### Unfurled Media Item Structure

| Field                  | Type      | Description                                                      |
| ---------------------- | --------- | ---------------------------------------------------------------- |
| url                    | string    | Supports arbitrary URLs and `attachment://<filename>` references |
| proxy_url?\*           | string    | Proxied URL of the media item                                    |
| height?\*              | ?integer  | Height of the media item (if image or video)                     |
| width?\*               | ?integer  | Width of the media item (if image or video)                      |
| placeholder?\*         | string    | Thumbhash placeholder (if image or video)                        |
| placeholder_version?\* | integer   | Version of the placeholder                                       |
| content_type?\*        | string    | MIME type of the content                                         |
| flags?\*               | integer   | Bitfield of unfurled media item flags (see below)                |
| attachment_id?\* \*\*  | snowflake | ID of the uploaded attachment                                    |

\* Read-only; provided by the API. \*\* Only present if uploaded as an attachment.

#### Unfurled Media Item Flags

| Flag        | Value    | Description            |
| ----------- | -------- | ---------------------- |
| IS_ANIMATED | `1 << 0` | This image is animated |

### Uploading a File

Send payload as `multipart/form-data` (not `application/json`) and include the file with a valid filename. Reference using `attachment://<filename>` in the `url` field.

---

## Legacy Message Component Behavior

Before `IS_COMPONENTS_V2`, components were used alongside `content` and `embeds`. This still works but offers less layout control.

- Legacy messages allow up to **5 Action Rows** as top-level components
- Components from legacy messages have `id: 0`

```json
{
  "content": "This is a message with legacy components",
  "components": [
    {
      "type": 1,
      "components": [
        {
          "type": 2,
          "style": 1,
          "label": "Click Me",
          "custom_id": "click_me_1"
        }
      ]
    }
  ]
}
```

---
> Source: [The-LukeZ/ComponentsV2ForLLMs](https://github.com/The-LukeZ/ComponentsV2ForLLMs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
