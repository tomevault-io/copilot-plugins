## tweet-sentiment-analysis

> >


# 🧠 Ensemble de Modelos — Análise de Sentimento em Redes Sociais
### Trabalho Final — Python em Projetos de IA

---

## 📌 Visão Geral do Projeto

**Problema real:** Classificar automaticamente o sentimento de tweets em positivo, negativo ou neutro,
com aplicação prática em monitoramento de marca, análise de crises e pesquisa social.

**Abordagem escolhida:** Ensemble de 3 modelos pré-treinados via Hugging Face Transformers, com
estratégia de votação ponderada por confiança, publicado no HF Hub e versionado no GitHub.

**Por que isso é portfólio nível sênior?**
- Usa modelos state-of-the-art (não treina do zero — escolha inteligente)
- Ensemble é técnica usada em competições e produção real
- Publicação no HF Hub demonstra conhecimento de MLOps
- Código limpo, documentado e reproduzível

---

## 🗂️ Estrutura do Repositório

```
sentiment-ensemble/
├── README.md                    # Card do projeto com badges e resultados
├── requirements.txt
├── .github/
│   └── workflows/
│       └── lint.yml             # CI básico com flake8
├── notebooks/
│   ├── 01_EDA.ipynb             # Análise exploratória do dataset
│   ├── 02_baseline.ipynb        # Modelos individuais + métricas
│   └── 03_ensemble.ipynb        # Ensemble + análise final
├── src/
│   ├── __init__.py
│   ├── data_loader.py           # Carregamento e split do dataset
│   ├── preprocessor.py          # Limpeza específica para tweets
│   ├── models.py                # Wrapper para cada modelo HF
│   ├── ensemble.py              # Lógica de votação ponderada
│   └── evaluate.py              # Métricas + relatório
├── scripts/
│   ├── run_inference.py         # Inferência standalone (demo)
│   └── push_to_hub.py           # Publica modelo no HF Hub
└── tests/
    └── test_preprocessor.py     # Testes unitários básicos
```

---

## 1️⃣ DEFINIÇÃO DO PROBLEMA

### O que está sendo resolvido
Tweets têm linguagem extremamente ruidosa: gírias, emojis, abreviações, ironia, hashtags e menções.
Modelos genéricos de sentimento falham nesse contexto. O projeto resolve isso usando modelos
**pré-treinados especificamente em dados de redes sociais**, combinados em ensemble para
maximizar robustez.

### Impacto prático
- Empresas monitoram percepção de marca em tempo real
- Campanhas políticas analisam reação do público
- Pesquisadores estudam saúde mental coletiva
- Jornalismo de dados usa para cobrir eventos ao vivo

### Solução atual e suas limitações
VADER e TextBlob são usados amplamente mas:
- VADER não lida bem com português ou contextos culturais específicos
- Modelos bag-of-words perdem contexto sequencial (ironia, negação)
- Fine-tuning individual de um único modelo é frágil a distribuições out-of-distribution

### Tipo de problema
Classificação multiclasse (3 classes: positivo, negativo, neutro) com textos curtos e ruidosos.

---

## 2️⃣ DATASET

### Dataset escolhido: `tweet_eval` (Hugging Face Datasets)

**Justificativa de escolha:**
- Benchmark padrão da comunidade NLP para tweets — usado em papers acadêmicos
- Subset `sentiment`: ~45k tweets em inglês já anotados (positivo/negativo/neutro)
- Split pré-definido (train/validation/test) — evita data leakage
- Carregamento direto via `datasets` lib sem necessidade de scraping ou API

**Alternativa PT-BR (opcional para diferencial):**
- `HeiReS/twitter_sentiment_pt` ou `ruanchaves/hatebr` (disponíveis no HF Hub)
- Se optar por PT-BR, os modelos mudam (ver seção 4)

**Carregamento:**
```python
from datasets import load_dataset

ds = load_dataset("tweet_eval", "sentiment")
# train: 45.615 | validation: 2.000 | test: 12.284
```

**Análise de desbalanceamento (EDA obrigatória):**
```python
import pandas as pd
import matplotlib.pyplot as plt

df_train = pd.DataFrame(ds['train'])
print(df_train['label'].value_counts())
# Espera-se: neutro > positivo > negativo (desequilíbrio moderado)

df_train['label'].value_counts().plot(kind='bar', title='Distribuição de Classes')
plt.savefig('assets/class_distribution.png')
```

