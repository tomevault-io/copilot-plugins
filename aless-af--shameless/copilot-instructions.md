## shameless

> When model needs to understand the project context


# Shameless - Project Context

## Project Mission
Crear una aplicación de análisis de sentimientos que permite analizar el perfil completo de un usuario en redes sociales. El usuario proporciona una URL o nombre de usuario, la aplicación recopila sus publicaciones y genera un perfil completo de sentimiento usando modelos ML entrenados en Kaggle.

## Core Concept

### 🎯 Problema que Resolvemos
**¿Qué tipo de persona es este usuario en redes sociales?**

- ¿Es mayormente positivo o negativo?
- ¿Sobre qué temas es más positivo/negativo?
- ¿Cómo ha evolucionado su sentimiento?
- ¿Su sentimiento afecta su engagement?

### 💡 Solución
Una aplicación local que:
1. Acepta URL o username
2. Scrape el perfil completo del usuario
3. Analiza cada tweet con un modelo entrenado en Kaggle
4. Genera un perfil de sentimiento completo
5. Produce un reporte visual y estadístico

## Workflow Completo

### 📊 En Kaggle (Entrenamiento)

```python
# 1. Preparación de datos
dataset = load_sentiment_dataset()  # Sentiment140, etc.
train, test = split_data(dataset)

# 2. Entrenamiento
model = train_sentiment_model(
    data=train,
    architecture="bert-base-uncased",
    epochs=3,
    batch_size=16
)

# 3. Evaluación
metrics = evaluate(model, test)
print(f"Accuracy: {metrics['accuracy']}")
print(f"F1 Score: {metrics['f1']}")

# 4. Guardar modelo
save_model(model, "sentiment_model_v1")
save_to_kaggle_dataset("shameless-sentiment-models")
```

### 💻 En Local (Aplicación)

```python
# 1. Usuario inicia análisis
from sentiment_analyser import UserAnalyzer

analyzer = UserAnalyzer()
result = analyzer.analyze("@username", tweets_limit=500)

# 2. Scraping automático
tweets = scraper.get_user_tweets("username", limit=500)

# 3. Descarga modelo de Kaggle (si no existe)
model = load_model_from_kaggle("sentiment_model_v1")

# 4. Inferencia
sentiments = model.predict_batch(tweets)

# 5. Agregación y análisis
profile = aggregate_sentiments(sentiments, tweets)

# 6. Reporte
profile.generate_report("username_report.pdf")
profile.show_interactive_dashboard()
```

## Use Cases Detallados

### 1. Análisis de Influencer
**Caso:** Una marca quiere contratar un influencer y necesita saber si su contenido es positivo.

```python
analyzer = UserAnalyzer()
result = analyzer.analyze("@influencer", tweets_limit=1000)

print(result.summary)
# Output:
# Overall Sentiment: Positive (73%)
# Negative tweets: 12%
# Neutral tweets: 15%
# Most positive topics: Technology, Gaming
# Most negative topics: Politics
```

### 2. Monitoreo de Marca Personal
**Caso:** Un profesional quiere analizar cómo se comunica en redes.

```python
result = analyzer.analyze("@professional")
result.show_timeline()  # Muestra evolución temporal
result.identify_patterns()  # Identifica patrones

# Sugerencias:
# - Tus tweets sobre "work" son 60% negativos
# - Considera cambiar el tono en temas profesionales
# - Tus tweets más populares son positivos
```

### 3. Análisis Comparativo
**Caso:** Comparar el sentimiento de varios usuarios.

```python
users = ["@user1", "@user2", "@user3"]
comparison = analyzer.compare_users(users)

comparison.show_comparison_chart()
comparison.export_excel("comparison.xlsx")
```

### 4. Análisis Temporal
**Caso:** Ver cómo ha cambiado el sentimiento de un usuario.

```python
result = analyzer.analyze_timeline(
    "@username",
    start_date="2023-01-01",
    end_date="2024-01-01"
)

result.plot_sentiment_evolution()
# Muestra gráfico de línea con sentimiento a lo largo del tiempo
```

## Technical Context

### Por qué Kaggle para Training

