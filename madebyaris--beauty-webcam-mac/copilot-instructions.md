## performance-optimization

> Performance optimization guidelines for BeautyWebcam video processing


# Performance Optimization Guidelines

## Target Performance Metrics
- **CPU Usage**: <15% on M1 MacBook Air during active processing
- **Memory Usage**: <150MB total application footprint
- **GPU Usage**: <30% Metal compute utilization
- **Latency**: <50ms from capture to virtual camera output
- **Frame Rate**: Consistent 30fps minimum, 60fps target

## Memory Management for Performance

### Buffer Management
```objc
// Use CVPixelBufferPool for efficient buffer reuse
@property (nonatomic, strong) CVPixelBufferPoolRef pixelBufferPool;

- (void)setupPixelBufferPool {
    NSDictionary *attributes = @{
        (NSString *)kCVPixelBufferPixelFormatTypeKey: @(kCVPixelFormatType_32BGRA),
        (NSString *)kCVPixelBufferWidthKey: @(1920),
        (NSString *)kCVPixelBufferHeightKey: @(1080),
        (NSString *)kCVPixelBufferMetalCompatibilityKey: @YES
    };
    
    CVPixelBufferPoolCreate(kCFAllocatorDefault, NULL, 
                           (__bridge CFDictionaryRef)attributes, 
                           &_pixelBufferPool);
}
```

### Autorelease Pool Usage
```objc
// Critical: Use autorelease pools in processing loops
- (void)processVideoFrames {
    while (self.isProcessing) {
        @autoreleasepool {
            CVPixelBufferRef frame = [self captureNextFrame];
            [self processFrame:frame];
            CVPixelBufferRelease(frame);
        }
    }
}
```

### Memory Pool Patterns
```objc
// Implement object pooling for frequently created objects
@interface BWObjectPool : NSObject
- (id)borrowObject;
- (void)returnObject:(id)object;
@end

// Use for expensive-to-create processing objects
@property (nonatomic, strong) BWObjectPool *processorPool;
```

## GPU Acceleration with Metal

### Compute Shader Best Practices
```objc
// Optimal threadgroup sizes for different operations
static const MTLSize kSkinSmoothingThreadgroupSize = {16, 16, 1};
static const MTLSize kColorCorrectionThreadgroupSize = {32, 8, 1};

// Use appropriate texture formats
MTLTextureDescriptor *descriptor = [MTLTextureDescriptor 
    texture2DDescriptorWithPixelFormat:MTLPixelFormatBGRA8Unorm
                                 width:width
                                height:height
                             mipmapped:NO];
descriptor.usage = MTLTextureUsageShaderRead | MTLTextureUsageShaderWrite;
```

### Command Buffer Optimization
```objc
// Batch operations into single command buffer
- (void)processFrameWithMetal:(CVPixelBufferRef)pixelBuffer {
    id<MTLCommandBuffer> commandBuffer = [self.commandQueue commandBuffer];
    
    // Batch multiple compute operations
    [self addSkinSmoothingToCommandBuffer:commandBuffer];
    [self addColorCorrectionToCommandBuffer:commandBuffer];
    [self addNoiseReductionToCommandBuffer:commandBuffer];
    
    [commandBuffer commit];
    [commandBuffer waitUntilCompleted];
}
```

## Core Image Optimization

### Filter Chain Optimization
```objc
// Chain filters efficiently to minimize intermediate textures
CIFilter *filter1 = [CIFilter filterWithName:@"CIBilateralFilter"];
CIFilter *filter2 = [CIFilter filterWithName:@"CIColorControls"];

// Connect filters directly instead of using intermediate images
filter2.inputImage = filter1.outputImage;
```

### Context Reuse
```objc
// Reuse CIContext instances - they're expensive to create
@property (nonatomic, strong) CIContext *metalContext;

- (void)setupCoreImageContext {
    self.metalContext = [CIContext contextWithMTLDevice:self.metalDevice
                                                options:@{
        kCIContextWorkingColorSpace: [NSNull null],
        kCIContextUseSoftwareRenderer: @NO
    }];
}
```

## Threading and Concurrency

### Processing Pipeline Threading
```objc
// Separate queues for different pipeline stages
@property (nonatomic, strong) dispatch_queue_t captureQueue;
@property (nonatomic, strong) dispatch_queue_t processingQueue;
@property (nonatomic, strong) dispatch_queue_t outputQueue;

- (void)setupProcessingPipeline {
    // High priority for capture to prevent frame drops
    dispatch_queue_attr_t captureAttr = dispatch_queue_attr_make_with_qos_class(
        DISPATCH_QUEUE_SERIAL, QOS_CLASS_USER_INTERACTIVE, 0);
    self.captureQueue = dispatch_queue_create("capture", captureAttr);
    
    // Background processing queue
    dispatch_queue_attr_t processAttr = dispatch_queue_attr_make_with_qos_class(
        DISPATCH_QUEUE_CONCURRENT, QOS_CLASS_DEFAULT, 0);
    self.processingQueue = dispatch_queue_create("processing", processAttr);
}
```

### Lock-Free Programming
```objc
// Use atomic operations for simple state management
@property (atomic, assign) BOOL isProcessing;
@property (atomic, assign) NSInteger frameCount;

// Use dispatch_semaphore for resource limiting
@property (nonatomic, strong) dispatch_semaphore_t frameSemaphore;

- (void)processFrame:(CVPixelBufferRef)frame {
    // Limit concurrent processing
    dispatch_semaphore_wait(self.frameSemaphore, DISPATCH_TIME_FOREVER);
    
    dispatch_async(self.processingQueue, ^{
        // Process frame
        [self enhanceFrame:frame];
        dispatch_semaphore_signal(self.frameSemaphore);
    });
}
```

