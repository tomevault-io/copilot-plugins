## watching

> When testing providers


1. Always create a new `ProviderContainer` (unit tests) or `ProviderScope` (widget tests) for each test to avoid shared state between tests. Use a utility like `createContainer()` to set up and automatically dispose containers (see `/references/riverpod/testing/create_container.dart`).
2. In unit tests, never share `ProviderContainer` instances between tests. Example:
   ```dart
   final container = createContainer();
   expect(container.read(provider), equals('some value'));
   ```
3. In widget tests, always wrap your widget tree with `ProviderScope` when using `tester.pumpWidget`. Example:
   ```dart
   await tester.pumpWidget(
     const ProviderScope(child: YourWidgetYouWantToTest()),
   );
   ```
4. Obtain a `ProviderContainer` in widget tests using `ProviderScope.containerOf(BuildContext)`. Example:
   ```dart
   final element = tester.element(find.byType(YourWidgetYouWantToTest));
   final container = ProviderScope.containerOf(element);
   ```
5. After obtaining the container, you can read or interact with providers as needed for assertions. Example:
   ```dart
   expect(container.read(provider), 'some value');
   ```
6. For providers with `autoDispose`, prefer `container.listen` over `container.read` to prevent the provider's state from being disposed during the test.
7. Use `container.read` to read provider values and `container.listen` to listen to provider changes in tests.
8. Use the `overrides` parameter on `ProviderScope` or `ProviderContainer` to inject mocks or fakes for providers in your tests.
9. Use `container.listen` to spy on changes in a provider for assertions or to combine with mocking libraries.
10. Await asynchronous providers in tests by reading the `.future` property (for `FutureProvider`) or listening to streams.
11. Prefer mocking dependencies (such as repositories) used by Notifiers rather than mocking Notifiers directly.
12. If you must mock a Notifier, subclass the original Notifier base class instead of using `implements` or `with Mock`.
13. Place Notifier mocks in the same file as the Notifier being mocked if code generation is used, to access generated classes.
14. Use the `overrides` parameter to swap out Notifiers or providers for mocks or fakes in tests.
15. Keep all test-specific setup and teardown logic inside the test body or test utility functions. Avoid global state.
16. Ensure your test environment closely matches your production environment for reliable results.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonhygg99)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jonhygg99)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