**Por que a distribuição importa:**
- Se neutro dominar, Accuracy sozinha vai enganar
- Precisamos de F1-Macro como métrica principal (ver seção 6)

---

## 3️⃣ PRÉ-PROCESSAMENTO

### Limpeza específica para tweets (src/preprocessor.py)

```python
import re

def clean_tweet(text: str) -> str:
    """
    Limpeza focada em preservar semântica emocional.
    NÃO remove emojis — eles carregam sentimento.
    """
    # Remove URLs
    text = re.sub(r'http\S+|www\S+', '', text)
    # Remove menções (@usuario) mas mantém o @ para contexto
    text = re.sub(r'@\w+', '@user', text)
    # Normaliza hashtags (remove # mas mantém a palavra)
    text = re.sub(r'#(\w+)', r'\1', text)
    # Remove caracteres repetidos excessivos (looooove -> loove)
    text = re.sub(r'(.)\1{3,}', r'\1\1', text)
    # Remove espaços extras
    text = re.sub(r'\s+', ' ', text).strip()
    return text
```

**Justificativa de cada escolha:**
| Técnica | Por quê |
|---|---|
| Substituir @user | Preserva estrutura sem vazar dados de usuários |
| Manter emojis | 😂 e 😢 são features riquíssimas de sentimento |
| Normalizar repetições | "looool" e "lol" são o mesmo token semanticamente |
| NÃO fazer stemming | Transformers tokenizam no nível de subpalavras — stemming atrapalha |
| NÃO remover stopwords | Negações como "não" e "never" são críticas para sentimento |

**Tokenização:** Cada modelo tem seu próprio tokenizer — usar sempre o tokenizer nativo do modelo.
Nunca tokenizar manualmente antes de passar para o pipeline do HF.

---

## 4️⃣ ESCOLHA DOS MODELOS

### Estratégia: diversidade + especialização

O ensemble funciona melhor quando os modelos erram em pontos diferentes.
Escolhemos 3 modelos com arquiteturas e treinamentos complementares:

| # | Modelo | Parâmetros | Treinamento | Ponto Forte |
|---|---|---|---|---|
| M1 | `cardiffnlp/twitter-roberta-base-sentiment-latest` | 125M | Tweets em inglês | Alta precisão em linguagem informal |
| M2 | `finiteautomata/bertweet-base-sentiment-analysis` | 110M | BERTweet (tweets) | Representa bem hashtags e emojis |
| M3 | `distilbert-base-uncased-finetuned-sst-2-english` | 66M | Resenhas SST-2 | Rapido + bom baseline em inglês formal |

**Justificativa técnica de cada modelo:**

**M1 — twitter-roberta-base-sentiment-latest**
- RoBERTa treinada especificamente em 58M tweets
- Versão "latest" foi atualizada com dados mais recentes
- Benchmark state-of-the-art em tweet_eval sentiment
- Referência: Barbieri et al., 2020 (TweetEval paper)

**M2 — bertweet-base-sentiment-analysis**
- BERTweet é um BERT pré-treinado do zero em 850M tweets
- Fine-tuned em análise de sentimento de tweets
- Diferente arquitetura de tokenização (otimizada para tweets)
- Complementa M1 por ter sido treinado de forma diferente

**M3 — distilbert-finetuned-sst-2**
- 40% menor e 60% mais rápido que BERT base
- Treinado em texto mais formal (resenhas de filmes)
- Age como "regularizador" do ensemble — ancora em linguagem padrão
- Binário (pos/neg), precisará de mapeamento para 3 classes

**Alternativas consideradas e descartadas:**
- `VADER`: Baseado em regras, sem aprendizado — teto de performance baixo
- GPT via API: Custo proibitivo para 45k exemplos + não reproduzível offline
- Treinar BERT do zero: Inviável computacionalmente sem GPU dedicada

---

## 5️⃣ TREINAMENTO / INFERÊNCIA — CÓDIGO

### Abordagem: zero-shot inference (sem fine-tuning adicional)

**Por que não fine-tunar?**
Os modelos já foram fine-tunados em dados de tweet sentiment. Re-treinar:
1. Exige GPU (T4 no Colab ou superior)
2. Risco alto de overfitting em dataset pequeno
3. Não agrega valor ao portfólio — o diferencial está no **ensemble**, não no fine-tuning

