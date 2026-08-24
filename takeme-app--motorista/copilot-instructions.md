## preparador-encomendas-nao-alterar

> Não alterar o ambiente preparador de encomendas sem pedido explícito


# Preparador de encomendas — não alterar por padrão

- **Não modificar** código, navegação, telas, libs nem migrations **do ambiente preparador de encomendas**, exceto quando o usuário pedir **explicitamente** algo nesse escopo.
- Trabalhos em motorista “clássico”, cliente, admin, excursões ou outras áreas **não** devem incluir mudanças colaterais nesse ambiente.

## Escopo (referência)

- App motorista: `**/encomendas/**`, `*Encomendas*`, `MainTabsEncomendas`, stacks/tipos de chat/pagamentos de encomendas, `preparerEncomendas*`, `preparer_shipment*`, `shipmentConversation` ligado ao preparador, etc.
- Supabase: migrations/policies/RPCs dedicados à **fila/base do preparador de envios** (`preparer_shipment_queue`, RLS `shipments` para preparador, seeds só desse fluxo), quando a tarefa não for sobre isso.

Se a correção exigir tocar nesse escopo, **pergunte** ou limite-se ao mínimo e avise o usuário.

---
> Source: [takeme-app/motorista](https://github.com/takeme-app/motorista) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
