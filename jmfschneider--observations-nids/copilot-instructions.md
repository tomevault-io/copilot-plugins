## observations-nids

> - **Framework CSS** : Bootstrap 5.3+


# Règles Frontend

## Stack Frontend

- **Framework CSS** : Bootstrap 5.3+
- **Formulaires** : Crispy Forms avec crispy-bootstrap5
- **Templates** : Django Template Language
- **JavaScript** : Vanilla JS (pas de framework frontend)
- **Icônes** : Font Awesome (CDN)

## Templates Django - Structure

### Template de Base

Tous les templates doivent étendre `base.html` :

```django
{% extends 'base.html' %}
{% load static %}
{% load crispy_forms_tags %}

{% block title %}Mon Titre - Observations Nids{% endblock %}

{% block extra_css %}
<link rel="stylesheet" href="{% static 'observations/css/custom.css' %}">
{% endblock %}

{% block content %}
<div class="container mt-4">
    <h1>Mon Contenu</h1>
</div>
{% endblock %}

{% block extra_js %}
<script src="{% static 'observations/js/custom.js' %}"></script>
{% endblock %}
```

### Règles de Structure

```django
{# ✅ BON - Espacement clair, commentaires utiles #}
{% extends 'base.html' %}

{# Section formulaire utilisateur #}
<div class="card">
    <div class="card-header">
        <h3>Formulaire</h3>
    </div>
    <div class="card-body">
        {% include 'partials/user_form.html' %}
    </div>
</div>

{# ❌ MAUVAIS - Pas de commentaires, imbrication complexe #}
<div><div><div><form>...</form></div></div></div>
```

## Bootstrap 5 - Conventions

### Grille et Layout

```django
{# Container principal #}
<div class="container">
    <div class="row">
        <div class="col-md-8">
            {# Contenu principal #}
        </div>
        <div class="col-md-4">
            {# Sidebar #}
        </div>
    </div>
</div>

{# Cards pour regrouper le contenu #}
<div class="card mb-3">
    <div class="card-header bg-primary text-white">
        <h4 class="mb-0">Titre</h4>
    </div>
    <div class="card-body">
        <p class="card-text">Contenu</p>
    </div>
    <div class="card-footer">
        <a href="#" class="btn btn-primary">Action</a>
    </div>
</div>
```

### Composants Standards

```django
{# Boutons avec classes sémantiques #}
<a href="{% url 'observations:creer_fiche' %}" class="btn btn-primary">
    <i class="fas fa-plus"></i> Nouvelle Fiche
</a>
<button type="submit" class="btn btn-success">
    <i class="fas fa-save"></i> Enregistrer
</button>
<a href="{% url 'observations:liste' %}" class="btn btn-secondary">
    <i class="fas fa-arrow-left"></i> Retour
</a>
<button type="button" class="btn btn-danger" onclick="confirmerSuppression()">
    <i class="fas fa-trash"></i> Supprimer
</button>

{# Badges pour les statuts #}
<span class="badge bg-success">Validé</span>
<span class="badge bg-warning">En attente</span>
<span class="badge bg-danger">Refusé</span>
<span class="badge bg-info">En cours</span>

{# Alerts pour les messages #}
<div class="alert alert-success alert-dismissible fade show" role="alert">
    <i class="fas fa-check-circle"></i> Opération réussie
    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
</div>
```

## Messages Django

### Affichage Automatique

```django
{# Dans base.html ou au début de chaque template #}
{% if messages %}
    <div class="messages-container">
        {% for message in messages %}
            <div class="alert alert-{{ message.tags }} alert-dismissible fade show" role="alert">
                {% if message.tags == 'error' %}
                    <i class="fas fa-exclamation-circle"></i>
                {% elif message.tags == 'success' %}
                    <i class="fas fa-check-circle"></i>
                {% elif message.tags == 'warning' %}
                    <i class="fas fa-exclamation-triangle"></i>
                {% else %}
                    <i class="fas fa-info-circle"></i>
                {% endif %}
                {{ message }}
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        {% endfor %}
    </div>
{% endif %}
```

### Tags Bootstrap Standards

- `success` → `alert-success` (vert)
- `error` → `alert-danger` (rouge)
- `warning` → `alert-warning` (orange)
- `info` → `alert-info` (bleu)

## Crispy Forms - Formulaires

### Usage Standard

```django
{% load crispy_forms_tags %}

{# Formulaire complet avec Crispy #}
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form|crispy }}
    
    <div class="form-actions mt-3">
        <button type="submit" class="btn btn-primary">
            <i class="fas fa-save"></i> Enregistrer
        </button>
        <a href="{% url 'observations:liste' %}" class="btn btn-secondary">
            <i class="fas fa-times"></i> Annuler
        </a>
    </div>
</form>
```

