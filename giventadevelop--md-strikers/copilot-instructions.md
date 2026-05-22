## givebutter-widget-integration

> Standard pattern for integrating Givebutter donation widgets with popup modal functionality in Next.js applications


# Givebutter Widget Integration Pattern

## **Overview**
This rule defines the standard pattern for integrating Givebutter donation widgets into Next.js applications, specifically for creating donate buttons that open popup modals with donation forms. This pattern addresses X-Frame-Options restrictions and provides reliable popup functionality across desktop and mobile devices.

## **Problem Solved**
- **X-Frame-Options Restriction**: Givebutter sets `X-Frame-Options: sameorigin`, preventing iframe embedding from other domains
- **Popup/New Tab Functionality**: Donate button opens the full campaign page in a popup (desktop) or new tab (mobile), so dashboard settings (suggested amounts, **Other amount**, minimum donation) are visible
- **Other Amount & Minimum**: The widget modal does not reliably show “Other” option or minimum donation from the dashboard; opening the full campaign URL ensures these options appear
- **Cross-Platform Compatibility**: Works consistently on both desktop and mobile browsers
- **Fallback**: If popup is blocked, opens campaign in new tab

## **Credentials and configuration from Givebutter dashboard**

To make the donate button (and optional widget script) work, you need the following from the Givebutter dashboard. **No API keys, Aadhaar, or other identity credentials are required in this application** for the home-page donate button.

| What you need | Where to get it in Givebutter | Where it’s used in the app | Required for donate button? |
|---------------|-------------------------------|----------------------------|-----------------------------|
| **Campaign ID** | Campaign URL: `https://givebutter.com/{CampaignID}` (e.g. `mhoZp0`). Or: open the campaign → Sharing → copy from campaign link or embed code. | Set in **`.env.local`** (and production env) as **`NEXT_PUBLIC_GIVEBUTTER_CAMPAIGN_ID`**. Read by `GivebutterDonateButton` and `GivebutterWidget`; optional `campaignId` prop overrides. Declared in `next.config.mjs` `env`. Builds URL: `https://givebutter.com/{campaignId}`. | **Yes** – without it the button would open the wrong or no campaign. |
| **Widget ID** | Givebutter embed for **event fund** (not donation): e.g. `<givebutter-widget id="j1ek6j">`. Get the ID from the widget embed code in Givebutter. | Set in **`.env.local`** (and production env) as **`NEXT_PUBLIC_GIVEBUTTER_WIDGET_ID`** (e.g. `j1ek6j`). Used by event GiveButter checkout page (`/events/[id]/givebutter-checkout`): when the event’s `donation_metadata` has no `givebutterWidgetId`, the app falls back to this env. Declared in `next.config.mjs` `env`. | **No** for the home-page donate button. **Yes** for the event fund embed when you want a single global widget ID without storing it per event. |
| **Account ID** | Dashboard → **Campaign** → **Sharing** → **Widgets** → **Installation**. Shown in the script snippet as `acct=...` (e.g. `mKoUpYQebNsn6RqA`). | `src/app/layout.tsx`: Givebutter script `src="...?acct={AccountID}&p=other"`. | **No** for the donate button (button only opens the campaign URL). **Yes** if you use Givebutter embed widgets (e.g. `givebutter-form`, `givebutter-button`) elsewhere. |
| **Webhook URL** | Not “brought” into the app; you **configure** in Givebutter: Campaign or Account → Integrations / Webhooks → add endpoint. Set URL to your app’s route, e.g. `https://your-domain.com/api/webhooks/givebutter`. | Givebutter sends donation events to this URL. App route: `src/app/api/webhooks/givebutter/route.ts` (forwards to backend). | No – only needed if you want server-side handling of donation events. |
| **Webhook signing secret** | Givebutter dashboard: when you add the webhook, Givebutter shows or lets you copy a **signing secret**. | Stored and used by the **backend** (not in Next.js env) to verify `X-Givebutter-Signature`. Backend matches secret per tenant in `payment_provider_config`. | No – backend only; not needed for the button to open. |

