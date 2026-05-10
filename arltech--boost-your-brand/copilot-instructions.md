## boost-your-brand

> Landing page com animacao 3D usando Three.js (WebGL) e GSAP. Efeito principal: Particle Morphing entre uma esfera (globo) e um astronauta.

# Boost Your Brand - 3D Particle Morphing Landing Page

## Sobre o Projeto
Landing page com animacao 3D usando Three.js (WebGL) e GSAP. Efeito principal: Particle Morphing entre uma esfera (globo) e um astronauta.

## Stack
- **Three.js** v0.160.0 (via CDN importmap, ES modules)
- **GSAP** v3.12.5 (via CDN)
- **Montserrat** (Google Fonts)
- HTML/CSS/JS puro (sem bundler)

## Estrutura
```
index.html          - Pagina principal (tudo inline por enquanto)
assets/models/      - Modelos 3D (.glb, .gltf, .obj)
assets/textures/    - Texturas (Earth, etc.)
css/                - Estilos (quando extrair do HTML)
js/                 - Scripts (quando modularizar)
```

## Como rodar localmente
```bash
# Precisa de servidor HTTP (CORS bloqueia file://)
python3 -m http.server 8000
# ou
npx serve .
```

## Carregar modelo GLTF externo
1. Colocar .glb em `assets/models/`
2. Descomentar `loadGLTFModel('assets/models/astronaut.glb')` no index.html
3. Acessar via http://localhost:8000

## Funcionalidades
- 10.000 particulas com Fibonacci sphere distribution
- Morphing esfera <-> astronauta com dispersao (scatter)
- Astronauta geometrico fallback (sem modelo externo)
- GLTFLoader + OBJLoader prontos para modelos externos
- GSAP power3.inOut easing
- Mouse parallax + auto-morph timer
- Estrelas de fundo 3D
- Blending aditivo para efeito de brilho

---
> Source: [arltech/boost-your-brand](https://github.com/arltech/boost-your-brand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