### Personnalisation par Champ

```django
{% load crispy_forms_tags %}

<form method="post">
    {% csrf_token %}
    
    {# Champ individuel avec helper #}
    {% crispy form.username %}
    
    {# Layout personnalisé #}
    <div class="row">
        <div class="col-md-6">
            {% crispy form.first_name %}
        </div>
        <div class="col-md-6">
            {% crispy form.last_name %}
        </div>
    </div>
    
    {# Champ avec aide contextuelle #}
    <div class="mb-3">
        <label for="{{ form.email.id_for_label }}" class="form-label">
            Email <span class="text-danger">*</span>
        </label>
        {{ form.email }}
        <small class="form-text text-muted">
            Votre email ne sera pas partagé publiquement.
        </small>
    </div>
    
    <button type="submit" class="btn btn-primary">Valider</button>
</form>
```

## PAS de Logique Métier dans les Templates

### ✅ BON - Logique dans la Vue

```python
# views.py
def liste_fiches(request):
    fiches = FicheObservation.objects.select_related('observateur', 'espece')
    
    # Calcul dans la vue
    fiches_avec_stats = []
    for fiche in fiches:
        fiches_avec_stats.append({
            'fiche': fiche,
            'est_complete': fiche.etat_correction.pourcentage_completion == 100,
            'nb_observations': fiche.observations.count(),
        })
    
    return render(request, 'liste.html', {'fiches': fiches_avec_stats})
```

```django
{# Template - Affichage simple #}
{% for item in fiches %}
    <tr>
        <td>{{ item.fiche.num_fiche }}</td>
        <td>
            {% if item.est_complete %}
                <span class="badge bg-success">Complète</span>
            {% else %}
                <span class="badge bg-warning">Incomplète</span>
            {% endif %}
        </td>
        <td>{{ item.nb_observations }} observation(s)</td>
    </tr>
{% endfor %}
```

### ❌ MAUVAIS - Logique dans le Template

```django
{# Ne JAMAIS faire de queries dans les templates #}
{% for fiche in fiches %}
    <td>{{ fiche.observations.all|length }}</td>  {# Query à chaque itération! #}
{% endfor %}

{# Ne JAMAIS faire de calculs complexes #}
{% for fiche in fiches %}
    {% if fiche.resume.nombre_oeufs_pondus > 0 and fiche.resume.nombre_oeufs_eclos > 0 %}
        {{ fiche.resume.nombre_oeufs_eclos|div:fiche.resume.nombre_oeufs_pondus|mul:100 }}%
    {% endif %}
{% endfor %}
```

## JavaScript - Conventions

### Vanilla JS - Pas de jQuery

```javascript
// ✅ BON - Vanilla JS moderne
document.addEventListener('DOMContentLoaded', function() {
    const bouton = document.getElementById('mon-bouton');
    
    bouton.addEventListener('click', function(e) {
        e.preventDefault();
        console.log('Bouton cliqué');
    });
});

// Utilitaire pour CSRF token
function getCsrfToken() {
    return document.querySelector('[name=csrfmiddlewaretoken]').value;
}
```

### Fetch API - Pattern Standard avec Gestion d'Erreurs

**IMPORTANT** : Toujours utiliser le wrapper `fetcherRobuste` pour les appels AJAX. Ne jamais appeler `fetch()` directement.

