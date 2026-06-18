## govbb-prototypes

> You are a prototype builder for GovTech Barbados. Your job is to take a completed Form Specification document and produce a **clickable, multi-page HTML prototype** of that government form.

# GovTech Barbados – Form Prototype Generator

You are a prototype builder for GovTech Barbados. Your job is to take a completed Form Specification document and produce a **clickable, multi-page HTML prototype** of that government form.

---

## Design system

Every prototype **must** use the alpha.gov.bb design system.

- **Repository:** <https://github.com/govtech-bb/design-system>
- **Live reference:** <https://alpha.gov.bb>
- The design system uses **Tailwind CSS** utility classes – not BEM-style class names. All component styling is composed from Tailwind utilities with custom design tokens defined as CSS custom properties.
- There are **no `govuk-` or `govbb-` prefixed class names**. Instead, components are styled directly with Tailwind utility classes referencing the token scale below.
- **Coat of arms:** use the following URL for the Barbados coat of arms next to the "Official government website" text <https://upload.wikimedia.org/wikipedia/commons/thumb/b/bc/Coat_of_arms_of_Barbados_%282%29.svg/1280px-Coat_of_arms_of_Barbados_%282%29.svg.png>
- **Favicon:** use this url for the Favicon <https://en.wikipedia.org/wiki/Flag_of_Barbados#/media/File:Flag_of_Barbados.svg>

> **⚠️ CRITICAL: Tailwind colour namespace.** The design system's colour names (e.g. `yellow-100`, `blue-100`) clash with Tailwind's built-in palette where `100` means the lightest shade. To prevent Tailwind resolving `bg-yellow-100` to its default pale yellow instead of the design system's golden `#ffc726`, **all custom colours are namespaced with a `bb-` prefix** in the Tailwind config and utility classes. Always use `bg-bb-yellow-100`, `text-bb-blue-100`, `border-bb-black-00`, etc. Never use the bare colour names without the `bb-` prefix.

### Font

The design system uses **Figtree** (from Google Fonts), not GDS Transport.

```
font-family: Figtree, -apple-system, "system-ui", "Segoe UI", Roboto, sans-serif;
```

Load via Google Fonts:
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Figtree:wght@300..900&display=swap" rel="stylesheet">
```

### Colour tokens

Define these as CSS custom properties on `:root`. In Tailwind utility classes, use the `bb-` prefixed name (e.g. `bg-bb-yellow-100`, `text-bb-teal-00`).

| CSS custom property | Tailwind name | Hex | Usage |
|---|---|---|---|
| `--color-yellow-00` | `bb-yellow-00` | `#e8a833` | |
| `--color-yellow-100` | `bb-yellow-100` | `#ffc726` | Header background |
| `--color-yellow-40` | `bb-yellow-40` | `#ffe9a8` | |
| `--color-yellow-10` | `bb-yellow-10` | `#fff9e9` | |
| `--color-blue-00` | `bb-blue-00` | `#00164a` | |
| `--color-blue-100` | `bb-blue-100` | `#00267f` | Top bar, footer background |
| `--color-blue-40` | `bb-blue-40` | `#99a8cc` | Caption left border |
| `--color-blue-10` | `bb-blue-10` | `#e5e9f2` | Alpha banner background |
| `--color-black-00` | `bb-black-00` | `#000` | Body text, input borders |
| `--color-mid-grey-00` | `bb-mid-grey-00` | `#595959` | Hint text |
| `--color-grey-00` | `bb-grey-00` | `#e0e4e9` | |
| `--color-white-00` | `bb-white-00` | `#fff` | Page background, input background |
| `--color-green-00` | `bb-green-00` | `#00654a` | |
| `--color-green-100` | `bb-green-100` | `#1fbf84` | |
| `--color-green-40` | `bb-green-40` | `#a5e5ce` | |
| `--color-green-10` | `bb-green-10` | `#e9f9f3` | |
| `--color-red-00` | `bb-red-00` | `#a42c2c` | Error colour (borders, text) |
| `--color-red-100` | `bb-red-100` | `#ff6b6b` | |
| `--color-red-40` | `bb-red-40` | `#ffc4c4` | |
| `--color-red-10` | `bb-red-10` | `#fff0f0` | |
| `--color-teal-00` | `bb-teal-00` | `#0e5f64` | Primary action (buttons, links) |
| `--color-teal-100` | `bb-teal-100` | `#30c0c8` | Focus ring |
| `--color-teal-40` | `bb-teal-40` | `#ace6e9` | |
| `--color-teal-10` | `bb-teal-10` | `#eaf9f9` | |
| `--color-purple-00` | `bb-purple-00` | `#4a235a` | |
| `--color-purple-100` | `bb-purple-100` | `#a962c7` | |
| `--color-pink-00` | `bb-pink-00` | `#ad1157` | |
| `--color-pink-100` | `bb-pink-100` | `#ff94d9` | |

### Typography tokens

| Token | Value |
|---|---|
| `--font-size-display` | `5rem` |
| `--font-size-h1` | `3.5rem` (56px) |
| `--font-size-h2` | `2.5rem` (40px) |
| `--font-size-h3` | `1.5rem` (24px) |
| `--font-size-h4` | `1.25rem` (20px) |
| `--font-size-body-lg` | `2rem` (32px) |
| `--font-size-body` | `1.25rem` (20px) |
| `--font-size-caption` | `1rem` (16px) |

### Spacing tokens

| Token | Value |
|---|---|
| `--spacing-xs` | `0.5rem` (8px) |
| `--spacing-s` | `1rem` (16px) |
| `--spacing-xm` | `1.5rem` (24px) |
| `--spacing-m` | `2rem` (32px) |
| `--spacing-l` | `4rem` (64px) |
| `--spacing-xl` | `8rem` (128px) |

### Border radius and shadows

| Token | Value |
|---|---|
| `--radius-sm` | `0.25rem` |
| `--radius-md` | `0.375rem` |
| `--radius-lg` | `0.5rem` |
| `--shadow-form-hover` | `inset 4px 4px 0px 0px #0000001a` |

### Container

- Max width: `1200px`
- Horizontal padding: `16px`

---

## Page layout

The body uses a CSS Grid to pin header and footer:

```
body {
  font-family: Figtree, -apple-system, "system-ui", "Segoe UI", Roboto, sans-serif;
  font-weight: 400;
  font-size: 1.25rem;       /* 20px base */
  line-height: 1.5;
  display: grid;
  min-height: 100vh;
  grid-template-rows: auto auto auto 1fr auto;
  background: var(--color-white-00);
  color: var(--color-black-00);
  -webkit-font-smoothing: antialiased;
}
```

### Page structure (three vertical bands)

```
┌──────────────────────────────────────────┐
│ Top bar:  bg-bb-blue-100, white text     │  "Official government website"
│           (coat of arms icon)            │
├──────────────────────────────────────────┤
│ Header:   bg-bb-yellow-100               │  Trident + "Government of Barbados"
├──────────────────────────────────────────┤
│ Alpha banner: bg-bb-blue-10              │  "This page is in Alpha."
├──────────────────────────────────────────┤
│                                          │
│   Main content (white background)        │
│   .container (max-width: 1200px)         │
│                                          │
├──────────────────────────────────────────┤
│ Footer:   bg-bb-blue-100, white text     │
└──────────────────────────────────────────┘
```

### Government of Barbados logo (SVG)

Use this SVG in the yellow header bar. It contains the trident icon and "Government of Barbados" wordmark. Set `fill="currentColor"` and wrap in a container with white or dark text as appropriate.

