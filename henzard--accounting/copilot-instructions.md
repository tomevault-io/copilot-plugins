## accounting

> **⚠️ CRITICAL**: React Native's `Alert.alert()` does NOT work on web. It silently fails.

---
globs:
  - "src/**/*.web.*"
  - "**/*.web.tsx"
alwaysApply: false
---

# Web-Specific Rules

## Alert Compatibility

**⚠️ CRITICAL**: React Native's `Alert.alert()` does NOT work on web. It silently fails.

**Solution**: Always use `showAlert` or `showConfirm` from `@/shared/utils/alert` instead of `Alert.alert` directly.

See [Rule 35: Web Alert Compatibility](35-web-alert-compatibility.mdc) for complete details.

---
 & Best Practices

## Responsive Design

### Mobile-First Approach
```typescript
// src/shared/constants/breakpoints.ts
export const BREAKPOINTS = {
  mobile: 0,       // 0-767px
  tablet: 768,     // 768-1023px
  desktop: 1024,   // 1024-1439px
  wide: 1440,      // 1440px+
} as const;

export function useMediaQuery() {
  const [breakpoint, setBreakpoint] = useState(getBreakpoint());

  useEffect(() => {
    const handleResize = () => setBreakpoint(getBreakpoint());
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return {
    breakpoint,
    isMobile: breakpoint === 'mobile',
    isTablet: breakpoint === 'tablet',
    isDesktop: breakpoint === 'desktop' || breakpoint === 'wide',
  };
}

function getBreakpoint() {
  const width = window.innerWidth;
  if (width >= BREAKPOINTS.wide) return 'wide';
  if (width >= BREAKPOINTS.desktop) return 'desktop';
  if (width >= BREAKPOINTS.tablet) return 'tablet';
  return 'mobile';
}
```

### Responsive Components
```typescript
export const AnimalGrid: React.FC = () => {
  const { breakpoint } = useMediaQuery();
  
  const columns = {
    mobile: 1,
    tablet: 2,
    desktop: 3,
    wide: 4,
  }[breakpoint];

  return (
    <FlatList
      data={animals}
      key={columns} // Force re-render on column change
      numColumns={columns}
      renderItem={({ item }) => <AnimalCard animal={item} />}
    />
  );
};
```

### Fluid Typography
```typescript
// Use viewport units for responsive text
const styles = StyleSheet.create({
  heading: {
    fontSize: 'clamp(24px, 5vw, 48px)', // Min 24px, max 48px
  },
});
```

## SEO Optimization

### Meta Tags
```typescript
// src/shared/utils/seo.web.ts
export interface PageMeta {
  title: string;
  description: string;
  keywords?: string[];
  ogImage?: string;
  canonical?: string;
}

export function setPageMeta(meta: PageMeta): void {
  // Title
  document.title = `${meta.title} | Weighsoft Animal Weigher`;
  
  // Description
  updateMetaTag('description', meta.description);
  
  // Keywords
  if (meta.keywords) {
    updateMetaTag('keywords', meta.keywords.join(', '));
  }
  
  // Open Graph
  updateMetaTag('og:title', meta.title, 'property');
  updateMetaTag('og:description', meta.description, 'property');
  if (meta.ogImage) {
    updateMetaTag('og:image', meta.ogImage, 'property');
  }
  
  // Canonical URL
  if (meta.canonical) {
    updateLinkTag('canonical', meta.canonical);
  }
}

function updateMetaTag(name: string, content: string, attr = 'name') {
  let tag = document.querySelector(`meta[${attr}="${name}"]`);
  if (!tag) {
    tag = document.createElement('meta');
    tag.setAttribute(attr, name);
    document.head.appendChild(tag);
  }
  tag.setAttribute('content', content);
}
```

