## nicotron-github-io

> Eres un asistente de desarrollo para el proyecto nicotron.cl

Eres un asistente de desarrollo para el proyecto nicotron.cl

## Stack
- HTML5 semántico, CSS con custom properties, JavaScript vanilla
- Sin frameworks, sin build tools

## Convenciones CSS
- Todos los colores usan variables de styles.css (nunca valores hardcodeados)
- Nomenclatura BEM: .bloque__elemento--modificador
- Mobile first

## Tokens disponibles
- Colores:
  /* Colores */
  --color-white_1: #f0f0f0; 
  --color-white_2: #FCFCFC;
  --color-white_3: #F6F7F9;
  --color-black_1: #020303;
  --color-black_2: #16191D;
  --color-black_3: #2E3338;
  --color-grey_0: #E5E6E7;
  --color-gold: #EBDABA;
  --color-green: #2ECC71;
  --color-blue: #6F96B2;
  --color-red: #E84920;

  /* Variables de colores light */
  --color-light-primary: var(--color-white_1);
  --color-light-secondary: var(--color-white_2);
  --color-light-tertiary: var(--color-white_3);
  
  /* Variables de colores dark */
  --color-dark-primary: var(--color-black_1);
  --color-dark-secondary: var(--color-black_2);
  --color-dark-tertiary: var(--color-black_3);
  
  /* Variables de colores de interacción */
  --color-accent: var(--color-gold);
  --color-success: var(--color-green);
  --color-info: var(--color-blue);
  --color-error: var(--color-red);
  --color-text-inverted: var(--color-grey_0);

  --color-background: var(--color-white);
- Fuentes:
  /* Sistema Tipográfico ----------------------- */
  /* Familia tipográfica */
  --font-primary: "Ubuntu";
  --font-weight-regular: 400;
  --font-weight-bold: 700;

  /* Escala tipográfica */
  --text-xs: 0.625rem;     /* 10px */
  --text-sm: 0.75rem;      /* 12px */
  --text-helper: 0.875rem;  /* 14px */
  --text-base: 1rem;       /* 16px */
  --text-lg: 1.125rem;     /* 18px */
  --text-xl: 1.25rem;      /* 20px */
  --text-2xl: 1.5rem;      /* 24px */
  --text-3xl: 2rem;        /* 32px */
  --text-4xl: 2.5rem;      /* 40px */
  --text-5xl: 3rem;        /* 48px */
  --text-6xl: 4rem;        /* 64px */
- Variables semánticas responsivas:
  /* Variables semánticas responsivas */
  --h1-mobile: var(--text-2xl);    /* 24px */
  --h1-tablet: var(--text-4xl);    /* 40px */
  --h1-desktop: var(--text-6xl);   /* 64px */
  
  --h2-mobile: var(--text-2xl);    /* 24px */
  --h2-tablet: var(--text-3xl);    /* 32px */
  --h2-desktop: var(--text-4xl);   /* 40px */
  
  --h3-mobile: var(--text-xl);     /* 20px */
  --h3-tablet: var(--text-2xl);    /* 24px */
  --h3-desktop: var(--text-3xl);   /* 32px */
  
  --p-mobile: var(--text-base);    /* 16px */
  --p-tablet: var(--text-lg);      /* 18px */
  --p-desktop: var(--text-xl);     /* 20px */

- Espaciado:
  /* Line heights */
  --leading-none: 1;        /* 16px  */
  --leading-tight: 1.25;    /* 20px  */
  --leading-snug: 1.375;    /* 22px  */
  --leading-normal: 1.5;    /* 24px  */
  --leading-relaxed: 1.625; /* 26px  */
  --leading-loose: 2;       /* 32px  */

  /* Letter spacing */
  --tracking-tighter: -0.05em;
  --tracking-tight: -0.025em;
  --tracking-normal: 0em;
  --tracking-wide: 0.025em;
  --tracking-wider: 0.05em;
  
  /* Espaciado */
  --spacing-xs: 0.5rem;    /* 8px */
  --spacing-sm: 0.75rem;   /* 12px */
  --spacing-md: 1rem;      /* 16px */
  --spacing-lg: 1.5rem;    /* 24px */
  --spacing-xl: 2rem;      /* 32px */
  --spacing-2xl: 3rem;     /* 48px */

- Responsividad
  /* Breakpoints */
  --breakpoint-sm: 48rem;     /* 768px  - Tablet */
  --breakpoint-md: 62rem;     /* 992px  - Desktop */
  --breakpoint-lg: 80rem;     /* 1280px - Large Desktop */
  --breakpoint-xl: 90rem;     /* 1440px - Extra Large */
  --breakpoint-xxl: 120rem;   /* 1920px - HD/4K */


## Estructura del proyecto
- styles.css: variables globales
- pages/: páginas completas

## Al recibir un link de Figma
1. Leer el diseño
2. Respetar los tokens existentes
3. Usar HTML semántico (nav, main, article, section, footer)
4. No instalar dependencias

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nicotron) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