Se quiser ir além (diferencial extra): fazer fine-tuning de M1 no subset de validação com apenas 3 epochs.

### Pipeline de inferência (src/models.py)

```python
from transformers import pipeline
import torch

MODELS = {
    "roberta_twitter": "cardiffnlp/twitter-roberta-base-sentiment-latest",
    "bertweet": "finiteautomata/bertweet-base-sentiment-analysis",
    "distilbert": "distilbert-base-uncased-finetuned-sst-2-english",
}

LABEL_MAP = {
    # Normaliza os labels diferentes de cada modelo para {0: neg, 1: neu, 2: pos}
    "roberta_twitter": {"negative": 0, "neutral": 1, "positive": 2},
    "bertweet":        {"NEG": 0, "NEU": 1, "POS": 2},
    "distilbert":      {"NEGATIVE": 0, "POSITIVE": 2},  # sem neutro
}

def load_pipelines(device: int = -1) -> dict:
    """
    device=-1: CPU | device=0: primeira GPU
    """
    pipes = {}
    for name, model_id in MODELS.items():
        pipes[name] = pipeline(
            "text-classification",
            model=model_id,
            top_k=None,          # retorna scores de todas as classes
            device=device,
            truncation=True,
            max_length=128,      # tweets raramente passam de 280 chars = ~60 tokens
        )
    return pipes
```

### Divisão treino/teste
O dataset `tweet_eval` já vem dividido. Usar os splits oficiais.
**Nunca misturar validation e test** — o test set é usado UMA única vez, na avaliação final.

---

## 6️⃣ ENSEMBLE — VOTAÇÃO PONDERADA POR CONFIANÇA

### Estratégia: Weighted Soft Voting

Em vez de votação majoritária simples (cada modelo vota 1 classe), usamos a **probabilidade de cada
classe** de cada modelo, multiplicada por um peso aprendido.

**Por que soft voting > hard voting:**
- Aproveita a incerteza do modelo ("75% positivo" é mais informativo que apenas "positivo")
- Modelos mais confiantes têm mais influência na decisão final

### Cálculo dos pesos (src/ensemble.py)

```python
import numpy as np
from sklearn.metrics import f1_score

def calibrate_weights(pipes: dict, val_dataset, label_map: dict) -> dict:
    """
    Calcula peso de cada modelo como seu F1-Macro no validation set.
    Modelos mais precisos recebem mais peso no ensemble.
    """
    weights = {}
    for name, pipe in pipes.items():
        preds, labels = [], []
        for item in val_dataset:
            result = pipe(clean_tweet(item['text']))[0]
            mapped = {label_map[name].get(r['label'], 1): r['score'] for r in result}
            pred = max(mapped, key=mapped.get)
            preds.append(pred)
            labels.append(item['label'])
        
        f1 = f1_score(labels, preds, average='macro')
        weights[name] = f1
        print(f"  {name}: F1-Macro = {f1:.4f} → peso = {f1:.4f}")
    
    # Normaliza para somar 1
    total = sum(weights.values())
    return {k: v / total for k, v in weights.items()}


def ensemble_predict(text: str, pipes: dict, weights: dict, label_map: dict) -> int:
    """
    Retorna classe predita pelo ensemble (0=neg, 1=neu, 2=pos).
    """
    combined = np.zeros(3)
    
    for name, pipe in pipes.items():
        result = pipe(clean_tweet(text))[0]
        scores = np.zeros(3)
        for r in result:
            idx = label_map[name].get(r['label'])
            if idx is not None:
                scores[idx] = r['score']
        
        # distilbert não tem neutro: distribui probabilidade restante
        if name == "distilbert":
            scores[1] = 1.0 - scores[0] - scores[2]
        
        combined += weights[name] * scores
    
    return int(np.argmax(combined))
```

---

## 7️⃣ AVALIAÇÃO

### Métricas e justificativas

| Métrica | Justificativa |
|---|---|
| **F1-Macro** | Métrica principal — penaliza igualmente erros em classes minoritárias |
| **F1 por classe** | Diagnóstico — mostra onde o ensemble falha (geralmente em neutro) |
| Accuracy | Reportar mas NÃO usar como critério principal (enganosa com desequilíbrio) |
| Confusion Matrix | Visualização obrigatória — mostra padrões de confusão |

**Por que F1-Macro e não F1-Weighted?**
F1-Weighted favorece classes majoritárias. Em sentimento, errar "negativo" (classe menor)
pode ter impacto real (ex: crise de imagem não detectada).

