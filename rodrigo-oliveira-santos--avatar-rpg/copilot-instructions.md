## avatar-rpg

> **Updated:** 2026-06-30

# Avatar RPG — GitHub Copilot Instructions

**Updated:** 2026-06-30

## Stack
- Frontend: HTML5 + CSS3 + JS ES6 modules (no framework, no bundler)
- Backend: Netlify Functions + Supabase (PostgreSQL) + Supabase Realtime
- Default: localStorage; opt-in Supabase via `public/config.js` (`useSupabase: true`) or `?supabase=1`
- Hosting: Netlify free tier
- **Cross-Origin Isolation** activado via `public/serve.json` (dev) +
  `netlify.toml` (prod) — necessário para o iframe Godot (mapa do Hub
  Avatar) usar SharedArrayBuffer.

## Status: Multi-game platform (Avatar + D&D + Minecraft) + admin cross-app

Três apps independentes a partilhar landing, login overlay e
infraestrutura (router, registry de utilizadores, persistência).

| App | Status | Tabs |
|-----|--------|------|
| Avatar | Fases 1-6 + GM tooling + combat + cross-browser trades + GM delegated modals + Hub map | Personagem · Skills (×5) · Itens · Loja · Hub · Importar · Admin · GM Control · Monstros |
| D&D 5e | MVP + Admin | Ficha · Perícias · Magias · Inventário · Trade · Hub · Importar · Admin |
| Minecraft | MVP + Admin | Galeria · Painel Pessoal · Minhas Listas · Admin |
| Landing | Game selector + Admin Global | botão admin se sessão admin activa |

### Recente (2026-06-30) — Combat tweaks + chi regen + item modifiers

- **Vocabulário de combate**: "ronda" → **Turno** (loop completo);
  "turno" individual → **Vez** (slot de cada combatente). Apenas
  strings de UI; colunas `current_round` / `current_turn_index`
  ficam como estão. Botões agora: `Próxima vez →`, `Fim da minha vez`,
  badge `Turno N`.
- **Status effects no fim da vez**: DoT (sangrando, queimadura,
  regeneração) movidos de `tick_when:'start'` para `'end'` — efeitos
  aplicam-se depois das ações do alvo. Effects de bloqueio de ação
  (stun/paralisia/medo/congelado) ficam em `'start'`. Default no
  engine (`processEffects`) também passou explicitamente a `'end'`.
- **Chi regen +20 a cada 2 turnos**: novo `combat/regen.js` cobre
  jogadores (via `updateVitals`) e monstros (via `tickPatch` quando
  têm `cp_max` definido). Disparado em rondas 3/5/7… no GM
  "Próxima vez" e no jogador "Fim da minha vez". Toast resume
  quantos foram afetados.
- **Monsters chi pool**: nova migração
  `20260630000000_monsters_chi.sql` adiciona `cp_max` + `cp_current`
  (nullable). Editor de monstros tem helper `_numberNullable` — em
  branco = monstro não usa chi (regen ignora-o).
- **Item modifiers**: `validators.js` aceita `item.modifiers`
  (string, opcional). UI mostra bloco "Modificadores" + aviso
  `⚠ Cálculos automáticos pendentes no backend.` no detalhe do item.
- **Skill chi_cost top-level**: `validators.js` aceita
  `skill.chi_cost` (number ≥ 0). `SkillCard` renderiza chip
  `Chi: N` ao lado do tier. Per-attack `attack.chi_cost` continua
  inalterado.
- **Ferramentas GM colapsáveis**: HubPage renomeou "Ferramentas GM
  (simulado)" → "Ferramentas GM" e ganhou botão
  `▲ Esconder / ▼ Mostrar` (mesma UX do mapa, persistido em
  `localStorage.avatar_rpg_hub_gmtools_collapsed`).

### Recente (anterior 2026-06-30) — Multi-game platform + admin panels