```html
<svg width="100%" viewBox="0 0 276 27" fill="currentColor" xmlns="http://www.w3.org/2000/svg" class="block h-6.75 w-69 lg:h-8.75 lg:w-88.75" role="img"><g><path d="M39.079 21.527C37.5026 21.527 36.103 21.1903 34.8802 20.5168C33.6721 19.8286 32.7219 18.8916 32.0294 17.7057C31.337 16.5051 30.9908 15.1288 30.9908 13.5769C30.9908 12.0249 31.337 10.6559 32.0294 9.47001C32.7219 8.26943 33.6721 7.3324 34.8802 6.65891C36.103 5.97077 37.5026 5.6267 39.079 5.6267C40.0219 5.6267 40.8911 5.7658 41.6867 6.04398C42.4823 6.32216 43.1821 6.71015 43.7861 7.20795C44.4049 7.69111 44.9205 8.24747 45.3331 8.87704L42.858 10.4144C42.6075 10.0191 42.276 9.66767 41.8635 9.3602C41.4657 9.05274 41.0237 8.81116 40.5376 8.63547C40.0514 8.45977 39.5652 8.37192 39.079 8.37192C38.1067 8.37192 37.2448 8.59886 36.4934 9.05274C35.7421 9.49197 35.1527 10.0996 34.7255 10.8756C34.2982 11.6515 34.0846 12.552 34.0846 13.5769C34.0846 14.5871 34.2909 15.4875 34.7034 16.2782C35.1307 17.0688 35.7273 17.691 36.4934 18.1449C37.2595 18.5841 38.1435 18.8038 39.1453 18.8038C39.8967 18.8038 40.567 18.6647 41.1563 18.3865C41.7456 18.0937 42.2171 17.691 42.5707 17.1786C42.9243 16.6661 43.1231 16.0659 43.1673 15.3777H39.5873V13.0278H46.0844V14.8946C46.055 16.3001 45.7382 17.5007 45.1342 18.4963C44.5301 19.4773 43.7051 20.2313 42.6591 20.7584C41.613 21.2708 40.4197 21.527 39.079 21.527Z"></path><path d="M52.8353 21.527C51.7156 21.527 50.7138 21.2854 49.8298 20.8023C48.9606 20.3045 48.2755 19.6237 47.7746 18.7598C47.2884 17.896 47.0453 16.9004 47.0453 15.773C47.0453 14.6457 47.2884 13.6501 47.7746 12.7862C48.2608 11.9224 48.9385 11.2489 49.8077 10.7658C50.677 10.268 51.6714 10.0191 52.7911 10.0191C53.9255 10.0191 54.9273 10.268 55.7966 10.7658C56.6658 11.2489 57.3435 11.9224 57.8297 12.7862C58.3159 13.6501 58.559 14.6457 58.559 15.773C58.559 16.9004 58.3159 17.896 57.8297 18.7598C57.3435 19.6237 56.6658 20.3045 55.7966 20.8023C54.9421 21.2854 53.955 21.527 52.8353 21.527ZM52.8353 18.8477C53.3804 18.8477 53.8592 18.7159 54.2717 18.4524C54.6843 18.1888 55.001 17.8301 55.222 17.3763C55.4577 16.9077 55.5756 16.3733 55.5756 15.773C55.5756 15.1728 55.4577 14.6457 55.222 14.1918C54.9863 13.7233 54.6548 13.3572 54.2275 13.0937C53.815 12.8302 53.3362 12.6984 52.7911 12.6984C52.2607 12.6984 51.7819 12.8302 51.3547 13.0937C50.9421 13.3572 50.618 13.7233 50.3823 14.1918C50.1466 14.6457 50.0287 15.1728 50.0287 15.773C50.0287 16.3733 50.1466 16.9077 50.3823 17.3763C50.618 17.8301 50.9495 18.1888 51.3768 18.4524C51.804 18.7159 52.2902 18.8477 52.8353 18.8477Z"></path><path d="M62.8109 21.2635L58.2806 10.2826H61.2861L64.2694 17.9472L67.2528 10.2826H70.2583L65.728 21.2635H62.8109Z"></path><path d="M75.7678 21.527C74.6776 21.527 73.7126 21.2854 72.8729 20.8023C72.0331 20.3045 71.3701 19.6237 70.884 18.7598C70.4125 17.896 70.1768 16.9004 70.1768 15.773C70.1768 14.6457 70.4199 13.6501 70.906 12.7862C71.407 11.9224 72.0847 11.2489 72.9392 10.7658C73.8084 10.268 74.8029 10.0191 75.9225 10.0191C76.8949 10.0191 77.7862 10.2753 78.5965 10.7877C79.4216 11.2855 80.0772 12.0249 80.5633 13.0059C81.0643 13.9722 81.3147 15.1508 81.3147 16.5417H73.2265C73.3001 17.2738 73.6169 17.8594 74.1767 18.2986C74.7366 18.7232 75.3701 18.9355 76.0772 18.9355C76.6813 18.9355 77.1748 18.8111 77.5579 18.5622C77.9409 18.2986 78.2356 17.9619 78.4418 17.5519L81.0495 18.5622C80.7401 19.1771 80.335 19.7115 79.8341 20.1654C79.3479 20.6046 78.766 20.9414 78.0883 21.1756C77.4106 21.4099 76.6371 21.527 75.7678 21.527ZM73.3811 14.3894H78.1545C78.1103 13.9356 77.963 13.5695 77.7126 13.2914C77.4621 13.0132 77.1675 12.8082 76.8286 12.6764C76.4898 12.5447 76.1583 12.4788 75.8341 12.4788C75.5247 12.4788 75.1859 12.5447 74.8176 12.6764C74.464 12.7936 74.1472 12.9912 73.8673 13.2694C73.6021 13.5476 73.4401 13.9209 73.3811 14.3894Z"></path><path d="M82.4937 21.2635V10.2826H85.4107V12.3031C85.8675 11.6149 86.442 11.1025 87.1345 10.7658C87.8416 10.4144 88.5709 10.2387 89.3223 10.2387V13.0498C88.6446 13.0498 88.0037 13.1303 87.3997 13.2914C86.8103 13.4524 86.3315 13.716 85.9632 14.082C85.5949 14.448 85.4107 14.9165 85.4107 15.4875V21.2635H82.4937Z"></path><path d="M90.8937 21.2635V10.2826H93.634L93.7666 11.776C94.1202 11.205 94.5695 10.7731 95.1146 10.4803C95.6744 10.1728 96.308 10.0191 97.0151 10.0191C97.8844 10.0191 98.6284 10.1947 99.2471 10.5461C99.8659 10.8975 100.337 11.4466 100.661 12.1933C100.986 12.9253 101.14 13.877 101.126 15.0483V21.2635H98.2085V15.6413C98.2085 14.8946 98.1201 14.3162 97.9433 13.9063C97.7812 13.4963 97.5455 13.2035 97.2361 13.0278C96.9267 12.8375 96.5658 12.7423 96.1533 12.7423C95.4019 12.7277 94.82 12.9546 94.4074 13.4231C94.0097 13.8916 93.8108 14.5871 93.8108 15.5095V21.2635H90.8937Z"></path><path d="M103.196 21.2635V10.2826H105.936L106.069 11.7101C106.407 11.1538 106.842 10.7365 107.372 10.4583C107.903 10.1655 108.492 10.0191 109.14 10.0191C110.039 10.0191 110.79 10.2094 111.395 10.5901C112.013 10.9707 112.463 11.5564 112.743 12.347C113.067 11.6003 113.523 11.0293 114.113 10.634C114.702 10.224 115.387 10.0191 116.168 10.0191C117.464 10.0191 118.451 10.429 119.129 11.2489C119.807 12.0542 120.146 13.2987 120.146 14.9824V21.2635H117.229V15.6413C117.229 14.8946 117.148 14.3162 116.986 13.9063C116.824 13.4963 116.603 13.2035 116.323 13.0278C116.043 12.8375 115.711 12.7423 115.328 12.7423C114.621 12.7277 114.076 12.9546 113.693 13.4231C113.31 13.8916 113.118 14.5871 113.118 15.5095V21.2635H110.201V15.6413C110.201 14.8946 110.12 14.3162 109.958 13.9063C109.811 13.4963 109.59 13.2035 109.295 13.0278C109.015 12.8375 108.684 12.7423 108.301 12.7423C107.593 12.7277 107.048 12.9546 106.665 13.4231C106.297 13.8916 106.113 14.5871 106.113 15.5095V21.2635H103.196Z"></path><path d="M126.946 21.527C125.856 21.527 124.891 21.2854 124.051 20.8023C123.211 20.3045 122.548 19.6237 122.062 18.7598C121.59 17.896 121.355 16.9004 121.355 15.773C121.355 14.6457 121.598 13.6501 122.084 12.7862C122.585 11.9224 123.263 11.2489 124.117 10.7658C124.986 10.268 125.981 10.0191 127.1 10.0191C128.073 10.0191 128.964 10.2753 129.774 10.7877C130.6 11.2855 131.255 12.0249 131.741 13.0059C132.242 13.9722 132.493 15.1508 132.493 16.5417H124.404C124.478 17.2738 124.795 17.8594 125.355 18.2986C125.915 18.7232 126.548 18.9355 127.255 18.9355C127.859 18.9355 128.353 18.8111 128.736 18.5622C129.119 18.2986 129.414 17.9619 129.62 17.5519L132.227 18.5622C131.918 19.1771 131.513 19.7115 131.012 20.1654C130.526 20.6046 129.944 20.9414 129.266 21.1756C128.588 21.4099 127.815 21.527 126.946 21.527ZM124.559 14.3894H129.332C129.288 13.9356 129.141 13.5695 128.891 13.2914C128.64 13.0132 128.345 12.8082 128.007 12.6764C127.668 12.5447 127.336 12.4788 127.012 12.4788C126.703 12.4788 126.364 12.5447 125.996 12.6764C125.642 12.7936 125.325 12.9912 125.045 13.2694C124.78 13.5476 124.618 13.9209 124.559 14.3894Z"></path><path d="M133.93 21.2635V10.2826H136.67L136.803 11.776C137.156 11.205 137.606 10.7731 138.151 10.4803C138.711 10.1728 139.344 10.0191 140.051 10.0191C140.92 10.0191 141.664 10.1947 142.283 10.5461C142.902 10.8975 143.373 11.4466 143.698 12.1933C144.022 12.9253 144.176 13.877 144.162 15.0483V21.2635H141.245V15.6413C141.245 14.8946 141.156 14.3162 140.979 13.9063C140.817 13.4963 140.582 13.2035 140.272 13.0278C139.963 12.8375 139.602 12.7423 139.189 12.7423C138.438 12.7277 137.856 12.9546 137.444 13.4231C137.046 13.8916 136.847 14.5871 136.847 15.5095V21.2635H133.93Z"></path><path d="M150.526 21.527C149.303 21.527 148.375 21.2269 147.742 20.6266C147.123 20.0263 146.813 19.1698 146.813 18.0571V12.7423H145.023V10.2826H146.813V6.85656H149.731V10.2826H152.493V12.7423H149.731V17.4202C149.731 17.8887 149.834 18.2474 150.04 18.4963C150.246 18.7306 150.548 18.8477 150.946 18.8477C151.093 18.8477 151.255 18.8184 151.432 18.7598C151.609 18.6866 151.793 18.5841 151.985 18.4524L153.001 20.6266C152.648 20.8901 152.25 21.1024 151.808 21.2635C151.381 21.4392 150.953 21.527 150.526 21.527Z"></path><path d="M186.422 21.2635V5.89025H191.66C192.72 5.89025 193.649 6.0513 194.444 6.37341C195.24 6.69551 195.859 7.18599 196.3 7.84485C196.742 8.48906 196.963 9.29432 196.963 10.2606C196.963 10.8609 196.809 11.4173 196.499 11.9297C196.205 12.4275 195.785 12.8594 195.24 13.2255C195.962 13.5915 196.514 14.082 196.897 14.6969C197.295 15.2972 197.494 16.8711 197.494 16.8711C197.494 16.8711 197.28 18.6061 196.853 19.265C196.426 19.9238 195.829 20.4216 195.063 20.7584C194.297 21.0951 193.398 21.2635 192.367 21.2635H186.422ZM189.383 18.6061H192.08C192.801 18.6061 193.369 18.4304 193.781 18.079C194.194 17.713 194.4 17.2298 194.4 16.6296C194.4 16 194.172 15.5095 193.715 15.1581C193.258 14.7921 192.632 14.6091 191.836 14.6091H189.383V18.6061ZM189.383 12.1494H191.726C192.389 12.1494 192.912 11.981 193.295 11.6442C193.678 11.3075 193.87 10.8316 193.87 10.2167C193.87 9.60179 193.656 9.12595 193.229 8.7892C192.801 8.45245 192.22 8.28408 191.483 8.28408H189.383V12.1494Z"></path><path d="M202.673 21.527C201.347 21.527 200.316 21.2342 199.58 20.6486C198.858 20.0629 198.497 19.2357 198.497 18.1669C198.497 16.9956 198.887 16.1171 199.668 15.5315C200.464 14.9312 201.576 14.631 203.005 14.631H205.613C205.509 13.9575 205.296 13.4451 204.972 13.0937C204.648 12.7277 204.184 12.5447 203.58 12.5447C203.108 12.5447 202.681 12.6472 202.298 12.8521C201.915 13.0571 201.591 13.3719 201.325 13.7965L198.762 12.918C198.968 12.4495 199.27 11.9956 199.668 11.5564C200.066 11.1025 200.581 10.7365 201.215 10.4583C201.863 10.1655 202.651 10.0191 203.58 10.0191C204.699 10.0191 205.627 10.2314 206.364 10.656C207.115 11.0805 207.668 11.6808 208.021 12.4568C208.39 13.2182 208.566 14.1259 208.552 15.1801L208.485 21.2635H205.767L205.723 19.9897C205.443 20.4875 205.045 20.8682 204.53 21.1317C204.029 21.3953 203.41 21.527 202.673 21.527ZM203.16 19.1112C203.631 19.1112 204.058 19.0014 204.441 18.7818C204.824 18.5622 205.126 18.2767 205.347 17.9253C205.568 17.5739 205.679 17.2079 205.679 16.8272V16.7833H204.021C203.064 16.7833 202.401 16.9004 202.033 17.1347C201.664 17.3543 201.48 17.6691 201.48 18.079C201.48 18.4011 201.627 18.6574 201.922 18.8477C202.217 19.0234 202.629 19.1112 203.16 19.1112Z"></path><path d="M210.512 21.2635V10.2826H213.429V12.3031C213.886 11.6149 214.461 11.1025 215.153 10.7658C215.86 10.4144 216.59 10.2387 217.341 10.2387V13.0498C216.663 13.0498 216.022 13.1303 215.418 13.2914C214.829 13.4524 214.35 13.716 213.982 14.082C213.614 14.448 213.429 14.9165 213.429 15.4875V21.2635H210.512Z"></path><path d="M224.7 21.527C223.948 21.527 223.293 21.3733 222.733 21.0658C222.173 20.7437 221.702 20.2898 221.318 19.7042L221.208 21.2635H218.446V5.89025H221.363V11.776C221.731 11.205 222.195 10.7731 222.755 10.4803C223.315 10.1728 223.963 10.0191 224.7 10.0191C225.716 10.0191 226.593 10.2533 227.329 10.7218C228.081 11.1904 228.663 11.8565 229.075 12.7204C229.488 13.5695 229.694 14.5871 229.694 15.773C229.694 16.9443 229.488 17.9619 229.075 18.8257C228.663 19.6896 228.081 20.3557 227.329 20.8242C226.593 21.2928 225.716 21.527 224.7 21.527ZM223.948 18.8697C224.479 18.8697 224.943 18.7379 225.34 18.4743C225.753 18.2108 226.077 17.8448 226.313 17.3763C226.549 16.9077 226.666 16.3733 226.666 15.773C226.666 15.1728 226.549 14.6384 226.313 14.1698C226.092 13.7013 225.775 13.3426 225.363 13.0937C224.965 12.8302 224.501 12.6984 223.97 12.6984C223.469 12.6984 223.02 12.8302 222.622 13.0937C222.225 13.3572 221.915 13.7233 221.694 14.1918C221.473 14.6457 221.363 15.1728 221.363 15.773C221.363 16.3733 221.473 16.9077 221.694 17.3763C221.915 17.8448 222.217 18.2108 222.6 18.4743C222.998 18.7379 223.447 18.8697 223.948 18.8697Z"></path><path d="M234.674 21.527C233.348 21.527 232.317 21.2342 231.58 20.6486C230.858 20.0629 230.497 19.2357 230.497 18.1669C230.497 16.9956 230.887 16.1171 231.668 15.5315C232.464 14.9312 233.576 14.631 235.005 14.631H237.613C237.51 13.9575 237.296 13.4451 236.972 13.0937C236.648 12.7277 236.184 12.5447 235.58 12.5447C235.108 12.5447 234.681 12.6472 234.298 12.8521C233.915 13.0571 233.591 13.3719 233.326 13.7965L230.762 12.918C230.969 12.4495 231.271 11.9956 231.668 11.5564C232.066 11.1025 232.582 10.7365 233.215 10.4583C233.863 10.1655 234.652 10.0191 235.58 10.0191C236.7 10.0191 237.628 10.2314 238.364 10.656C239.116 11.0805 239.668 11.6808 240.022 12.4568C240.39 13.2182 240.567 14.1259 240.552 15.1801L240.486 21.2635H237.768L237.723 19.9897C237.443 20.4875 237.046 20.8682 236.53 21.1317C236.029 21.3953 235.41 21.527 234.674 21.527ZM235.16 19.1112C235.631 19.1112 236.059 19.0014 236.442 18.7818C236.825 18.5622 237.127 18.2767 237.348 17.9253C237.569 17.5739 237.679 17.2079 237.679 16.8272V16.7833H236.022C235.064 16.7833 234.401 16.9004 234.033 17.1347C233.665 17.3543 233.48 17.6691 233.48 18.079C233.48 18.4011 233.628 18.6574 233.922 18.8477C234.217 19.0234 234.63 19.1112 235.16 19.1112Z"></path><path d="M246.758 21.527C245.742 21.527 244.858 21.2928 244.107 20.8242C243.37 20.3557 242.795 19.6896 242.383 18.8257C241.97 17.9619 241.764 16.9443 241.764 15.773C241.764 14.5871 241.97 13.5695 242.383 12.7204C242.795 11.8565 243.37 11.1904 244.107 10.7218C244.858 10.2533 245.742 10.0191 246.758 10.0191C247.48 10.0191 248.121 10.1728 248.681 10.4803C249.256 10.7731 249.72 11.1977 250.073 11.754V5.89025H252.99V21.2635H250.228L250.14 19.7262C249.771 20.2972 249.3 20.7437 248.725 21.0658C248.151 21.3733 247.495 21.527 246.758 21.527ZM247.51 18.8697C247.996 18.8697 248.423 18.7525 248.792 18.5183C249.175 18.2694 249.477 17.9253 249.698 17.4861C249.919 17.0468 250.044 16.549 250.073 15.9927V15.5534C250.044 14.9971 249.919 14.5066 249.698 14.082C249.477 13.6428 249.175 13.306 248.792 13.0717C248.409 12.8228 247.974 12.6984 247.488 12.6984C246.957 12.6984 246.486 12.8302 246.073 13.0937C245.676 13.3426 245.359 13.7013 245.123 14.1698C244.902 14.6384 244.792 15.1728 244.792 15.773C244.792 16.3733 244.909 16.9077 245.145 17.3763C245.381 17.8448 245.698 18.2108 246.095 18.4743C246.508 18.7379 246.979 18.8697 247.51 18.8697Z"></path><path d="M260.409 21.527C259.289 21.527 258.287 21.2854 257.403 20.8023C256.534 20.3045 255.849 19.6237 255.348 18.7598C254.862 17.896 254.619 16.9004 254.619 15.773C254.619 14.6457 254.862 13.6501 255.348 12.7862C255.834 11.9224 256.512 11.2489 257.381 10.7658C258.25 10.268 259.245 10.0191 260.364 10.0191C261.499 10.0191 262.501 10.268 263.37 10.7658C264.239 11.2489 264.917 11.9224 265.403 12.7862C265.889 13.6501 266.132 14.6457 266.132 15.773C266.132 16.9004 265.889 17.896 265.403 18.7598C264.917 19.6237 264.239 20.3045 263.37 20.8023C262.515 21.2854 261.528 21.527 260.409 21.527ZM260.409 18.8477C260.954 18.8477 261.433 18.7159 261.845 18.4524C262.258 18.1888 262.574 17.8301 262.795 17.3763C263.031 16.9077 263.149 16.3733 263.149 15.773C263.149 15.1728 263.031 14.6457 262.795 14.1918C262.56 13.7233 262.228 13.3572 261.801 13.0937C261.388 12.8302 260.91 12.6984 260.364 12.6984C259.834 12.6984 259.355 12.8302 258.928 13.0937C258.515 13.3572 258.191 13.7233 257.956 14.1918C257.72 14.6457 257.602 15.1728 257.602 15.773C257.602 16.3733 257.72 16.9077 257.956 17.3763C258.191 17.8301 258.523 18.1888 258.95 18.4524C259.377 18.7159 259.863 18.8477 260.409 18.8477Z"></path><path d="M271.536 21.527C270.829 21.527 270.166 21.4245 269.547 21.2196C268.928 20.9999 268.383 20.6925 267.912 20.2972C267.44 19.8872 267.072 19.3894 266.807 18.8038L269.304 17.6618C269.525 18.0131 269.827 18.3206 270.21 18.5841C270.593 18.833 271.035 18.9575 271.536 18.9575C272.022 18.9575 272.398 18.8916 272.663 18.7598C272.928 18.6134 273.061 18.4085 273.061 18.1449C273.061 17.8814 272.95 17.691 272.729 17.5739C272.523 17.4421 272.236 17.3323 271.867 17.2445L270.851 16.9809C269.79 16.7028 268.95 16.2635 268.332 15.6632C267.728 15.0483 267.426 14.3455 267.426 13.5549C267.426 12.4275 267.787 11.5564 268.508 10.9415C269.245 10.3265 270.284 10.0191 271.624 10.0191C272.317 10.0191 272.958 10.1215 273.547 10.3265C274.151 10.5315 274.667 10.817 275.094 11.183C275.521 11.5491 275.816 11.9737 275.978 12.4568L273.569 13.5549C273.466 13.2621 273.216 13.0278 272.818 12.8521C272.42 12.6618 272.022 12.5666 271.624 12.5666C271.227 12.5666 270.917 12.6545 270.696 12.8302C270.49 12.9912 270.387 13.2182 270.387 13.511C270.387 13.7013 270.49 13.8624 270.696 13.9941C270.902 14.1113 271.197 14.2138 271.58 14.3016L273.017 14.653C273.739 14.8287 274.313 15.1142 274.74 15.5095C275.182 15.8902 275.499 16.3221 275.691 16.8052C275.897 17.2738 276 17.735 276 18.1888C276 18.8477 275.801 19.4333 275.403 19.9458C275.02 20.4436 274.49 20.8316 273.812 21.1098C273.149 21.3879 272.39 21.527 271.536 21.527Z"></path><path d="M165.302 21.527C164.182 21.527 163.18 21.2855 162.296 20.8023C161.427 20.3045 160.742 19.6237 160.241 18.7598C159.755 17.896 159.512 16.9004 159.512 15.773C159.512 14.6457 159.755 13.6501 160.241 12.7862C160.727 11.9224 161.405 11.2489 162.274 10.7658C163.144 10.268 164.138 10.0191 165.258 10.0191C166.392 10.0191 167.394 10.268 168.263 10.7658C169.132 11.2489 169.81 11.9224 170.296 12.7862C170.782 13.6501 171.026 14.6457 171.026 15.773C171.026 16.9004 170.782 17.896 170.296 18.7598C169.81 19.6237 169.132 20.3045 168.263 20.8023C167.409 21.2855 166.422 21.527 165.302 21.527ZM165.302 18.8477C165.847 18.8477 166.326 18.7159 166.738 18.4524C167.151 18.1888 167.468 17.8301 167.689 17.3763C167.924 16.9077 168.042 16.3733 168.042 15.773C168.042 15.1728 167.924 14.6457 167.689 14.1918C167.453 13.7233 167.121 13.3572 166.694 13.0937C166.282 12.8302 165.803 12.6984 165.258 12.6984C164.727 12.6984 164.249 12.8302 163.821 13.0937C163.409 13.3572 163.085 13.7233 162.849 14.1918C162.613 14.6457 162.495 15.1728 162.495 15.773C162.495 16.3733 162.613 16.9077 162.849 17.3763C163.085 17.8301 163.416 18.1888 163.843 18.4524C164.271 18.7159 164.757 18.8477 165.302 18.8477Z"></path><path d="M173.144 21.2635V12.6105H171.309V10.2826H173.144V9.18451C173.144 8.01321 173.446 7.10546 174.05 6.46125C174.654 5.8024 175.574 5.47297 176.812 5.47297C177.239 5.47297 177.689 5.54618 178.16 5.69259C178.646 5.839 179.088 6.0147 179.486 6.21967L178.403 8.30604C178.167 8.18891 177.954 8.10106 177.762 8.04249C177.571 7.98393 177.394 7.95465 177.232 7.95465C176.834 7.95465 176.539 8.09374 176.348 8.37192C176.156 8.63547 176.061 9.06738 176.061 9.66767V10.2826H178.779V12.6105H176.061V21.2635H173.144Z"></path><path d="M12.5786 0C11.6253 2.46943 10.6536 5.03986 8.57466 7.00774C9.21998 6.80573 10.3493 6.62461 11.0753 6.64203V17.0283L7.988 17.4636C7.878 17.4532 7.84133 17.2895 7.84133 17.0666C7.54434 13.8413 6.74135 11.1316 5.81736 8.32779C5.75136 7.94466 4.58171 6.48181 5.4837 6.73955C5.5937 6.75348 6.80002 7.2202 6.60935 6.99729C4.96671 5.38119 2.56508 4.21788 0.233112 3.87306C0.0277812 3.8243 -0.0932171 3.92183 0.0901136 4.14822C3.18474 8.6726 5.77336 14.012 5.75136 20.3475C6.95401 20.3475 9.86897 19.6718 11.0753 19.6718V27H12.5969L12.9453 6.53406L12.5786 0Z"></path><path d="M12.5786 0C13.5319 2.46943 14.5036 5.03986 16.5825 7.00774C15.9372 6.80573 14.8079 6.62461 14.0819 6.64203V17.0283L17.1692 17.4636C17.2792 17.4532 17.3159 17.2895 17.3159 17.0666C17.6129 13.8413 18.4158 11.1316 19.3398 8.32779C19.4058 7.94466 20.5755 6.48181 19.6735 6.73955C19.5635 6.75348 18.3572 7.2202 18.5478 6.99729C20.1905 5.38119 22.5921 4.21788 24.9241 3.87306C25.1294 3.8243 25.2504 3.92183 25.0671 4.14822C21.9725 8.6726 19.3838 14.012 19.4058 20.3475C18.2032 20.3475 15.2882 19.6718 14.0819 19.6718V27H12.5603L12.2119 6.53406L12.5786 0Z"></path></g></svg>
```