### Código de avaliação (src/evaluate.py)

```python
from sklearn.metrics import classification_report, confusion_matrix, f1_score
import matplotlib.pyplot as plt
import seaborn as sns

def full_report(y_true, y_pred, model_name: str):
    labels = ['Negativo', 'Neutro', 'Positivo']
    
    print(f"\n{'='*50}")
    print(f"  {model_name}")
    print('='*50)
    print(classification_report(y_true, y_pred, target_names=labels))
    
    # Confusion matrix
    cm = confusion_matrix(y_true, y_pred)
    fig, ax = plt.subplots(figsize=(6, 5))
    sns.heatmap(cm, annot=True, fmt='d', cmap='Blues',
                xticklabels=labels, yticklabels=labels, ax=ax)
    ax.set_title(f'Confusion Matrix — {model_name}')
    ax.set_ylabel('Real')
    ax.set_xlabel('Predito')
    plt.tight_layout()
    plt.savefig(f'assets/cm_{model_name.lower().replace(" ", "_")}.png', dpi=150)
    plt.close()
    
    return f1_score(y_true, y_pred, average='macro')
```

### Comparação esperada de resultados

| Modelo | F1-Macro esperado |
|---|---|
| DistilBERT (baseline) | ~0.65–0.70 |
| BERTweet | ~0.72–0.76 |
| RoBERTa Twitter | ~0.74–0.78 |
| **Ensemble (weighted)** | **~0.76–0.80** |

O ensemble deve superar todos os modelos individuais — se não superar, reavalie os pesos.

---

## 8️⃣ PUBLICAÇÃO NO HUGGING FACE HUB

### Estrutura do model card (README.md do modelo)

```markdown
---
language: en
tags:
  - sentiment-analysis
  - twitter
  - ensemble
  - text-classification
datasets:
  - tweet_eval
metrics:
  - f1
license: mit
---

# Sentiment Ensemble — Twitter

Ensemble de 3 modelos transformer para análise de sentimento em tweets.
Combina RoBERTa-Twitter, BERTweet e DistilBERT via soft voting ponderado por F1-Macro.

## Resultados

| Modelo | F1-Macro |
|--------|----------|
| RoBERTa Twitter | 0.XX |
| BERTweet | 0.XX |
| DistilBERT | 0.XX |
| **Ensemble** | **0.XX** |

## Uso rápido

\`\`\`python
from sentiment_ensemble import predict_sentiment
result = predict_sentiment("I love this product! 🎉")
# {'label': 'positive', 'confidence': 0.94}
\`\`\`
```

### Script de publicação (scripts/push_to_hub.py)

```python
from huggingface_hub import HfApi, login
import json, os

def push_ensemble_to_hub(weights: dict, repo_name: str = "seu-usuario/sentiment-ensemble"):
    login(token=os.environ["HF_TOKEN"])  # nunca coloque token no código
    api = HfApi()
    
    # Cria o repositório se não existir
    api.create_repo(repo_id=repo_name, repo_type="model", exist_ok=True)
    
    # Salva os pesos do ensemble
    with open("ensemble_weights.json", "w") as f:
        json.dump(weights, f, indent=2)
    
    api.upload_file(
        path_or_fileobj="ensemble_weights.json",
        path_in_repo="ensemble_weights.json",
        repo_id=repo_name,
    )
    
    api.upload_file(
        path_or_fileobj="src/ensemble.py",
        path_in_repo="ensemble.py",
        repo_id=repo_name,
    )
    
    print(f"✅ Publicado em: https://huggingface.co/{repo_name}")
```

---

## 9️⃣ README.md DO GITHUB (template)

