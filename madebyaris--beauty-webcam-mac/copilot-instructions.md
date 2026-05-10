## metal-shaders

> Metal shader coding standards and optimization guidelines for BeautyWebcam


# Metal Shader Guidelines

## Shader Architecture

### File Organization
```metal
// BeautyWebcam_Shaders.metal
#include <metal_stdlib>
using namespace metal;

// Constants
constant float kSkinSmoothingRadius = 2.0;
constant float kColorEnhancementStrength = 0.3;

// Shared utility functions
float luminance(float3 color);
float3 rgb2yuv(float3 rgb);
float3 yuv2rgb(float3 yuv);

// Main processing kernels
kernel void skinSmoothingKernel(...);
kernel void colorEnhancementKernel(...);
kernel void noiseReductionKernel(...);
```

### Naming Conventions
- **Kernels**: Use descriptive names ending with "Kernel" - e.g., `skinSmoothingKernel`
- **Constants**: Use `k` prefix with camelCase - e.g., `kBilateralSigmaD`
- **Structs**: Use `BW` prefix - e.g., `BWProcessingParams`
- **Functions**: Use camelCase - e.g., `calculateBilateralWeight`

## Performance Optimization

### Thread Group Sizing
```metal
// Optimal thread group sizes for different operations
// 16x16 for 2D image processing (256 threads per group)
kernel void imageProcessingKernel(texture2d<float, access::read> inputTexture [[texture(0)]],
                                 texture2d<float, access::write> outputTexture [[texture(1)]],
                                 uint2 gid [[thread_position_in_grid]]) {
    // Process pixel at gid
}

// Use from Objective-C:
// MTLSize threadgroupSize = MTLSizeMake(16, 16, 1);
// MTLSize threadgroupCount = MTLSizeMake((width + 15) / 16, (height + 15) / 16, 1);
```

### Memory Access Patterns
```metal
// Prefer coalesced memory access
kernel void efficientKernel(texture2d<float, access::read> input [[texture(0)]],
                           texture2d<float, access::write> output [[texture(1)]],
                           uint2 gid [[thread_position_in_grid]]) {
    if (gid.x >= input.get_width() || gid.y >= input.get_height()) return;
    
    // Good: Access neighboring pixels in a pattern that maximizes cache hits
    float4 center = input.read(gid);
    float4 right = input.read(gid + uint2(1, 0));
    float4 down = input.read(gid + uint2(0, 1));
    
    // Process and write result
    output.write(processPixels(center, right, down), gid);
}
```

### Threadgroup Memory Usage
```metal
// Use threadgroup memory for shared computations
kernel void bilateralFilterKernel(texture2d<float, access::read> input [[texture(0)]],
                                 texture2d<float, access::write> output [[texture(1)]],
                                 uint2 gid [[thread_position_in_grid]],
                                 uint2 tid [[thread_position_in_threadgroup]],
                                 threadgroup float4 *sharedMemory [[threadgroup(0)]]) {
    
    // Load data into shared memory with border handling
    uint sharedIndex = tid.y * 18 + tid.x; // 18x18 for 16x16 + 1-pixel border
    if (tid.x < 18 && tid.y < 18) {
        uint2 loadPos = gid + tid - uint2(1, 1); // Offset for border
        sharedMemory[sharedIndex] = input.read(loadPos);
    }
    
    threadgroup_barrier(mem_flags::mem_threadgroup);
    
    // Use shared data for filtering
    if (tid.x < 16 && tid.y < 16) {
        float4 result = bilateralFilter(sharedMemory, tid);
        output.write(result, gid);
    }
}
```

## Image Processing Algorithms

### Bilateral Filter Implementation
```metal
// Optimized bilateral filter for skin smoothing
float bilateralWeight(float2 spatialDistance, float colorDistance, 
                     float sigmaD, float sigmaR) {
    float spatialWeight = exp(-dot(spatialDistance, spatialDistance) / (2.0 * sigmaD * sigmaD));
    float colorWeight = exp(-(colorDistance * colorDistance) / (2.0 * sigmaR * sigmaR));
    return spatialWeight * colorWeight;
}

kernel void skinSmoothingKernel(texture2d<float, access::read> input [[texture(0)]],
                               texture2d<float, access::write> output [[texture(1)]],
                               constant BWProcessingParams &params [[buffer(0)]],
                               uint2 gid [[thread_position_in_grid]]) {
    
    if (gid.x >= input.get_width() || gid.y >= input.get_height()) return;
    
    float4 centerPixel = input.read(gid);
    float3 centerColor = centerPixel.rgb;
    
    float3 filteredColor = float3(0.0);
    float totalWeight = 0.0;
    
    // Sample in kernel radius
    int radius = int(params.smoothingRadius);
    for (int dy = -radius; dy <= radius; dy++) {
        for (int dx = -radius; dx <= radius; dx++) {
            uint2 samplePos = uint2(max(0, min(int(input.get_width()) - 1, int(gid.x) + dx)),
                                   max(0, min(int(input.get_height()) - 1, int(gid.y) + dy)));
            
            float4 samplePixel = input.read(samplePos);
            float3 sampleColor = samplePixel.rgb;
            
            float colorDist = distance(centerColor, sampleColor);
            float2 spatialDist = float2(dx, dy);
            
            float weight = bilateralWeight(spatialDist, colorDist, 
                                         params.sigmaD, params.sigmaR);
            
            filteredColor += weight * sampleColor;
            totalWeight += weight;
        }
    }
    
    filteredColor /= totalWeight;
    
    // Blend with original based on intensity
    float3 finalColor = mix(centerColor, filteredColor, params.smoothingIntensity);
    
    output.write(float4(finalColor, centerPixel.a), gid);
}
```