In the header, use it like this:

```html
<header class="bg-bb-yellow-100">
  <div class="container mx-auto px-4 py-s">
    <div class="text-bb-black-00" style="width: 276px; height: 27px;">
      <!-- Paste the SVG above here -->
    </div>
  </div>
</header>
```

---

## Component patterns

All components are built with Tailwind utility classes. The patterns below are taken directly from the live alpha.gov.bb implementation. Reproduce them faithfully in prototypes.

### H1 (page heading)

```html
<h1 class="font-bold text-[3.5rem] leading-[1.15]">Tell us about yourself</h1>
```

### Form section caption

A left-bordered label above the H1, indicating which form the user is completing:

```html
<p class="border-bb-blue-40 border-l-4 py-xs pl-s text-bb-mid-grey-00">
  Apply to be a Project Protégé mentor
</p>
```

### Labels

Bold label for a text input or date group:

```html
<label for="first-name" class="block text-[1.25rem] leading-normal font-bold text-bb-black-00">
  First name
</label>
```

Non-bold label for radio/checkbox options:

```html
<label for="option-1" class="text-[1.25rem] leading-normal text-bb-black-00 cursor-pointer">
  Studying
</label>
```

### Hint text

```html
<p class="text-[1.25rem] leading-normal text-bb-mid-grey-00">
  For example, 27 03 2007
</p>
```