### Semantic HTML
```typescript
// Use semantic HTML elements when targeting web
export const AnimalCard: React.FC = ({ animal }) => {
  if (Platform.OS === 'web') {
    return (
      <article>
        <header>
          <h2>{animal.name}</h2>
        </header>
        <section>
          <p>{animal.species}</p>
        </section>
      </article>
    );
  }
  
  // Mobile version
  return (
    <View>
      <Text>{animal.name}</Text>
      <Text>{animal.species}</Text>
    </View>
  );
};
```

## Web Routing

### React Router Setup
```typescript
// src/presentation/navigation/web-router.tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

export const WebRouter: React.FC = () => (
  <BrowserRouter>
    <Routes>
      {/* Public routes */}
      <Route path="/" element={<HomeScreen />} />
      <Route path="/login" element={<LoginScreen />} />
      
      {/* Protected routes */}
      <Route element={<ProtectedRoute />}>
        <Route path="/animals" element={<AnimalListScreen />} />
        <Route path="/animals/:id" element={<AnimalDetailScreen />} />
        <Route path="/weighing" element={<WeighingScreen />} />
        <Route path="/settings" element={<SettingsScreen />} />
      </Route>
      
      {/* 404 */}
      <Route path="*" element={<NotFoundScreen />} />
    </Routes>
  </BrowserRouter>
);

// Protected route wrapper
const ProtectedRoute = () => {
  const { isAuthenticated } = useAuth();
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" />;
};
```

### URL Parameters & Query Strings
```typescript
// Using react-router hooks
import { useParams, useSearchParams } from 'react-router-dom';

export const AnimalDetailScreen = () => {
  const { id } = useParams<{ id: string }>();
  const [searchParams, setSearchParams] = useSearchParams();
  
  const tab = searchParams.get('tab') ?? 'details';
  
  return (
    <View>
      <Tabs
        activeTab={tab}
        onTabChange={(newTab) => setSearchParams({ tab: newTab })}
      />
    </View>
  );
};
```

### Browser History
```typescript
// Handle browser back/forward
useEffect(() => {
  const handlePopState = () => {
    // Handle navigation
  };
  
  window.addEventListener('popstate', handlePopState);
  return () => window.removeEventListener('popstate', handlePopState);
}, []);
```

## Progressive Web App (PWA)

### Service Worker Setup
```typescript
// public/service-worker.js
const CACHE_NAME = 'weighsoft-v1';
const urlsToCache = [
  '/',
  '/static/css/main.css',
  '/static/js/main.js',
  '/manifest.json',
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => cache.addAll(urlsToCache))
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => response || fetch(event.request))
  );
});
```

### Web App Manifest
```json
// public/manifest.json
{
  "name": "Weighsoft Animal Weigher",
  "short_name": "Weighsoft",
  "description": "Professional animal weighing application",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#1976D2",
  "background_color": "#FFFFFF",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

### Offline Support
```typescript
// src/infrastructure/services/offline-detector.web.ts
export class OfflineDetector {
  private listeners: Array<(online: boolean) => void> = [];

  constructor() {
    window.addEventListener('online', () => this.notify(true));
    window.addEventListener('offline', () => this.notify(false));
  }

  isOnline(): boolean {
    return navigator.onLine;
  }

  subscribe(callback: (online: boolean) => void) {
    this.listeners.push(callback);
    return () => {
      this.listeners = this.listeners.filter(l => l !== callback);
    };
  }

  private notify(online: boolean) {
    this.listeners.forEach(listener => listener(online));
  }
}

// Usage
export const useOnlineStatus = () => {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const detector = new OfflineDetector();
    return detector.subscribe(setIsOnline);
  }, []);

  return isOnline;
};
```

## Web Performance

### Code Splitting
```typescript
// Lazy load screens
import { lazy, Suspense } from 'react';

const AnimalDetailScreen = lazy(() => import('./screens/animal-detail.screen'));
const SettingsScreen = lazy(() => import('./screens/settings.screen'));

