## site-eduardo

> Regras para integração com Contentful CMS


# Regras Contentful — CMS do Blog

## Status Atual

⚠️ **ATENÇÃO:** Ver TBD-001 em DECISIONS_TBD.md

O Contentful está configurado mas pode ter problemas com variáveis de ambiente (usa `REACT_APP_*` em vez de `VITE_*`).

## Arquivos Envolvidos

```
src/lib/contentful.ts      # Cliente e serviços Contentful
src/pages/BlogPage.tsx     # Lista de posts
src/pages/BlogPostPage.tsx # Post individual
```

## Configuração

### Variáveis de ambiente necessárias:

```bash
# .env.local
REACT_APP_CONTENTFUL_SPACE_ID=xxx       # ⚠️ Deveria ser VITE_*
REACT_APP_CONTENTFUL_ACCESS_TOKEN=xxx   # ⚠️ Deveria ser VITE_*
REACT_APP_CONTENTFUL_PREVIEW_TOKEN=xxx  # ⚠️ Deveria ser VITE_*
```

### Se corrigir para Vite:

```typescript
// src/lib/contentful.ts
const spaceId = import.meta.env.VITE_CONTENTFUL_SPACE_ID;
const accessToken = import.meta.env.VITE_CONTENTFUL_ACCESS_TOKEN;
```

## Content Types

### blogPost:

```typescript
interface BlogPostFields {
  title: string;
  slug: string;
  description: string;
  content: RichText;
  featuredImage: Asset;
  author: string;
  category: string;
  tags: string[];
  publishedDate: string;
  readingTime: number;
  featured: boolean;
}
```

### category:

```typescript
interface CategoryFields {
  name: string;
  slug: string;
  description?: string;
  color?: string;
}
```

## Serviços Disponíveis

### contentfulService (singleton):

```typescript
import { contentfulService } from '../lib/contentful';

// Buscar posts
const posts = await contentfulService.getBlogPosts({
  limit: 10,
  category: 'investimentos',
  featured: true
});

// Buscar post por slug
const post = await contentfulService.getBlogPostBySlug('meu-post');

// Posts relacionados
const related = await contentfulService.getRelatedPosts(postId, category);

// Categorias
const categories = await contentfulService.getCategories();

// Busca
const results = await contentfulService.searchPosts('termo');

// Rich text para HTML
const html = contentfulService.richTextToHtml(post.content);

// Otimizar imagem
const imageUrl = contentfulService.getOptimizedImageUrl(asset, {
  width: 800,
  format: 'webp'
});
```

## Uso em Componentes

### Buscar posts:

```typescript
const [posts, setPosts] = useState<any[]>([]);
const [loading, setLoading] = useState(true);

useEffect(() => {
  async function fetchPosts() {
    try {
      const data = await contentfulService.getBlogPosts({ limit: 10 });
      setPosts(data);
    } catch (error) {
      console.error('Erro ao buscar posts:', error);
    } finally {
      setLoading(false);
    }
  }
  fetchPosts();
}, []);
```

### Renderizar rich text:

```tsx
<div 
  className="prose"
  dangerouslySetInnerHTML={{ 
    __html: contentfulService.richTextToHtml(post.fields.content) 
  }} 
/>
```

## Otimização de Imagens

### Parâmetros disponíveis:

```typescript
contentfulService.getOptimizedImageUrl(asset, {
  width: 800,      // Largura em pixels
  height: 600,     // Altura em pixels
  quality: 80,     // Qualidade (1-100)
  format: 'webp'   // Formato (jpg, png, webp)
});
```

### Padrão: WebP automático

## Limitações

- Não há cache implementado
- Preview client requer token separado
- Rich text com imagens pode ter URLs relativas

## Se Contentful Não Estiver em Uso

Se decidido em TBD-001 que Contentful não está em uso:

1. Remover `src/lib/contentful.ts`
2. Atualizar páginas do blog para usar outra fonte
3. Remover dependências do package.json:
   - `@contentful/rich-text-html-renderer`
   - `@contentful/rich-text-react-renderer`
   - `@contentful/rich-text-types`
   - `contentful`

## Proibições

- ❌ Hardcoded Space ID ou tokens
- ❌ Usar preview client em produção sem intenção
- ❌ Ignorar erros de fetch (sempre try/catch)
- ❌ Não otimizar imagens (sempre usar getOptimizedImageUrl)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EduardoSousaPO)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EduardoSousaPO)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