### Color Enhancement
```metal
// HSV color space manipulation for natural color enhancement
float3 rgb2hsv(float3 rgb) {
    float maxVal = max(max(rgb.r, rgb.g), rgb.b);
    float minVal = min(min(rgb.r, rgb.g), rgb.b);
    float delta = maxVal - minVal;
    
    float3 hsv;
    hsv.z = maxVal; // Value
    hsv.y = (maxVal > 0.0) ? (delta / maxVal) : 0.0; // Saturation
    
    // Hue calculation
    if (delta > 0.0) {
        if (maxVal == rgb.r) {
            hsv.x = (rgb.g - rgb.b) / delta;
        } else if (maxVal == rgb.g) {
            hsv.x = 2.0 + (rgb.b - rgb.r) / delta;
        } else {
            hsv.x = 4.0 + (rgb.r - rgb.g) / delta;
        }
        hsv.x /= 6.0;
        if (hsv.x < 0.0) hsv.x += 1.0;
    } else {
        hsv.x = 0.0;
    }
    
    return hsv;
}

float3 hsv2rgb(float3 hsv) {
    float h = hsv.x * 6.0;
    float s = hsv.y;
    float v = hsv.z;
    
    int i = int(floor(h));
    float f = h - float(i);
    float p = v * (1.0 - s);
    float q = v * (1.0 - s * f);
    float t = v * (1.0 - s * (1.0 - f));
    
    switch (i % 6) {
        case 0: return float3(v, t, p);
        case 1: return float3(q, v, p);
        case 2: return float3(p, v, t);
        case 3: return float3(p, q, v);
        case 4: return float3(t, p, v);
        case 5: return float3(v, p, q);
        default: return float3(v, v, v);
    }
}

kernel void colorEnhancementKernel(texture2d<float, access::read> input [[texture(0)]],
                                  texture2d<float, access::write> output [[texture(1)]],
                                  constant BWColorParams &params [[buffer(0)]],
                                  uint2 gid [[thread_position_in_grid]]) {
    
    if (gid.x >= input.get_width() || gid.y >= input.get_height()) return;
    
    float4 pixel = input.read(gid);
    float3 hsv = rgb2hsv(pixel.rgb);
    
    // Enhance saturation
    hsv.y = saturate(hsv.y * params.saturationMultiplier);
    
    // Adjust brightness
    hsv.z = saturate(hsv.z * params.brightnessMultiplier);
    
    // Convert back to RGB
    float3 enhancedRGB = hsv2rgb(hsv);
    
    // Apply warmth adjustment
    enhancedRGB.r = saturate(enhancedRGB.r * params.warmth);
    enhancedRGB.b = saturate(enhancedRGB.b / params.warmth);
    
    output.write(float4(enhancedRGB, pixel.a), gid);
}
```

### Noise Reduction
```metal
// Edge-preserving noise reduction
float edgeStrength(texture2d<float, access::read> input, uint2 pos) {
    float4 center = input.read(pos);
    float4 right = input.read(pos + uint2(1, 0));
    float4 down = input.read(pos + uint2(0, 1));
    
    float dx = length(center.rgb - right.rgb);
    float dy = length(center.rgb - down.rgb);
    
    return sqrt(dx * dx + dy * dy);
}

kernel void noiseReductionKernel(texture2d<float, access::read> input [[texture(0)]],
                                texture2d<float, access::write> output [[texture(1)]],
                                constant BWNoiseParams &params [[buffer(0)]],
                                uint2 gid [[thread_position_in_grid]]) {
    
    if (gid.x >= input.get_width() || gid.y >= input.get_height()) return;
    
    float4 centerPixel = input.read(gid);
    float edge = edgeStrength(input, gid);
    
    // Reduce noise reduction near edges
    float adaptiveStrength = params.noiseReductionStrength * (1.0 - edge);
    
    // Simple box filter for noise reduction
    float3 filteredColor = float3(0.0);
    int kernelSize = 3;
    int count = 0;
    
    for (int dy = -kernelSize/2; dy <= kernelSize/2; dy++) {
        for (int dx = -kernelSize/2; dx <= kernelSize/2; dx++) {
            uint2 samplePos = uint2(max(0, min(int(input.get_width()) - 1, int(gid.x) + dx)),
                                   max(0, min(int(input.get_height()) - 1, int(gid.y) + dy)));
            
            filteredColor += input.read(samplePos).rgb;
            count++;
        }
    }
    
    filteredColor /= float(count);
    
    // Blend based on adaptive strength
    float3 finalColor = mix(centerPixel.rgb, filteredColor, adaptiveStrength);
    
    output.write(float4(finalColor, centerPixel.a), gid);
}
```