## Algorithm Optimization

### Bilateral Filter Implementation
```objc
// Use separable filters when possible
- (void)applySkinSmoothingOptimized:(CVPixelBufferRef)pixelBuffer {
    // Two-pass separable bilateral filter is faster than full kernel
    [self applyHorizontalBilateralFilter:pixelBuffer];
    [self applyVerticalBilateralFilter:pixelBuffer];
}
```

### Lookup Table Usage
```objc
// Pre-compute expensive operations in lookup tables
@property (nonatomic, strong) NSData *gammaCorrectionLUT;
@property (nonatomic, strong) NSData *skinToneLUT;

- (void)generateLookupTables {
    // Generate 256-entry LUT for gamma correction
    uint8_t *gammaLUT = malloc(256);
    for (int i = 0; i < 256; i++) {
        gammaLUT[i] = (uint8_t)(255.0 * pow(i / 255.0, 1.0/2.2));
    }
    self.gammaCorrectionLUT = [NSData dataWithBytes:gammaLUT length:256];
    free(gammaLUT);
}
```

## Performance Monitoring

### Real-time Metrics Collection
```objc
// Track performance metrics
@property (nonatomic, assign) CFAbsoluteTime lastFrameTime;
@property (nonatomic, assign) NSInteger processedFrameCount;
@property (nonatomic, assign) double averageProcessingTime;

- (void)updatePerformanceMetrics {
    CFAbsoluteTime currentTime = CFAbsoluteTimeGetCurrent();
    double frameTime = currentTime - self.lastFrameTime;
    
    // Exponential moving average
    self.averageProcessingTime = 0.1 * frameTime + 0.9 * self.averageProcessingTime;
    
    // Log performance warnings
    if (frameTime > 0.033) { // >33ms indicates dropped frames at 30fps
        NSLog(@"Performance warning: Frame processing took %.2fms", frameTime * 1000);
    }
}
```

### Memory Usage Monitoring
```objc
// Monitor memory usage and pressure
- (void)checkMemoryPressure {
    dispatch_source_t memoryPressureSource = 
        dispatch_source_create(DISPATCH_SOURCE_TYPE_MEMORYPRESSURE, 0, 
                              DISPATCH_MEMORYPRESSURE_WARN | DISPATCH_MEMORYPRESSURE_CRITICAL, 
                              dispatch_get_global_queue(QOS_CLASS_BACKGROUND, 0));
    
    dispatch_source_set_event_handler(memoryPressureSource, ^{
        dispatch_source_memorypressure_flags_t flags = 
            dispatch_source_get_data(memoryPressureSource);
        
        if (flags & DISPATCH_MEMORYPRESSURE_CRITICAL) {
            [self reduceMemoryUsage];
        }
    });
    
    dispatch_resume(memoryPressureSource);
}
```

## Optimization Patterns

### Adaptive Quality Scaling
```objc
// Automatically reduce quality under load
- (void)adaptProcessingQuality {
    double cpuUsage = [self getCurrentCPUUsage];
    
    if (cpuUsage > 80.0) {
        self.processingQuality = BWProcessingQualityLow;
    } else if (cpuUsage > 60.0) {
        self.processingQuality = BWProcessingQualityMedium;
    } else {
        self.processingQuality = BWProcessingQualityHigh;
    }
}
```

### Frame Rate Management
```objc
// Dynamic frame rate adjustment
- (void)adjustFrameRate {
    if (self.averageProcessingTime > 0.033) {
        // Reduce to 24fps if we can't maintain 30fps
        self.targetFrameRate = 24;
    } else if (self.averageProcessingTime < 0.016) {
        // Increase to 60fps if we have headroom
        self.targetFrameRate = 60;
    }
}
```

## Cache Optimization

### Intelligent Caching
```objc
// Cache processed results based on input parameters
@property (nonatomic, strong) NSCache *processedFrameCache;

- (CIImage *)getCachedProcessedFrame:(CIImage *)inputImage 
                          parameters:(BWProcessingParameters *)params {
    NSString *cacheKey = [self generateCacheKey:inputImage parameters:params];
    return [self.processedFrameCache objectForKey:cacheKey];
}
```

### Texture Reuse
```objc
// Maintain texture pools for different sizes
@property (nonatomic, strong) NSMutableDictionary *texturePoolsBySize;

- (id<MTLTexture>)getReusableTextureWithSize:(MTLSize)size {
    NSString *sizeKey = [NSString stringWithFormat:@"%lux%lu", size.width, size.height];
    NSMutableArray *pool = self.texturePoolsBySize[sizeKey];
    
    if (pool.count > 0) {
        id<MTLTexture> texture = pool.lastObject;
        [pool removeLastObject];
        return texture;
    }
    
    // Create new texture if pool is empty
    return [self createTextureWithSize:size];
}
```

## Performance Testing Requirements

1. **Benchmark all critical paths** with Instruments
2. **Profile memory allocations** regularly
3. **Test on minimum spec hardware** (Intel MacBook Air 2020)
4. **Monitor thermal throttling** during extended use
5. **Validate frame timing** under various system loads

---
> Source: [madebyaris/beauty-webcam-mac](https://github.com/madebyaris/beauty-webcam-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