- **Multi-game platform**: landing (#/) + router (`#/<game>/<page>`) +
  per-game session isolation (`avatar_rpg_user` / `dnd_user` /
  `mc_user`). Single-game mode via `?game=<id>` (`npm run
  dev:avatar|dnd|minecraft`).
- **D&D 5e MVP+Admin**: full sheet, multiclass simples, trade com
  pre-validation, pack import 4 domínios (spells / subclasses / magic
  items / races), tab Admin com modo **impersonate** (admin edita
  ficha de outro user com banner roxo).
- **Minecraft MVP+Admin**: galeria pública 2 vistas (grelha + lista),
  tags, links sociais (YT/IG), likes/dislikes, playlists "Minhas
  Listas", tab Admin com role mgmt + transferência de builds
  single/bulk.
- **Admin Global** (landing): painel agregador visível quando alguma
  sessão activa for admin. Stats por jogo + role mgmt cross-app +
  apagar conta cascata.
- **Módulo partilhado `games/lib/users-registry.js`**: registry
  `avatar_rpg_users_registry` + validação + sincronização das **4
  chaves de sessão** (`avatar_rpg_user`, `dnd_user`, `mc_user`,
  `landing_user`) quando uma role muda. Avatar `AdminPanel` delega
  aqui o role-change (bug-fix: antes só sincronizava sessão Avatar).
- **Login overlay partilhado** (`#login-overlay`) com brand+cor por
  jogo via CSS var `--login-accent` + botão "← Início" global.

### Recente (Avatar — pré-multi-game)

- **GM Control delegated modals**: `PlayerShopModal` (comprar por
  jogador), `PlayerInventoryModal` (equip/unequip/usar),
  `PlayerSkillsModal` (skill tree). Nome do jogador + 👤 abre
  `CharacterModal` read-only. `💰 Recompensas em Grupo` + `🎁 Entregar
  Loot` no header (reusa Hub).
- **Admin delete account**: cascata users + cleanup orphan trades +
  guard self / last-admin.
- **Hub Avatar map**: iframe Godot (`html.itch.zone/html/8396265/`)
  cacheado + colapsável. Requer COI; loader supabase-js movido para
  jsdelivr (envia CORP).
- **Lazy non-bender path picker**: só aparece quando user abre tab
  "Sem Dobra"; dismissable; tabs de preview antes de committar.
- **Encounters** + EncounterPanel + status-effect tick engine +
  `endOwnTurn` do lado do jogador.
- **Persistência cirúrgica**: colunas `hp_current` / `cp_current` /
  `sp_current` / `status_effects` / `player_notes` / `gm_notes` em
  `characters`.
- **Trades cross-browser** com Realtime + histórico.
- **Loja GM**: modo Gerir + `shop_profiles` (bundles).
- **Skill system v2**: 5 tiers, 5 branches (sp/ag/cb/pr/br); `cb`
  parte em `pr`/`br` no tier 3+ (mutually exclusive, persiste em
  `character.combat_path`); `element='none'` parte em `chiblocker`/
  `weapons` (`character.non_bender_path`); mastery M0-M3 automática
  por número de usos. Source-of-truth `docs/skill-trees/*.html` →
  `data/skills/*.json` via `scripts/extract-skill-trees.mjs`.

## Módulos

### Avatar
| Module | Path | Purpose |
|--------|------|---------|
| auth | js/auth/ | AuthManager Avatar |
| character | js/character/ | Character class, stats, XP, level-up, subclasses, slots, AttributeStrip, NotesEditor |
| skills | js/skills/ | SkillTree (canvas + cards), SkillCard, SkillPanel, PathPicker (lazy non-bender), CanvasTreeView, mastery (M0-M3), 5 tiers, 5 branches |
| items | js/items/ | InventoryPage, equip/unequip, scrolls |
| shop | js/shop/ | ShopPage (player) + ShopManager (GM CRUD), dual-currency |
| hub | js/hub/ | HubPage (mapa Godot + jogadores + ferramentas), CharacterModal, GroupRewards, LootDelivery, GiftTransfer, StatusEffectManager, EncounterPanel |
| monsters | js/monsters/ | MonstersPage (staged/biblioteca/cemitério), BattleLauncher trigger |
| gm-control | js/gm-control/ | GMControlPage dashboard + delegated modals (Shop/Inventory/Skills) |
| combat | js/combat/ | Dice (rollExpression + promptRoll manual/auto), EncounterPanel, BattleLauncher, statusTicks |
| import | js/import/ | GM JSON import (validators, ImportPage) |
| trade | js/trade/ | TradeManager (Avatar cross-browser via Realtime), TradeModal, notifications, TradeHistoryPanel |
| admin | js/admin/ | AdminPanel (delega role-change a `lib/users-registry`; tem 🗑 apagar conta), BackupRestore, LogService, LogViewer |
| storage | js/storage/ | AutoSave debounced + flush no logout + opções `omitStatusEffects/omitGmNotes/omitVitals` |
| api/* | js/api/ | auth, characters, skills, items, monsters, encounters, notifications, gm-characters (Supabase-first cross-user), shopProfiles, trades — todos com fallback localStorage |
| utils | js/utils/ | dom helpers, constants, statusEffects, toast (× por toast + "Limpar tudo") |

### Multi-game scaffolding
| File | Purpose |
|------|---------|
| js/router.js | Hash router, serializa unmount → mount |
| js/main.js | Registra jogos, single-game mode, botão "← Início" global |
| js/games/landing/ | Game selector (público) + Admin Global |
| js/games/landing/pages/AdminLandingPage.js | Painel admin global cross-app |
| js/games/back-widget.js | "← Início" flutuante (oculto em single-game) |
| js/games/lib/shared-auth.js | Factory de auth p/ D&D+MC (brand+title+session) |
| js/games/lib/users-registry.js | Registry partilhado, role rules, session sync, deleteUser cascata |
| js/games/lib/delete-user-ui.js | Helper UI: confirmação dupla + summary |
| js/games/lib/user-picker.js | Modal escolha de username |

### D&D 5e
| File | Purpose |
|------|---------|
| games/dnd/index.js | Entry: tabs, autosave, GM rewards, admin impersonate-mode |
| games/dnd/data/srd.js | Tabelas SRD (skills, classes, races, XP) |
| games/dnd/dnd-character.js | Modelo + cálculos (mods, prof, saves, multiclass) |
| games/dnd/dnd-trade.js | State machine + pre-validation + applyTransfer |
| games/dnd/dnd-import.js | Validators + storage de 4 domínios |
| games/dnd/pages/ | Character, Skills, Spells, Inventory, Trade, Hub, Import, Admin |
| api/dnd-characters.js | CRUD + `deleteCharacter(username)` |
| api/dnd-character-mapper.js | Flat row ↔ nested model |

### Minecraft Builds
| File | Purpose |
|------|---------|
| games/minecraft/index.js | Entry: tabs Galeria/Painel/Listas/Admin |
| games/minecraft/lib/drive.js | Helpers Drive URL + probe assíncrono |
| games/minecraft/lib/social.js | Detect YT/IG/generic link |
| games/minecraft/components/reactions-bookmarks.js | Like/dislike + popover playlist |
| games/minecraft/pages/ | Gallery, MyPanel, BuildForm, Lists, Admin |
| api/mc-builds.js | CRUD + `transferBuild` + `transferAllBuildsFromUser` |
| api/mc-reactions.js | Like/dislike (localStorage; schema Supabase pronto) |
| api/mc-lists.js | Playlists (localStorage; schema Supabase pronto) |

## Key Patterns

- Role check: `authManager.hasRole('gm')` (player < gm < admin).
- Per-app storage namespaces (sem fallback cross-app):
  - `avatar_rpg_*` (auth, character, trades, logs, registry)
  - `dnd_*` (auth, character, trades, characters_registry, imported_*)
  - `mc_*` (auth, builds, reactions, lists_{username})
- Registry partilhado: `avatar_rpg_users_registry` consumido pelos
  4 admin panels via `games/lib/users-registry.js`.
- Login overlay partilhado (`#login-overlay`) com brand contextual.
- Data priority Avatar: API > localStorage imported > mock.
- Tests: `npm test` (vitest).

## Fórmulas

**Avatar:**
- HP = 10 + (nivel × 8) + (FOR × 3)
- Chi = 6 + (nivel × 5) + (CHI × 4)
- Spirit = 8 + (nivel × 6) + (ESP × 3)
- Defense = (RES × 2) + nivel + armor_bonus
- Dodge = 10 + ((AGI × 2) + PER) × 0.2 - armor_penalty (cap 15)
- XP next = round(200 × (nivel-1)^1.55) — máx nível 40
- Mastery thresholds: M1=15, M2=50, M3=150 usos

**D&D 5e:**
- Modifier = floor((score − 10) / 2)
- Proficiency = ⌈level / 4⌉ + 1
- Save = ab_mod + (prof ? prof_bonus : 0)
- Skill = ab_mod + prof_bonus × (prof ? (expertise ? 2 : 1) : 0)
- Spell DC = 8 + prof + cast_ability_mod
- Multiclass level = soma dos níveis por classe
- XP table = standard 5e (0 → 355 000)

## Skill system v2 (Avatar)

Source of truth: `docs/skill-trees/*.html` →
`data/skills/*.json` via `scripts/extract-skill-trees.mjs`.

- **5 tiers** (1..5; 5 = Lendário). **5 branches** por elemento:
  - `sp` Espírito, `ag` Agilidade — sempre disponíveis
  - `cb` Combate (tiers 1-2 partilhados)
  - `pr` Preciso, `br` Bruto — **mutually exclusive at tier 3+**; locked via `character.combat_path`
- **`element='none'`** parte em duas paths via `character.non_bender_path`:
  `chiblocker` ou `weapons` (árvores separadas).
- **Mastery** progride automaticamente M0 → M3 conforme usos
  (`character.recordSkillUse(id)` + `character.getMasteryLevel(id)`).
- Persistido em `characters.combat_path` / `characters.non_bender_path` +
  `character_skills.uses` / `mastery_level`.
- Migration `20260629100000_skill_system_v2.sql`.
- JSON import schema: `skill-import-v2` (ver `docs/DIAGRAMAS-TECNICOS.md` §9.1).

## Roles

| Role | Avatar | D&D | Minecraft |
|------|--------|-----|-----------|
| player | Own char, shop, trade | Own sheet, trade | Bookmark, react |
| gm | + All chars, give gold/XP/loot, import, GM Control, monstros | + All sheets, give XP/gold, import packs | (sem privilégio extra) |
| admin | + Users, backup, logs, **apagar contas** | + Admin tab: gerir users, impersonate ou apagar ficha, **apagar contas** | + Admin tab: gerir users, editar/apagar/**transferir builds (single/bulk)**, **apagar contas** |

> **Admin Global** (landing): painel agregador visível se alguma
> sessão activa for admin. Stats por jogo + role mgmt cross-app +
> apagar conta.
>
> "Apagar conta" (cascata): registry + ficha Avatar + ficha D&D +
> builds Minecraft + reactions/listas + sessões activas + cleanup
> orphan trades (Avatar). Confirmação DUPLA. Regras: não apagar a
> si próprio, não apagar último admin.

## Dev scripts

```bash
npm run dev            # multi-game (landing + 3 apps)
npm run dev:avatar     # só Avatar
npm run dev:dnd        # só D&D
npm run dev:minecraft  # só Minecraft
npm test               # vitest
```

## File Structure

```text
public/
├── index.html
├── config.js                  (gitignored — Supabase config)
├── serve.json                 (COI headers — dev)
├── data/skills/*.json         (per-element trees, copied for serving)
├── css/                       main.css + components/ (incl. multi-game, dnd-sheet, mc-gallery)
└── js/
    ├── main.js                Entry: registra jogos + single-game mode
    ├── router.js              Hash router serializado
    ├── app.js                 Avatar App class
    ├── games/
    │   ├── landing/           LandingPage + AdminLandingPage
    │   ├── lib/               shared-auth, users-registry, delete-user-ui, user-picker
    │   ├── back-widget.js     "← Início" flutuante
    │   ├── avatar/            delega no App existente
    │   ├── dnd/               (index, data, pages — inclui AdminPage)
    │   └── minecraft/         (index, lib, pages, components — inclui AdminPage)
    ├── admin/                 AdminPanel (delega role-change), BackupRestore, LogService, LogViewer
    ├── auth, character, skills, items, shop, hub, import, trade,
    │   combat, storage, monsters, gm-control, utils  — Avatar
    └── api/                   auth, characters, skills, items, monsters, encounters,
                               notifications, gm-characters, shopProfiles, trades,
                               dnd-characters/+mapper, mc-builds/reactions/lists

data/skills/                   source-of-truth JSONs (espelhado em public/)
scripts/
  dev-game.js                  launcher single-game
  extract-skill-trees.mjs      gerador HTML → JSON
docs/                          documentação humana
  AVATAR-APP.md, DND-APP.md, MINECRAFT-APP.md, MULTI-GAME-DESIGN.md
  FEATURES.md, DECISIONS.md, DEV-LOCAL.md
  DIAGRAMAS-NAO-TECNICOS.md, DIAGRAMAS-TECNICOS.md
  skill-trees/*.html
supabase/migrations/           init + relax-RLS + multi-game + skills-v2 +
                               monsters + status-effects + encounters +
                               vitals-realtime + shop_profiles + trades + …
tests/                         vitest
```

## Docs

Documentação humana vive em `docs/`:

- `docs/AVATAR-APP.md`, `docs/DND-APP.md`, `docs/MINECRAFT-APP.md` — por app.
- `docs/MULTI-GAME-DESIGN.md` — landing, router, shared-auth, schema, admin panels.
- `docs/FEATURES.md` — features Avatar.
- `docs/DECISIONS.md` — decisões aplicadas/pendentes.
- `docs/DEV-LOCAL.md` — setup local + Supabase opt-in.
- `docs/DIAGRAMAS-NAO-TECNICOS.md`, `docs/DIAGRAMAS-TECNICOS.md` —
  arquitetura, schema, API, JSON imports.
- `docs/skill-trees/*.html` — source-of-truth das skills Avatar.

`README.md` (raiz) é o índice de alto nível. `CLAUDE.md` e
`.github/copilot-instructions.md` ficam na raiz para os agentes
encontrarem.

## Doc Update Rules

Depois de implementar nova fase/feature set:
- Atualizar `docs/FEATURES.md` (Avatar) ou doc da app correspondente.
- Atualizar `README.md` (status, scripts).
- Atualizar este ficheiro **e** `.github/copilot-instructions.md`
  (mesma estrutura).
- Adicionar/atualizar doc dedicado em `docs/` se for cross-app.

## Working Style

- Ask clarifying questions before implementing when design decisions have multiple valid options.
- Prefer multiple choice questions for faster decisions.
- Don't assume — confirm scope, behavior, and edge cases when ambiguous.

## Backlog

- Avatar: companions, Supabase Auth real, cooldown engine,
  automated combat resolution.
- D&D: motor de combate (rolls automáticos, status effects).
- Minecraft: wirar reactions/lists em Supabase (schema pronto em
  `20260629250000_…`; UI precisa de passar de sync para async).

---
> Source: [Rodrigo-Oliveira-Santos/avatar-rpg](https://github.com/Rodrigo-Oliveira-Santos/avatar-rpg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
