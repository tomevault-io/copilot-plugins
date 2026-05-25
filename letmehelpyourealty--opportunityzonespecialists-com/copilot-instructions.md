## calendly-widget-integration

> Calendly widget integration patterns for Next.js


# Calendly Widget Integration (Next.js)

## Script Loading

Load Calendly script **once globally** in root layout using `strategy="afterInteractive"`:

```tsx
// app/layout.tsx
<Script
  src="https://assets.calendly.com/assets/external/widget.js"
  strategy="afterInteractive"
/>
```

**Important**: Use `afterInteractive` (not `beforeInteractive`) because the widget needs DOM to be ready.

## Widget Component Pattern

Use `useEffect` with `Calendly.initInlineWidget()` API:

```tsx
// ✅ CORRECT - Use initInlineWidget API
"use client";
import { useEffect, useRef } from "react";

export default function CalendlyWidget({ url = "https://calendly.com/drjanduffy/showing" }) {
  const widgetRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const initWidget = () => {
      if ((window as any).Calendly && widgetRef.current) {
        widgetRef.current.innerHTML = "";
        const widgetDiv = document.createElement("div");
        widgetDiv.className = "calendly-inline-widget";
        widgetDiv.setAttribute("data-url", url);
        widgetDiv.style.height = "700px";
        widgetRef.current.appendChild(widgetDiv);
        
        (window as any).Calendly.initInlineWidget({
          url: url,
          parentElement: widgetDiv,
        });
      }
    };

    if ((window as any).Calendly) {
      initWidget();
    } else {
      const check = setInterval(() => {
        if ((window as any).Calendly) {
          clearInterval(check);
          initWidget();
        }
      }, 100);
      setTimeout(() => clearInterval(check), 10000);
    }
  }, [url]);

  return <div ref={widgetRef} style={{ height: "700px", width: "100%" }} />;
}
```

```tsx
// ❌ WRONG - dangerouslySetInnerHTML doesn't initialize widget
<div dangerouslySetInnerHTML={{
  __html: `<div class="calendly-inline-widget" data-url="${url}"></div>`
}} />
```

## CSP Headers Required

Add these domains to `next.config.js` CSP:

```js
"script-src": "https://assets.calendly.com",
"style-src": "https://assets.calendly.com", 
"font-src": "https://assets.calendly.com",
"connect-src": "https://calendly.com",
"frame-src": "https://calendly.com https://assets.calendly.com"
```

## Current Calendly URL

Dr. Jan Duffy's scheduling link: `https://calendly.com/drjanduffy/showing`

---
> Source: [LetMeHelpYouREALTY/opportunityzonespecialists.com](https://github.com/LetMeHelpYouREALTY/opportunityzonespecialists.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
