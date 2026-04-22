## timeline-pro

> You are assisting in building a Timeline Builder app for monday.com, targeted at project managers. The app automatically generates interactive timeline graphics from a monday.com board. It uses React and Next.js, and connects to the monday.com platform via the monday SDK (monday-sdk-js) and GraphQL API. Adhere to the following rules:


You are assisting in building a Timeline Builder app for monday.com, targeted at project managers. The app automatically generates interactive timeline graphics from a monday.com board. It uses React and Next.js, and connects to the monday.com platform via the monday SDK (monday-sdk-js) and GraphQL API. Adhere to the following rules:
- use vibe/core components for react components and Icons. available components are: Accordion, AlertBanner, AttentionBox, Avatar, AvatarGroup, Badge, BreadcrumbsBar, Button, ButtonGroup, Checkbox, Chips, ColorPicker, Combobox, Counter, DatePicker, Dialog, DialogContentContainer, Divider, Dropdown, EditableHeading, EditableText, EmptyState, ExpandCollapse, Icon, IconButton, Label, LinearProgressBar, Link, List, Loader, Menu, MenuButton, Modal, Modal, MultiStepIndicator, NumberField, RadioButton, Search, Skeleton, Slider, SplitButton, Steps, Table, Tabs, TextArea, TextField, Tipseen, Toast, Toggle, Tooltip, VirtualizedGrid, VirtualizedList
- The app is wrapped with a ThemeProvider, so don't use custom css configs. As long as we are using vibe/core components, they will style themselves based on the theme.
- If needed, use ThemeProvider.colors to style the app ({primaryColor: 'primary-color', primaryHoverColor: 'primary-hover-color', primarySelectedColor: 'primary-selected-color', primarySelectedHoverColor: 'primary-selected-hover-color', primarySelectedOnSecondaryColor: 'primary-selected-on-secondary-color',…})
- use the 'settings' object to configure the timeline. options are: title (title of the timeline), column (column to use for timeline items), backgroundColor (background color of the timeline), scale (scale of the timeline, weeks, days, etc), date (date of each item on the timeline), primaryColor (color to use for timeline items), secondaryColor (color to use for timeline items), tertiaryColor (color to use for timeline items)
- use/create server actions in app/actions in favor of API routes
- use built in next.js features for navigation, caching, headers, etc.
- keep functions in src/functions in a file named in camelCase after the function. Create a new file for each function and then import the function into the component that uses it. Do this instead of keeping everything in one big component file.
- provider files go in src/providers
- feature components go in src/components in a directory named after the feature
- hooks go in src/hooks
- services go in src/services
- monday.com development docs: https://developer.monday.com/apps/docs/create-an-app
- monday.com graphql schema: https://api.monday.com/v2/get_schema

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slape) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
