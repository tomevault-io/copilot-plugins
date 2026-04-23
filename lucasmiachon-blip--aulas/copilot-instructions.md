## aulas

> USE WHEN editing CSS. Flexbox, display, cores, semântica. Evitar space-between assimétrico, display inline em section.


# Erros CSS Comuns

## Flexbox & Layout
- `space-between` com N≠M items = PROIBIDO
- `flex:1` igualitário em containers desiguais = PROIBIDO
- ≤3 children para space-between
- Diagnosticar assimetria ANTES de escolher layout

## Display
- NUNCA `display` inline no `<section class="slide">` (quebra viewer)
- Cache-busting `?v=date` em todos CSS/JS linkados

## Cores
- Warning/gold em bg claro = usar `--warning-on-light`
- Cor = semântica clínica (ler texto antes de escolher)
- Em dark mode: verificar contraste sobre true black (#000) — ratio ≥ 4.5:1
- Semântica: teal=safe | copper=cautela | danger=risco | navy=estrutural

## Processo
- Gerar PDF/screenshot antes de declarar "pronto"
- NUNCA remover overflow:hidden do @media print

## E-codes (registro completo)

Para erros E01–E31 com prioridade MUST/SHOULD/MAY e reincidências: ver .claude/rules/css-errors.md (referência completa). Clusters: Flexbox (E06,E10,E18,E20,E22,E26,E27,E28), Display (E07,E12,E23,E24), Dados (E21,E25), Cores (E08,E13–E15,E17,E31), Processo (E01–E03,E05,E09,E11,E16,E19,E30).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lucasmiachon-blip) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
