## lt-prompter-rules

> 1.	UI Language is Always German


General Principles
	1.	UI Language is Always German
	•	All UI labels, messages, and instructional texts must be in German.
	•	Ensure consistency in terminology across the entire application.
	•	Example: Use “Einstellungen speichern” instead of “Save settings”.
	2.	Adopt Standalone Components
	•	With Angular 18, components, directives, and pipes are standalone by default, eliminating the need for NgModules.  ￼
	•	Define components with the standalone: true property:

@Component({
  selector: 'app-example',
  templateUrl: './example.component.html',
  styleUrls: ['./example.component.scss'],
  standalone: true,
})
export class ExampleComponent {}


	3.	Implement Zoneless Change Detection
	•	Angular 18 introduces experimental support for zoneless change detection, reducing reliance on zone.js and potentially enhancing performance.  ￼
	•	To enable zoneless change detection: ￼

bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});


	•	Remove zone.js from your polyfills in angular.json. ￼

	4.	Utilize Built-in Control Flow Syntax
	•	Angular 18 stabilizes the new control flow syntax, allowing for more concise templates: ￼

@if (condition) {
  <!-- Content -->
} @else {
  <!-- Alternative Content -->
}


	•	Migrate existing templates using the provided schematics: ￼

ng g @angular/core:control-flow



⸻

Frontend Development (Angular)
	5.	Use Angular Material for UI Components
	•	Adhere to Material Design principles for a consistent look and feel.
	•	Utilize Angular Material components, ensuring they are compatible with zoneless change detection. ￼
	6.	Component Structure & Reusability
	•	Adopt the Smart (container) and Dumb (presentational) components approach.  ￼
	•	Extract reusable UI elements into dedicated components to promote maintainability.
	7.	Efficient State Management
	•	Leverage RxJS for state management, using the async pipe to handle subscriptions automatically.
	•	Consider using Signals for local state that doesn’t leave the browser, as recommended by the community.  ￼
	8.	IndexedDB for Local Data Storage
	•	Store user preferences and templates in IndexedDB using Angular’s idb library.
	•	Ensure fallback mechanisms are in place if IndexedDB is unavailable.
	9.	Performance Optimization
	•	Implement lazy loading for feature components to improve initial load times.
	•	Use the @defer directive to load content conditionally, enhancing performance.  ￼

⸻

Backend & Data Handling
	10.	Data Persistence & Future Cloud Syncing
	•	Design data structures compatible with both client-side storage and potential server-side storage, facilitating future migration to services like Supabase.
	11.	Form Handling & Validation
	•	Utilize Reactive Forms for scalable and robust form management.
	•	Implement both client-side and server-side validation to ensure data integrity.
	12.	Security Best Practices
	•	Sanitize all user inputs to prevent XSS and injection attacks.
	•	Implement Content Security Policy (CSP) headers to enhance security.
	•	Apply proper rate limiting for API calls to mitigate abuse.

⸻

User Experience (UX) & Accessibility
	13.	Intuitive Sidebar Navigation
	•	Ensure the sidebar is structured with clear section labels in German, facilitating easy navigation.
	14.	Dark Mode & Theme Management
	•	Use Angular Material’s Theming System to support light/dark mode switching.
	•	Allow users to set their preferred theme, storing preferences in IndexedDB.
	15.	Offline Support with PWA Features
	•	Configure service workers to cache essential assets for offline use.
	•	Implement background sync to allow users to continue working offline.
	16.	Localization & German Text Consistency
	•	All hardcoded text should be in German, ensuring a consistent user experience.
	•	Example: const errorMessage = "Fehler beim Laden der Daten"; instead of English error messages.

⸻

Deployment & Hosting
	17.	Netlify for Hosting & CI/CD
	•	Automate deployments via Netlify’s Git-based CI/CD pipeline.
	•	Optimize build settings for efficient deployment processes.
	18.	Service Workers for Performance
	•	Configure service workers for offline mode and caching using Angular’s PWA support.
	•	Ensure updates are automatically detected and prompted to users.

⸻

---
> Source: [BensAppForge/lt-prompter](https://github.com/BensAppForge/lt-prompter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
