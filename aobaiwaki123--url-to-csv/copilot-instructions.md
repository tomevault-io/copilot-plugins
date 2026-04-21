## url-to-csv

> Shared utility functions and helper modules


# Shared Utilities & Helper Functions

The [header-utils.js](mdc:header-utils.js) provides common utility functions for HTTP header processing and other shared functionality across the image URL collection toolkit.

## Utility Module Overview
- **Purpose**: Centralized helper functions for HTTP operations and data processing
- **Architecture**: Pure JavaScript module with no external dependencies
- **Target**: Shared functionality between Net2Sheet, CSV Checker, and URL to CSV tools

## HTTP Header Processing

### Header Parsing Utilities
```javascript
// Parse HTTP headers from response objects
const parseHeaders = (headerString) => {
  const headers = new Map();
  if (!headerString) return headers;
  
  headerString.split('\r\n').forEach(line => {
    const colonIndex = line.indexOf(':');
    if (colonIndex > 0) {
      const name = line.substring(0, colonIndex).trim().toLowerCase();
      const value = line.substring(colonIndex + 1).trim();
      headers.set(name, value);
    }
  });
  
  return headers;
};

// Extract content type from headers
const getContentType = (headers) => {
  const contentType = headers.get('content-type') || '';
  return contentType.split(';')[0].trim().toLowerCase();
};

// Check if response contains image content
const isImageContentType = (contentType) => {
  const imageTypes = new Set([
    'image/png', 'image/jpeg', 'image/jpg', 'image/gif',
    'image/webp', 'image/svg+xml', 'image/avif',
    'image/bmp', 'image/x-icon', 'image/vnd.microsoft.icon'
  ]);
  return imageTypes.has(contentType);
};
```

### Request Enhancement
```javascript
// Add standard headers for web scraping
const enhanceRequestHeaders = (baseHeaders = {}) => {
  return {
    'User-Agent': 'Mozilla/5.0 (compatible; ImageURLCollector/1.0)',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/*;q=0.8,*/*;q=0.7',
    'Accept-Language': 'ja,en-US;q=0.9,en;q=0.8',
    'Accept-Encoding': 'gzip, deflate, br',
    'DNT': '1',
    'Upgrade-Insecure-Requests': '1',
    ...baseHeaders
  };
};

// Timeout wrapper for fetch requests
const fetchWithTimeout = async (url, options = {}, timeoutMs = 10000) => {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);
  
  try {
    const response = await fetch(url, {
      ...options,
      signal: controller.signal,
      headers: enhanceRequestHeaders(options.headers)
    });
    clearTimeout(timeoutId);
    return response;
  } catch (error) {
    clearTimeout(timeoutId);
    if (error.name === 'AbortError') {
      throw new Error(`Request timeout after ${timeoutMs}ms`);
    }
    throw error;
  }
};
```

## URL Processing Utilities

### URL Validation & Normalization
```javascript
// Comprehensive URL validation
const validateUrl = (urlString) => {
  try {
    const url = new URL(urlString);
    
    // Check for supported protocols
    if (!['http:', 'https:'].includes(url.protocol)) {
      return { valid: false, error: 'Unsupported protocol' };
    }
    
    // Check for suspicious patterns
    if (url.hostname.includes('localhost') || url.hostname.startsWith('127.')) {
      return { valid: false, error: 'Local URLs not supported' };
    }
    
    return { valid: true, url };
  } catch (error) {
    return { valid: false, error: error.message };
  }
};

// Convert relative URLs to absolute
const resolveUrl = (relativeUrl, baseUrl) => {
  try {
    return new URL(relativeUrl, baseUrl).href;
  } catch {
    return null;
  }
};

// Extract domain information
const getDomainInfo = (url) => {
  try {
    const urlObj = new URL(url);
    return {
      protocol: urlObj.protocol,
      hostname: urlObj.hostname,
      port: urlObj.port || (urlObj.protocol === 'https:' ? '443' : '80'),
      domain: urlObj.hostname.replace(/^www\./, ''),
      subdomain: urlObj.hostname.split('.').slice(0, -2).join('.')
    };
  } catch {
    return null;
  }
};
```

