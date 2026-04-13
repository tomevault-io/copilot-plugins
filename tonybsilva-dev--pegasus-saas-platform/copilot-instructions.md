## pegasus-saas-platform

> SEO Guidelines - Diretrizes completas para otimização de mecanismos de busca


# SEO Guidelines - Diretrizes de SEO

## **Visão Geral**

Diretrizes completas para otimização de mecanismos de busca (SEO), incluindo meta tags, estrutura HTML semântica, Open Graph, Schema.org, e otimização de imagens e links.

---

## 📋 Meta Tags Essenciais

### **1. Title Tag**

O título deve ser descritivo, único por página, e ter entre 50-60 caracteres.

#### **✅ DO: Títulos Descritivos e Únicos**

```tsx
// ✅ DO: Usar títulos específicos por página
import { useEffect } from 'react';

export function SettingsPage() {
  useEffect(() => {
    document.title = 'Configurações - Pegasus Dashboard';
  }, []);
  
  return <div>...</div>;
}

// ✅ DO: Hook reutilizável para SEO
export function usePageTitle(title: string) {
  useEffect(() => {
    const previousTitle = document.title;
    document.title = `${title} - Pegasus`;
    return () => {
      document.title = previousTitle;
    };
  }, [title]);
}

// Uso:
export function DashboardPage() {
  usePageTitle('Dashboard');
  return <div>...</div>;
}
```

#### **❌ DON'T: Títulos Genéricos ou Curtos**

```tsx
// ❌ DON'T: Título muito curto ou genérico
export function SettingsPage() {
  // document.title = "Pegasus"; // Muito curto (7 caracteres)
  // document.title = "Settings"; // Genérico, não inclui contexto
  return <div>...</div>;
}
```

### **2. Meta Description**

Meta description é obrigatória e deve ter entre 120-160 caracteres, descrevendo o conteúdo da página.

#### **✅ DO: Descriptions Descritivas**

```tsx
// ✅ DO: Hook para meta description
export function useMetaDescription(description: string) {
  useEffect(() => {
    let metaDescription = document.querySelector('meta[name="description"]');
    
    if (!metaDescription) {
      metaDescription = document.createElement('meta');
      metaDescription.setAttribute('name', 'description');
      document.head.appendChild(metaDescription);
    }
    
    metaDescription.setAttribute('content', description);
  }, [description]);
}

// Uso:
export function SettingsPage() {
  useMetaDescription(
    'Gerencie suas configurações de perfil, equipe, veículos e canais no Pegasus Dashboard. Personalize sua experiência e configure preferências.'
  );
  return <div>...</div>;
}
```

#### **❌ DON'T: Deixar Description Vazia**

```tsx
// ❌ DON'T: Não definir meta description
export function SettingsPage() {
  // Sem meta description - extensão SEO vai reportar erro
  return <div>...</div>;
}
```

### **3. Meta Keywords**

Embora menos crítico que antes, ainda deve ser incluído com palavras-chave relevantes.

#### **✅ DO: Keywords Relevantes**

```tsx
// ✅ DO: Hook para keywords
export function useMetaKeywords(keywords: string[]) {
  useEffect(() => {
    const keywordsString = keywords.join(', ');
    let metaKeywords = document.querySelector('meta[name="keywords"]');
    
    if (!metaKeywords) {
      metaKeywords = document.createElement('meta');
      metaKeywords.setAttribute('name', 'keywords');
      document.head.appendChild(metaKeywords);
    }
    
    metaKeywords.setAttribute('content', keywordsString);
  }, [keywords]);
}

// Uso:
export function SettingsPage() {
  useMetaKeywords([
    'configurações',
    'dashboard',
    'perfil',
    'equipe',
    'veículos',
    'pegasus',
    'dilis'
  ]);
  return <div>...</div>;
}
```

### **4. Meta Robots**

Controla como os mecanismos de busca indexam a página.

#### **✅ DO: Robots Tag Configurada**

```tsx
// ✅ DO: Hook para robots meta tag
export function useMetaRobots(content: string = 'index, follow') {
  useEffect(() => {
    let metaRobots = document.querySelector('meta[name="robots"]');
    
    if (!metaRobots) {
      metaRobots = document.createElement('meta');
      metaRobots.setAttribute('name', 'robots');
      document.head.appendChild(metaRobots);
    }
    
    metaRobots.setAttribute('content', content);
  }, [content]);
}

// Uso:
export function SettingsPage() {
  useMetaRobots('index, follow'); // Página pública
  // useMetaRobots('noindex, nofollow'); // Para páginas privadas se necessário
  return <div>...</div>;
}
```

### **5. Canonical URL**

Define a URL canônica para evitar conteúdo duplicado.

#### **✅ DO: Canonical URL Definida**

```tsx
// ✅ DO: Hook para canonical URL
export function useCanonicalUrl(url: string) {
  useEffect(() => {
    let linkCanonical = document.querySelector('link[rel="canonical"]');
    
    if (!linkCanonical) {
      linkCanonical = document.createElement('link');
      linkCanonical.setAttribute('rel', 'canonical');
      document.head.appendChild(linkCanonical);
    }
    
    linkCanonical.setAttribute('href', url);
  }, [url]);
}

// Uso:
export function SettingsPage() {
  const baseUrl = 'https://pegasus.dilis.com.br';
  useCanonicalUrl(`${baseUrl}/dashboard/settings`);
  return <div>...</div>;
}
```

### **6. Meta Author e Publisher**

