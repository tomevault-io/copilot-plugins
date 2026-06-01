## test-page

> This rule provides guidance on when to use the TestPage component and example components versus creating new components.

# Test Page Component Usage Guidelines

This rule provides guidance on when to use the TestPage component and example components versus creating new components.

<rule>
name: test_page_component_usage
description: Guidelines for using TestPage and example components versus creating new components
filters:
  # Match React component files
  - type: file_extension
    pattern: "\\.(jsx|tsx)$"
  # Match import statements that reference TestPage or example components
  - type: content
    pattern: "import.*(?:TestPage|CounterButton|ClaudeChat|OpenAIChat|GeminiChat|FireflyImageGenerator|FireflyVideoGenerator|FalImageGenerator|StockComponent|AudioMusicGenerator)"
  # Match component usage in JSX
  - type: content
    pattern: "<(?:TestPage|CounterButton|ClaudeChat|OpenAIChat|GeminiChat|FireflyImageGenerator|FireflyVideoGenerator|FalImageGenerator|StockComponent|AudioMusicGenerator)"

actions:
  - type: suggest
    conditions:
      # When TestPage is not displayed in App.tsx
      - pattern: "^(?!.*<TestPage.*/>).*$"
        context: "src/App.tsx"
        message: "TestPage is not displayed - prioritize creating new components"
      
    message: |
      ## Test Page Component Usage Guidelines

      ### When TestPage is NOT displayed in App.tsx:
      1. **Prioritize creating new components** based on user requirements
      2. **Do not use existing example components** unless specifically requested
      3. **Use example components only as reference** for:
         - Understanding API patterns
         - Learning service integration
         - Seeing component structure examples
      4. **Create fresh, purpose-built components** that:
         - Meet the specific user requirements
         - Follow Adobe Spectrum design patterns
         - Are tailored to the user's needs

      ### When TestPage IS displayed in App.tsx:
      1. **Example components are available for use** and testing
      2. **Can modify existing components** if they meet user needs
      3. **Can use example components** for rapid prototyping

      ### Example Components (for reference only when TestPage not displayed):
      - `CounterButton` - Basic state management example
      - `ClaudeChat` - AI chat integration pattern
      - `OpenAIChat` - OpenAI API usage example
      - `GeminiChat` - Gemini API integration example
      - `FireflyImageGenerator` - Image generation workflow
      - `FireflyVideoGenerator` - Video generation workflow
      - `FalImageGenerator` - Fal AI service usage
      - `StockComponent` - Stock API integration
      - `AudioMusicGenerator` - Audio service usage

      ### Best Practices:
      1. **Always check App.tsx first** to see if TestPage is displayed
      2. **Create new components** that are specific to user requirements
      3. **Use services directly** rather than copying example component patterns
      4. **Focus on user experience** over reusing existing examples
      5. **Build components from scratch** using Adobe Spectrum for consistency

examples:
  - input: |
      // When TestPage is not displayed in App.tsx
      // User asks: "Create a music player component"
      
      // Bad approach:
      import AudioMusicGenerator from './components/AudioMusicGenerator'
      
      // Good approach:
      // Create a new MusicPlayer component from scratch
      import { View, Heading, Button } from '@adobe/react-spectrum'
      import { generateMusic } from '../services'
      
      const MusicPlayer: React.FC = () => {
        // Build component specific to user needs
      }
    output: "Created purpose-built component instead of reusing example"

  - input: |
      // When TestPage IS displayed in App.tsx
      // User asks: "Show me how to generate music"
      
      // Acceptable approach:
      import AudioMusicGenerator from './components/AudioMusicGenerator'
      // Can use or modify existing example component
    output: "Using example component is acceptable when TestPage is displayed"

metadata:
  priority: high
  version: 1.0
  tags:
    - component-usage
    - development-workflow
    - user-experience
</rule>

---
> Source: [alexawilcox1120/TestingCursor](https://github.com/alexawilcox1120/TestingCursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
