## pngcutter

> **NUNCA usar barras laterais verticais (vertical side bars) no design.**

# PNG Cutter - Regras de Design e Desenvolvimento

## 🎨 Regras de Design

### ❌ PROIBIDO: Barras Laterais Verticais
**NUNCA usar barras laterais verticais (vertical side bars) no design.**

Esta é uma regra fundamental do projeto. Barras laterais verticais são consideradas design "vulgar" na era moderna de "vibe coding" e devem ser completamente evitadas.

**O que NÃO fazer:**
- ❌ `border-left` colorido em elementos de navegação
- ❌ `::before` ou `::after` pseudo-elementos criando barras verticais
- ❌ Indicadores visuais laterais (left-side indicators)
- ❌ Qualquer elemento visual vertical na lateral esquerda de itens de menu, modais, ou componentes

**O que fazer em vez disso:**
- ✅ Usar apenas background colorido para estados ativos/hover
- ✅ Usar bordas arredondadas completas
- ✅ Usar sombras suaves para profundidade
- ✅ Usar cores de texto para indicar estado ativo
- ✅ Usar ícones ou outros elementos internos para feedback visual

**Exemplos de implementação correta:**
```css
/* ✅ CORRETO - Apenas background */
.sidebar-item.active {
  background: var(--color-primary-light);
  color: var(--color-primary);
  font-weight: 600;
}

/* ❌ ERRADO - Barra lateral */
.sidebar-item.active::before {
  width: 4px;
  background: var(--color-primary);
}
```

## 🛠️ Regras de Desenvolvimento

### Framework MOZ-CHOP
- Todos os componentes devem seguir a estrutura do framework MOZ-CHOP
- Usar translation keys para todo o texto visível
- Componentes devem ser reutilizáveis e modulares
- Sistema de temas (dark/light) deve ser respeitado

### Ícones
- **OBRIGATÓRIO:** Usar apenas ícones SVG minimalistas e "roundish"
- **PROIBIDO:** NUNCA usar emojis (ℹ️, ⚠️, 🔴, ✅, 📷, etc.) como ícones
- Ícones devem estar em `src/utils/icons.js`
- Todos os ícones devem seguir o padrão visual estabelecido (traços suaves, cantos arredondados)
- Ícones devem usar `stroke="currentColor"` para herdar a cor do texto
- Tamanho padrão: 20x20px (24x24px para modais)

**Exemplos:**
```javascript
// ✅ CORRETO - SVG minimalista
info: `<svg width="24" height="24" viewBox="0 0 20 20" fill="none">
  <circle cx="10" cy="10" r="7" stroke="currentColor" stroke-width="1.5" fill="none"/>
</svg>`

// ❌ ERRADO - Emoji
info: 'ℹ️'
```

### Cores e Temas
- Usar variáveis CSS definidas em `src/themes/base.css`
- Suportar dark mode e light mode
- Cores primárias: gradientes suaves (indigo/purple)
- Evitar cores muito saturadas

### Animações
- Animações devem ser suaves e naturais
- Usar `cubic-bezier` para transições
- Evitar animações excessivas ou distraentes
- Transições padrão: 200ms-500ms

### Responsividade
- Design deve ser responsivo
- Mobile-first approach quando possível
- Testar em diferentes tamanhos de tela
- Sidebar deve colapsar em mobile

### Acessibilidade
- Usar `aria-label` em botões sem texto
- Manter contraste adequado
- Suportar navegação por teclado
- Sempre fornecer feedback visual para ações

## 📝 Convenções de Código

### JavaScript
- Usar ES6+ modules
- Classes para componentes
- Async/await para operações assíncronas
- Destructuring quando apropriado

### CSS
- Usar variáveis CSS para valores reutilizáveis
- BEM ou similar para organização
- Mobile-first media queries
- Comentários para seções complexas

### Estrutura de Arquivos
- Componentes em `src/components/`
- Utilitários em `src/utils/`
- Páginas em `src/pages/`
- Temas em `src/themes/`
- Traduções em `src/i18n/`

## 🚫 Anti-padrões

1. **NUNCA** criar barras laterais verticais
2. **NUNCA** usar emojis como ícones principais
3. **NUNCA** hardcode strings de texto (usar i18n)
4. **NUNCA** ignorar o sistema de temas
5. **NUNCA** criar componentes não reutilizáveis sem justificativa

## ✅ Checklist Antes de Commitar

- [ ] Nenhuma barra lateral vertical foi adicionada
- [ ] Nenhum emoji foi usado como ícone (apenas SVG)
- [ ] Todos os textos usam translation keys
- [ ] Dark mode e light mode funcionam corretamente
- [ ] Componentes são responsivos
- [ ] Ícones seguem o padrão minimalista (SVG roundish)
- [ ] Animações são suaves e não excessivas
- [ ] Código segue as convenções estabelecidas

---

**Lembre-se:** Design minimalista e limpo é a prioridade. Menos é mais.

---
> Source: [kceryeus/pngcutter](https://github.com/kceryeus/pngcutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