Informações sobre autor e editor da página.

#### **✅ DO: Author e Publisher Definidos**

```tsx
// ✅ DO: Hooks para author e publisher
export function useMetaAuthor(author: string) {
  useEffect(() => {
    let metaAuthor = document.querySelector('meta[name="author"]');
    
    if (!metaAuthor) {
      metaAuthor = document.createElement('meta');
      metaAuthor.setAttribute('name', 'author');
      document.head.appendChild(metaAuthor);
    }
    
    metaAuthor.setAttribute('content', author);
  }, [author]);
}

export function useMetaPublisher(publisher: string) {
  useEffect(() => {
    let metaPublisher = document.querySelector('meta[name="publisher"]');
    
    if (!metaPublisher) {
      metaPublisher = document.createElement('meta');
      metaPublisher.setAttribute('name', 'publisher');
      document.head.appendChild(metaPublisher);
    }
    
    metaPublisher.setAttribute('content', publisher);
  }, [publisher]);
}

// Uso:
export function SettingsPage() {
  useMetaAuthor('Dilis Studio');
  useMetaPublisher('Dilis Studio');
  return <div>...</div>;
}
```

### **7. Language (Lang)**

Idioma da página deve estar correto no HTML.

#### **✅ DO: Lang Correto no HTML**

```html
<!-- ✅ DO: index.html com lang correto -->
<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Pegasus</title>
</head>
```

#### **❌ DON'T: Lang Incorreto**

```html
<!-- ❌ DON'T: Lang em inglês quando o conteúdo é em português -->
<html lang="en">
```

---

## 🌐 Open Graph (Facebook) e Twitter Cards

### **Open Graph Tags**

Essenciais para compartilhamento em redes sociais.

#### **✅ DO: Open Graph Completo**

```tsx
// ✅ DO: Hook completo para Open Graph
export function useOpenGraph(og: {
  title: string;
  description: string;
  image: string;
  url: string;
  type?: string;
  siteName?: string;
}) {
  useEffect(() => {
    const ogTags = [
      { property: 'og:title', content: og.title },
      { property: 'og:description', content: og.description },
      { property: 'og:image', content: og.image },
      { property: 'og:url', content: og.url },
      { property: 'og:type', content: og.type || 'website' },
      { property: 'og:site_name', content: og.siteName || 'Pegasus' },
      { property: 'og:locale', content: 'pt_BR' },
    ];

    ogTags.forEach(({ property, content }) => {
      let meta = document.querySelector(`meta[property="${property}"]`);
      
      if (!meta) {
        meta = document.createElement('meta');
        meta.setAttribute('property', property);
        document.head.appendChild(meta);
      }
      
      meta.setAttribute('content', content);
    });
  }, [og]);
}

// Uso:
export function SettingsPage() {
  useOpenGraph({
    title: 'Configurações - Pegasus Dashboard',
    description: 'Gerencie suas configurações no Pegasus Dashboard',
    image: 'https://pegasus.dilis.com.br/og-image.png',
    url: 'https://pegasus.dilis.com.br/dashboard/settings',
    type: 'website',
    siteName: 'Pegasus',
  });
  return <div>...</div>;
}
```

### **Twitter Cards**

#### **✅ DO: Twitter Cards Completo**

```tsx
// ✅ DO: Hook para Twitter Cards
export function useTwitterCard(twitter: {
  card: 'summary' | 'summary_large_image';
  title: string;
  description: string;
  image: string;
  site?: string;
  creator?: string;
}) {
  useEffect(() => {
    const twitterTags = [
      { name: 'twitter:card', content: twitter.card },
      { name: 'twitter:title', content: twitter.title },
      { name: 'twitter:description', content: twitter.description },
      { name: 'twitter:image', content: twitter.image },
      { name: 'twitter:site', content: twitter.site || '@dilisstudio' },
      { name: 'twitter:creator', content: twitter.creator || '@dilisstudio' },
    ];

    twitterTags.forEach(({ name, content }) => {
      let meta = document.querySelector(`meta[name="${name}"]`);
      
      if (!meta) {
        meta = document.createElement('meta');
        meta.setAttribute('name', name);
        document.head.appendChild(meta);
      }
      
      meta.setAttribute('content', content);
    });
  }, [twitter]);
}

// Uso:
export function SettingsPage() {
  useTwitterCard({
    card: 'summary_large_image',
    title: 'Configurações - Pegasus Dashboard',
    description: 'Gerencie suas configurações no Pegasus Dashboard',
    image: 'https://pegasus.dilis.com.br/twitter-image.png',
  });
  return <div>...</div>;
}
```

---

## 📊 Schema.org (Structured Data)

### **✅ DO: Schema.org para Páginas**

