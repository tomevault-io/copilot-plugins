## veriqan

> Get best practices for Entity Framework Core

**mode**: 'agent'
**Objective**: 'Ensure that C# types are documented with XML comments and follow best practices for documentation.'
---

## C# Documentation Best Practices

- Document all public members using XML comments
- Prefer documenting internal members, especially for non-trivial logic
- Use `<summary>` to briefly explain what the member does
- Use `<param name="...">` to describe each method parameter
- Use `<returns>` to explain what the method returns
- Use `<remarks>` for extended notes or usage guidance
- Use `<example>` to provide usage demonstrations
- Use `<exception cref="...">` to list exceptions the method may throw
- Use `<see cref="...">` and `<seealso cref="...">` to reference related members or types
- Use `<inheritdoc/>` to inherit comments from base class or interface unless behavior differs
- Use `<typeparam name="...">` for describing generic type parameters
- Use `<typeparamref name="..."/>` to reference a type parameter inline
- Use `<c>inline code</c>` for small code snippets
- Use `<code>...multiline code...</code>` for code blocks

### Example

```csharp
/// <summary>
/// Calculates the area of a rectangle.
/// </summary>
/// <param name="width">The width of the rectangle.</param>
/// <param name="height">The height of the rectangle.</param>
/// <returns>The computed area.</returns>
/// <remarks>Width and height must be non-negative.</remarks>
/// <example>
/// <code>
/// var area = calculator.CalculateArea(5, 10);
/// </code>
/// </example>
/// <exception cref="ArgumentOutOfRangeException">
/// Thrown if width or height is negative.
/// </exception>
public double CalculateArea(double width, double height) { ... }

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hebelmx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