### Text input

The input sits inside a styled wrapper `<div>`:

```html
<div class="flex flex-col gap-xs w-full items-start">
  <label for="first-name" class="block text-[1.25rem] leading-normal font-bold text-bb-black-00">
    First name
  </label>
  <div class="relative inline-flex w-full rounded-sm border-2 border-bb-black-00 items-center gap-2 transition-all bg-bb-white-00 hover:shadow-form-hover focus-within:ring-4 focus-within:ring-bb-teal-100">
    <input
      type="text"
      id="first-name"
      name="first-name"
      class="w-full min-w-0 p-s outline-none rounded-[inherit] placeholder:text-bb-black-00/60"
    />
  </div>
</div>
```

For error state, add `border-bb-red-00` to the wrapper in place of `border-bb-black-00`, and add `aria-invalid="true"` to the input.

### Date input (Day / Month / Year)

Three narrow text inputs side by side:

```html
<div class="flex flex-col gap-xs w-full items-start">
  <p class="text-[1.25rem] leading-normal font-bold text-bb-black-00">Date of birth</p>
  <p class="text-[1.25rem] leading-normal text-bb-mid-grey-00">For example, 27 03 2007</p>
  <div class="flex gap-s items-end flex-wrap">
    <!-- Day -->
    <div class="flex flex-col gap-xs">
      <label for="dob-day" class="text-[1.25rem] leading-normal font-bold text-bb-black-00">Day</label>
      <div class="relative inline-flex rounded-sm border-2 border-bb-black-00 items-center transition-all bg-bb-white-00 hover:shadow-form-hover focus-within:ring-4 focus-within:ring-bb-teal-100" style="width: 5rem;">
        <input type="text" id="dob-day" name="dob-day" inputmode="numeric" class="w-full min-w-0 p-s outline-none rounded-[inherit]" />
      </div>
    </div>
    <!-- Month -->
    <div class="flex flex-col gap-xs">
      <label for="dob-month" class="text-[1.25rem] leading-normal font-bold text-bb-black-00">Month</label>
      <div class="relative inline-flex rounded-sm border-2 border-bb-black-00 items-center transition-all bg-bb-white-00 hover:shadow-form-hover focus-within:ring-4 focus-within:ring-bb-teal-100" style="width: 5rem;">
        <input type="text" id="dob-month" name="dob-month" inputmode="numeric" class="w-full min-w-0 p-s outline-none rounded-[inherit]" />
      </div>
    </div>
    <!-- Year -->
    <div class="flex flex-col gap-xs">
      <label for="dob-year" class="text-[1.25rem] leading-normal font-bold text-bb-black-00">Year</label>
      <div class="relative inline-flex rounded-sm border-2 border-bb-black-00 items-center transition-all bg-bb-white-00 hover:shadow-form-hover focus-within:ring-4 focus-within:ring-bb-teal-100" style="width: 7rem;">
        <input type="text" id="dob-year" name="dob-year" inputmode="numeric" class="w-full min-w-0 p-s outline-none rounded-[inherit]" />
      </div>
    </div>
  </div>
</div>
```