## Utility Functions

### Common Image Processing Utilities
```metal
// Luminance calculation
float luminance(float3 color) {
    return dot(color, float3(0.299, 0.587, 0.114));
}

// Gaussian weight calculation
float gaussian(float x, float sigma) {
    return exp(-(x * x) / (2.0 * sigma * sigma));
}

// Safe texture sampling with bounds checking
float4 sampleTextureSafe(texture2d<float, access::read> tex, int2 coord) {
    coord = clamp(coord, int2(0), int2(tex.get_width() - 1, tex.get_height() - 1));
    return tex.read(uint2(coord));
}

// Bilinear interpolation
float4 sampleBilinear(texture2d<float, access::read> tex, float2 uv) {
    float2 texSize = float2(tex.get_width(), tex.get_height());
    float2 coord = uv * texSize - 0.5;
    int2 coordInt = int2(floor(coord));
    float2 f = coord - float2(coordInt);
    
    float4 a = sampleTextureSafe(tex, coordInt);
    float4 b = sampleTextureSafe(tex, coordInt + int2(1, 0));
    float4 c = sampleTextureSafe(tex, coordInt + int2(0, 1));
    float4 d = sampleTextureSafe(tex, coordInt + int2(1, 1));
    
    return mix(mix(a, b, f.x), mix(c, d, f.x), f.y);
}
```

## Parameter Structures

### Consistent Parameter Passing
```metal
// Processing parameters structure
struct BWProcessingParams {
    float smoothingRadius;
    float smoothingIntensity;
    float sigmaD;
    float sigmaR;
};

struct BWColorParams {
    float saturationMultiplier;
    float brightnessMultiplier;
    float warmth;
    float contrast;
};

struct BWNoiseParams {
    float noiseReductionStrength;
    float edgeThreshold;
    int kernelSize;
};
```

## Error Handling and Validation

### Bounds Checking
```metal
// Always validate thread position
kernel void safeKernel(texture2d<float, access::read> input [[texture(0)]],
                      texture2d<float, access::write> output [[texture(1)]],
                      uint2 gid [[thread_position_in_grid]]) {
    
    // Early exit for out-of-bounds threads
    if (gid.x >= input.get_width() || gid.y >= input.get_height()) {
        return;
    }
    
    // Safe to proceed with processing
    float4 pixel = input.read(gid);
    // ... process pixel ...
    output.write(pixel, gid);
}
```

### Parameter Validation
```metal
// Validate and clamp parameters
kernel void parameterValidatedKernel(texture2d<float, access::read> input [[texture(0)]],
                                    texture2d<float, access::write> output [[texture(1)]],
                                    constant BWProcessingParams &params [[buffer(0)]],
                                    uint2 gid [[thread_position_in_grid]]) {
    
    if (gid.x >= input.get_width() || gid.y >= input.get_height()) return;
    
    // Validate and clamp parameters
    float intensity = clamp(params.smoothingIntensity, 0.0, 1.0);
    float radius = clamp(params.smoothingRadius, 1.0, 10.0);
    
    // Use validated parameters
    float4 result = processWithParams(input.read(gid), intensity, radius);
    output.write(result, gid);
}
```

## Performance Testing

### Profiling Considerations
1. **Use GPU frame capture** in Xcode for detailed analysis
2. **Monitor memory bandwidth** - texture reads/writes are often the bottleneck
3. **Test threadgroup occupancy** - aim for high GPU utilization
4. **Profile on target hardware** - different GPUs have different characteristics
5. **Measure end-to-end latency** including CPU-GPU synchronization

### Optimization Checklist
- [ ] Minimize texture reads per thread
- [ ] Use appropriate data types (half vs float)
- [ ] Optimize threadgroup sizes for target hardware
- [ ] Minimize divergent branches
- [ ] Use shared memory for repeated data access
- [ ] Profile memory access patterns

---
> Source: [madebyaris/beauty-webcam-mac](https://github.com/madebyaris/beauty-webcam-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
