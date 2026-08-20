## ugc-studio

> Generate 40 different static ad designs for any brand using Claude Code + Google Gemini API (Nano Banana 2).

# UGC Studio — AI Static Ad Generator

Generate 40 different static ad designs for any brand using Claude Code + Google Gemini API (Nano Banana 2).

## Quick Start

1. Open this folder in any IDE with Claude Code (Cursor, VS Code, Google Antigravity, or Terminal)
2. Set your Google API key:
   ```
   export GOOGLE_API_KEY="your-key-here"
   ```
   Get a free key at: https://aistudio.google.com/apikey
3. Install dependencies:
   ```
   python3 -m venv .venv && source .venv/bin/activate && pip install google-genai Pillow
   ```
4. Tell Claude Code:
   ```
   Run the full pipeline for [brand name]. URL: [brand website]. Product: [specific product].
   ```

That's it. Claude handles everything from here.

## How to Use

### Full Pipeline (Recommended)
```
Run the full pipeline for [brand]. URL: [url]. Product: [product name].
```
Claude will research the brand, generate 40 ad prompts, and create all images automatically.

### Step by Step
- `Run Phase 1 for [brand]` — Research brand + create DNA document
- `Run Phase 2 for [brand]` — Generate 40 ad prompts from templates
- `Run Phase 3 for [brand], templates 1, 7, 13, 15` — Generate specific ads
- `Run Phase 3 for [brand], all` — Generate all 40 ads

### Adding Product Images (Optional but Recommended)
Drop your product photos (PNG/JPG) into `brands/[brand-name]/product-images/` before running Phase 3. The AI will use them as reference for more accurate bottle/packaging rendering.

### Other Commands
- `Show my brands` — List all brands
- `Show prompts for [brand]` — Display generated prompts
- `Regenerate template [N] for [brand]` — Re-run specific template

## Project Structure
```
UGC-Studio/
├── CLAUDE.md                        ← You are here
├── generate-ads.py                  ← Image generation script (Nano Banana 2)
├── skills/
│   ├── skill.md                     ← Full process instructions
│   └── references/
│       └── template-prompts.md      ← 40 ad templates
└── brands/                          ← Your brands go here
    └── [brand-name]/
        ├── brand-dna.md             ← Phase 1 output
        ├── prompts.json             ← Phase 2 output
        ├── product-images/          ← Your product photos
        └── outputs/                 ← Generated ad images
```

## The 40 Ad Types
1. Headline  2. Offer/Promotion  3. Testimonial  4. Feature/Benefits  5. Before/After  6. Ingredient Breakdown  7. Us vs. Them  8. Social Proof/Stats  9. Lifestyle  10. Unboxing  11. Problem/Agitation  12. Minimalist  13. Social Comment  14. Countdown/Urgency  15. Negative Marketing  16. How It Works  17. Founder's Story  18. Bundle  19. Infographic  20. Seasonal  21. Meme  22. Press/As Seen In  23. Ingredient Spotlight  24. Review Collage  25. Routine/Ritual  26. Size Comparison  27. Text-Only  28. Satisfaction Guarantee  29. Texture Close-Up  30. Endorsement  31. Flavor Lineup  32. Quick Tip  33. Challenge/Dare  34. Subscription  35. Comparison Table  36. Mood Board  37. UGC-Style  38. Award/Badge  39. Scarcity  40. Thank You/Community

## Notes
- Model: Nano Banana 2 (`gemini-3.1-flash-image-preview`) via Google Gemini API
- Free tier available with Google AI Studio API key
- Convert AVIF/WebP images to PNG before adding to product-images
- ~10 seconds per image generation

---
> Source: [Yuzzyuk/UGC-Studio](https://github.com/Yuzzyuk/UGC-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
