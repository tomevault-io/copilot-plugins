## html-structure-hydration

> - **NEVER** put block-level elements inside inline elements

# HTML Structure & Hydration Error Prevention (MANDATORY)

## 🚨 **Critical HTML Structure Rules (MUST FOLLOW)**

### **1. Block vs Inline Element Rules**

- **NEVER** put block-level elements inside inline elements
- **NEVER** put `<div>`, `<section>`, `<article>` inside `<p>`, `<span>`, `<a>`
- **ALWAYS** ensure proper HTML nesting hierarchy

```typescript
// ✅ CORRECT - Proper HTML structure
<span className="block">
  <div className="relative">
    <Image src={src} alt={alt} />
  </div>
</span>

// ❌ WRONG - Block element inside inline element
<p>
  <div>  {/* This causes hydration error */}
    <Image src={src} alt={alt} />
  </div>
</p>
```

### **2. Markdown Rendering HTML Structure**

- **ALWAYS** handle images as block-level elements in markdown
- **NEVER** let markdown parser wrap images in `<p>` tags
- **ALWAYS** use custom components to override default markdown behavior

```typescript
// ✅ CORRECT - Custom image rendering that prevents p tag wrapping
components={{
  img: ({ src, alt, ...props }) => {
    return (
      <span className="block my-4">
        <Image src={src} alt={alt} {...props} />
      </span>
    );
  },
  // Override p tag rendering for images
  p: ({ children, ...props }) => {
    const hasOnlyImage = React.Children.count(children) === 1 && 
      React.isValidElement(children) && 
      children.type === 'img';
    
    if (hasOnlyImage) {
      return <div {...props}>{children}</div>; // Use div instead of p
    }
    return <p {...props}>{children}</p>;
  }
}}
```

## 🔧 **Hydration Error Prevention (CRITICAL)**

### **1. Server/Client Component Consistency**

- **ALWAYS** ensure server and client render identical HTML
- **NEVER** use browser-only APIs in server components
- **ALWAYS** handle dynamic content with proper client boundaries

```typescript
// ✅ CORRECT - Proper client boundary for dynamic content
'use client';

export const DynamicImage = ({ src, alt }: { src: string; alt: string }) => {
  const [isLoaded, setIsLoaded] = useState(false);
  
  return (
    <div className="relative">
      <Image 
        src={src} 
        alt={alt}
        onLoad={() => setIsLoaded(true)}
      />
      {!isLoaded && <div className="loading-placeholder" />}
    </div>
  );
};

// ❌ WRONG - Browser API in server component
export const ServerComponent = () => {
  const [windowSize, setWindowSize] = useState({}); // Hydration error!
  return <div>{windowSize.width}</div>;
};
```

### **2. Conditional Rendering Best Practices**

- **ALWAYS** use consistent conditional rendering patterns
- **NEVER** mix server and client conditional logic
- **ALWAYS** handle loading states properly

```typescript
// ✅ CORRECT - Consistent conditional rendering
export const PostCard = ({ post, showThumbnail = true }) => {
  return (
    <Card>
      {showThumbnail && post.thumbnail_url ? (
        <div className="thumbnail-container">
          <Image src={post.thumbnail_url} alt={post.title} />
        </div>
      ) : showThumbnail && !post.thumbnail_url ? (
        <div className="placeholder-container">
          <div className="placeholder-image" />
        </div>
      ) : null}
    </Card>
  );
};

// ❌ WRONG - Inconsistent conditional rendering
export const BadPostCard = ({ post }) => {
  return (
    <Card>
      {post.thumbnail_url && (
        <div className="thumbnail">
          <Image src={post.thumbnail_url} alt={post.title} />
        </div>
      )}
      {/* Missing else case - can cause hydration mismatch */}
    </Card>
  );
};
```

## 🎯 **Component Structure Guidelines**

### **1. Image Component Best Practices**

- **ALWAYS** use Next.js Image component for optimization
- **ALWAYS** provide proper width and height props
- **NEVER** use onError/onLoad with Next.js Image (not supported)
- **ALWAYS** handle image loading states with CSS or state

```typescript
// ✅ CORRECT - Next.js Image with proper structure
export const OptimizedImage = ({ src, alt, className }: ImageProps) => {
  return (
    <div className="image-container">
      <Image
        src={src}
        alt={alt}
        width={800}
        height={600}
        className={className}
        style={{
          display: 'block',
          maxWidth: '100%',
          height: 'auto',
        }}
      />
    </div>
  );
};

// ❌ WRONG - Unsupported events with Next.js Image
<Image
  src={src}
  alt={alt}
  onError={handleError}  // Not supported!
  onLoad={handleLoad}    // Not supported!
/>
```