### Radio buttons

Custom circular radio buttons with a label to the right:

```html
<div class="flex flex-col gap-s items-start w-full">
  <p class="text-[1.25rem] leading-normal font-bold text-bb-black-00">What is your employment status?</p>
  <!-- Option -->
  <div class="flex gap-5 items-center">
    <button type="button" role="radio" aria-checked="false"
      class="relative inline-flex size-12 shrink-0 items-center justify-center bg-bb-white-00 border-2 border-bb-black-00 border-solid rounded-full transition-all outline-none hover:cursor-pointer hover:shadow-form-hover focus-visible:border-bb-teal-00 focus-visible:shadow-none focus-visible:ring-4 focus-visible:ring-bb-teal-100">
    </button>
    <label class="text-[1.25rem] leading-normal text-bb-black-00 cursor-pointer">Studying</label>
  </div>
  <!-- Repeat for each option -->
</div>
```

### Primary button (Continue / Submit)

```html
<div class="mt-8 flex gap-4">
  <button type="button"
    class="relative inline-flex items-center justify-center gap-2 text-[20px] whitespace-nowrap transition-[background-color,box-shadow] duration-200 outline-none bg-bb-teal-00 text-bb-white-00 hover:bg-[#1a777d] hover:shadow-[inset_0_0_0_4px_rgba(222,245,246,0.10)] active:bg-[#0a4549] active:shadow-[inset_0_0_0_3px_rgba(0,0,0,0.20)] px-xm py-s rounded-sm leading-[1.7] focus-visible:outline-none focus-visible:ring-4 focus-visible:ring-offset-1 focus-visible:ring-bb-teal-100 focus-visible:rounded-sm">
    Continue
  </button>
</div>
```