## Data Processing Helpers

### Array & Object Utilities
```javascript
// Remove duplicates while preserving order
const uniqueBy = (array, keyFunction) => {
  const seen = new Set();
  return array.filter(item => {
    const key = keyFunction(item);
    if (seen.has(key)) return false;
    seen.add(key);
    return true;
  });
};

// Chunk array into smaller batches
const chunkArray = (array, chunkSize) => {
  const chunks = [];
  for (let i = 0; i < array.length; i += chunkSize) {
    chunks.push(array.slice(i, i + chunkSize));
  }
  return chunks;
};

// Deep clone with JSON fallback
const deepClone = (obj) => {
  try {
    return structuredClone(obj);
  } catch {
    // Fallback for older browsers
    return JSON.parse(JSON.stringify(obj));
  }
};
```

### String Processing
```javascript
// Safe filename generation from URLs
const sanitizeFilename = (filename) => {
  return filename
    .replace(/[<>:"/\\|?*\x00-\x1f]/g, '_') // Remove invalid chars
    .replace(/^\.+/, '_') // No leading dots
    .replace(/\.+$/, '') // No trailing dots
    .substring(0, 255); // Limit length
};

// Extract file extension with validation
const getFileExtension = (filename) => {
  const lastDot = filename.lastIndexOf('.');
  if (lastDot === -1 || lastDot === filename.length - 1) return '';
  return filename.substring(lastDot).toLowerCase();
};

// Generate timestamp strings
const getTimestamp = (date = new Date(), format = 'iso') => {
  switch (format) {
    case 'iso':
      return date.toISOString();
    case 'filename':
      return date.toISOString().slice(0, 19).replace(/[T:]/g, '_');
    case 'human':
      return date.toLocaleString('ja-JP');
    default:
      return date.toString();
  }
};
```

## Error Handling Utilities

### Robust Error Processing
```javascript
// Standardized error object creation
const createError = (type, message, details = {}) => {
  const error = new Error(message);
  error.type = type;
  error.details = details;
  error.timestamp = new Date().toISOString();
  return error;
};

// Retry logic with exponential backoff
const retryWithBackoff = async (fn, maxRetries = 3, baseDelay = 1000) => {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      const delay = baseDelay * Math.pow(2, attempt - 1);
      console.warn(`Attempt ${attempt} failed, retrying in ${delay}ms:`, error.message);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
};

// Safe JSON parsing with fallback
const safeJsonParse = (jsonString, fallback = null) => {
  try {
    return JSON.parse(jsonString);
  } catch {
    return fallback;
  }
};
```

## Performance Monitoring

### Timing & Metrics
```javascript
// Simple performance timing
const createTimer = (label) => {
  const start = performance.now();
  return {
    stop: () => {
      const end = performance.now();
      const duration = end - start;
      console.log(`${label}: ${duration.toFixed(2)}ms`);
      return duration;
    }
  };
};

// Memory usage tracking (where available)
const getMemoryUsage = () => {
  if (performance.memory) {
    return {
      used: Math.round(performance.memory.usedJSHeapSize / 1024 / 1024),
      total: Math.round(performance.memory.totalJSHeapSize / 1024 / 1024),
      limit: Math.round(performance.memory.jsHeapSizeLimit / 1024 / 1024)
    };
  }
  return null;
};
```

## Browser Compatibility