**Ventajas:**
- ✅ **GPU gratuita**: P100, T4 (30h/semana)
- ✅ **Datasets públicos**: Sentiment140, Twitter datasets
- ✅ **Colaboración**: Notebooks públicos
- ✅ **Versionado**: Kaggle Datasets para modelos
- ✅ **Reproducibilidad**: Ambiente consistente

**Flujo:**
```
Kaggle Notebook → Train Model → Save to Dataset → Download Locally
```

### Por qué Local Application

**Ventajas:**
- ✅ **Sin límites de tiempo**: No hay timeout de notebook
- ✅ **Mejor UX**: Streamlit/CLI personalizado
- ✅ **Integración**: Fácil de distribuir
- ✅ **Control**: Código privado si es necesario

### Model Management

#### Versionado de Modelos
```
Kaggle Dataset: shameless-sentiment-models
├── v1.0/
│   ├── model.pt                  # 250 MB
│   ├── tokenizer/
│   ├── config.json
│   └── metrics.json             # accuracy: 0.85
│
├── v1.1/                        # Improved
│   ├── model.pt                 # 250 MB
│   ├── metrics.json             # accuracy: 0.87
│
└── v2.0/                        # New architecture
    ├── model.pt                 # 180 MB (distilled)
    └── metrics.json             # accuracy: 0.89
```

#### Local Cache
```
data/models/
├── current -> v2.0/            # Symlink al modelo actual
├── v1.0/
├── v1.1/
└── v2.0/
```

## Data Schema

### User Profile Analysis Result

```json
{
  "username": "@elonmusk",
  "analysis_date": "2024-10-18T12:00:00Z",
  "tweets_analyzed": 500,
  "date_range": {
    "start": "2024-09-01",
    "end": "2024-10-18"
  },
  "sentiment_summary": {
    "overall": "positive",
    "positive_pct": 67.2,
    "negative_pct": 18.5,
    "neutral_pct": 14.3,
    "confidence": 0.89
  },
  "sentiment_timeline": [
    {"date": "2024-09-01", "sentiment": 0.65},
    {"date": "2024-09-02", "sentiment": 0.70},
    ...
  ],
  "topic_sentiments": {
    "technology": {"score": 0.82, "positive": 89, "negative": 5},
    "politics": {"score": 0.45, "positive": 32, "negative": 41}
  },
  "engagement_analysis": {
    "positive_tweets_avg_likes": 1250,
    "negative_tweets_avg_likes": 890,
    "sentiment_engagement_correlation": 0.65
  },
  "insights": [
    "User is predominantly positive (67%)",
    "Most negative about politics",
    "Positive tweets get 40% more engagement",
    "Sentiment stable over time"
  ]
}
```

### Tweet Data (Internal)

```json
{
  "id": "1234567890",
  "content": "Original tweet text",
  "clean_content": "preprocessed text",
  "sentiment": {
    "label": "positive",
    "score": 0.95,
    "probabilities": {
      "positive": 0.95,
      "negative": 0.03,
      "neutral": 0.02
    }
  },
  "metadata": {
    "date": "2024-10-18T10:00:00Z",
    "likes": 1200,
    "retweets": 450,
    "replies": 89
  }
}
```

## Configuration

### Kaggle Setup

```bash
# 1. Crear cuenta en Kaggle
# 2. Ir a Account → API → Create New API Token
# 3. Descargar kaggle.json

# 4. Configurar credenciales
mkdir ~/.kaggle
cp kaggle.json ~/.kaggle/
chmod 600 ~/.kaggle/kaggle.json
```

### Local Setup

```env
# .env
KAGGLE_USERNAME=your_username
KAGGLE_KEY=your_key

# Model settings
MODEL_DATASET=your_username/shameless-sentiment-models
MODEL_VERSION=v2.0
AUTO_UPDATE_MODEL=true

# Scraper settings
MAX_TWEETS_PER_USER=500
CACHE_EXPIRY_HOURS=24
RATE_LIMIT=1.0

# Output
REPORTS_DIR=data/reports
CACHE_DIR=data/cache
```

## Common Workflows

### 1. Train New Model in Kaggle