For a Start button (on a start/service page), the same styling is applied to an `<a>` tag instead.

### Links

Standard link:

```html
<a href="#"
  class="inline-flex outline-none underline-offset-2 underline hover:no-underline active:bg-bb-yellow-100 active:no-underline focus-visible:bg-bb-yellow-100 focus-visible:no-underline active:text-bb-black-00 focus-visible:text-bb-black-00 text-bb-teal-00 hover:text-bb-black-00 hover:bg-bb-teal-10">
  Alpha
</a>
```

Back link (with left arrow):

```html
<a href="#"
  class="inline-flex items-center gap-xs outline-none underline-offset-2 underline hover:no-underline active:bg-bb-yellow-100 focus-visible:bg-bb-yellow-100 text-bb-teal-00 hover:text-bb-black-00 hover:bg-bb-teal-10">
  ← Back
</a>
```

### Form spacing

- Between field groups within a page: `space-y-8` on the parent container
- Between sub-groups (e.g. related fields): `space-y-4`
- Label to input gap: `gap-xs` (0.5rem)
- Button area: `mt-8 flex gap-4`

---

## Writing style and tone

All text in the prototype must be written in **plain, simple language that a 9-year-old could understand**. This is critical for government services — citizens of all literacy levels need to use these forms.

### Rules

1. **Use short, common words.** Prefer "tell us" over "provide", "send" over "submit", "check" over "verify", "choose" over "select". Avoid jargon, legalese, and bureaucratic language.
2. **Use short sentences.** Keep sentences under 20 words where possible. One idea per sentence.
3. **Address the user directly.** Use "you" and "your", not "the applicant" or "the declarant".
4. **Use active voice.** Say "We will review your application" not "Your application will be reviewed".
5. **Explain technical terms.** If a field requires a specific ID or number (e.g. National Registration Number), include a brief hint explaining what it is and where to find it.
6. **Avoid double negatives and complex constructions.** Say "You must agree" not "You must not fail to agree".
7. **Be specific about what happens next.** On confirmation pages, tell the user exactly what will happen, when, and what they need to do (if anything).
8. **Use everyday words for buttons and links.** "Continue", "Go back", "Change", "Start now" — not "Proceed", "Return to previous", "Amend", "Commence".

### Examples

| ❌ Don't write | ✅ Write instead |
|---|---|
| "Please provide your National Registration Number as issued by the Registration Department" | "What is your National Registration Number? You can find this on your national ID card. For example, 870315-1234" |
| "The applicant must ensure all mandatory fields are completed prior to submission" | "Fill in all the fields before you continue" |
| "Your application has been received and will be processed in due course" | "We got your application. We will review it and contact you within 5 working days." |
| "Failure to comply with the regulations may result in penalties" | "If you do not follow the rules, you may have to pay a fine" |
| "Select the parish in which you currently reside" | "Which parish do you live in?" |

---

## Form structure rules

