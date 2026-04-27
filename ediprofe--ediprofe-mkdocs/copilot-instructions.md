## ediprofe-mkdocs

> Guía: Crear Sitio Educativo con MkDocs Material

Guía: Crear Sitio Educativo con MkDocs Material
Objetivo
Crear un sitio web educativo con:

✅ LaTeX nativo (ecuaciones matemáticas)
✅ Videos de YouTube embebidos
✅ Navegación: MATERIA → UNIDAD → BLOQUE → LECCIÓN
✅ Deploy automático con GitHub Actions
✅ Modo oscuro automático
✅ 100% responsive
Estructura de Navegación
🏠 INICIO (página con cards de materias)
    └── 📚 MATERIA (tabs: Física, Química, Matemáticas, Ciencias)
        └── 📖 UNIDAD (navegación lateral nivel 1)
            └── 📦 BLOQUE (navegación lateral nivel 2)
                └── 📝 LECCIÓN (video + notas con LaTeX)
Paso 1: Crear Estructura de Carpetas
mkdir ediprofe && cd ediprofe
# Crear estructura
mkdir -p docs/{fisica,quimica,matematicas,ciencias}
mkdir -p docs/fisica/{introduccion,cinematica}
mkdir -p docs/quimica/{la-materia,estructura-atomica}
mkdir -p docs/javascripts
mkdir -p docs/stylesheets
mkdir -p .github/workflows
Estructura final:

📁 ediprofe/
├── 📄 mkdocs.yml
├── 📁 docs/
│   ├── 📄 index.md                 ← Página de inicio
│   ├── 📁 javascripts/
│   │   └── 📄 katex.js             ← Configuración LaTeX
│   ├── 📁 stylesheets/
│   │   └── 📄 extra.css            ← Estilos personalizados
│   ├── 📁 fisica/
│   │   ├── 📄 index.md
│   │   └── 📁 introduccion/
│   │       ├── 📄 index.md
│   │       └── 📄 leccion-ejemplo.md
│   ├── 📁 quimica/
│   ├── 📁 matematicas/
│   └── 📁 ciencias/
└── 📁 .github/workflows/
    └── 📄 deploy.yml
Paso 2: Crear mkdocs.yml
site_name: Ediprofe
site_url: https://TU_USUARIO.github.io/ediprofe
site_description: Recursos educativos de Física, Química, Matemáticas y Ciencias
theme:
  name: material
  language: es
  palette:
    - scheme: default
      primary: indigo
      accent: purple
      toggle:
        icon: material/brightness-7
        name: Modo oscuro
    - scheme: slate
      primary: indigo
      accent: purple
      toggle:
        icon: material/brightness-4
        name: Modo claro
  features:
    - navigation.tabs
    - navigation.sections
    - navigation.expand
    - navigation.top
    - search.highlight
    - content.tabs.link
markdown_extensions:
  - pymdownx.arithmatex:
      generic: true
  - pymdownx.superfences
  - pymdownx.tabbed:
      alternate_style: true
  - tables
  - admonition
  - pymdownx.details
  - attr_list
  - md_in_html
extra_javascript:
  - javascripts/katex.js
  - https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.16.9/katex.min.js
  - https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.16.9/contrib/auto-render.min.js
extra_css:
  - https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.16.9/katex.min.css
  - stylesheets/extra.css
nav:
  - 🏠 Inicio: index.md
  
  - 🔬 Física:
    - fisica/index.md
    - 📖 Introducción:
      - fisica/introduccion/index.md
      # Agregar lecciones aquí
    - 📖 Cinemática:
      - fisica/cinematica/index.md
  
  - ⚗️ Química:
    - quimica/index.md
    - 📖 La Materia:
      - quimica/la-materia/index.md
  
  - 📐 Matemáticas:
    - matematicas/index.md
  
  - 🌿 Ciencias:
    - ciencias/index.md
Paso 3: Crear docs/javascripts/katex.js
document.addEventListener("DOMContentLoaded", function() {
  renderMathInElement(document.body, {
    delimiters: [
      {left: "$$", right: "$$", display: true},
      {left: "$", right: "$", display: false},
      {left: "\\(", right: "\\)", display: false},
      {left: "\\[", right: "\\]", display: true}
    ],
    throwOnError: false
  });
});
Paso 4: Crear docs/stylesheets/extra.css
/* Contenedor responsive para videos */
.video-container {
  position: relative;
  padding-bottom: 56.25%;
  height: 0;
  overflow: hidden;
  max-width: 100%;
  margin: 1rem 0;
  border-radius: 8px;
}
.video-container iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: none;
  border-radius: 8px;
}
/* Cards para página de inicio */
.grid-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  margin: 2rem 0;
}
.card {
  padding: 1.5rem;
  border-radius: 12px;
  text-decoration: none;
  transition: transform 0.2s, box-shadow 0.2s;
}
.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0,0,0,0.15);
}
Paso 5: Crear .github/workflows/deploy.yml
name: Deploy MkDocs
on:
  push:
    branches: [main]
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: 3.x
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
Paso 6: Crear Página de Inicio docs/index.md
# 🎓 Bienvenido a Ediprofe
Tu plataforma de recursos educativos en Física, Química, Matemáticas y Ciencias.
<div class="grid-cards" markdown>
[🔬 **Física**](fisica/index.md){ .card style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white;" }
[⚗️ **Química**](quimica/index.md){ .card style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); color: white;" }
[📐 **Matemáticas**](matematicas/index.md){ .card style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%); color: white;" }
[🌿 **Ciencias**](ciencias/index.md){ .card style="background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%); color: white;" }
</div>
Paso 7: Template de una Lección
Cada lección es un archivo .md con este formato:

# 🚀 Título de la Lección
<div class="video-container">
  <iframe 
    src="https://www.youtube.com/embed/VIDEO_ID" 
    allowfullscreen>
  </iframe>
</div>
[Ver en TikTok](https://tiktok.com/...)
---
## Contenido
La **aceleración** se define como:
$$
a = \frac{\Delta v}{t}
$$
Donde:
- $a$ = aceleración en $\mathrm{m/s^2}$
- $\Delta v$ = cambio de velocidad
- $t$ = tiempo
### Tabla de Ejemplo
| Tiempo ($t$) | Velocidad ($v$) |
|:------------:|:---------------:|
| 0 s | 0 m/s |
| 1 s | 5 m/s |
| 2 s | 10 m/s |
Paso 8: Probar Localmente (Opcional)
# Instalar MkDocs Material
pip install mkdocs-material
# Ejecutar servidor de desarrollo
mkdocs serve
# Abrir http://localhost:8000
Paso 9: Deploy a GitHub
# Inicializar git
git init
git add .
git commit -m "Initial commit"
# Crear repo en GitHub y push
gh repo create ediprofe --public --push
# GitHub Actions se ejecutará automáticamente
# Sitio disponible en: https://TU_USUARIO.github.io/ediprofe
Mantenimiento Diario
Para agregar contenido nuevo:

Crear archivo .md en la carpeta correspondiente
Agregar a nav en mkdocs.yml
Push a GitHub:
git add .
git commit -m "Nueva lección: Nombre"
git push
✅ Publicado automáticamente en ~2 minutos
Checklist de Verificación
 LaTeX inline ($x^2$) renderiza
 LaTeX en bloque ($$..$$) renderiza
 Tablas con LaTeX funcionan
 Videos de YouTube se muestran
 Navegación MATERIA → UNIDAD → BLOQUE funciona
 Modo oscuro funciona
 Página responsive en móvil

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ediprofe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
