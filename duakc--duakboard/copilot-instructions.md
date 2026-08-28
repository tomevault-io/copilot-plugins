## duakboard

> - Build a new React dashboard for sing-box and Clash API backends.

# Agent Notes

## User goals

- Build a new React dashboard for sing-box and Clash API backends.
- Use zashboard as the reference for API behavior and page ideas.
- Keep the layout responsive and single-column on the connections and proxies pages.
- Add multi-instance management and mobile adaptation.
- Use a light business-style theme with minimal border radius.
- Keep the implementation incremental; more features will come later.
- Ship a self-contained static build and a zip artifact for offline use.

## Important constraints

- Use Node + pnpm.
- Avoid creating unnecessary system clutter.
- Document that the project is largely AI-built in the README.
- Keep future work compatible with zashboard-style API flows.

## Current implementation notes

- Connections use dense clickable rows with search, status tabs, sorting, close actions, and a detail modal.
- Proxy groups stay in one vertical column while their nodes use a compact responsive grid. Groups can collapse, nodes and groups can be tested, and provider health checks are supported.
- The desktop sidebar quick-settings area contains only the fast mode switch. Mihomo/Clash backends expose their `GLOBAL`/`全局` group in Proxies and show only that group while the backend mode is `global`; sing-box backends hide those reserved groups and use per-instance Settings values for the custom global mode and global group, showing only that configured group while the custom mode is active. Mobile keeps the instance picker in the top toolbar.
- Telemetry history is kept per instance in React memory, so instance switching preserves charts while a browser refresh clears them.
- Rule Flow UI state is kept in an in-memory store keyed by instance ID, so route navigation restores positions, selections, filters, zoom, and dynamic color allocation; a full browser refresh clears it. Full-layout positions are kept separately from the temporary compact filtered layout, so leaving a filter restores the complete canvas without reusing filtered row/column coordinates. Initial placement, new-node placement after a data refresh, and explicit layout reset share the same full-layout position generator and the instance's first-render layout baseline; later DOM measurements do not expand the full-view row spacing. A refresh with unchanged nodes reuses the existing position object, while a changed rule/target set creates a new baseline.
- Display names are `Clash API` and `sing-box API`.
- Rules are visualized in Overview as a compact draggable rule-flow board; Logs remains a separate page alongside Connections, Proxies, and Settings. Instance editing is only exposed through the sidebar picker.
- The desktop topbar is a page toolbar. Connections and Proxies mount their search and actions into it; the instance picker stays in the sidebar on desktop and appears in the topbar only on mobile.
- The API client covers the zashboard-style Clash REST surface, connection actions, proxy/provider/rule operations, DNS/cache/Geo/config actions, maintenance/storage/smart/honk helpers, and traffic/memory/connections/logs WebSockets.

## Latest UI behavior