```tsx
// ✅ DO: Hook para Schema.org
export function useSchemaOrg(schema: {
  type: string;
  data: Record<string, unknown>;
}) {
  useEffect(() => {
    let script = document.querySelector('script[type="application/ld+json"]');
    
    if (!script) {
      script = document.createElement('script');
      script.setAttribute('type', 'application/ld+json');
      document.head.appendChild(script);
    }
    
    const schemaData = {
      '@context': 'https://schema.org',
      '@type': schema.type,
      ...schema.data,
    };
    
    script.textContent = JSON.stringify(schemaData);
  }, [schema]);
}

// Uso para página de configurações:
export function SettingsPage() {
  useSchemaOrg({
    type: 'WebPage',
    data: {
      name: 'Configurações - Pegasus Dashboard',
      description: 'Gerencie suas configurações no Pegasus Dashboard',
      url: 'https://pegasus.dilis.com.br/dashboard/settings',
      isPartOf: {
        '@type': 'WebSite',
        name: 'Pegasus',
        url: 'https://pegasus.dilis.com.br',
      },
    },
  });
  return <div>...</div>;
}

// Uso para organização:
export function useOrganizationSchema() {
  useSchemaOrg({
    type: 'Organization',
    data: {
      name: 'Dilis Studio',
      url: 'https://dilis.com.br',
      logo: 'https://pegasus.dilis.com.br/logo.png',
      sameAs: [
        'https://www.linkedin.com/company/dilis',
        'https://github.com/dilis',
      ],
    },
  });
}
```

---

## 🖼️ Otimização de Imagens

### **1. Alt Text Obrigatório**

Todas as imagens devem ter `alt` descritivo.

#### **✅ DO: Alt Text Descritivo**

```tsx
// ✅ DO: Alt text descritivo e contextual
<img
  src="/assets/icons/logo-icon-slim.svg"
  alt="Logo do Pegasus - Plataforma de gerenciamento"
  title="Logo do Pegasus"
  loading="lazy"
/>

// ✅ DO: Alt text para avatares
<img
  src={user.avatar}
  alt={`Foto de perfil de ${user.name}`}
  title={`Foto de perfil de ${user.name}`}
  loading="lazy"
/>
```

#### **❌ DON'T: Alt Text Vazio ou Genérico**

```tsx
// ❌ DON'T: Alt text vazio ou genérico
<img src="/logo.png" alt="" />
<img src="/logo.png" alt="logo" />
<img src="/logo.png" /> {/* Sem alt */}
```

### **2. Title Attribute para Imagens**

Todas as imagens devem ter `title` attribute.

#### **✅ DO: Title Attribute em Imagens**

```tsx
// ✅ DO: Title attribute descritivo
<img
  src="/assets/icons/logo-icon-slim.svg"
  alt="Logo do Pegasus"
  title="Logo do Pegasus - Plataforma de gerenciamento Dilis Studio"
  loading="lazy"
/>
```

### **3. Nomes de Arquivo Descritivos**

#### **✅ DO: Nomes de Arquivo Descritivos**

```tsx
// ✅ DO: Nomes descritivos
// assets/images/logo-pegasus-dashboard.svg
// assets/images/avatar-user-profile.jpg
// assets/images/icon-settings-gear.png

// ❌ DON'T: Nomes genéricos ou hash
// assets/images/d33c457dbd2c043d8d3e-1A.jpg
// assets/images/image1.png
```

### **4. Otimização de Imagens**

#### **✅ DO: Otimização de Performance**

```tsx
// ✅ DO: Lazy loading e formatos modernos
<img
  src="/logo.webp"
  srcSet="/logo.webp 1x, /logo@2x.webp 2x"
  alt="Logo do Pegasus"
  title="Logo do Pegasus"
  loading="lazy"
  decoding="async"
/>

// ✅ DO: Picture element para responsividade
<picture>
  <source media="(max-width: 768px)" srcSet="/logo-mobile.webp" />
  <source media="(min-width: 769px)" srcSet="/logo-desktop.webp" />
  <img
    src="/logo.webp"
    alt="Logo do Pegasus"
    title="Logo do Pegasus"
    loading="lazy"
  />
</picture>
```

---

## 🔗 Otimização de Links

### **1. Title Attribute em Links**

Todos os links devem ter `title` attribute descritivo.

#### **✅ DO: Title em Links**

```tsx
// ✅ DO: Title attribute descritivo
<a
  href="/dashboard/settings"
  title="Acessar página de configurações do dashboard"
  aria-label="Ir para configurações"
>
  Configurações
</a>

// ✅ DO: Links externos com title
<a
  href="https://dilis.com.br"
  target="_blank"
  rel="noopener noreferrer"
  title="Abrir site oficial da Dilis em nova aba"
  aria-label="Site oficial da Dilis (nova aba)"
>
  Dilis Studio
</a>
```

#### **❌ DON'T: Links sem Title**

```tsx
// ❌ DON'T: Links sem title attribute
<a href="/dashboard/settings">Configurações</a>
<a href="/dashboard">Dashboard</a>
```

### **2. Links Internos vs Externos**

#### **✅ DO: Rel Attributes Corretos**

```tsx
// ✅ DO: Links internos
<a href="/dashboard" title="Ir para dashboard">Dashboard</a>

// ✅ DO: Links externos
<a
  href="https://dilis.com.br"
  target="_blank"
  rel="noopener noreferrer"
  title="Abrir site da Dilis em nova aba"
>
  Dilis
</a>
```

---

## 📐 Estrutura de Headers (H1-H6)

### **Hierarquia Semântica**

#### **✅ DO: Estrutura Hierárquica Correta**

```tsx
// ✅ DO: Hierarquia clara e semântica
export function SettingsPage() {
  return (
    <main>
      <h1>Configurações</h1>
      
      <section>
        <h2>Meu Perfil</h2>
        
        <div>
          <h3>Informações Pessoais</h3>
          <h4>Dados Básicos</h4>
        </div>
      </section>
      
      <section>
        <h2>Veículos</h2>
        <h3>Lista de Veículos Cadastrados</h3>
      </section>
    </main>
  );
}
```

#### **❌ DON'T: Pular Níveis ou Estrutura Incorreta**

