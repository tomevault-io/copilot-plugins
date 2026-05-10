## objc-standards

> Objective-C coding standards and best practices for BeautyWebcam


# Objective-C Coding Standards

## Code Style Guidelines

### Naming Conventions
- **Classes**: Use `BW` prefix (BeautyWebcam) - e.g., `BWCaptureManager`, `BWVideoProcessor`
- **Methods**: Use descriptive, action-oriented names - e.g., `startCaptureSessionWithDevice:`
- **Properties**: Use camelCase without prefixes - e.g., `captureSession`, `isProcessing`
- **Constants**: Use `k` prefix with class prefix - e.g., `kBWMaxFrameRate`, `kBWDefaultQuality`
- **Enums**: Use NS_ENUM with descriptive names - e.g., `BWProcessingQuality`, `BWCameraState`

### Method Declarations
```objc
// Preferred: Descriptive parameter names
- (BOOL)startProcessingWithQuality:(BWProcessingQuality)quality 
                         forDevice:(AVCaptureDevice *)device 
                             error:(NSError **)error;

// Avoid: Generic parameter names
- (BOOL)start:(int)q device:(id)d error:(NSError **)e;
```

### Property Declarations
```objc
// Use atomic/nonatomic explicitly
@property (nonatomic, strong) AVCaptureSession *captureSession;
@property (nonatomic, assign) BOOL isProcessing;
@property (nonatomic, copy) NSString *deviceIdentifier;

// Use nullable/nonnull annotations
@property (nonatomic, strong, nullable) NSError *lastError;
@property (nonatomic, strong, nonnull) BWVideoProcessor *processor;
```

## Memory Management

### ARC Best Practices
- Always use `@autoreleasepool` for intensive loops
- Use `__weak` references to break retain cycles
- Use `__unsafe_unretained` only when absolutely necessary
- Implement proper cleanup in `dealloc`

```objc
// Weak reference to avoid retain cycles
@property (nonatomic, weak) id<BWCaptureDelegate> delegate;

// Cleanup in dealloc
- (void)dealloc {
    [self stopCapture];
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}
```

### Memory-Intensive Operations
```objc
// Use autorelease pools for image processing
- (void)processFrameBuffer:(CVPixelBufferRef)pixelBuffer {
    @autoreleasepool {
        CIImage *image = [[CIImage alloc] initWithCVPixelBuffer:pixelBuffer];
        // Process image...
    }
}
```

## Error Handling

### NSError Pattern
```objc
- (BOOL)performOperationWithError:(NSError **)error {
    // Validate parameters
    if (!self.isReady) {
        if (error) {
            *error = [NSError errorWithDomain:BWErrorDomain 
                                         code:BWErrorNotReady 
                                     userInfo:@{NSLocalizedDescriptionKey: @"Device not ready"}];
        }
        return NO;
    }
    
    // Perform operation
    return YES;
}
```

### Exception Handling
```objc
// Use exceptions only for programmer errors
@try {
    [self criticalOperation];
} @catch (NSException *exception) {
    NSLog(@"Critical error: %@", exception);
    // Log and potentially crash in debug builds
    NSAssert(NO, @"Unrecoverable error: %@", exception);
}
```

## Performance Guidelines

### Method Implementation
- Keep methods focused and under 50 lines
- Use early returns to reduce nesting
- Avoid deep inheritance hierarchies
- Use composition over inheritance

```objc
- (BOOL)validateCaptureDevice:(AVCaptureDevice *)device {
    // Early return pattern
    if (!device) return NO;
    if (!device.connected) return NO;
    if ([device lockForConfiguration:nil] == NO) return NO;
    
    [device unlockForConfiguration];
    return YES;
}
```

### Dispatch Queues
```objc
// Use appropriate queue types
@property (nonatomic, strong) dispatch_queue_t processingQueue;
@property (nonatomic, strong) dispatch_queue_t captureQueue;

// Initialize in init method
_processingQueue = dispatch_queue_create("com.beautywebcam.processing", 
                                        DISPATCH_QUEUE_SERIAL);
_captureQueue = dispatch_queue_create("com.beautywebcam.capture", 
                                     DISPATCH_QUEUE_SERIAL);
```

## Documentation Standards

### Header Documentation
```objc
/**
 * Manages video capture from USB webcams with real-time processing capabilities.
 *
 * This class handles the complete capture pipeline from device selection through
 * frame processing to virtual camera output. It ensures optimal performance
 * through GPU acceleration and proper memory management.
 *
 * @warning This class must be used from the main thread for UI updates.
 */
@interface BWCaptureManager : NSObject
```

### Method Documentation
```objc
/**
 * Starts video capture with specified quality settings.
 *
 * @param quality The processing quality level to use
 * @param device The capture device to use for input
 * @param error On failure, contains an NSError describing the problem
 * @return YES if capture started successfully, NO otherwise
 *
 * @note This method performs validation and may take several seconds to complete
 */
- (BOOL)startCaptureWithQuality:(BWProcessingQuality)quality
                         device:(AVCaptureDevice *)device
                          error:(NSError **)error;
```

## Thread Safety

### Synchronization
```objc
// Use atomic properties for simple thread safety
@property (atomic, assign) BOOL isProcessing;

// Use dispatch_sync for critical sections
- (void)updateConfiguration:(void (^)(void))block {
    dispatch_sync(self.configurationQueue, block);
}

// Use dispatch_barrier for read/write operations
- (void)setConfiguration:(BWConfiguration *)config {
    dispatch_barrier_async(self.configurationQueue, ^{
        self->_configuration = config;
    });
}
```

## Framework Integration

### CoreMediaIO Usage
```objc
// Proper plugin registration
CMIOObjectPropertyAddress propertyAddress = {
    kCMIOHardwarePropertyDevices,
    kCMIOObjectPropertyScopeGlobal,
    kCMIOObjectPropertyElementMaster
};
```

### Metal Framework
```objc
// Proper Metal device initialization
@property (nonatomic, strong) id<MTLDevice> metalDevice;
@property (nonatomic, strong) id<MTLCommandQueue> commandQueue;

- (void)setupMetal {
    self.metalDevice = MTLCreateSystemDefaultDevice();
    NSAssert(self.metalDevice, @"Metal not supported on this device");
    self.commandQueue = [self.metalDevice newCommandQueue];
}
```

## Code Quality Rules

1. **No force unwrapping**: Always check for nil before using objects
2. **Use modern Objective-C**: Prefer properties over instance variables
3. **Consistent indentation**: Use 4 spaces, no tabs
4. **Line length**: Maximum 100 characters per line
5. **Import order**: System frameworks first, then local headers
6. **Semicolons**: Always use semicolons, even when optional

## Testing Requirements

- Every public method should have corresponding tests
- Use dependency injection for testability
- Mock external dependencies (cameras, hardware)
- Test error conditions and edge cases

---
> Source: [madebyaris/beauty-webcam-mac](https://github.com/madebyaris/beauty-webcam-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
