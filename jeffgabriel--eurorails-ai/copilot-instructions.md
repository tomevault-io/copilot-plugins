## phaserrules

> - Phaser converts SVG files to bitmap textures at load time

# Phaser Engine Rules and Behaviors

## SVG Handling in Phaser
- Phaser converts SVG files to bitmap textures at load time
- SVGs are not rendered as vector graphics in real-time
- Modifying SVG files after they are loaded will not affect the in-game sprites
- SVG modifications require a game reload to take effect
- The main benefits of SVGs in Phaser are:
  - Smaller file sizes for distribution
  - Clean source files for asset management
  - Initial high-quality conversion to textures

## Best Practices for SVG Assets in Phaser
1. **Asset Loading**
   - SVGs should be properly sized and formatted before loading
   - Ensure SVG viewBox and dimensions are correct in the source file
   - Test SVG appearance before implementing in Phaser
   - use this.load.svg for svg based images
   - scale an svg at load time if at all possible

2. **Runtime Behavior**
   - Treat loaded SVG sprites like any other bitmap sprite
   - Use Phaser's sprite manipulation methods for runtime changes
   - Scale, rotate, and transform using Phaser's sprite properties
   - Do not attempt to modify SVG properties at runtime

3. **Troubleshooting Display Issues**
   - If sprite appears incorrect, fix the source SVG file
   - Clear browser cache and reload the game to see SVG changes
   - Use Phaser's sprite debugging tools to check texture bounds
   - Consider sprite scale and anchor points for positioning

4. **Performance Considerations**
   - SVGs are converted to textures only once at load time
   - Runtime performance is identical to PNG/JPG sprites
   - Memory usage is based on the converted texture size
   - Choose appropriate SVG complexity for target texture resolution 


5. **Container visibility**
   - Containers which need to show above others should use container.setDepth(9998);
   - Containers which should not interact with zoom or pan should use container.setScrollFactor(0);

   ## UI Element and Camera Management

### Container and Camera Organization
1. **Single Source of Truth**
   - Each UI element should be created in exactly one place
   - Avoid creating the same UI element in multiple components
   - Use a dedicated UI manager class to handle all UI element creation
   - Example: Don't create settings button in both GameScene and UIManager

2. **Camera Visibility**
   - Create a separate UI camera that ignores the map container
   - Main camera should ignore all UI containers
   - When creating new UI elements, ensure they are added to the correct container AND properly configured for camera visibility
   - Example:
     ```typescript
     // Create UI camera
     const uiCamera = this.cameras.add(0, 0, width, height);
     uiCamera.setScroll(0, 0);
     uiCamera.ignore(this.mapContainer);

     // Main camera ignores UI
     this.cameras.main.ignore([this.uiContainer, this.playerHandContainer]);
     ```

3. **Container Hierarchy**
   - Map container: For game world elements that move with the camera
   - UI container: For fixed UI elements that stay on screen
   - Player hand container: For player-specific UI elements
   - Modal containers: For temporary overlays and dialogs

4. **Common Pitfalls**
   - Creating UI elements in multiple places (e.g., both scene and manager)
   - Forgetting to set camera visibility for new UI elements
   - Not properly cleaning up containers before adding new elements
   - Mixing world and UI elements in the same container
   - Not setting appropriate depth values for UI layers

5. **Container Cleanup**
   - Always clean up containers before adding new elements
   - Use `container.removeAll(true)` to properly destroy children
   - When switching scenes or updating UI, ensure old elements are properly destroyed
   

---
> Source: [jeffgabriel/eurorails_ai](https://github.com/jeffgabriel/eurorails_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