```tsx
// ❌ DON'T: Pular níveis de header
<h1>Configurações</h1>
<h3>Perfil</h3> {/* Pulou H2 */}

// ❌ DON'T: Múltiplos H1 na mesma página
<h1>Configurações</h1>
<h1>Perfil</h1> {/* Deve ter apenas um H1 por página */}
```

### **Regras de Headers**

1. **Um único H1 por página** - Define o tópico principal
2. **H2 para seções principais** - Subsections do H1
3. **H3-H6 para subseções** - Hierarquia descendente
4. **Não pular níveis** - H1 → H2 → H3 (não H1 → H3)

---

## 🎯 Hook Unificado de SEO

### **✅ DO: Hook Completo para SEO**

```tsx
// ✅ DO: Hook unificado para todas as meta tags
interface SEOConfig {
  title: string;
  description: string;
  keywords?: string[];
  canonicalUrl?: string;
  robots?: string;
  author?: string;
  publisher?: string;
  og?: {
    title?: string;
    description?: string;
    image?: string;
    url?: string;
    type?: string;
    siteName?: string;
  };
  twitter?: {
    card?: 'summary' | 'summary_large_image';
    title?: string;
    description?: string;
    image?: string;
    site?: string;
    creator?: string;
  };
  schema?: {
    type: string;
    data: Record<string, unknown>;
  };
}

export function useSEO(config: SEOConfig) {
  const {
    title,
    description,
    keywords,
    canonicalUrl,
    robots = 'index, follow',
    author = 'Dilis Studio',
    publisher = 'Dilis Studio',
    og,
    twitter,
    schema,
  } = config;

  // Title
  useEffect(() => {
    document.title = title.includes('Pegasus') ? title : `${title} - Pegasus`;
  }, [title]);

  // Description
  useMetaDescription(description);

  // Keywords
  useEffect(() => {
    if (keywords && keywords.length > 0) {
      useMetaKeywords(keywords);
    }
  }, [keywords]);

  // Canonical
  useEffect(() => {
    if (canonicalUrl) {
      useCanonicalUrl(canonicalUrl);
    }
  }, [canonicalUrl]);

  // Robots
  useMetaRobots(robots);

  // Author
  useMetaAuthor(author);

  // Publisher
  useMetaPublisher(publisher);

  // Open Graph
  useEffect(() => {
    if (og) {
      useOpenGraph({
        title: og.title || title,
        description: og.description || description,
        image: og.image || '',
        url: og.url || canonicalUrl || window.location.href,
        type: og.type,
        siteName: og.siteName,
      });
    }
  }, [og, title, description, canonicalUrl]);

  // Twitter
  useEffect(() => {
    if (twitter) {
      useTwitterCard({
        card: twitter.card || 'summary_large_image',
        title: twitter.title || title,
        description: twitter.description || description,
        image: twitter.image || og?.image || '',
        site: twitter.site,
        creator: twitter.creator,
      });
    }
  }, [twitter, title, description, og]);

  // Schema.org
  useEffect(() => {
    if (schema) {
      useSchemaOrg(schema);
    }
  }, [schema]);
}

// Uso completo:
export function SettingsPage() {
  useSEO({
    title: 'Configurações',
    description: 'Gerencie suas configurações de perfil, equipe, veículos e canais no Pegasus Dashboard.',
    keywords: ['configurações', 'dashboard', 'perfil', 'equipe', 'veículos', 'pegasus'],
    canonicalUrl: 'https://pegasus.dilis.com.br/dashboard/settings',
    robots: 'index, follow',
    author: 'Dilis Studio',
    publisher: 'Dilis Studio',
    og: {
      title: 'Configurações - Pegasus Dashboard',
      description: 'Gerencie suas configurações no Pegasus Dashboard',
      image: 'https://pegasus.dilis.com.br/og-settings.png',
      url: 'https://pegasus.dilis.com.br/dashboard/settings',
      type: 'website',
      siteName: 'Pegasus',
    },
    twitter: {
      card: 'summary_large_image',
      title: 'Configurações - Pegasus Dashboard',
      description: 'Gerencie suas configurações no Pegasus Dashboard',
      image: 'https://pegasus.dilis.com.br/twitter-settings.png',
    },
    schema: {
      type: 'WebPage',
      data: {
        name: 'Configurações - Pegasus Dashboard',
        description: 'Gerencie suas configurações no Pegasus Dashboard',
        url: 'https://pegasus.dilis.com.br/dashboard/settings',
      },
    },
  });

  return <div>...</div>;
}
```

---

## ⚡ JavaScript SEO e Aplicações SPA

Para aplicações React/SPA, é crucial garantir que o conteúdo seja acessível aos crawlers do Google.

### **✅ DO: Renderização para Crawlers**