```javascript
// ✅ Wrapper standard pour tous les appels fetch
// À placer dans common.js et réutiliser partout
async function fetcherRobuste(url, options = {}) {
    try {
        const response = await fetch(url, options);
        
        // 1. Vérifier le statut HTTP AVANT de parser
        if (!response.ok) {
            // Tenter de lire le message d'erreur si JSON
            let errorMessage = `Erreur HTTP ${response.status}`;
            try {
                const errorData = await response.json();
                errorMessage = errorData.message || errorMessage;
            } catch {
                // Si ce n'est pas du JSON (ex: page HTML d'erreur Django)
                errorMessage = `${errorMessage}: ${response.statusText}`;
            }
            throw new Error(errorMessage);
        }
        
        // 2. Parser le JSON seulement si OK
        const data = await response.json();
        return data;
        
    } catch (error) {
        // 3. Distinguer les types d'erreurs
        if (error instanceof TypeError) {
            // Erreur réseau (pas de connexion, CORS, etc.)
            console.error('Erreur réseau:', error);
            throw new Error('Impossible de contacter le serveur. Vérifiez votre connexion.');
        } else if (error instanceof SyntaxError) {
            // Erreur de parsing JSON
            console.error('Erreur parsing JSON:', error);
            throw new Error('Réponse serveur invalide (JSON attendu).');
        } else {
            // Erreur HTTP ou autre
            console.error('Erreur:', error);
            throw error;
        }
    }
}

// ✅ Exemple d'utilisation - Géocodage
async function geocoderCommune(ficheId, commune) {
    try {
        const data = await fetcherRobuste('/geo/geocoder/', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'X-CSRFToken': getCsrfToken(),
            },
            body: new URLSearchParams({
                fiche_id: ficheId,
                commune: commune,
            }),
        });
        
        if (data.success) {
            afficherCoordonnees(data.coords);
        } else {
            afficherErreur(data.message || 'Erreur inconnue');
        }
    } catch (error) {
        afficherErreur(error.message);
    }
}

// ✅ Exemple d'utilisation - GET simple
async function chargerFiche(numFiche) {
    try {
        const data = await fetcherRobuste(`/api/fiches/${numFiche}/`);
        afficherFiche(data);
    } catch (error) {
        afficherErreur(`Impossible de charger la fiche: ${error.message}`);
    }
}
```

### Anti-Patterns Fetch - À ÉVITER

```javascript
// ❌ MAUVAIS - Appeler fetch() directement sans wrapper
async function mauvaiseFonction() {
    const response = await fetch('/api/data/');
    const data = await response.json(); // Plante si erreur 500 avec HTML
}

// ❌ MAUVAIS - Parse JSON sans vérifier response.ok
async function autreMauvaiseFonction() {
    const response = await fetch('/api/data/');
    const data = await response.json(); // Erreur si non-JSON
    if (data.success) { ... }
}

// ❌ MAUVAIS - Catch générique sans message utilisateur
async function encoreMauvais() {
    try {
        const response = await fetch('/api/data/');
        const data = await response.json();
    } catch (error) {
        console.error(error); // L'utilisateur ne voit rien!
    }
}

// ✅ BON - Toujours utiliser fetcherRobuste
async function bonneFonction() {
    try {
        const data = await fetcherRobuste('/api/data/');
        traiterDonnees(data);
    } catch (error) {
        afficherErreur(error.message);
    }
}
```

### Intégration avec Django

```django
{# Template avec JavaScript inline (minimal) #}
<button 
    type="button" 
    class="btn btn-primary"
    onclick="geocoderCommune({{ fiche.num_fiche }}, '{{ fiche.localisation.commune|escapejs }}')"
>
    Géocoder
</button>

<script>
// ✅ Passer des données Django au JS
const ficheData = {
    numFiche: {{ fiche.num_fiche }},
    annee: {{ fiche.annee }},
    observateur: "{{ fiche.observateur.username|escapejs }}"
};

// Utiliser dans le code JS
console.log(`Fiche ${ficheData.numFiche} de ${ficheData.observateur}`);
</script>
```

### Organisation des Fichiers JS

```
observations/static/observations/js/
├── common.js          # Fonctions utilitaires communes
├── sidebar.js         # Gestion du sidebar
├── geocoding.js       # Fonctionnalités géocodage
└── stats.js           # Graphiques et statistiques
```

```django
{# Charger les scripts dans l'ordre #}
{% block extra_js %}
<script src="{% static 'observations/js/common.js' %}"></script>
<script src="{% static 'observations/js/geocoding.js' %}"></script>
{% endblock %}
```

## CSS - Organisation

### Fichiers Statiques

```
observations/static/observations/css/
├── custom.css         # Styles globaux personnalisés
├── sidebar.css        # Styles du sidebar
└── print.css          # Styles d'impression
```

### Convention de Nommage

```css
/* Classes BEM-like pour composants custom */
.fiche-card {
    border: 1px solid #ddd;
}

.fiche-card__header {
    background-color: #f8f9fa;
    padding: 1rem;
}

.fiche-card__title {
    margin: 0;
    font-size: 1.25rem;
}

.fiche-card--validated {
    border-color: #28a745;
}

/* Préfixer les classes custom pour éviter conflits Bootstrap */
.obs-sidebar {
    width: 250px;
}

.obs-table-striped tbody tr:nth-of-type(odd) {
    background-color: #f8f9fa;
}
```

### Surcharges Bootstrap Minimales

```css
/* Personnalisation des variables Bootstrap si nécessaire */
:root {
    --bs-primary: #0d6efd;
    --bs-success: #198754;
}

/* Surcharges spécifiques - toujours justifier par un commentaire */
.btn-primary {
    /* Ajustement pour meilleure lisibilité sur fond sombre */
    font-weight: 600;
}
```

