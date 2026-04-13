## alergy

> - **Framework:** React Native (Expo Managed Workflow for speed, but keep architecture clean for potential ejection).

# Project: YumSafe (React Native + Gemini 3)
# Goal: Build a high-performance, EU-compliant AI Food Allergy Scanner.

## 1. Tech Stack
- **Framework:** React Native (Expo Managed Workflow for speed, but keep architecture clean for potential ejection).
- **Navigation:** React Navigation (Native Stack).
- **UI:** NativeWind (Tailwind CSS for React Native) for styling.
- **Camera:** expo-camera or react-native-vision-camera for real-time label capture.
- **AI Integration:** Google Generative AI SDK (Gemini 1.5 Flash for speed and cost-efficiency).
- **State Management:** React Context or Zustand for user allergy profiles.

## 2. Core Functional Requirements
- **Allergy Profile:** Allow users to select allergens from the 14 EU-mandated categories (Gluten, Peanuts, Milk, etc.).
- **Camera Scanner:** A clean UI that captures food ingredient labels.
- **Gemini Intelligence:**
    - Input: Image (Base64) + User Allergy Profile + System Instruction.
    - Logic: Use the "YumSafe" system logic to identify allergens in English, German, French, and Spanish.
    - Handling: Must detect bolded text and "May contain" (Precautionary Allergen Labelling) phrases.
- **Result UI:** A high-impact "Traffic Light" system (Red/Yellow/Green) with clear explanations.

## 3. Product Rules & Guidelines
- **EU Compliance:** Ensure all analysis aligns with EU Regulation No 1169/2011.
- **Privacy First:** Design for local-first storage. User allergy data should remain on the device.
- **Multilingual Support:** The UI and AI responses must support EN, DE, FR, ES. Use i18next for app strings.
- **Error Handling:** Gracefully handle "Blurry image," "No ingredients found," or "API timeout."

## 4. Cursor Execution Steps (Phase 1)
1. Initialize a new Expo project with TypeScript and NativeWind.
2. Setup the directory structure: `/src/components`, `/src/screens`, `/src/services/ai`, `/src/context`.
3. Create a `geminiService.ts` to handle the `google-generativeai` configuration and prompt logic.
4. Implement the 'Allergy Setup' onboarding screen.
5. Create the 'Scanner' screen with real-time camera preview and "Capture" logic.

## 5. Coding Style
- Use Functional Components and Hooks.
- Strict TypeScript: No `any` types.
- Modularize the AI logic so I can easily swap models or update system prompts later.
- Add descriptive comments for each logic block, especially for image-to-base64 conversion.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zaiyong) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