```tsx
// ✅ DO: Usar History API corretamente
import { useNavigate } from 'react-router-dom';

export function ProductPage() {
  const navigate = useNavigate();
  
  const handleNavigation = () => {
    // Usar History API do React Router
    navigate('/products/123');
    // Não usar window.location.hash para navegação principal
  };
  
  return <div>...</div>;
}

// ✅ DO: Evitar soft 404s
export function ProductPage({ productId }: { productId: string }) {
  const { data, isLoading, error } = useProduct(productId);
  
  useEffect(() => {
    if (error?.status === 404) {
      // Adicionar noindex para páginas não encontradas
      const metaRobots = document.querySelector('meta[name="robots"]') || 
        document.createElement('meta');
      metaRobots.setAttribute('name', 'robots');
      metaRobots.setAttribute('content', 'noindex');
      document.head.appendChild(metaRobots);
    }
  }, [error]);
  
  if (error?.status === 404) {
    return <NotFoundPage />;
  }
  
  return <div>...</div>;
}

// ✅ DO: JSON-LD para structured data (não depende de JS)
export function useProductSchema(product: Product) {
  useEffect(() => {
    const script = document.createElement('script');
    script.type = 'application/ld+json';
    script.textContent = JSON.stringify({
      '@context': 'https://schema.org',
      '@type': 'Product',
      name: product.name,
      description: product.description,
      image: product.image,
    });
    document.head.appendChild(script);
    
    return () => {
      document.head.removeChild(script);
    };
  }, [product]);
}
```

#### **❌ DON'T: Navegação Apenas com JavaScript**

```tsx
// ❌ DON'T: Links apenas com JavaScript
<button onClick={() => navigate('/products')}>Ver Produtos</button>

// ✅ DO: Usar <a> tags para links importantes
<a href="/products" title="Ver lista de produtos">Ver Produtos</a>
```

---

## 🎯 Core Web Vitals e Performance

### **Métricas Essenciais**

Core Web Vitals são métricas de performance que afetam diretamente o ranking do Google.

#### **✅ DO: Otimizar Core Web Vitals**

```tsx
// ✅ DO: Lazy loading de imagens
<img
  src="/hero-image.webp"
  alt="Hero image"
  title="Hero image"
  loading="lazy"
  decoding="async"
/>

// ✅ DO: Minimizar JavaScript crítico
// Usar code splitting e lazy loading de componentes
const HeavyComponent = lazy(() => import('./HeavyComponent'));

export function Page() {
  return (
    <Suspense fallback={<Skeleton />}>
      <HeavyComponent />
    </Suspense>
  );
}

// ✅ DO: Otimizar LCP (Largest Contentful Paint)
// - Usar formatos modernos de imagem (WebP, AVIF)
// - Priorizar carregamento de recursos críticos
// - Minimizar render-blocking resources
```

#### **Métricas Alvo**

- **LCP (Largest Contentful Paint)**: < 2.5 segundos
- **FID (First Input Delay)**: < 100 milissegundos
- **CLS (Cumulative Layout Shift)**: < 0.1

---

## 👤 E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness)

### **Importância para Qualidade**

O Google avalia a qualidade do conteúdo baseado em E-E-A-T, especialmente importante para conteúdo YMYL.

#### **✅ DO: Demonstrar E-E-A-T**

```tsx
// ✅ DO: Schema.org com informações do autor
export function ArticlePage() {
  useSchemaOrg({
    type: 'Article',
    data: {
      headline: 'Título do Artigo',
      author: {
        '@type': 'Person',
        name: 'Nome do Autor',
        jobTitle: 'Cargo/Posição',
        url: 'https://pegasus.dilis.com.br/authors/author-name',
      },
      publisher: {
        '@type': 'Organization',
        name: 'Dilis Studio',
        logo: {
          '@type': 'ImageObject',
          url: 'https://pegasus.dilis.com.br/logo.png',
        },
      },
      datePublished: '2024-01-15',
      dateModified: '2024-01-20',
    },
  });
  
  return <article>...</article>;
}

// ✅ DO: Informações sobre o autor na página
export function ArticleContent({ author }: { author: Author }) {
  return (
    <article>
      <div className="author-info">
        <img src={author.avatar} alt={author.name} title={author.name} />
        <div>
          <h3>{author.name}</h3>
          <p>{author.bio}</p>
          <p>Experiência: {author.experience} anos</p>
        </div>
      </div>
      {/* Conteúdo do artigo */}
    </article>
  );
}
```

---

## 🏥 YMYL (Your Money or Your Life)

### **Conteúdo que Requer Alta Confiança**

Páginas YMYL (saúde, finanças, segurança) precisam de E-E-A-T muito alto.

#### **✅ DO: Schema.org para YMYL**

```tsx
// ✅ DO: MedicalWebPage para conteúdo médico
export function MedicalPage() {
  useSchemaOrg({
    type: 'MedicalWebPage',
    data: {
      name: 'Informações sobre Saúde',
      author: {
        '@type': 'MedicalPerson',
        name: 'Dr. João Silva',
        medicalSpecialty: 'Cardiologia',
        credential: 'CRM 12345',
      },
      publisher: {
        '@type': 'MedicalOrganization',
        name: 'Hospital Exemplo',
      },
      datePublished: '2024-01-15',
    },
  });
  
  return <div>...</div>;
}

// ✅ DO: FinancialProduct para conteúdo financeiro
export function FinancialProductPage() {
  useSchemaOrg({
    type: 'FinancialProduct',
    data: {
      name: 'Produto Financeiro',
      provider: {
        '@type': 'FinancialService',
        name: 'Banco Exemplo',
      },
    },
  });
  
  return <div>...</div>;
}
```

---

## 📊 Structured Data Específicos

### **BreadcrumbList**

#### **✅ DO: Breadcrumbs com Structured Data**

