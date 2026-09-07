## farmacore

> **Não criar complexidade desnecessária. Ser o mais enxuto possível para que

# Farmacore — princípios de código

## Princípio fundamental: simplicidade

**Não criar complexidade desnecessária. Ser o mais enxuto possível para que
outros humanos ou agentes consigam entender o código de maneira simples,
fácil e rápida.**

Aplicação prática:

- **Antes de adicionar uma abstração**, pergunte: ela carrega o seu peso? Três
  linhas similares quase sempre superam uma abstração prematura.
- **Antes de adicionar uma camada de defesa** (try/catch, validação, fallback),
  pergunte: esse caso pode mesmo acontecer? Confiar em código interno e
  contratos de framework; validar apenas em fronteiras (entrada de usuário,
  APIs externas).
- **Zero dead code, zero lógica desnecessária**: imports não usados, helpers
  chamados uma vez (inline), guards pra casos impossíveis, exports não
  consumidos, constantes "para o futuro" — tudo isso é peso. Remova ou não
  adicione.
- **Antes de adicionar um comentário**, pergunte: o nome do identificador já
  diz isso? Comentar só o "porquê" não-óbvio (constraint escondida, bug
  conhecido, decisão de design surpreendente).
- **Antes de adicionar um arquivo**, pergunte: faz sentido em um arquivo
  existente? Menos arquivos é melhor quando o escopo é o mesmo.
- **Antes de criar uma feature flag ou um shim de compatibilidade**, pergunte:
  posso só mudar o código?
- **JSDoc longo é sinal**: se precisei explicar muita coisa, talvez o código
  esteja fazendo demais. Reescrever costuma ser melhor que documentar.

O leitor é um humano cansado às 11 da noite, ou um agente IA com 5 minutos
de contexto. O código deve ser legível para eles, não para o autor original.

---
> Source: [cinegagliamarco/farmacore](https://github.com/cinegagliamarco/farmacore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