```markdown
# 🧠 Sentiment Ensemble — Twitter NLP

![Python](https://img.shields.io/badge/Python-3.10-blue)
![HuggingFace](https://img.shields.io/badge/🤗%20HuggingFace-Models-yellow)
![License](https://img.shields.io/badge/License-MIT-green)
![F1-Macro](https://img.shields.io/badge/F1--Macro-0.XX-brightgreen)

> Ensemble de modelos transformer para análise de sentimento em tweets,
> combinando RoBERTa-Twitter, BERTweet e DistilBERT via soft voting ponderado.

## 🎯 Resultado Principal
O ensemble alcança **F1-Macro de 0.XX** no benchmark tweet_eval,
superando o melhor modelo individual em +X.X pontos percentuais.

## 🏗️ Arquitetura

\`\`\`
Tweet → Limpeza → [M1: RoBERTa] ─┐
                  [M2: BERTweet] ─┼─ Weighted Soft Voting → Sentimento
                  [M3: DistilBERT]─┘
\`\`\`

## 🚀 Instalação

\`\`\`bash
git clone https://github.com/seu-usuario/sentiment-ensemble
cd sentiment-ensemble
pip install -r requirements.txt
\`\`\`

## 📊 Notebooks
| Notebook | Descrição |
|---|---|
| [01_EDA](notebooks/01_EDA.ipynb) | Análise exploratória do tweet_eval |
| [02_baseline](notebooks/02_baseline.ipynb) | Avaliação individual dos modelos |
| [03_ensemble](notebooks/03_ensemble.ipynb) | Ensemble + análise de erros |

## 🤗 Modelo no HuggingFace Hub
[seu-usuario/sentiment-ensemble](https://huggingface.co/seu-usuario/sentiment-ensemble)
```

---

## 🔟 LIMITAÇÕES E MELHORIAS FUTURAS

### Limitações honestas (obrigatório no portfólio)
- Dataset em inglês: modelos perdem performance em tweets PT-BR sem adaptação
- Ironia e sarcasmo continuam sendo o maior desafio para qualquer modelo
- Inferência lenta: 3 modelos em sequência (~2s/tweet em CPU)
- Os 3 modelos base foram treinados em dados que podem ter vieses históricos

### Melhorias futuras (mostra maturidade técnica)
- Adicionar modelo PT-BR (`neuralmind/bert-base-portuguese-cased`) para cobertura multilíngue
- Stacking com meta-learner (Logistic Regression sobre as probabilidades dos 3 modelos)
- Quantização dos modelos com `optimum` para 4x mais velocidade em CPU
- Monitoramento de drift via Evidently AI em produção
- Dataset aumentado com Backtranslation para a classe minoritária

---

## 1️⃣1️⃣ CONCLUSÃO TÉCNICA

| Critério | Avaliação |
|---|---|
| O modelo resolve o problema? | ✅ Sim — F1-Macro > 0.75 é referência de produção |
| É viável em produção? | ⚠️ Com restrições — precisa de caching e batching |
| Próximo passo real | Deploy em FastAPI + Redis cache para latência <200ms |

**Argumento central do projeto:**
O ensemble não é apenas uma técnica — é uma decisão de engenharia. Modelos individuais
têm pontos cegos. Ao combinar perspectivas diversas (tweets vs. resenhas, RoBERTa vs. BERT),
o sistema se torna mais robusto a variações linguísticas do mundo real.

---

## ⚙️ requirements.txt

```
transformers==4.40.0
datasets==2.19.0
torch==2.3.0
scikit-learn==1.4.2
pandas==2.2.2
matplotlib==3.8.4
seaborn==0.13.2
huggingface_hub==0.23.0
tqdm==4.66.4
```

---

## 📋 CHECKLIST DO TRABALHO FINAL

### Estrutura obrigatória da atividade

- [ ] **Definição do Problema** — coberto na seção 1
- [ ] **Dataset** — `tweet_eval` via HF Datasets, seção 2
- [ ] **Pré-processamento** — limpeza de tweets justificada, seção 3
- [ ] **Escolha do Modelo** — 3 modelos + justificativa técnica, seção 4
- [ ] **Treinamento / Código** — pipelines HF + ensemble, seções 5–6
- [ ] **Avaliação** — F1-Macro, confusion matrix, comparativo, seção 7
- [ ] **Resultados** — tabela comparativa + análise de erros
- [ ] **Limitações** — listadas honestamente, seção 10
- [ ] **Melhorias Futuras** — seção 10
- [ ] **Conclusão Técnica** — seção 11

### Diferenciais de portfólio (além da atividade)

- [ ] Modelo publicado no HuggingFace Hub com model card completo
- [ ] Repositório GitHub com README profissional e badges
- [ ] GitHub Actions com lint básico (`.github/workflows/lint.yml`)
- [ ] Análise de erros qualitativa (exemplos que o ensemble ainda erra)
- [ ] Notebook EDA com visualizações exportadas como imagens

---
> Source: [cavalcanteprofissional/tweet-sentiment-analysis](https://github.com/cavalcanteprofissional/tweet-sentiment-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