```tsx
// ✅ DO: BreadcrumbList Schema.org
export function useBreadcrumbSchema(breadcrumbs: Breadcrumb[]) {
  useEffect(() => {
    const script = document.createElement('script');
    script.type = 'application/ld+json';
    script.textContent = JSON.stringify({
      '@context': 'https://schema.org',
      '@type': 'BreadcrumbList',
      itemListElement: breadcrumbs.map((crumb, index) => ({
        '@type': 'ListItem',
        position: index + 1,
        name: crumb.name,
        item: crumb.url,
      })),
    });
    document.head.appendChild(script);
    
    return () => {
      document.head.removeChild(script);
    };
  }, [breadcrumbs]);
}
```

### **VideoObject**

#### **✅ DO: Vídeo com Structured Data**

```tsx
// ✅ DO: VideoObject Schema.org
export function VideoPage({ video }: { video: Video }) {
  useSchemaOrg({
    type: 'VideoObject',
    data: {
      name: video.title,
      description: video.description,
      thumbnailUrl: video.thumbnail,
      uploadDate: video.uploadDate,
      contentUrl: video.url,
      duration: video.duration, // ISO 8601 format: PT1M30S
      embedUrl: video.embedUrl,
    },
  });
  
  return (
    <div>
      {/* ✅ DO: Usar tags HTML padrão para vídeo */}
      <video controls>
        <source src={video.url} type="video/mp4" />
      </video>
    </div>
  );
}
```

### **Product**

#### **✅ DO: Product Schema.org**

```tsx
// ✅ DO: Product Schema.org para e-commerce
export function ProductPage({ product }: { product: Product }) {
  useSchemaOrg({
    type: 'Product',
    data: {
      name: product.name,
      description: product.description,
      image: product.images,
      brand: {
        '@type': 'Brand',
        name: product.brand,
      },
      offers: {
        '@type': 'Offer',
        price: product.price,
        priceCurrency: 'BRL',
        availability: product.inStock ? 'https://schema.org/InStock' : 'https://schema.org/OutOfStock',
      },
    },
  });
  
  return <div>...</div>;
}
```

---

## 🔧 Meta Tags Específicas do Google

### **✅ DO: Tags Especiais**

```tsx
// ✅ DO: data-nosnippet - Excluir elementos do snippet
<div data-nosnippet>
  <p>Este conteúdo não aparecerá no snippet do Google</p>
</div>

// ✅ DO: max-video-preview - Controlar preview de vídeos
useEffect(() => {
  const meta = document.createElement('meta');
  meta.setAttribute('name', 'robots');
  meta.setAttribute('content', 'max-video-preview:10'); // 10 segundos
  document.head.appendChild(meta);
}, []);

// ✅ DO: google-site-verification
useEffect(() => {
  const meta = document.createElement('meta');
  meta.setAttribute('name', 'google-site-verification');
  meta.setAttribute('content', 'your_verification_code');
  document.head.appendChild(meta);
}, []);

// ✅ DO: Prevenir tradução automática
useEffect(() => {
  const meta = document.createElement('meta');
  meta.setAttribute('name', 'google');
  meta.setAttribute('content', 'notranslate');
  document.head.appendChild(meta);
}, []);

// ✅ DO: Desabilitar text-to-speech
useEffect(() => {
  const meta = document.createElement('meta');
  meta.setAttribute('name', 'google');
  meta.setAttribute('content', 'nopagereadaloud');
  document.head.appendChild(meta);
}, []);

// ✅ DO: Marcar conteúdo adulto (se aplicável)
useEffect(() => {
  const meta = document.createElement('meta');
  meta.setAttribute('name', 'rating');
  meta.setAttribute('content', 'adult');
  // ou
  // meta.setAttribute('content', 'RTA-5042-1996-1400-1577-RTA');
  document.head.appendChild(meta);
}, []);
```

---

## 📱 Mobile-First Indexing

### **✅ DO: Otimização Mobile**

```tsx
// ✅ DO: Títulos e descriptions idênticos em mobile e desktop
// Garantir que mobile e desktop tenham o mesmo conteúdo meta

// ✅ DO: Meta tags específicas para Googlebot-Mobile
useEffect(() => {
  const meta = document.createElement('meta');
  meta.setAttribute('name', 'googlebot-mobile');
  meta.setAttribute('content', 'index, follow');
  document.head.appendChild(meta);
}, []);

// ✅ DO: Viewport correto
// index.html deve ter:
// <meta name="viewport" content="width=device-width, initial-scale=1.0" />

// ✅ DO: Imagens otimizadas para mobile
<picture>
  <source media="(max-width: 768px)" srcSet="/image-mobile.webp" />
  <source media="(min-width: 769px)" srcSet="/image-desktop.webp" />
  <img src="/image.webp" alt="Descrição" title="Descrição" />
</picture>
```

---

## 🔗 Paginação e Navegação Facetada

### **✅ DO: Paginação Otimizada**

```tsx
// ✅ DO: Cada página paginada com seu próprio canonical
export function PaginatedList({ page, totalPages }: Props) {
  const canonicalUrl = `https://pegasus.dilis.com.br/products?page=${page}`;
  
  useEffect(() => {
    useCanonicalUrl(canonicalUrl);
  }, [canonicalUrl]);
  
  return (
    <div>
      {/* ✅ DO: Links sequenciais com <a> tags */}
      {page > 1 && (
        <a href={`/products?page=${page - 1}`} title="Página anterior">
          Anterior
        </a>
      )}
      {page < totalPages && (
        <a href={`/products?page=${page + 1}`} title="Próxima página">
          Próxima
        </a>
      )}
    </div>
  );
}