**Summary for the home-page donate button only:**

1. **Campaign ID** – Get from the campaign’s Givebutter URL or Sharing/Widgets. Put it in the app as:
   - **Option A:** Pass `campaignId` to `<GivebutterDonateButton campaignId="mhoZp0" />` (e.g. in `HeroSection.tsx`), or
   - **Option B:** Set `NEXT_PUBLIC_GIVEBUTTER_CAMPAIGN_ID` in `.env.local` and use it in the component so the same env can be used in production.
2. **Account ID** – Only if you use the widget script (e.g. for other Givebutter embeds). Put it in the script URL in `layout.tsx`: `acct=YOUR_ACCOUNT_ID`.

**What is not needed in this app for the button:**

- No Givebutter API key in the frontend (backend may use API for donation status if needed).
- No Aadhaar or other identity credentials – Givebutter does not use Aadhaar for this integration.
- No webhook secret in the Next.js app – the backend holds and verifies it.

## **Core Pattern**

### **1. Script Installation in Layout**

```tsx
// ✅ DO: Add Givebutter script to root layout using Next.js Script component
// src/app/layout.tsx
import Script from "next/script";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        {/* Other content */}

        {/* Givebutter Widget Script */}
        <Script
          id="givebutter-widget"
          src="https://widgets.givebutter.com/latest.umd.cjs?acct={YOUR_ACCOUNT_ID}&p=other"
          strategy="afterInteractive"
          async
        />
      </body>
    </html>
  );
}
```

**Key Requirements:**
- **`strategy="afterInteractive"`**: Loads script after page becomes interactive
- **`async`**: Allows script to load asynchronously
- **Account ID**: Replace `{YOUR_ACCOUNT_ID}` with your Givebutter account ID (e.g., `mKoUpYQebNsn6RqA`)

### **2. Donate Button Component**

