## phaser-perf

> 1. Excessive Render Calls

1. Excessive Render Calls
javascriptCopy// PROBLEMATIC: Redrawing entire canvas on every mouse move
this.input.on('pointermove', (pointer) => {
    if (pointer.isDown) {
        this.graphics.clear();
        this.redrawEverything(); // Heavy operation on every frame
    }
});
Solution: Use incremental drawing or dirty rectangle updates.
2. Memory Leaks from Graphics Objects
javascriptCopy// PROBLEMATIC: Creating new graphics objects without cleanup
this.input.on('pointermove', (pointer) => {
    let newGraphics = this.add.graphics(); // Never destroyed
    newGraphics.fillCircle(pointer.x, pointer.y, 5);
});
Solution: Reuse graphics objects or properly destroy them.
3. Unbounded Event Listeners
javascriptCopy// PROBLEMATIC: High-frequency events without throttling
this.input.on('pointermove', this.draw); // Fires 60+ times per second
Solution: Implement throttling or debouncing:
javascriptCopylet lastUpdate = 0;
this.input.on('pointermove', (pointer) => {
    const now = Date.now();
    if (now - lastUpdate > 16) { // ~60fps throttle
        this.draw(pointer);
        lastUpdate = now;
    }
});
4. Inefficient Path Building
javascriptCopy// PROBLEMATIC: Complex path calculations on every move
this.input.on('pointermove', (pointer) => {
    this.graphics.lineStyle(2, 0xff0000);
    this.graphics.strokePoints(this.allPoints); // Redraws entire path
});
Solution: Use line segments between points:
javascriptCopythis.input.on('pointermove', (pointer) => {
    if (this.lastPoint) {
        this.graphics.lineBetween(this.lastPoint.x, this.lastPoint.y, pointer.x, pointer.y);
    }
    this.lastPoint = pointer;
});
5. Large Texture/RenderTexture Updates
javascriptCopy// PROBLEMATIC: Updating large render textures frequently
this.input.on('pointermove', (pointer) => {
    this.renderTexture.draw(this.brush, pointer.x, pointer.y);
    this.renderTexture.saveTexture('drawing'); // Heavy operation
});
6. Collision Detection on Every Pixel
javascriptCopy// PROBLEMATIC: Checking collisions with all drawn elements
this.input.on('pointermove', (pointer) => {
    this.drawnElements.forEach(element => {
        if (this.checkPixelCollision(pointer, element)) {
            // Process collision
        }
    });
});
Best Practices to Avoid These Issues:
1. Use Object Pooling
javascriptCopyclass DrawingSystem {
    constructor() {
        this.brushPool = [];
        this.activeStrokes = [];
    }
    
    getBrush() {
        return this.brushPool.pop() || this.scene.add.graphics();
    }
    
    returnBrush(brush) {
        brush.clear();
        this.brushPool.push(brush);
    }
}
2. Implement Dirty Rectangle Updates
javascriptCopyupdateDrawing(pointer) {
    const bounds = this.calculateDirtyRect(pointer);
    this.renderTexture.drawFrame('brush', 0, pointer.x, pointer.y);
    // Only update the affected area
}
3. Use RequestAnimationFrame Pattern
javascriptCopyclass OptimizedDrawing {
    constructor() {
        this.needsUpdate = false;
        this.pendingPoints = [];
    }
    
    onPointerMove(pointer) {
        this.pendingPoints.push({x: pointer.x, y: pointer.y});
        if (!this.needsUpdate) {
            this.needsUpdate = true;
            requestAnimationFrame(() => this.processPendingDrawing());
        }
    }
    
    processPendingDrawing() {
        // Process all pending points in one batch
        this.drawPoints(this.pendingPoints);
        this.pendingPoints = [];
        this.needsUpdate = false;
    }
}
4. Monitor Performance
javascriptCopy// Add performance monitoring
const perfMonitor = {
    frameCount: 0,
    lastTime: performance.now(),
    
    update() {
        this.frameCount++;
        const now = performance.now();
        if (now - this.lastTime >= 1000) {
            console.log(`FPS: ${this.frameCount}`);
            this.frameCount = 0;
            this.lastTime = now;
        }
    }
};
The key is to minimize redundant operations, batch updates when possible, and always clean up resources properly to maintain smooth performance in your Phaser drawing applications.

---
> Source: [jeffgabriel/eurorails_ai](https://github.com/jeffgabriel/eurorails_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
