## lailms

> - **Wrap Key Components with `LunaContextElement`**

- **Wrap Key Components with `LunaContextElement`**
    - Any UI component that displays significant information (text, data summaries, lists) or provides key interaction points (forms, complex controls) MUST be wrapped with [`LunaContextElement`](mdc:src/components/luna/LunaContextElement.tsx).
    - This ensures the AI assistant (Luna) has the necessary context about the user's current view and available actions.

- **Provide Meaningful Context Data**
    - **`type` (string, required):** A descriptive identifier for the component type (e.g., `course-structure`, `lesson-content`, `user-profile-form`).
    - **`role` (string, required):** The role this component plays in the UI (e.g., `display`, `form`, `navigation`, `control`).
    - **`content` (object, optional):** Include the core textual or structured data displayed by the component. This is crucial for Luna to understand *what* the user is seeing.
        - Example: `{ title: "Module 1", topics: ["Topic A", "Topic B"] }`
    - **`state` (object, optional):** Include relevant state information about the component (e.g., `{ isOpen: true, currentTab: 'details' }`).
    - **`metadata` (object, optional):** Include identifiers or other non-display data useful for context or actions (e.g., `{ lessonId: 'uuid-123', sectionId: 'sec-456' }`).
    - **`actionable` (boolean, optional):** Set to `true` (default) if Luna should be able to potentially interact with this component (requires corresponding backend functions).

- **Examples**

    ```typescript
    // ✅ DO: Wrap components displaying important data and provide context
    import LunaContextElement from '@/components/luna/LunaContextElement';

    function CourseDetails({ course }) {
      return (
        <LunaContextElement
          type="course-details"
          role="display"
          content={{ title: course.title, description: course.description }}
          metadata={{ courseId: course.id }}
        >
          <div>
            <h1>{course.title}</h1>
            <p>{course.description}</p>
          </div>
        </LunaContextElement>
      );
    }

    // ❌ DON'T: Forget to wrap key components or omit crucial 'content'
    function CourseDetails({ course }) {
      // This component's content is invisible to Luna
      return (
        <div>
          <h1>{course.title}</h1>
          <p>{course.description}</p>
        </div>
      );
    }
    ```

- **Rationale**
    - Consistent use of `LunaContextElement` makes the application significantly more intuitive for AI assistance, allowing Luna to provide more relevant help, answer questions about on-screen content, and potentially automate actions.
    - Refer to [`LunaContextProvider.tsx`](mdc:src/context/LunaContextProvider.tsx) for the context management logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dbawebdesign) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
