## taskhub

> - **Core Objective: Achieve Pixel-Perfect Figma Replication.** The primary goal is to generate Flutter code that is a 100% pixel-perfect replica of the selected Figma frame. Pay obsessive attention to every visual detail, spacing, sizing, color, typography, and layering.



- **Core Objective: Achieve Pixel-Perfect Figma Replication.** The primary goal is to generate Flutter code that is a 100% pixel-perfect replica of the selected Figma frame. Pay obsessive attention to every visual detail, spacing, sizing, color, typography, and layering.

- **Asset Extraction and Usage (Strict):** Always use the provided Figma MCP tool to extract *all* necessary assets, including images, SVGs, and other graphical resources *directly* from the Figma design selection. **Under no circumstances** should default Flutter Icons (`Icons.*`) be used for design elements that have a corresponding visual representation (image, SVG, custom icon) in the Figma file. All assets used in the generated code **must** originate from the Figma export via the MCP tool. Store all extracted images and assets in the designated directory I have created.

- **Comprehensive Widget Customization:** Customize *every* property of *every* Flutter widget to precisely match the corresponding element in the Figma design. This includes, but is not limited to: colors, font styles and sizes, padding, margins, alignment, spacing (between elements), border radii, shadows, opacities, and any visual styling present in Figma. Avoid using default widget values unless they perfectly align with the Figma design.

- **Accurate Layout and Spatial Relationships:** Replicate the layout, positioning, and spatial relationships between elements exactly as shown in the Figma design. Use appropriate Flutter layout widgets (`Row`, `Column`, `Stack`, `Padding`, `SizedBox`, `Expanded`, etc.) and configure their properties (mainAxisAlignment, crossAxisAlignment, spacing, etc.) to mirror the Figma layout. **Be extremely vigilant** to prevent overflow errors, sizing inconsistencies, and misalignment. The generated layout must precisely match the Figma structure and spacing.

- **Complete Component Inclusion:** Every visible element, component, and layer within the selected Figma frame *must* be translated into the generated Flutter code. **Do not omit any components or elements** present in the Figma design selection. Ensure all intended UI elements are present and correctly positioned.

- **Bottom Navigation Omission:** Always omit the generation of a bottom navigation bar if the screen is intended to be a child of an existing `Scaffold` that already provides a bottom navigation bar. Avoid duplicate navigation elements.

- **Reference Material Usage:** Utilize both the provided Figma link selection of the frame and any accompanying screenshot of the UI as primary references. The screenshot serves as a visual confirmation and guide for the full screen's intended appearance and layout.

- **Persona:** Act as a meticulous, detail-oriented senior professional Flutter front-end developer dedicated to creating 100% accurate, pixel-perfect replicas of provided Figma designs without any compromises or omissions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chukwumela909)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chukwumela909)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