```tsx
// ✅ DO: Always open full campaign page (popup on desktop, new tab on mobile)
// Do NOT use <givebutter-button> modal — it does not show "Other amount" or minimum donation from dashboard
'use client';

import React from 'react';

export default function GivebutterDonateButton({ campaignId: campaignIdProp, className = '', children }: GivebutterDonateButtonProps) {
  const campaignId = campaignIdProp ?? process.env.NEXT_PUBLIC_GIVEBUTTER_CAMPAIGN_ID?.trim() || 'mhoZp0';
  const handleClick = (e: React.MouseEvent) => {
    e.preventDefault();
    e.stopPropagation();

    const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent) || (window.innerWidth <= 768);

    if (isMobile) {
      window.open(`https://givebutter.com/${campaignId}`, '_blank');
      return;
    }

    const width = 800, height = 900;
    const left = (window.screen.width - width) / 2;
    const top = (window.screen.height - height) / 2;
    const popup = window.open(
      `https://givebutter.com/${campaignId}`,
      'GivebutterDonation',
      `width=${width},height=${height},left=${left},top=${top},resizable=yes,scrollbars=yes,toolbar=no,menubar=no,location=no`
    );

    if (popup) {
      popup.focus();
      const checkClosed = setInterval(() => { if (popup.closed) clearInterval(checkClosed); }, 500);
      setTimeout(() => clearInterval(checkClosed), 600000);
    } else {
      window.open(`https://givebutter.com/${campaignId}`, '_blank');
    }
  };

  return (
    <button onClick={handleClick} className={className} title="Donate Now" aria-label="Donate Now" type="button">
      {children}
    </button>
  );
}
```

## **Key Implementation Details**

### **Why Popup Window Instead of Iframe?**

**Problem**: Givebutter sets `X-Frame-Options: sameorigin` header, which prevents embedding their pages in iframes from other domains. This is a security measure to prevent clickjacking attacks.

**Error Message**: `Refused to display 'https://givebutter.com/' in a frame because it set 'X-Frame-Options' to 'sameorigin'.`

**Solution**: Use `window.open()` to create a popup window instead of an iframe. Popup windows are not subject to X-Frame-Options restrictions.

### **Popup Window Configuration**

**Desktop Configuration:**
- **Width**: `800px` - Comfortable width for donation forms
- **Height**: `900px` - Sufficient height for form and content
- **Position**: Centered on screen using `(screen.width - width) / 2` and `(screen.height - height) / 2`

**Window Features:**
- **`resizable=yes`**: Allows users to resize if needed
- **`scrollbars=yes`**: Enables scrolling for long forms
- **`toolbar=no`**: Removes browser toolbar for cleaner appearance
- **`menubar=no`**: Removes menu bar
- **`location=no`**: Removes address bar (optional, improves UX)

**Mobile Configuration:**
- **Strategy**: Open in new tab instead of popup window
- **Rationale**: Mobile browsers don't support popup windows well and often block them
- **PRB Support**: Opening in new tab ensures Apple Pay/Google Pay PRB buttons work correctly
- **Detection**: Use user agent and screen width to detect mobile devices

### **Donation Flow (No Widget Modal)**

1. **Desktop**: Open full campaign page in popup window with `window.open(\`https://givebutter.com/${campaignId}\`, ...)` so users see suggested amounts, **Other amount**, and minimum donation from the dashboard
2. **Mobile**: Open full campaign page in new tab with `window.open(url, '_blank')` (better UX, PRB support)
3. **Popup blocked**: Fallback to new tab with `window.open(url, '_blank')`

We do **not** use the `<givebutter-button>` widget modal because it does not show the “Other amount” option or minimum donation from the Givebutter dashboard.

### **Script Loading Detection**

**Polling Strategy:**
- Check every 100ms for script and custom element availability
- Timeout after 5 seconds if not detected
- Still allow button to work with popup fallback even if script doesn't load

**Detection Methods:**
- Check for script tag: `document.querySelector('script[src*="widgets.givebutter.com"]')`
- Check for custom element: `customElements.get('givebutter-button')`
- Check for global object: `(window as any).Givebutter` or `(window as any).givebutter`

## **TypeScript Declarations**

### **Custom Element Types**

```typescript
// src/types/givebutter.d.ts
declare namespace JSX {
  interface IntrinsicElements {
    'givebutter-widget': React.DetailedHTMLProps<React.HTMLAttributes<HTMLElement> & {
      id?: string;
    }, HTMLElement>;
    'givebutter-form': React.DetailedHTMLProps<React.HTMLAttributes<HTMLElement> & {
      campaign?: string;
    }, HTMLElement>;
    'givebutter-button': React.DetailedHTMLProps<React.HTMLAttributes<HTMLElement> & {
      campaign?: string;
      label?: string;
      'label-color'?: string;
      'background-color'?: string;
      'drop-shadow'?: string | boolean;
      'border-radius'?: string | number;
      'border-color'?: string;
      'border-width'?: string | number;
      'givebutter-theme'?: string;
    }, HTMLElement>;
  }
}

declare global {
  interface Window {
    Givebutter?: {
      init?: (config: {
        account: string;
        campaign?: string;
        container?: HTMLElement;
      }) => void;
    };
  }
}

export {};
```

## **Usage Examples**

### **Homepage Donate Button**

```tsx
// src/app/page.tsx
import GivebutterDonateButton from '@/components/GivebutterDonateButton';

export default function HomePage() {
  return (
    <div>
      <div className="w-full bg-green-700 text-white py-6 md:py-8 mt-[10px]">
        <div className="max-w-6xl mx-auto px-4 sm:px-6 lg:px-8 text-center">
          <span className="text-2xl md:text-4xl font-bold tracking-wider">MOSC-TEMP</span>

          {/* Donate Button - Opens Givebutter Popup Modal */}
          <div className="mt-4">
            <GivebutterDonateButton
              campaignId="mhoZp0"
              className="inline-flex items-center justify-center gap-2 px-6 py-3 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-lg shadow-md hover:shadow-lg transition-all duration-300 hover:scale-105 cursor-pointer"
            >
              <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" />
              </svg>
              <span>Donate Now</span>
            </GivebutterDonateButton>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### **Embedded Widget (Full Page)**

```tsx
// src/app/donate/page.tsx
import GivebutterWidget from '@/components/GivebutterWidget';

export default function DonatePage() {
  return (
    <div className="min-h-screen bg-gray-50 py-12">
      <div className="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="bg-white rounded-lg shadow-lg p-6 md:p-8">
          <GivebutterWidget campaignId="mhoZp0" />
        </div>
      </div>
    </div>
  );
}
```

## **Common Anti-Patterns**

```tsx
// ❌ DON'T: Use iframe for Givebutter campaigns
<iframe src={`https://givebutter.com/${campaignId}`} />
// Error: X-Frame-Options: sameorigin blocks this

// ❌ DON'T: Navigate to donation page instead of popup
<Link href="/donate">Donate</Link>
// This breaks the popup modal UX pattern

// ❌ DON'T: Use givebutter-theme on regular buttons without script check
<button givebutter-theme="click-only">Donate</button>
// Won't work if script hasn't loaded

// ❌ DON'T: Load script in component instead of layout
useEffect(() => {
  const script = document.createElement('script');
  script.src = 'https://widgets.givebutter.com/...';
  // This can cause duplicate script loads
}, []);
```

## **Best Practices**

### **DO:**
- ✅ Load Givebutter script in root layout using Next.js `Script` component (optional; donate button opens full campaign URL)
- ✅ **Always open full campaign page** (popup on desktop, new tab on mobile) so “Other amount” and minimum donation from dashboard are visible
- ✅ Handle popup blocking gracefully (fallback to new tab)
- ✅ Monitor popup closure for analytics/callbacks
- ✅ Center popup windows on screen
- ✅ Set appropriate popup dimensions (800x900px recommended)
- ✅ Include cleanup for interval timers
- ✅ Enable “Show an ‘Other’ option” and “Require minimum donation” in Givebutter Dashboard → Campaign → Suggested amounts

### **DON'T:**
- ❌ Don't use iframe for Givebutter campaigns (X-Frame-Options blocks it)
- ❌ **Don't use `<givebutter-button>` widget modal** for donate button — it does not show “Other amount” or minimum donation from dashboard
- ❌ Don't navigate to donation page (use popup/tab instead)
- ❌ Don't forget to handle popup blocking
- ❌ Don't use arbitrary popup dimensions
- ❌ Don't forget cleanup for timers/intervals

## **Mobile Device Support & PRB Compatibility**

### **Apple Pay & Google Pay PRB Support**

**✅ YES - Givebutter supports Apple Pay and Google Pay automatically**

Givebutter's donation forms automatically display Apple Pay and Google Pay buttons when:
- **Apple Pay**: User is on iOS Safari or macOS Safari with Apple Pay configured
- **Google Pay**: User is on Chrome/Android with Google Pay configured
- Payment methods are enabled in Givebutter campaign settings

### **Mobile Browser Behavior**

**Mobile Strategy:**
- **Desktop**: Opens popup window (800x900px, centered)
- **Mobile**: Opens in new tab (better UX, ensures PRB works)

**Why New Tab on Mobile?**
1. Mobile browsers often block or mishandle popup windows
2. New tab provides better user experience on small screens
3. Ensures Apple Pay/Google Pay PRB buttons work correctly
4. Avoids popup blocking issues common on mobile

**PRB Compatibility:**
- ✅ **Apple Pay**: Works when Givebutter page opens in new tab on iOS Safari
- ✅ **Google Pay**: Works when Givebutter page opens in new tab on Chrome/Android
- ✅ **Automatic Detection**: Givebutter automatically detects device capabilities and shows appropriate PRB buttons
- ✅ **No Configuration Needed**: Givebutter handles PRB display automatically

### **Testing Mobile PRB**

1. **Test on iOS Safari**:
   - Open donation page on iPhone/iPad
   - Verify Apple Pay button appears (if Apple Pay is set up)
   - Complete donation using Apple Pay

2. **Test on Android Chrome**:
   - Open donation page on Android device
   - Verify Google Pay button appears (if Google Pay is set up)
   - Complete donation using Google Pay

3. **Verify in Givebutter Dashboard**:
   - Check that donations are recorded correctly
   - Verify payment method shows as Apple Pay/Google Pay

## **Full Campaign Page (Popup/Tab) – Current Behavior**

**We always open the full Givebutter campaign page** (popup on desktop, new tab on mobile). We do **not** use the `<givebutter-button>` widget modal.

### **Why Full Campaign Page Instead of Widget Modal**

The **widget modal** (`<givebutter-button>`) shows a limited checkout flow that **does not** include all campaign settings from the Givebutter dashboard, including:

- **"Show an 'Other' option to allow supporters to enter their own amount"** – even when enabled in Dashboard → Campaign → Suggested amounts, the modal may not show the Other amount field.
- **"Require a minimum donation amount"** – minimum amount configured in the dashboard may not apply in the modal.
- Cover image and campaign description also do not appear in the modal.

By opening the **full campaign URL** (`https://givebutter.com/{campaignId}`) in a popup or new tab, users see the complete campaign form as configured in the dashboard: suggested amounts, **Other amount** field, minimum donation, cover image, and description.

### **Modal vs Full Campaign Page**

| Approach | What opens | Other amount? | Min donation? | Cover & description? |
|----------|------------|---------------|--------------|----------------------|
| **Widget modal** | Overlay on our site → checkout | **No** (not reliably) | **No** (not reliably) | **No** |
| **Full campaign page** (current) | `https://givebutter.com/{campaignId}` in popup/tab | **Yes** | **Yes** | **Yes** |

### **Implementation**

- **GivebutterDonateButton** does **not** render `<givebutter-button>`.
- On click it always calls `window.open(\`https://givebutter.com/${campaignId}\`, ...)` (popup on desktop, `_blank` on mobile).
- Dashboard settings (Suggested amounts, “Other” option, Require minimum) apply only on the full campaign page; they are not fully respected in the widget modal.

### **Dashboard Configuration (Other Amount & Minimum)**

- In **Givebutter Dashboard** → Campaign → **Suggested amounts**:
  - Enable **"Show an 'Other' option to allow supporters to enter their own amount"** so users can enter any amount.
  - Enable **"Require a minimum donation amount"** and set **Minimum amount** (e.g. $2.00) if desired.
- These options take effect when donors use the **full campaign page** (our popup/tab). They do not reliably apply in the widget modal.

**References:** Givebutter Help – [How to use Givebutter Widgets on your website](https://help.givebutter.com/en/articles/6464859-how-to-use-givebutter-widgets-on-your-website); [How to customize suggested donation amounts](https://help.givebutter.com/en/articles/2859751-how-to-customize-suggested-donation-amounts).

## **Troubleshooting**

### **Popup Not Opening?**
1. Check browser popup blocker settings
2. Verify popup is triggered by user interaction (not programmatic)
3. Check console for errors
4. Verify campaign ID is correct
5. Test popup in different browsers
6. **On Mobile**: Popup will open in new tab instead (this is expected behavior)

### **PRB Buttons Not Showing on Mobile?**
1. Verify you're testing on actual device (not just browser dev tools)
2. Check that Apple Pay/Google Pay is set up on the device
3. Verify payment methods are enabled in Givebutter campaign settings
4. Ensure Givebutter page opens in new tab (not iframe)
5. Check browser console for any errors
6. Verify HTTPS is being used (required for Apple Pay)

### **Custom Element Not Working?**
1. Verify script is loaded: `document.querySelector('script[src*="widgets.givebutter.com"]')`
2. Check if custom element is defined: `customElements.get('givebutter-button')`
3. Verify account ID in script URL matches your Givebutter account
4. Check browser console for Givebutter initialization errors
5. Fallback to popup window should still work

### **X-Frame-Options Error?**
- **This is expected** - Givebutter blocks iframe embedding
- **Solution**: Use popup window instead of iframe
- **Error Message**: `Refused to display 'https://givebutter.com/' in a frame because it set 'X-Frame-Options' to 'sameorigin'.`

### **Script Not Loading?**
1. Check network tab for script request
2. Verify script URL is correct
3. Check for CORS or CSP errors
4. Verify `strategy="afterInteractive"` is set
5. Check if script is being blocked by ad blockers

## **Environment Variables**

### **Optional (recommended for production)**
- **`NEXT_PUBLIC_GIVEBUTTER_CAMPAIGN_ID`**: Campaign ID for donate button and campaign URL (e.g. `mhoZp0`). Used when no `campaignId` prop is passed.
- **`NEXT_PUBLIC_GIVEBUTTER_WIDGET_ID`**: Widget ID for the **event fund** embed (e.g. `j1ek6j`). Used on `/events/[id]/givebutter-checkout` when the event’s `donation_metadata` does not include `givebutterWidgetId`. This externalizes the `<givebutter-widget id="...">` ID so it can be set per environment without storing it per event.

### **Other**
- Account ID is embedded in the Givebutter script URL in `layout.tsx` (optional for donate button; required for embed widgets).

### **Optional (for Webhooks)**
- `DEV_GBTTR_API_KEY`: Givebutter API key for webhook verification
- `DEV_GBTTR_WEBHOOK_SECRET`: Givebutter webhook secret for signature verification

## **Reference Implementations**

- **Donate Button Component**: [`src/components/GivebutterDonateButton.tsx`](mdc:src/components/GivebutterDonateButton.tsx)
- **Widget Component**: [`src/components/GivebutterWidget.tsx`](mdc:src/components/GivebutterWidget.tsx)
- **Donation Page**: [`src/app/donate/page.tsx`](mdc:src/app/donate/page.tsx)
- **Homepage Integration**: [`src/app/page.tsx`](mdc:src/app/page.tsx) - Lines 268-279
- **Layout Script**: [`src/app/layout.tsx`](mdc:src/app/layout.tsx) - Lines 374-380
- **TypeScript Declarations**: [`src/types/givebutter.d.ts`](mdc:src/types/givebutter.d.ts)

## **Related Patterns**

- See [Next.js API Routes](mdc:.cursor/rules/nextjs_api_routes.mdc) for webhook handling patterns
- See [Icon Standards](mdc:.cursor/rules/icons_buttons_styles.mdc) for button styling
- See [MOSC Styling Standards](mdc:.cursor/rules/mosc_styling_standards.mdc) for design system

## **Summary**

**Key Rules:**
1. **Never use iframe** - Givebutter blocks iframe embedding with X-Frame-Options
2. **Always open full campaign page** - Use popup (desktop) or new tab (mobile) so “Other amount” and minimum donation from the dashboard are visible; do **not** use the `<givebutter-button>` widget modal
3. **Use popup window on desktop** - `window.open(\`https://givebutter.com/${campaignId}\`, ...)` so users get the full campaign form
4. **Use new tab on mobile** - Mobile browsers handle popups poorly; new tab ensures PRB works
5. **Handle popup blocking** - Gracefully fallback to new tab if popup is blocked
6. **Monitor popup closure** - Use intervals to detect when popup closes (for analytics)
7. **Detect mobile devices** - Use user agent and screen width to determine mobile vs desktop

**Implementation Checklist:**
- [ ] **Credentials**: Campaign ID set in **`.env.local`** (and production env) as **`NEXT_PUBLIC_GIVEBUTTER_CAMPAIGN_ID`**; **Widget ID** (for event fund embed) as **`NEXT_PUBLIC_GIVEBUTTER_WIDGET_ID`** (e.g. `j1ek6j`); both declared in `next.config.mjs` `env`. No hardcoded campaign or widget IDs; optional props/event metadata override. Optionally Account ID in layout script if using widgets. See **Credentials and configuration from Givebutter dashboard** above.
- [ ] Givebutter script added to root layout with `strategy="afterInteractive"` (optional; donate button opens full campaign URL and does not require widget script)
- [ ] TypeScript declarations file created for custom elements (for GivebutterWidget / givebutter-form if used elsewhere)
- [ ] Donate button **always** opens full campaign page: popup (desktop) or new tab (mobile) — do **not** use `<givebutter-button>` modal so “Other amount” and minimum donation from dashboard are visible
- [ ] Mobile device detection implemented (user agent + screen width)
- [ ] Fallback to new tab if popup is blocked
- [ ] Popup dimensions set to 800x900px and centered (desktop only)
- [ ] Cleanup for timers/intervals implemented (popup closure)
- [ ] Error handling and logging included

**Mobile PRB Support:**
- ✅ **Apple Pay**: Automatically supported when Givebutter page opens in new tab on iOS Safari
- ✅ **Google Pay**: Automatically supported when Givebutter page opens in new tab on Chrome/Android
- ✅ **No Additional Configuration**: Givebutter handles PRB display automatically based on device capabilities
- ✅ **HTTPS Required**: Ensure your site uses HTTPS (required for Apple Pay)

**Cover image & description:**
- **Full campaign page:** Cover image and description are shown on the campaign page opened in popup/tab. Upload the cover in the Givebutter dashboard (Campaign → Settings / Design).
- **Givebutter cover image specs (from Givebutter Help):**
  - **Dimensions:** 1200 × 630 px (recommended)
  - **Aspect ratio:** ~1.9:1 (1200:630); "Maintain Aspect Ratio" can be enabled in campaign details to avoid cropping
  - **File size:** Max 1 MB
  - **Format:** JPEG recommended
- **Creating a cover from a logo (e.g. for Nano Banana / AI image editor):** Use the prompt in **Cover image – AI editor prompt** below to produce a 1200×630 image from `public/images/logos/giventa/logo_png.png` (or your logo path) for upload to Givebutter.

**Cover image – AI editor prompt (Nano Banana / generic):**
Use this prompt with the logo image attached so the AI outputs a Givebutter-ready cover image:

```
Using the attached GIVENTA logo image, create a single cover/banner image for a donation campaign page with these exact specs:

1. Output dimensions: 1200 pixels wide × 630 pixels tall (aspect ratio 1200:630). No other size.
2. Content: Place the GIVENTA logo (the emblem with the white "G" and blue/green shapes plus the lime-green "GIVENTA" text) centered both horizontally and vertically on the canvas. Do not stretch, skew, or change the logo’s proportions or colors.
3. Background: Use the same solid black as in the original logo. Extend this black to fill the entire 1200×630 canvas so the logo sits on a clean black banner.
4. Clear space: Leave enough padding around the logo so it is clearly visible and not cramped; the logo should occupy roughly the middle 50–60% of the height with equal margins on top and bottom.
5. Output: One image only, 1200×630 px, high quality, suitable for web. Prefer JPEG under 1 MB for upload to a campaign platform, or PNG if the tool does not export JPEG.
```

After generating, upload the result to Givebutter (Campaign → cover image). If Givebutter offers "Maintain aspect ratio," enable it so the image is not cropped.

This ensures reliable Givebutter donation widget integration that works across all browsers and devices while respecting security restrictions and enabling mobile payment methods (Apple Pay/Google Pay PRB).

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
