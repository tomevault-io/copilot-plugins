## st-gcn-jules

> O sistema é uma plataforma de predição de crimes (CVLI) no estado do Ceará, utilizando modelos de Grafos Espaço-Temporais (ST-GAT). Atualmente, o foco é superar o patamar de **40% de P@10** (Precisão no Top 10) para a capital Fortaleza, com objetivo final de **60-70% de P@10** em Fortaleza.

# 🧠 Estado Atual do Projeto - Report Preview

## 🌍 Contexto Geral
O sistema é uma plataforma de predição de crimes (CVLI) no estado do Ceará, utilizando modelos de Grafos Espaço-Temporais (ST-GAT). Atualmente, o foco é superar o patamar de **40% de P@10** (Precisão no Top 10) para a capital Fortaleza, com objetivo final de **60-70% de P@10** em Fortaleza.

## 🚀 Status do Modelo (T33 - Elite P10)
- **Arquitetura:** `DeepSTGAT_64` (3 camadas ST-GAT + Heads de Atenção Temporal).
- **Dados:** 29 canais de entrada (Sazonalidade, CVLI, Veículos, Tensão de Facções, Intel Triggers).
- **Últimas Otimizações:**
    - Aumento da janela temporal (`WINDOW`) para **120 dias**.
    - Implementação de `OneCycleLR` com `max_lr=0.02` para evitar platôs precoces.
    - Refinamento da `RankLoss` com **Hard Negative Mining** (comparação entre Top-10 e Mid-Tier).
    - Aumento do peso de ranking (`ranking_weight`) para **40.0**.
- **Treinamento Atual:** Iniciado em 10/03/2026. Primeiro epoch atingiu **29% P@10 (Val)**.

## Logs de Tentativas e Métricas 
- As métricas e tentativas de otimização estão contidas em TRAINING_LOGS. 

## 🛠️ Diretrizes de Desenvolvimento
- **Venv:** Sempre utilizar o Python do ambiente virtual: `.\.venv\Scripts\python.exe`.
- **Logs:** Monitorar progresso em `logs/training_ELITE_P10.log`.
- **Checkpoint:** Modelos salvos em `models/active/` baseados no melhor P@10.

## 🇧🇷 Idioma de Interação
- **Mandato:** Todas as interações, explicações e respostas devem ser realizadas **obrigatoriamente em Português**.

---
*Última atualização: 10 de Março de 2026*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Boanerges20297)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Boanerges20297)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