// ✅ DO: URL parameters padrão (usar & como separador)
// ✅ DO: Retornar 404 para combinações de filtros inválidas
// ✅ DO: Usar noindex para URLs filtradas/ordenadas duplicadas
export function FilteredProducts({ filters }: { filters: Filters }) {
  useEffect(() => {
    // Se for uma URL duplicada (mesmo conteúdo com filtros diferentes)
    if (isDuplicateFilterUrl(filters)) {
      const meta = document.createElement('meta');
      meta.setAttribute('name', 'robots');
      meta.setAttribute('content', 'noindex');
      document.head.appendChild(meta);
    }
  }, [filters]);
  
  return <div>...</div>;
}
```

---

## 🎥 Otimização de Vídeos

### **✅ DO: Vídeos Otimizados**

```tsx
// ✅ DO: Usar tags HTML padrão
<video controls>
  <source src="/video.mp4" type="video/mp4" />
  Seu navegador não suporta vídeo.
</video>

// ✅ DO: Open Graph para vídeos
export function useVideoOpenGraph(video: Video) {
  useEffect(() => {
    const ogTags = [
      { property: 'og:type', content: 'video.other' },
      { property: 'og:video', content: video.url },
      { property: 'og:video:width', content: '1280' },
      { property: 'og:video:height', content: '720' },
    ];
    
    ogTags.forEach(({ property, content }) => {
      let meta = document.querySelector(`meta[property="${property}"]`);
      if (!meta) {
        meta = document.createElement('meta');
        meta.setAttribute('property', property);
        document.head.appendChild(meta);
      }
      meta.setAttribute('content', content);
    });
  }, [video]);
}

// ❌ DON'T: Evitar fragment identifiers (#) para carregamento de vídeo
// ❌ DON'T: Não usar CSS background-image para imagens importantes
```

---

## 🖼️ Otimização Avançada de Imagens

### **✅ DO: Técnicas Modernas**

```tsx
// ✅ DO: Evitar CSS background-image para imagens importantes
// ❌ DON'T: background-image não é indexado
// <div style={{ backgroundImage: 'url(/image.jpg)' }} />

// ✅ DO: Usar <img> tag
<img src="/image.jpg" alt="Descrição" title="Descrição" />

// ✅ DO: Suporte a SVG (mas não imagem JPG dentro de SVG inline)
<img src="/logo.svg" alt="Logo" title="Logo" />

// ❌ DON'T: SVG inline com imagem JPG dentro
// <svg>
//   <image href="image.jpg" /> {/* Não é indexado */}
// </svg>

// ✅ DO: Picture element para responsividade
<picture>
  <source media="(max-width: 768px)" srcSet="/image-mobile.webp" />
  <source media="(min-width: 769px)" srcSet="/image-desktop.webp" />
  <img src="/image.webp" alt="Descrição" title="Descrição" loading="lazy" />
</picture>

// ✅ DO: srcSet para diferentes densidades
<img
  src="/logo.webp"
  srcSet="/logo.webp 1x, /logo@2x.webp 2x"
  alt="Logo"
  title="Logo"
  loading="lazy"
/>
```

---

## 🔗 Links Avançados e Navegação

### **✅ DO: Links Otimizados**

```tsx
// ✅ DO: Usar <a> tags (não JavaScript navigation) para links importantes
<a href="/products" title="Ver lista de produtos">
  Ver Produtos
</a>

// ✅ DO: Anchor text descritivo e contextual
<a href="/products/widgets" title="Ver widgets disponíveis">
  Widgets Disponíveis
</a>

// ❌ DON'T: Evitar anchor text genérico
<a href="/products">Clique aqui</a>
<a href="/products">Leia mais</a>
<a href="/products">Ver artigo</a>

// ✅ DO: Links internos para estrutura do site
<nav>
  <a href="/" title="Página inicial">Home</a>
  <a href="/products" title="Produtos">Produtos</a>
  <a href="/about" title="Sobre nós">Sobre</a>
</nav>

// ✅ DO: Breadcrumbs implementados
<nav aria-label="breadcrumb">
  <ol>
    <li><a href="/" title="Home">Home</a></li>
    <li><a href="/products" title="Produtos">Produtos</a></li>
    <li>Widget Azul</li>
  </ol>
</nav>
```

---

## 🤖 Robots e Crawling

### **✅ DO: Controle de Crawling**

```tsx
// ✅ DO: Robots meta tag dinâmico via JavaScript (quando necessário)
export function DynamicPage({ shouldIndex }: { shouldIndex: boolean }) {
  useEffect(() => {
    let metaRobots = document.querySelector('meta[name="robots"]');
    
    if (!metaRobots) {
      metaRobots = document.createElement('meta');
      metaRobots.setAttribute('name', 'robots');
      document.head.appendChild(metaRobots);
    }
    
    metaRobots.setAttribute('content', shouldIndex ? 'index, follow' : 'noindex');
  }, [shouldIndex]);
  
  return <div>...</div>;
}