### **2. Markdown Component Structure**

- **ALWAYS** override default markdown component behavior
- **ALWAYS** prevent invalid HTML nesting
- **ALWAYS** use semantic HTML elements appropriately

```typescript
// ✅ CORRECT - Safe markdown rendering
export const SafeMarkdownRenderer = ({ content }: { content: string }) => {
  return (
    <ReactMarkdown
      components={{
        // Prevent p tag wrapping for images
        img: ({ src, alt }) => (
          <span className="block my-4">
            <Image src={src} alt={alt} width={800} height={600} />
          </span>
        ),
        // Override p tag for image-only content
        p: ({ children, ...props }) => {
          const hasImage = React.Children.toArray(children).some(
            child => React.isValidElement(child) && child.type === 'img'
          );
          return hasImage ? <div {...props}>{children}</div> : <p {...props}>{children}</p>;
        }
      }}
    >
      {content}
    </ReactMarkdown>
  );
};
```

## 🚫 **Forbidden HTML Patterns**

### **1. Invalid Nesting (NEVER DO)**

```typescript
// ❌ NEVER - Block inside inline
<p><div>Content</div></p>
<span><section>Content</section></span>
<a><article>Content</article></a>

// ❌ NEVER - Complex nesting in simple elements
<p>
  <div>
    <span>
      <div>Content</div>  {/* Multiple block levels */}
    </span>
  </div>
</p>
```

### **2. Hydration Mismatch Patterns (NEVER DO)**

```typescript
// ❌ NEVER - Different server/client rendering
export const BadComponent = () => {
  const [isClient, setIsClient] = useState(false);
  
  useEffect(() => setIsClient(true), []);
  
  return (
    <div>
      {isClient ? <ClientOnlyContent /> : <ServerContent />} {/* Hydration mismatch! */}
    </div>
  );
};

// ❌ NEVER - Conditional rendering based on environment
export const EnvironmentComponent = () => {
  if (typeof window !== 'undefined') {
    return <ClientContent />; // Different from server render
  }
  return <ServerContent />;
};
```

## ✅ **Required HTML Structure Practices**

### **1. Always Use Semantic HTML**

- **ALWAYS** use `<article>` for blog posts
- **ALWAYS** use `<section>` for content sections
- **ALWAYS** use `<header>`, `<footer>` appropriately
- **ALWAYS** use proper heading hierarchy (`<h1>` to `<h6>`)

### **2. Always Handle Dynamic Content Safely**

- **ALWAYS** use `useEffect` for client-side only logic
- **ALWAYS** provide fallback content for loading states
- **ALWAYS** ensure consistent initial render between server and client

### **3. Always Validate HTML Structure**

- **ALWAYS** check HTML nesting in custom components
- **ALWAYS** test markdown rendering with various content types
- **ALWAYS** verify hydration consistency in development

## 🔍 **Debugging Checklist**

When encountering hydration errors, check:

1. **HTML Structure**: Are block elements nested inside inline elements?
2. **Server/Client Consistency**: Does server render match client render?
3. **Conditional Rendering**: Are conditional patterns consistent?
4. **Dynamic Content**: Is client-side logic properly isolated?
5. **Markdown Rendering**: Are custom components overriding defaults correctly?
6. **Image Components**: Are Next.js Image components used properly?

## 📝 **Code Review Checklist**

- [ ] No block elements inside inline elements
- [ ] Markdown components properly override defaults
- [ ] Server and client render identical HTML
- [ ] Dynamic content uses proper client boundaries
- [ ] Image components follow Next.js best practices
- [ ] Conditional rendering is consistent
- [ ] HTML structure is semantically correct
- [ ] No hydration mismatches in development

## 🎯 **Key Takeaways**

1. **HTML nesting rules are critical** - never put `<div>` inside `<p>`
2. **Markdown rendering needs custom overrides** - prevent default p tag wrapping
3. **Server/client consistency is mandatory** - ensure identical initial render
4. **Next.js Image has limitations** - don't use unsupported events
5. **Semantic HTML improves accessibility** - use appropriate elements
6. **Conditional rendering must be consistent** - avoid hydration mismatches

description: HTML 구조와 hydration 에러 방지를 위한 필수 규칙
globs: ["**/*.tsx", "**/*.ts", "**/*.jsx", "**/*.js"]
alwaysApply: true

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