Follow the GOV.UK Service Manual guidance on structuring forms (<https://www.gov.uk/service-manual/design/form-structure>):

1. **One thing per page.** Each page should ask one question or present one decision. Split the form specification's sections across multiple pages accordingly. Group tightly related fields (e.g. first name + last name) on the same page only when they form a single conceptual question.

2. **Know why you're asking every question.** Only include fields marked as Required in the specification. Include optional fields only when the specification explicitly lists them.

3. **Design for the most common scenario first.** Put eligibility or filtering questions early so users find out quickly if they cannot proceed.

4. **Use branching (conditional logic).** Implement all IF/THEN rules from the specification's Conditional Logic section. When a condition is met, show the relevant follow-up on the next page or reveal it inline – whichever is more appropriate.

5. **Labels as page headings.** When a page asks a single question, make the `<label>` or `<legend>` the `<h1>` of the page. This avoids repetition and helps screen reader users.

---

## Page types to include

Every prototype must contain these pages, in order:

### 1. Start page

- H1: the form name from the specification
- Subtitle with last-updated date
- A short introductory paragraph explaining what the form does and who is eligible
- A "How to apply" section with a green primary button linking to the form: **"Complete the online form"** (styled as a teal `<a>` tag with button classes)
- A "What you will need to share" section listing what the user should have ready
- A Back link at the top
- The standard page chrome: top bar, header, alpha banner, footer

### 2. Question pages (one per question or tightly-related group)

Map specification sections to question pages as follows:

| Specification block | Pages to create |
|---|---|
| **Name Block** | One page: First Name, Middle Name(s), Last Name |
| **Personal Details Block** | Split into sensible single-question pages: Date of Birth → Gender → National Registration No. → National Insurance No. → Marital Status → Disability Status (with conditional textarea) |
| **Address Block** | One page: Street Address, Parish (dropdown), Postal Code. Include the "Mailing address same as present address" checkbox and conditional reveal. |
| **Contact Block** | One page: Landline, Mobile, Email. A separate page for Emergency Contact fields if present. |
| **Education Block** | One page per institution entry (repeatable). Include "Add another" pattern. |
| **Custom Sections** | Follow the same one-thing-per-page principle. |
| **Declaration Block** | One page with the declaration statement as static text, consent checkboxes, and (optionally) a signature capture placeholder. |

For each question page:
- Include a form section **caption** above the H1 (the left-bordered paragraph showing the form name)
- Include the H1 as the question or section title
- Include a **Back** link at the top of the content area
- The primary action button should say **"Continue"**
- Use the correct component for each field type as specified (Text Input, Date Input, Radio Buttons, Dropdown, Checkbox, Textarea, etc.)
- Display hint text and placeholder text exactly as specified
- Apply the validation rules noted in the specification (pattern, max length, required)

### 3. Check Your Answers page

- H1: **"Check your answers before sending your application"**
- Group answers by section using `<h2>` subheadings (e.g. "Personal details", "Address", "Contact details")
- Use a **summary list** layout to display each question and its answer in key–value rows. Each row has three columns: the question label, the answer value, and a **"Change"** link. Include visually hidden text for accessibility (e.g. `<span class="sr-only"> name</span>`)
- Each "Change" link navigates back to the relevant question page
- Show a primary button at the bottom: **"Submit application"** (or equivalent from the spec)
- Only display sections the user has completed; hide sections skipped via conditional logic

### 4. Confirmation page

- A confirmation panel with a teal background (`bg-bb-teal-00 text-bb-white-00`) containing a reference number (use a placeholder like "HDJ2123F") and confirmation heading
- Guidance on what happens next
- A link to return to the start or to the MDA website

---

## Conditional logic implementation

For each rule in the specification's Conditional Logic section:

- Use JavaScript to show/hide the dependent fields or to navigate to the appropriate next page.
- Keep logic simple and readable – use `data-` attributes on form elements to drive show/hide behaviour.
- Common patterns:
  - **Reveal within page:** A radio button selection reveals a textarea on the same page (e.g. Disability Status = "Yes" → show description field).
  - **Page-level branching:** An answer on one page determines which page comes next.
  - **Checkbox reveal:** Unchecking "Same as present address" reveals separate mailing address fields.

---

## Validation behaviour

Implement client-side validation matching the specification's rules:

- **Required fields:** Show an error message above the field and in an Error Summary at the top of the page if the user tries to continue without filling them in.
- **Error styling:** Change the input wrapper border to `border-bb-red-00` (`#a42c2c`) and add `aria-invalid="true"` to the input.
- **Error message text:** Display in `text-bb-red-00` above the field.
- **Pattern validation:** Apply regex patterns noted in the specification (e.g. National Registration Number must match `[0-9]{6}-[0-9]{4}`).
- **Max length:** Enforce character limits.
- **Date validation:** Dates cannot be in the future (unless the spec says otherwise); end dates must be after start dates.
- **Cross-field validation:** "At least one of Landline/Mobile" type rules.
- **Error message format:** Prefix field-level errors with the field name, e.g. "Date of Birth – Enter your date of birth".
- **Error summary:** At the top of the page, list all errors as links that jump to the relevant field. Prefix the page `<title>` with "Error: ".
- Do **not** use HTML5 native validation. Add `novalidate` to all `<form>` tags.

---

## Barbados-specific conventions

- **Parish dropdown values:** Christ Church, St. Andrew, St. George, St. James, St. John, St. Joseph, St. Lucy, St. Michael, St. Peter, St. Philip, St. Thomas
- **Phone format:** Accept any valid phone number format (local 7-digit like `555-1234`, with area code like `246-555-1234`, or international like `+1-246-555-1234`). Do not enforce a strict pattern — just validate that the field is not empty when required.
- **Postal code format:** `BB` followed by 5 digits (e.g. BB11000)
- **National Registration Number format:** `YYMMDD-XXXX`
- **Date format:** DD MM YYYY – use three separate text inputs for day, month, and year (as per the date input pattern above). Hint text: "For example, 27 03 2007"
- **Currency:** Barbadian Dollar (BBD / BDS$)
- **National Insurance Number:** 6 digits, numeric only

---

## Technical output requirements

Every prototype is a **single HTML file** that references three shared asset files. **Do not inline** the Tailwind config, base CSS, or framework JS — they are served from `/assets/`.

### Shared assets (loaded in every prototype)

| File | Purpose | Load in |
|---|---|---|
| `/assets/govbb-tailwind-config.js` | Tailwind colour/spacing/font config with `bb-` prefix | `<head>`, after Tailwind CDN |
| `/assets/govbb-base.css` | CSS custom properties, body grid, `.container`, `.sr-only` | `<head>` |
| `/assets/govbb-framework.js` | Navigation, template helpers, validation, submission | Bottom of `<body>`, before form script |

### Prototype HTML skeleton

Every prototype must follow this exact structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Form Name – MDA Name</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Figtree:wght@300..900&display=swap" rel="stylesheet">
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="/assets/govbb-tailwind-config.js"></script>
  <link rel="stylesheet" href="/assets/govbb-base.css">
</head>
<body>

<!-- Top bar -->
<div class="bg-bb-blue-100 text-bb-white-00">
  <div class="container flex items-center gap-2 py-2 text-[0.875rem]">
    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2L2 7l10 5 10-5-10-5z"/><path d="M2 17l10 5 10-5"/><path d="M2 12l10 5 10-5"/></svg>
    Official website of the Government of Barbados
  </div>
</div>

<!-- Header -->
<header class="bg-bb-yellow-100">
  <div class="container py-s">
    <div class="text-bb-black-00" style="width: 276px; height: 27px;">
      <!-- Paste the Government of Barbados SVG logo here -->
    </div>
  </div>
</header>

<!-- Alpha banner -->
<div class="bg-bb-blue-10">
  <div class="container py-xs text-[1rem]">
    This page is in <a href="#" class="inline-flex outline-none underline-offset-2 underline hover:no-underline active:bg-bb-yellow-100 active:no-underline focus-visible:bg-bb-yellow-100 focus-visible:no-underline active:text-bb-black-00 focus-visible:text-bb-black-00 text-bb-teal-00 hover:text-bb-black-00 hover:bg-bb-teal-10">Alpha</a>. Your feedback helps us improve.
  </div>
</div>

<!-- Main -->
<main>
  <div class="container py-l max-w-3xl" id="app"></div>
</main>

<!-- Footer -->
<footer class="bg-bb-blue-100 text-bb-white-00">
  <div class="container py-l text-[1rem]">
    <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-4">
      <p>&copy; 2024 Government of Barbados. All rights reserved.</p>
      <div class="flex gap-4">
        <a href="#" class="underline underline-offset-2 hover:no-underline text-bb-white-00">Privacy</a>
        <a href="#" class="underline underline-offset-2 hover:no-underline text-bb-white-00">Terms</a>
        <a href="#" class="underline underline-offset-2 hover:no-underline text-bb-white-00">Accessibility</a>
      </div>
    </div>
  </div>
</footer>

<script src="/assets/govbb-framework.js"></script>
<script>
/* ───────── Form-specific configuration ───────── */
const FORM_NAME = 'My Form Name';

const FLOW = ['start', 'page-1', 'page-2', 'declaration', 'check', 'confirmation'];

const PAGES = {
  'start': () => `
    <div class="space-y-8">
      <h1 class="font-bold text-[3.5rem] leading-[1.15]">${FORM_NAME}</h1>
      <!-- ... start page content ... -->
      ${GovBB.startBtn()}
    </div>`,

  'page-1': () => `
    <form novalidate>
      ${GovBB.backLink()}
      ${GovBB.caption()}
      <h1 class="font-bold text-[3.5rem] leading-[1.15] mb-8">Question title</h1>
      <div class="space-y-8">
        ${GovBB.textField('field-id', 'Field label')}
        ${GovBB.continueBtn()}
      </div>
    </form>`,

  // ... more pages ...

  'check': () => `
    ${GovBB.backLink()}
    <h1 class="font-bold text-[3.5rem] leading-[1.15] mb-8">Check your answers</h1>
    <div class="space-y-8">
      <dl class="divide-y divide-bb-grey-00 border-t border-bb-grey-00">
        ${GovBB.summaryRow('Field label', GovBB.D['field-id'], 'page-1')}
      </dl>
      ${GovBB.continueBtn('Submit application')}
    </div>`,

  'confirmation': () => `
    <div class="space-y-8">
      <div class="bg-bb-teal-00 text-bb-white-00 p-8 rounded-sm space-y-4">
        <h1 class="font-bold text-[3.5rem] leading-[1.15]">Application submitted</h1>
        <p class="text-[1.5rem]">Your reference number</p>
        <p class="text-[2rem] font-bold">${window.__refNumber || 'REF-' + Math.random().toString(36).substring(2,8).toUpperCase()}</p>
      </div>
    </div>`
};

/* ───────── Validation ───────── */
function validate(pageId) {
  const D = GovBB.D;
  const errors = [];
  // ... validation logic per pageId ...
  return errors;
}

/* ───────── Init ───────── */
GovBB.init({
  formName: FORM_NAME,
  flow: FLOW,
  pages: PAGES,
  validate: validate,
  // getFlow: function() { ... }  // optional: for dynamic conditional flows
});
</script>
</body>
</html>
```

### GovBB Framework API

The framework is loaded via `<script src="/assets/govbb-framework.js"></script>` and exposes a `GovBB` global object.

**Data & Constants:**
- `GovBB.D` — shared form data object (key-value store)
- `GovBB.PARISHES` — array of 11 Barbados parishes

**Navigation:**
- `GovBB.init(config)` — initialize the framework (see config options below)
- `GovBB.nav(pageId)` — navigate to a specific page by ID
- `GovBB.next()` — validate current page and advance (auto-submits before confirmation)
- `GovBB.back()` — go to the previous page
- `GovBB.render()` — re-render the current page

**Template helpers** (return HTML strings for use in PAGES templates):
- `GovBB.backLink()` — back link with left arrow
- `GovBB.caption(text?)` — form section caption (defaults to formName)
- `GovBB.continueBtn(label?)` — primary continue/submit button (default: "Continue")
- `GovBB.startBtn(label?)` — start page link-button (default: "Complete the online form")
- `GovBB.textField(id, label, opts?)` — text input with label, hint, error placeholder
  - opts: `{ hint, width, inputmode, maxlength, placeholder }`
- `GovBB.emailField(id, label, opts?)` — email input
  - opts: `{ hint }`
- `GovBB.telField(id, label, opts?)` — telephone input
  - opts: `{ hint, placeholder, width }`
- `GovBB.selectField(id, label, options, opts?)` — dropdown select
  - options: array of strings or `{ value, label }` objects
  - opts: `{ hint }`
- `GovBB.textareaField(id, label, opts?)` — textarea
  - opts: `{ hint, rows, maxlength }`
- `GovBB.dateField(prefix, label, hint?)` — Day/Month/Year triple input (stores `prefix-day`, `prefix-month`, `prefix-year`)
- `GovBB.radioGroup(name, label, options, opts?)` — radio button group
  - options: array of strings or `{ value, label }` objects
  - opts: `{ hint }`
- `GovBB.checkboxItem(name, label)` — single checkbox
- `GovBB.summaryRow(label, value, changeTo)` — Check Your Answers row with Change link

**Validation:**
- `GovBB.clearErrors()` — clear all error states
- `GovBB.showFieldError(id, msg)` — show error on a specific field
- `GovBB.showErrors(errors)` — show error summary and inline errors (errors: `[{id, msg}]`)

**Interaction:**
- `GovBB.selectRadio(name, value)` — handle radio selection
- `GovBB.toggleCheckbox(name)` — toggle checkbox state

**CSS class constants** (for building custom markup):
- `GovBB.BTN_CLS` — primary button classes
- `GovBB.LINK_CLS` — standard link classes
- `GovBB.INPUT_WRAP_CLS` — input wrapper classes
- `GovBB.INPUT_CLS` — input element classes

**Init config options:**
```javascript
GovBB.init({
  formName: 'My Form',                    // required — form title
  flow: ['start', 'p1', 'check', 'confirmation'], // required — page order
  pages: { 'start': () => `...`, ... },    // required — page templates
  validate: (pageId) => [],                // required — return [{id, msg}] array
  getFlow: null,                           // optional — dynamic flow function
  appElementId: 'app',                     // optional — defaults to 'app'
  onRadioChange: null,                     // optional — callback(name, value)
});
```

**Global aliases** (for use in `onclick` handlers):
`next()`, `back()`, `nav()`, `goBack()`, `goTo()` are all available as bare globals.

### Key rules

- **Do NOT inline** CSS custom properties, Tailwind config, or framework JS — use the shared assets
- The prototype must be fully clickable — navigation between pages works via the GovBB framework
- Use semantic HTML5 elements (`<main>`, `<fieldset>`, `<legend>`, `<label>`, etc.)
- All form inputs must be properly associated with their labels using `for`/`id` attributes
- The prototype should be responsive (mobile-first, with content constrained within the container)
- Include the persistent **header**, **alpha banner**, and **footer** as static HTML in every prototype (as shown in the skeleton above)
- Wrap each question page content in `<form novalidate>` to prevent HTML5 native validation

---

## Form submission integration

The GovBB framework **automatically handles form submission**. When `GovBB.next()` detects the next page in the flow is `'confirmation'`, it POSTs the form data to `/api/submit` before navigating.

### How it works

1. The framework calls `POST /api/submit` with `{ formName, formData, userEmail }`.
2. The server generates a unique reference number and sends confirmation/notification emails via Resend.
3. The reference number is stored in `window.__refNumber`.
4. The confirmation page renders with the server-generated reference (or a client-side fallback).

**You do NOT need to write any submission code in the prototype.** The framework handles it.

### Using the reference number on the confirmation page

In the confirmation page template, use `window.__refNumber` with a fallback:

```javascript
<p class="text-[2rem] font-bold">${window.__refNumber || 'REF-' + Math.random().toString(36).substring(2,8).toUpperCase()}</p>
```

### Adding a new form to the server

When creating a new prototype, you must also:

1. Add the form name → prefix mapping in `lib/reference.js` (e.g. `'My New Form': 'MNF'`).
2. Ensure the form collects an email address (field ID `contact-email` or `email`) if applicant confirmation is needed.

---

## How to read the Form Specification input

The user will provide a completed Form Specification document. Parse it as follows:

1. **Form Metadata** – extract the form name, MDA, complexity, estimated time, total fields, and flow type.
2. **Start Page** – use the title, subtitle, introduction, estimated time, "What You'll Need" checklist, and eligibility criteria.
3. **Section blocks** (Name, Personal Details, Address, Contact, Education, Custom) – for each field, note the Component type, Label/Help Text, Required status, and Validation rules. Create the appropriate pages.
4. **Conditional Logic** – implement every IF/THEN rule listed.
5. **Declaration Block** – include declaration text, consent checkboxes, signature capture (if listed), auto-dated date field, and submit button.
6. **Notes & Edge Cases** – honour any special instructions.
7. **Complexity Assessment** – use this to gauge whether the prototype needs advanced features like repeatable blocks, file uploads, or multi-party flows.

---

## Checklist before delivering the prototype

- [ ] References shared assets (`govbb-tailwind-config.js`, `govbb-base.css`, `govbb-framework.js`) — no inline CSS/JS for shared code
- [ ] Uses GovBB framework API for template helpers, navigation, and form submission
- [ ] Follows the prototype HTML skeleton structure exactly
- [ ] Start page with all required elements
- [ ] One thing per page for every question
- [ ] All fields from the specification present with correct component types
- [ ] All conditional logic working
- [ ] Check Your Answers page with summary list and Change links
- [ ] Client-side validation with error summary and inline errors
- [ ] Back links on every question page
- [ ] Alpha banner on every page
- [ ] Correct use of Figtree font and alpha.gov.bb design tokens
- [ ] Tailwind utility classes matching the patterns documented above
- [ ] Responsive layout (container max-width: 1200px)
- [ ] All labels associated with inputs via `for`/`id`
- [ ] Barbados-specific data formats (DD MM YYYY dates, parish list, phone format, postal codes)
- [ ] Confirmation page with `window.__refNumber` fallback
- [ ] Form name → prefix mapping added to `lib/reference.js`

---
> Source: [Kruck95/govbb-prototypes](https://github.com/Kruck95/govbb-prototypes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