// ✅ DO: Evitar URLs com fragmentos (#) para indexação
// ❌ DON'T: /products#filter=price
// ✅ DO: /products?filter=price
```

---

## ⚠️ SafeSearch e Conteúdo Adulto

### **✅ DO: Marcação de Conteúdo Adulto**

```tsx
// ✅ DO: Meta tag para conteúdo adulto
export function useAdultContentTag() {
  useEffect(() => {
    const meta = document.createElement('meta');
    meta.setAttribute('name', 'rating');
    meta.setAttribute('content', 'adult');
    // ou para RTA
    // meta.setAttribute('content', 'RTA-5042-1996-1400-1577-RTA');
    document.head.appendChild(meta);
  }, []);
}
```

---

## 📋 Checklist de SEO por Página

### **Meta Tags Essenciais**

- [ ] Title tag (50-60 caracteres, único por página)
- [ ] Meta description (120-160 caracteres)
- [ ] Meta keywords (relevantes ao conteúdo)
- [ ] Canonical URL (evita conteúdo duplicado)
- [ ] Meta robots (index, follow ou noindex, nofollow)
- [ ] Meta author
- [ ] Meta publisher

### **Open Graph e Social**

- [ ] og:title
- [ ] og:description
- [ ] og:image (URL completa)
- [ ] og:url
- [ ] og:type
- [ ] og:site_name
- [ ] og:locale (pt_BR)
- [ ] twitter:card
- [ ] twitter:title
- [ ] twitter:description
- [ ] twitter:image

### **Schema.org**

- [ ] Schema.org markup (JSON-LD)
- [ ] itemtype definido
- [ ] Dados estruturados validados

### **Imagens**

- [ ] Todas as imagens têm alt text
- [ ] Todas as imagens têm title attribute
- [ ] Nomes de arquivo descritivos
- [ ] Imagens otimizadas (WebP, lazy loading)

### **Links**

- [ ] Todos os links têm title attribute
- [ ] Links externos têm rel="noopener noreferrer"
- [ ] Links internos corretos

### **Headers**

- [ ] Um único H1 por página
- [ ] Hierarquia H1 → H2 → H3 → H4 (sem pular níveis)
- [ ] Headers descritivos e semânticos

### **HTML Base**

- [ ] Lang correto no `<html>` tag (pt-BR)
- [ ] Charset UTF-8
- [ ] Viewport meta tag

### **JavaScript SEO**

- [ ] Conteúdo crítico renderizado no HTML inicial
- [ ] History API usado corretamente (não hash navigation)
- [ ] Sem soft 404s (páginas não encontradas retornam 404 real)
- [ ] Structured data em JSON-LD (não depende de JS)
- [ ] Links importantes usam `<a>` tags (não apenas JavaScript)

### **Core Web Vitals**

- [ ] LCP < 2.5 segundos
- [ ] FID < 100 milissegundos
- [ ] CLS < 0.1
- [ ] Imagens otimizadas (WebP, lazy loading)
- [ ] JavaScript crítico minimizado

### **E-E-A-T**

- [ ] Informações do autor/criador presentes
- [ ] Credenciais e experiência demonstradas
- [ ] Informações sobre a organização
- [ ] Data de publicação e atualização
- [ ] Schema.org com author/publisher

### **YMYL (se aplicável)**

- [ ] Conteúdo YMYL identificado
- [ ] E-E-A-T muito alto para YMYL
- [ ] Schema.org específico (MedicalWebPage, FinancialProduct, etc.)
- [ ] Credenciais verificáveis

### **Structured Data**

- [ ] BreadcrumbList implementado
- [ ] VideoObject (se houver vídeos)
- [ ] Product (se e-commerce)
- [ ] Article (se conteúdo editorial)
- [ ] Organization (informações da empresa)

### **Meta Tags Especiais**

- [ ] data-nosnippet (se necessário)
- [ ] max-video-preview (se houver vídeos)
- [ ] google-site-verification
- [ ] google: notranslate (se necessário)
- [ ] rating: adult (se conteúdo adulto)

### **Mobile-First**

- [ ] Títulos idênticos mobile/desktop
- [ ] Descriptions idênticas mobile/desktop
- [ ] Meta tags para Googlebot-Mobile
- [ ] Imagens otimizadas para mobile

### **Paginação**

- [ ] Cada página com seu próprio canonical
- [ ] Links sequenciais com `<a>` tags
- [ ] URL parameters padrão (`&`)
- [ ] 404 para filtros inválidos
- [ ] noindex para URLs duplicadas

---

## 🔍 Validação e Ferramentas

### **Ferramentas Recomendadas**

1. **Google Search Console** - Monitoramento de indexação
2. **Google Rich Results Test** - Validação de Schema.org
3. **Facebook Sharing Debugger** - Validação de Open Graph
4. **Twitter Card Validator** - Validação de Twitter Cards
5. **Lighthouse** - Auditoria completa de SEO
6. **SEO Extension** - Validação em tempo real

### **✅ DO: Validação Contínua**

```tsx
// ✅ DO: Validar SEO em desenvolvimento
if (import.meta.env.DEV) {
  // Log de validação de SEO
  console.log('[SEO] Validando meta tags...');
  console.log('[SEO] Title:', document.title);
  console.log('[SEO] Description:', document.querySelector('meta[name="description"]')?.getAttribute('content'));
  console.log('[SEO] Canonical:', document.querySelector('link[rel="canonical"]')?.getAttribute('href'));
}
```

---

## 📚 Referências

- [Google Search Central](https://developers.google.com/search)
- [Open Graph Protocol](https://ogp.me/)
- [Twitter Cards](https://developer.twitter.com/en/docs/twitter-for-websites/cards)
- [Schema.org](https://schema.org/)
- [MDN - SEO Basics](https://developer.mozilla.org/en-US/docs/Glossary/SEO)
- [Web.dev - SEO](https://web.dev/learn/seo/)

---

**Última atualização**: 2024  
**Versão**: 1.0.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tonybsilva-dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Tonybsilva-dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
