## hero-featured-event-banner-section

> This rule defines the section shown **directly under the hero** on the home page when there is a **future** event marked as **featured** (`isFeaturedEvent`) and that event has at least one media file marked as **featured event image** (`isFeaturedEventImage`). The section displays a **single image** (no rotation/carousel) with the same "Buy ticket click here" / "Fundraiser ticket click here" overlay at the **bottom right** as the hero section.

# Featured Event Banner Section (Under Hero)

## **Overview**
This rule defines the section shown **directly under the hero** on the home page when there is a **future** event marked as **featured** (`isFeaturedEvent`) and that event has at least one media file marked as **featured event image** (`isFeaturedEventImage`). The section displays a **single image** (no rotation/carousel) with the same "Buy ticket click here" / "Fundraiser ticket click here" overlay at the **bottom right** as the hero section.

## **Problem Solved**
- **Single featured spotlight**: One prominent featured event image below the hero when criteria are met
- **No looping**: Unlike the hero, this block shows one image only (no rotation)
- **Same overlay behavior**: Reuses hero overlay logic (ticketed fundraiser → fundraiser image + donation checkout; regular ticketed → red "buy tickets" + checkout/manual-checkout)
- **Conditional visibility**: Section is hidden when there is no qualifying featured event with featured event image

## **Display Criteria (All required)**

1. **Future event**: Event `startDate` is today or in the future (upcoming events from `useEventsData` already satisfy this).
2. **Event-level flag**: Event has `isFeaturedEvent === true`.
3. **Media-level flag**: Event has at least one media item with `isFeaturedEventImage === true`.
4. **Image URL**: The chosen media has a valid `fileUrl`.

When all are satisfied, show the section; otherwise render nothing.

## **Core Pattern**

### **Data source and selection**
- Use `useFilteredEvents('featured')` to get events that have media with `isFeaturedEventImage === true`.
- Filter results to events where `event.isFeaturedEvent === true`.
- Sort by `event.featuredEventPriorityRanking` (ascending: lower number = higher priority).
- Take the **first** item; use its `event` and first matching `media` (with `isFeaturedEventImage`).
- If no item remains or media has no `fileUrl`, do not render the section.

### **Single image (no rotation)**
- Render **one** image: the featured event image (`media.fileUrl`).
- No carousel, no automatic rotation, no multiple slides.
- Image should be contained (e.g. `object-contain`, full width, auto height) and link to the event page (`/events/[id]`).

### **Overlay (same as hero)**
- Use the **same overlay logic and position** as the hero section:
  - **Position**: Bottom right of the image (reuse `hero-ticket-overlay` class from `globals.css`).
  - **Logic**: Use `getOverlayInfo(event)` from `@/lib/heroOverlay` (shared with `HeroSection`).
  - **Behavior**:
    - **Ticketed fundraiser**: Show fundraiser CTA image; link to `/events/[id]/givebutter-checkout`.
    - **Regular ticketed**: Show red "Buy tickets" image; link to `/events/[id]/checkout` or `/events/[id]/manual-checkout` based on `manualPaymentEnabled` and `paymentFlowMode`.
    - **Non-upcoming or non-ticketed**: No overlay (`getOverlayInfo` returns `null`).
- **Overlay image sizes**: Match hero (e.g. `w-[140px] h-[48px] sm:w-[180px] sm:h-[62px] md:w-[200px] md:h-[70px]`).

### **Placement**
- Section is a **direct sibling below the hero** on the home page (e.g. after `<HeroSection />`, before the next content).
- Only one such section; it shows at most one featured event image.

## **Implementation References**

- **Component**: [`src/components/FeaturedEventBannerSection.tsx`](mdc:src/components/FeaturedEventBannerSection.tsx) – uses `useFilteredEvents('featured')`, filters by `isFeaturedEvent`, sorts by `featuredEventPriorityRanking`, renders one image + overlay.
- **Overlay util**: [`src/lib/heroOverlay.ts`](mdc:src/lib/heroOverlay.ts) – `getOverlayInfo(event)` used by both hero and featured event banner.
- **Hero overlay / rotation**: [hero_section_image_rotation.mdc](mdc:.cursor/rules/hero_section_image_rotation.mdc) – overlay logic and "Overlay Logic (Buy Tickets Click Here Image Pattern)".
- **Home page**: [`src/app/page.tsx`](mdc:src/app/page.tsx) – `<FeaturedEventBannerSection />` rendered immediately after `<HeroSection />`.
- **CSS**: `hero-ticket-overlay` in [`src/app/globals.css`](mdc:src/app/globals.css) – bottom-right positioning for the CTA.

## **Best Practices**

### **DO**
- Reuse `getOverlayInfo` from `@/lib/heroOverlay` for overlay content and links.
- Reuse `hero-ticket-overlay` class for overlay position and styling.
- Restrict to events with `isFeaturedEvent === true` and media with `isFeaturedEventImage === true`.
- Use a single image and a single event (first after sorting by `featuredEventPriorityRanking`).
- Link the main image to the event page and the overlay to checkout/donation as per `getOverlayInfo`.

### **DON'T**
- Do not add rotation/carousel for this section.
- Do not show the section when there is no qualifying featured event with featured event image.
- Do not implement separate overlay logic; use the shared hero overlay util and CSS.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
