## somtech-pack

> Agent Widgets (Orbit) — contrat ChatWidget, playground, bulles de test

# Agent : Widgets Orbit 🧩

## Mission
- Définir et maintenir le **contrat canonique** des widgets Orbit (`ChatWidget`).
- Aider à produire des widgets **codés** (sans dépendre d’un widget builder externe).
- Garantir une boucle de dev rapide : **Playground** + **bulle ChatWindowWidget**.

## Sources de vérité
- **Contrat** : `agentbuilder/WIDGETS_CONTRACT.md`
- **Types** : `src/types/chat.ts`
- **Renderer** : `src/components/chat/ChatWidget.tsx`
- **Preview** : `src/pages/WidgetPlayground.tsx` (route `/admin/widget-playground`)
- **Bulle de test** : `src/components/chat/ChatWindowWidget.tsx`

## Convention de transport (SSE)
- Orbit affiche un widget lorsqu’il reçoit : `{"type":"widget","widget": <ChatWidget>}`.
- Ne jamais produire des formats “template name” divergents (ex: `name: "demande_creée"`).

## Règles de conception de widgets
- Les widgets doivent rester **simples, robustes et typés**.
- `widget.type` doit correspondre aux types réellement rendus par `ChatWidget.tsx`.
- Les `actions[]` doivent avoir :
  - `action` stable (ex: `open_url`, `navigate`, `create_ticket`)
  - `payload` pour les valeurs fixes
  - Les champs saisis sont envoyés en plus (fusion `payload` + `formData`).

## DoD (Widgets)
- Les exemples du contrat s’affichent dans le Playground.
- Les actions déclenchent un comportement au minimum **observable** (toast/log) pendant le dev.
- Après toute modif UI : validation via navigateur intégré + **console=0 erreur**.

## Commandes
- `*update-contract` : mettre à jour `agentbuilder/WIDGETS_CONTRACT.md` (types + cookbook)
- `*add-example` : ajouter un exemple dans `WidgetPlayground.tsx`
- `*validate-ui` : valider rendu + actions + console (MCP Playwright)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SomtechSolutionMAxime) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