### Feature Detection
```javascript
// Check for required browser features
const checkBrowserSupport = () => {
  const features = {
    fetch: typeof fetch !== 'undefined',
    urlApi: typeof URL !== 'undefined',
    domParser: typeof DOMParser !== 'undefined',
    blob: typeof Blob !== 'undefined',
    asyncAwait: (async () => {})().constructor === Promise,
    es6Sets: typeof Set !== 'undefined',
    es6Maps: typeof Map !== 'undefined'
  };
  
  const unsupported = Object.entries(features)
    .filter(([_, supported]) => !supported)
    .map(([feature, _]) => feature);
  
  return {
    allSupported: unsupported.length === 0,
    unsupported,
    features
  };
};

// Polyfill detection and loading
const loadPolyfillsIfNeeded = async () => {
  const support = checkBrowserSupport();
  if (support.allSupported) return;
  
  console.warn('Some browser features are missing:', support.unsupported);
  
  // In a real implementation, load polyfills here
  // For now, just warn the user
  if (typeof fetch === 'undefined') {
    throw new Error('このブラウザはfetch APIをサポートしていません');
  }
};
```

## Integration Guidelines

### Module Usage Patterns
```javascript
// Import pattern for tools using header-utils
const headerUtils = (() => {
  // Include header-utils.js content here or import it
  return {
    parseHeaders,
    fetchWithTimeout,
    validateUrl,
    sanitizeFilename,
    // ... other exported functions
  };
})();

// Usage in Net2Sheet, CSV Checker, and URL to CSV
const processImageUrl = async (url) => {
  const validation = headerUtils.validateUrl(url);
  if (!validation.valid) {
    throw new Error(`Invalid URL: ${validation.error}`);
  }
  
  const response = await headerUtils.fetchWithTimeout(url);
  // Process response...
};
```

### Configuration & Settings
```javascript
// Global configuration object
const CONFIG = {
  requests: {
    timeout: 10000,
    maxRetries: 3,
    batchSize: 5
  },
  csv: {
    encoding: 'utf-8',
    bom: true,
    headers: ['ファイル名', 'URL']
  },
  images: {
    extensions: ['.png', '.jpg', '.jpeg', '.gif', '.webp', '.svg', '.avif', '.bmp', '.ico'],
    maxFileSize: 50 * 1024 * 1024 // 50MB
  }
};

// Environment detection
const getEnvironment = () => {
  if (typeof chrome !== 'undefined' && chrome.devtools) {
    return 'extension';
  } else if (typeof window !== 'undefined') {
    return 'web';
  } else {
    return 'node';
  }
};
```

## Testing & Validation

### Unit Test Helpers
```javascript
// Test data generators
const generateTestUrls = (count = 10) => {
  const domains = ['example.com', 'test.org', 'sample.net'];
  const paths = ['images', 'assets', 'media', 'content'];
  const files = ['image', 'photo', 'logo', 'banner'];
  const extensions = ['.png', '.jpg', '.gif', '.webp'];
  
  return Array.from({ length: count }, (_, i) => {
    const domain = domains[i % domains.length];
    const path = paths[i % paths.length];
    const file = files[i % files.length];
    const ext = extensions[i % extensions.length];
    return `https://${domain}/${path}/${file}${i}${ext}`;
  });
};

// Validation helpers
const validateCsvOutput = (csvString) => {
  const lines = csvString.split('\n').filter(line => line.trim());
  if (lines.length < 2) return false; // Need header + at least one data row
  
  return lines.every(line => {
    const fields = line.split(',');
    return fields.length >= 2 && fields.every(field => 
      field.startsWith('"') && field.endsWith('"')
    );
  });
};
```

## Security Considerations

### Input Sanitization
```javascript
// Sanitize user input to prevent XSS
const sanitizeInput = (input) => {
  if (typeof input !== 'string') return '';
  
  return input
    .replace(/[<>]/g, '') // Remove angle brackets
    .replace(/javascript:/gi, '') // Remove javascript: protocol
    .replace(/data:/gi, '') // Remove data: protocol
    .trim()
    .substring(0, 2048); // Limit length
};

// Validate URLs for security
const isSecureUrl = (url) => {
  try {
    const urlObj = new URL(url);
    return urlObj.protocol === 'https:' || 
           (urlObj.protocol === 'http:' && urlObj.hostname === 'localhost');
  } catch {
    return false;
  }
};
```

This shared utility module provides robust, reusable functionality across all tools in the image URL collection ecosystem while maintaining consistency and reliability.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AobaIwaki123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
