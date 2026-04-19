## dw-sitepref-extractor

> For full project context, read AGENTS.md in the project root first.

For full project context, read AGENTS.md in the project root first.

CRITICAL API CONSTRAINTS:
- Chakra UI v3 (NOT v2): Use compound components — Checkbox.Root, Accordion.Root, Dialog.Root, Card.Root/Card.Body. Use colorPalette not colorScheme.
- react-window v2 (NOT v1): Use <List rowComponent={Comp} rowCount={n} rowHeight={h} rowProps={obj} />. NOT FixedSizeList/itemCount/children-render-function.
- React 19 with createRoot API.
- React Router v7: BrowserRouter with basename="/dw_sitepref_extractor". Use useNavigate() and useLocation() hooks.
- No backend. Client-side only. IndexedDB via idb library for persistence.
- Build tooling: CRA + react-app-rewired (not raw react-scripts).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gigor911) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
