## godot-ui-automation

> **IMPORTANT**: As of Godot 4.5, `modulate` does NOT work on SVG files. To change SVG icon colors, you must edit the SVG file directly and change the `fill` attribute.

# Godot UI Automation - Project Context

## Godot-Specific Notes

### SVG Icon Colors
**IMPORTANT**: As of Godot 4.5, `modulate` does NOT work on SVG files. To change SVG icon colors, you must edit the SVG file directly and change the `fill` attribute.

Example:
```xml
<!-- Dark icon (won't respond to modulate) -->
<path ... fill="#212121"/>

<!-- Change to white by editing the file -->
<path ... fill="#FFFFFF"/>
```

Always edit SVG files directly when changing colors - do not attempt to use `modulate` or `self_modulate` on Button icons.

---
> Source: [graydwarf/godot-ui-automation](https://github.com/graydwarf/godot-ui-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