```python
# En Kaggle Notebook

# Load dataset
from kaggle import api
api.dataset_download_files('kazanova/sentiment140')

# Train
model = train_model()

# Save
torch.save(model.state_dict(), 'model.pt')
save_tokenizer(tokenizer, 'tokenizer/')

# Create dataset
!kaggle datasets version -p . -m "Updated model v2.0"
```

### 2. Use Model Locally

```bash
# Download model (automático si no existe)
python -m sentiment_analyser.models download-model

# Analyze user
python -m sentiment_analyser analyze @username --output report.pdf

# Or use Python API
python
>>> from sentiment_analyser import UserAnalyzer
>>> analyzer = UserAnalyzer()
>>> result = analyzer.analyze("@username")
>>> result.show()
```

### 3. Update Model

```bash
# Download nueva versión
python -m sentiment_analyser.models update-model --version v2.0

# Validate
python -m sentiment_analyser.models validate-model

# Set as current
python -m sentiment_analyser.models set-current v2.0
```

## Performance Considerations

### Scraping
- **Rate Limiting**: 1 request/segundo (configurable)
- **Cache**: 24h por usuario
- **Batch Size**: 100 tweets por request
- **Timeout**: 30s por request

### Inference
- **Batch Processing**: 32 tweets por batch
- **Device**: CPU por defecto, GPU opcional
- **Max Length**: 512 tokens
- **Quantization**: Opcional para modelos grandes

### Storage
- **Models**: ~250 MB cada uno
- **Cache**: ~10 MB por 500 tweets
- **Reports**: ~2 MB PDF por análisis

## Development Tips

### Testing with Sample Users

```python
# Use public test accounts
TEST_USERS = [
    "@TestUser123",      # Small account (100 tweets)
    "@MediumUser456",    # Medium (1000 tweets)
    "@LargeUser789"      # Large (10000+ tweets)
]

# Quick test
result = analyzer.analyze(TEST_USERS[0], tweets_limit=50)
```

### Debugging Model Issues

```python
# Load model manually
from sentiment_analyser.models import ModelLoader

loader = ModelLoader()
model = loader.load("v2.0")

# Test single prediction
text = "I love this!"
result = model.predict(text)
print(result)  # Should be positive
```

### Monitoring Performance

```python
import time

start = time.time()
result = analyzer.analyze("@username", tweets_limit=500)
duration = time.time() - start

print(f"Analysis took: {duration:.2f}s")
print(f"Tweets/second: {500/duration:.1f}")
```

## Resources

### Kaggle Datasets for Training
- [Sentiment140](https://www.kaggle.com/datasets/kazanova/sentiment140) - 1.6M tweets
- [Twitter Sentiment](https://www.kaggle.com/datasets/jp797498e/twitter-entity-sentiment-analysis) - Entity-level
- [IMDB Reviews](https://www.kaggle.com/datasets/lakshmi25npathi/imdb-dataset-of-50k-movie-reviews) - For transfer learning

### Pre-trained Models
- `bert-base-uncased` - General purpose
- `distilbert-base-uncased` - Faster, smaller
- `cardiffnlp/twitter-roberta-base-sentiment` - Twitter-specific

### Tools
- **Kaggle API**: [Documentation](https://www.kaggle.com/docs/api)
- **snscrape**: [GitHub](https://github.com/JustAnotherArchivist/snscrape)
- **transformers**: [HuggingFace Docs](https://huggingface.co/docs/transformers)
- **Streamlit**: [Documentation](https://docs.streamlit.io/)

## Roadmap

### MVP (Current Phase)
- [x] Project structure
- [ ] Kaggle notebook template
- [ ] Model download from Kaggle
- [ ] User scraping
- [ ] Basic sentiment analysis
- [ ] CLI interface

### Phase 2
- [ ] Streamlit web UI
- [ ] PDF report generation
- [ ] Temporal analysis
- [ ] Topic analysis
- [ ] Model auto-update

### Phase 3
- [ ] Multi-user comparison
- [ ] Real-time monitoring
- [ ] Email alerts
- [ ] Dashboard analytics
- [ ] API REST

### Phase 4
- [ ] Multi-platform (Instagram, Reddit)
- [ ] Image analysis
- [ ] Emotion detection
- [ ] Influencer metrics

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aless-af) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