- The default sidebar order is Overview, Proxies, Connections, Logs, Settings. Desktop navigation items are draggable and persist in `localStorage`; mobile navigation follows the same order.
- Logs filters and clearing are mounted in the compact top toolbar. Connections filters, status tabs, sorting, and close-all are also in the top toolbar.
- Connections default to descending download traffic, then Chains and Host as deterministic tie-breakers. The dense table has a stable horizontal scroll width, sortable headers, neutral X close actions, and a centered floating detail card whose body scrolls independently.
- Connection Raw details render the complete JSON payload with lightweight syntax colors for keys, strings, numbers, booleans, and null values.
- Proxy nodes use fixed rectangular two-row cards in a single-column group list. Every node card owns an independent loading key, so one test never disables selection or another node's test. Individual, group, all-group, and provider health checks preserve latency results in page state, animate values from zero only when a test result arrives, show existing values immediately after page navigation, keep the test button available, omit the redundant `ms` suffix, and color the latency number by health.
- Overview renders the three telemetry charts and the compact rule-flow board in the main content area; there is no separate Rules page in navigation. The charts use a fixed latest-sample window, progressively reveal from the newest point at the right, expose readable Y-axis labels, hover crosshairs, and value tooltips; traffic and memory use filled areas. Runtime status, instance/API details, theme, mode, and global-group selection remain in the sidebar quick-settings/telemetry area.
- Desktop instance management is a compact, bounded, scrollbar-hidden sidebar list: each instance is probed through `/version`, shows a colored availability dot, and exposes API, endpoint, version, and note details on hover. Clicking the active row opens its editor, other rows switch the active instance, and Add instance opens the same modal. Mobile keeps the compact topbar picker. There is no Instances navigation entry.
- The generic instance editor modal uses its own entrance animation; centered detail dialogs use a separate transform so they cannot shift generic modals.
- Panels use subtle surface contrast and no elevation shadow; main content panels are borderless and flush with the page. Rules search/count is a compact inline summary rather than a full-width notice bar.
- Main content panels are full-width, borderless, and flush with the page; their low-contrast surface tint blends into the light business background. Scrolling remains available while browser scrollbars are globally hidden.
- Log filtering and result counts stay in one compact top toolbar; there is no separate count panel. The level select and search field use fixed/minimum grid tracks so neither can cover the other.
- Proxy Groups/Providers are tabs in the Proxies top toolbar. The page body is a single workspace of group sections; group content remains mounted while collapsing so the height and opacity animation can complete without replaying page-load animations.
- Group health tests run each member node independently through the proxy delay API, so each latency value is painted as soon as its own request completes instead of waiting for the group batch.
- Proxy group types such as `selector` and `urltest` are low-emphasis helper text beside their icons. The selected node is a single small line in the group header, and the header stays identical while expanding except for the arrow rotation.
- Proxy group health dots are larger, right-to-left, wrapping indicators shown in both collapsed and expanded headers. Each dot is an independent button that selects its node, has an immediate CSS name tooltip, and keeps an accent ring without being clipped at either edge.
- The web shell does not show project branding or explanatory notes. The sidebar quick-settings area keeps the mode selector but does not repeat the current mode as a separate `Mode ...` status line.
- Connection details support Overview/Raw switching with Mac trackpad horizontal wheel gestures and touch swipe gestures while retaining vertical scrolling. Overview's rule-flow cards can be dragged within the current session and use local SVG connectors without external assets.
- The Overview rule-flow calculates its row count from the available desktop height, grows horizontally as rules and route targets increase, and keeps the page within the viewport. Rule numbers are generated client-side when an API omits indexes; clicking a rule, route target, or connector highlights its related endpoints and connectors.
- Rule Flow route targets are sorted by target content while rule order remains the backend/table order. Cards size themselves from their content within compact min/max width and height bounds without a forced equal height; long payloads scroll inside the card with hidden scrollbars, and measured DOM dimensions are used for connector endpoints. Cards use compact top-left index bubbles and a reset-layout control. The initial layout uses one shared row baseline for rules and route targets, then scores candidate row counts against card height, viewport aspect ratio, and horizontal/vertical overflow so the result stays rectangular instead of forming an L-shape. Overflow remains inside the Flow scroll area. Users can then drag cards freely. Clicking any rule, route target, or connector selects the complete target relationship: every rule routed to that target, its connector, and the target card share one highlight color. Ctrl/Command adds or removes complete target relationships with distinct color groups generated dynamically from the selection index. Holding Ctrl/Command while dragging moves only the highlighted nodes in the same color group and same side: rules move with same-color rules, and route targets move with same-color route targets. Dragging without the modifier moves one node and never alters selection. The flow toolbar can clear selection or show only the current selection; while filtering, the canvas renders from the filter snapshot so clearing a live selection does not immediately blank the filtered view. Leaving the filter restores the saved full-layout positions, and reset clears all positions so the normal full-layout initializer computes them again. The flow canvas owns its horizontal and vertical scrolling with overscroll containment so trackpad gestures do not scroll the main page, and supports local zoom controls plus Ctrl/Command-wheel and Safari gesture zoom without browser page zoom.
- The top toolbar is a collapsible hover/focus surface. Its page-specific tools are individually draggable, show a drop target, and persist their order per route in local storage; icon buttons retain their labels through native tooltips.
- Search inputs use a subtle square treatment with no visible border until focus. Logs default to Info, retain a maximum 1000-entry rolling buffer, support pointer-drag and Ctrl/Command multi-selection with indeterminate select-all state and copy, and expose automatic/manual refresh controls. Proxy node switching can optionally close active connections whose chain contains the selected group.
- Disabled buttons use a static unavailable cursor; only explicitly rendered request indicators use the spinning animation.
- Backend-specific behavior must be gated by capability detection: `isSingBoxBackend` returns true when the configured API kind is `singbox` or the `/version` response contains `sing-box`. Sing-box log ID grouping is enabled only under that check; mihomo/Clash keeps ordinary per-row log selection.
- Complex modal behavior uses `@radix-ui/react-dialog` for focus trapping, escape/outside close, and centered positioning; simple side-scrolling instance navigation remains in the sidebar document flow.

---
> Source: [duakc/duakboard](https://github.com/duakc/duakboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