## Tableaux et Listes

### Tables Responsives

```django
<div class="table-responsive">
    <table class="table table-striped table-hover">
        <thead class="table-dark">
            <tr>
                <th>Fiche</th>
                <th>Observateur</th>
                <th>Espèce</th>
                <th>Date</th>
                <th class="text-end">Actions</th>
            </tr>
        </thead>
        <tbody>
            {% for fiche in fiches %}
            <tr>
                <td><a href="{% url 'observations:detail_fiche' fiche.num_fiche %}">#{{ fiche.num_fiche }}</a></td>
                <td>{{ fiche.observateur.get_full_name }}</td>
                <td><em>{{ fiche.espece.nom_scientifique }}</em></td>
                <td>{{ fiche.date_creation|date:"d/m/Y" }}</td>
                <td class="text-end">
                    <a href="{% url 'observations:modifier_fiche' fiche.num_fiche %}" 
                       class="btn btn-sm btn-outline-primary">
                        <i class="fas fa-edit"></i>
                    </a>
                </td>
            </tr>
            {% empty %}
            <tr>
                <td colspan="5" class="text-center text-muted">
                    Aucune fiche trouvée.
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</div>
```

### Pagination

```django
{# Pagination Bootstrap 5 #}
{% if page_obj.has_other_pages %}
<nav aria-label="Pagination">
    <ul class="pagination justify-content-center">
        {% if page_obj.has_previous %}
        <li class="page-item">
            <a class="page-link" href="?page={{ page_obj.previous_page_number }}">Précédent</a>
        </li>
        {% else %}
        <li class="page-item disabled">
            <span class="page-link">Précédent</span>
        </li>
        {% endif %}
        
        {% for num in page_obj.paginator.page_range %}
            {% if page_obj.number == num %}
            <li class="page-item active">
                <span class="page-link">{{ num }}</span>
            </li>
            {% else %}
            <li class="page-item">
                <a class="page-link" href="?page={{ num }}">{{ num }}</a>
            </li>
            {% endif %}
        {% endfor %}
        
        {% if page_obj.has_next %}
        <li class="page-item">
            <a class="page-link" href="?page={{ page_obj.next_page_number }}">Suivant</a>
        </li>
        {% endif %}
    </ul>
</nav>
{% endif %}
```

## Accessibilité et Bonnes Pratiques

```django
{# Labels explicites pour les formulaires #}
<label for="{{ form.username.id_for_label }}" class="form-label">
    Nom d'utilisateur <span class="text-danger">*</span>
</label>
{{ form.username }}

{# Attributs ARIA pour les composants dynamiques #}
<button 
    type="button" 
    class="btn btn-primary"
    aria-label="Géocoder la commune"
    data-bs-toggle="modal" 
    data-bs-target="#geocodageModal"
>
    <i class="fas fa-map-marker-alt" aria-hidden="true"></i> Géocoder
</button>

{# Textes alternatifs pour les images #}
<img src="{% static 'images/logo.png' %}" alt="Logo Observations Nids" class="img-fluid">

{# Navigation avec liens actifs #}
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <ul class="navbar-nav">
        <li class="nav-item {% if request.resolver_match.url_name == 'home' %}active{% endif %}">
            <a class="nav-link" href="{% url 'observations:home' %}">Accueil</a>
        </li>
    </ul>
</nav>
```

## Anti-Patterns Frontend

```django
{# ❌ Ne JAMAIS mélanger logique Python et HTML #}
<div>
    {% for i in "12345" %}  {# Mauvais hack #}
        <span>{{ i }}</span>
    {% endfor %}
</div>

{# ❌ Ne JAMAIS faire de queries dans les templates #}
{{ fiche.observations.all.count }}  {# Query à chaque rendu #}

{# ❌ Ne JAMAIS utiliser de styles inline #}
<div style="color: red; margin: 10px;">  {# Utiliser des classes CSS #}

{# ❌ Ne JAMAIS dupliquer du code HTML #}
{# Utiliser {% include %} ou {% block %} #}
```

## Checklist Frontend

- [ ] Template étend `base.html`
- [ ] Tous les formulaires ont `{% csrf_token %}`
- [ ] Classes Bootstrap 5 utilisées
- [ ] Formulaires rendus avec Crispy Forms
- [ ] Pas de logique métier dans les templates
- [ ] JavaScript vanilla (pas de jQuery)
- [ ] Accessibilité : labels, ARIA, alt text
- [ ] Responsive (grille Bootstrap)
- [ ] Icônes Font Awesome avec aria-hidden="true"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmFschneider) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