export const App = () => (
  <Suspense fallback={<LoadingScreen />}>
    <Routes>
      <Route path="/animals/:id" element={<AnimalDetailScreen />} />
      <Route path="/settings" element={<SettingsScreen />} />
    </Routes>
  </Suspense>
);
```

### Image Optimization
```typescript
// Lazy load images
export const OptimizedImage: React.FC<ImageProps> = ({ src, alt }) => {
  return (
    <img
      src={src}
      alt={alt}
      loading="lazy"
      decoding="async"
      srcSet={`${src}?w=400 400w, ${src}?w=800 800w, ${src}?w=1200 1200w`}
      sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
    />
  );
};
```

### Bundle Size Optimization
```typescript
// Import only what you need
import { format } from 'date-fns'; // Tree-shakeable

// Use dynamic imports for large libraries
async function exportToPDF() {
  const { jsPDF } = await import('jspdf');
  // Use jsPDF
}
```

## Web Accessibility

### Keyboard Navigation
```typescript
// Handle keyboard events
export const Button: React.FC = ({ onPress, children }) => {
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      onPress();
    }
  };

  return (
    <div
      role="button"
      tabIndex={0}
      onClick={onPress}
      onKeyDown={handleKeyDown}
      aria-label="Save animal"
    >
      {children}
    </div>
  );
};
```

### Focus Management
```typescript
// Trap focus in modals
export const Modal: React.FC = ({ isOpen, onClose, children }) => {
  const modalRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!isOpen) return;

    const focusableElements = modalRef.current?.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );

    const firstElement = focusableElements?.[0] as HTMLElement;
    const lastElement = focusableElements?.[focusableElements.length - 1] as HTMLElement;

    const handleTab = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return;

      if (e.shiftKey && document.activeElement === firstElement) {
        e.preventDefault();
        lastElement?.focus();
      } else if (!e.shiftKey && document.activeElement === lastElement) {
        e.preventDefault();
        firstElement?.focus();
      }
    };

    document.addEventListener('keydown', handleTab);
    firstElement?.focus();

    return () => document.removeEventListener('keydown', handleTab);
  }, [isOpen]);

  return <div ref={modalRef}>{children}</div>;
};
```

### Skip Links
```typescript
// Allow keyboard users to skip navigation
export const Layout: React.FC = ({ children }) => (
  <>
    <a href="#main-content" className="skip-link">
      Skip to main content
    </a>
    <nav>...</nav>
    <main id="main-content">
      {children}
    </main>
  </>
);

// CSS
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  z-index: 100;
  padding: 8px;
  background: #000;
  color: #fff;
}

.skip-link:focus {
  top: 0;
}
```

## Web Security

### Content Security Policy (CSP)
```html
<!-- index.html -->
<meta
  http-equiv="Content-Security-Policy"
  content="
    default-src 'self';
    script-src 'self' 'unsafe-inline' 'unsafe-eval';
    style-src 'self' 'unsafe-inline';
    img-src 'self' data: https:;
    font-src 'self' data:;
    connect-src 'self' https://api.weighsoft.com;
  "
/>
```

### XSS Prevention
```typescript
// Sanitize user input before rendering
import DOMPurify from 'dompurify';

export const SafeHTML: React.FC<{ html: string }> = ({ html }) => {
  const sanitized = DOMPurify.sanitize(html);
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
};
```

## Web Analytics

### Google Analytics Setup
```typescript
// src/infrastructure/analytics/google-analytics.web.ts
export class GoogleAnalytics {
  track(event: string, params?: Record<string, any>) {
    if (typeof window.gtag !== 'undefined') {
      window.gtag('event', event, params);
    }
  }

  pageView(path: string) {
    this.track('page_view', { page_path: path });
  }
}

// Track page views
useEffect(() => {
  analytics.pageView(window.location.pathname);
}, [location.pathname]);
```

## Web Build Configuration

### webpack.config.js
```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
        },
      },
    },
  },
  performance: {
    maxAssetSize: 512000, // 500kb
    maxEntrypointSize: 512000,
  },
};
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/henzard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
