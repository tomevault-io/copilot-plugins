## coremediaio-virtualcam

> CoreMediaIO virtual camera implementation guidelines and best practices


# CoreMediaIO Virtual Camera Guidelines

## Overview
CoreMediaIO is the framework for creating virtual camera devices on macOS. This guide covers best practices for implementing a virtual camera that appears as a standard webcam to all applications.

## Plugin Architecture

### Basic Plugin Structure
```objc
// BWVirtualCameraPlugin.h
@interface BWVirtualCameraPlugin : NSObject

@property (nonatomic, strong, readonly) NSUUID *pluginUUID;
@property (nonatomic, strong, readonly) NSString *pluginName;

+ (instancetype)sharedInstance;
- (BOOL)initializePluginWithError:(NSError **)error;
- (void)teardownPlugin;

@end

// Plugin registration
- (BOOL)initializePluginWithError:(NSError **)error {
    // Register with CoreMediaIO
    CMIOObjectPropertyAddress propertyAddress = {
        kCMIOHardwarePropertyAllowScreenCaptureDevices,
        kCMIOObjectPropertyScopeGlobal,
        kCMIOObjectPropertyElementMaster
    };
    
    UInt32 allow = 1;
    OSStatus status = CMIOObjectSetPropertyData(kCMIOObjectSystemObject,
                                               &propertyAddress,
                                               0, NULL,
                                               sizeof(allow), &allow);
    
    return status == noErr;
}
```

### Device Implementation
```objc
// BWVirtualCameraDevice.h
@interface BWVirtualCameraDevice : NSObject

@property (nonatomic, strong, readonly) NSString *deviceName;
@property (nonatomic, strong, readonly) NSString *deviceID;
@property (nonatomic, assign, readonly) CMIODeviceID deviceID;
@property (nonatomic, assign, getter=isRunning) BOOL running;

- (BOOL)createDeviceWithError:(NSError **)error;
- (BOOL)startStreamingWithError:(NSError **)error;
- (void)stopStreaming;
- (void)destroyDevice;

- (void)sendVideoFrame:(CVPixelBufferRef)pixelBuffer
             timestamp:(CMTime)timestamp;

@end
```

### Stream Management
```objc
// BWVirtualCameraStream.m
@implementation BWVirtualCameraStream

- (BOOL)startStreamingWithFormat:(CMVideoFormatDescriptionRef)formatDescription
                           error:(NSError **)error {
    
    // Validate format
    if (!formatDescription) {
        if (error) {
            *error = [NSError errorWithDomain:BWErrorDomain
                                         code:BWErrorInvalidFormat
                                     userInfo:@{NSLocalizedDescriptionKey: @"Invalid format description"}];
        }
        return NO;
    }
    
    // Set up timing information
    self.frameRate = 30; // Default to 30fps
    self.frameDuration = CMTimeMake(1, self.frameRate);
    
    // Initialize stream state
    self.isStreaming = YES;
    self.frameCount = 0;
    
    return YES;
}

- (void)sendFrame:(CVPixelBufferRef)pixelBuffer {
    if (!self.isStreaming) return;
    
    // Calculate timestamp
    CMTime timestamp = CMTimeMultiply(self.frameDuration, self.frameCount);
    
    // Create sample buffer
    CMSampleBufferRef sampleBuffer = NULL;
    OSStatus status = CMSampleBufferCreateForImageBuffer(
        kCFAllocatorDefault,
        pixelBuffer,
        true, NULL, NULL,
        self.formatDescription,
        &self.sampleTiming,
        &sampleBuffer
    );
    
    if (status == noErr && sampleBuffer) {
        // Send to CoreMediaIO
        [self enqueueSampleBuffer:sampleBuffer];
        CFRelease(sampleBuffer);
    }
    
    self.frameCount++;
}

@end
```

## Property Management

### Device Properties
```objc
// Essential device properties
static const CMIOObjectPropertyAddress kDevicePropertyName = {
    kCMIOObjectPropertyName,
    kCMIOObjectPropertyScopeGlobal,
    kCMIOObjectPropertyElementMaster
};

static const CMIOObjectPropertyAddress kDevicePropertyUID = {
    kCMIODevicePropertyDeviceUID,
    kCMIOObjectPropertyScopeGlobal,
    kCMIOObjectPropertyElementMaster
};

// Property getter implementation
- (OSStatus)getPropertyWithAddress:(const CMIOObjectPropertyAddress *)address
                         dataSize:(UInt32)dataSize
                         dataUsed:(UInt32 *)dataUsed
                             data:(void *)data {
    
    OSStatus result = kCMIOHardwareNoError;
    
    switch (address->mSelector) {
        case kCMIOObjectPropertyName:
            *dataUsed = [self copyStringProperty:@"BeautyWebcam Virtual Camera"
                                          toData:data
                                        dataSize:dataSize];
            break;
            
        case kCMIODevicePropertyDeviceUID:
            *dataUsed = [self copyStringProperty:self.deviceUID
                                          toData:data
                                        dataSize:dataSize];
            break;
            
        case kCMIODevicePropertyStreams:
            *dataUsed = [self copyStreamsProperty:data dataSize:dataSize];
            break;
            
        default:
            result = kCMIOHardwareUnknownPropertyError;
            break;
    }
    
    return result;
}
```

### Stream Properties
```objc
// Stream format management
- (OSStatus)setFormatDescription:(CMVideoFormatDescriptionRef)formatDescription {
    // Validate format
    CMVideoDimensions dimensions = CMVideoFormatDescriptionGetDimensions(formatDescription);
    FourCharCode codec = CMFormatDescriptionGetMediaSubType(formatDescription);
    
    // Support common formats
    switch (codec) {
        case kCVPixelFormatType_32BGRA:
        case kCVPixelFormatType_32ARGB:
        case kCVPixelFormatType_24RGB:
        case kCVPixelFormatType_420YpCbCr8Planar:
            break;
        default:
            return kCMIOHardwareUnsupportedOperationError;
    }
    
    // Update format
    if (self.formatDescription) {
        CFRelease(self.formatDescription);
    }
    self.formatDescription = formatDescription;
    CFRetain(self.formatDescription);
    
    return kCMIOHardwareNoError;
}
```

## Frame Delivery

### Efficient Frame Handling
```objc
// BWFrameDelivery.m
@implementation BWFrameDelivery

- (void)deliverFrame:(CVPixelBufferRef)pixelBuffer
           timestamp:(CMTime)timestamp {
    
    // Validate inputs
    if (!pixelBuffer || !self.isActive) return;
    
    @autoreleasepool {
        // Create sample timing
        CMSampleTimingInfo timing = {
            .duration = self.frameDuration,
            .presentationTimeStamp = timestamp,
            .decodeTimeStamp = kCMTimeInvalid
        };
        
        // Create sample buffer
        CMSampleBufferRef sampleBuffer;
        OSStatus status = CMSampleBufferCreateReadyWithImageBuffer(
            kCFAllocatorDefault,
            pixelBuffer,
            self.formatDescription,
            &timing,
            &sampleBuffer
        );
        
        if (status == noErr) {
            // Deliver to all connected clients
            [self deliverSampleBuffer:sampleBuffer];
            CFRelease(sampleBuffer);
        } else {
            NSLog(@"Failed to create sample buffer: %d", (int)status);
        }
    }
}

- (void)deliverSampleBuffer:(CMSampleBufferRef)sampleBuffer {
    // Thread-safe delivery to multiple consumers
    dispatch_sync(self.deliveryQueue, ^{
        for (BWStreamConsumer *consumer in self.consumers) {
            if (consumer.isActive) {
                [consumer receiveSampleBuffer:sampleBuffer];
            }
        }
    });
}

@end
```

### Frame Rate Management
```objc
// Adaptive frame rate based on system performance
@interface BWFrameRateController : NSObject

@property (nonatomic, assign) NSInteger targetFrameRate;
@property (nonatomic, assign) NSInteger currentFrameRate;
@property (nonatomic, assign) NSTimeInterval lastFrameTime;

- (BOOL)shouldDeliverFrameAtTime:(NSTimeInterval)currentTime;
- (void)adjustFrameRateForPerformance:(double)processingTime;

@end

@implementation BWFrameRateController

- (BOOL)shouldDeliverFrameAtTime:(NSTimeInterval)currentTime {
    NSTimeInterval targetInterval = 1.0 / self.targetFrameRate;
    NSTimeInterval elapsed = currentTime - self.lastFrameTime;
    
    if (elapsed >= targetInterval) {
        self.lastFrameTime = currentTime;
        return YES;
    }
    
    return NO;
}

- (void)adjustFrameRateForPerformance:(double)processingTime {
    // Reduce frame rate if processing takes too long
    NSTimeInterval targetFrameTime = 1.0 / self.targetFrameRate;
    
    if (processingTime > targetFrameTime * 0.8) {
        // Processing is taking 80%+ of frame time, reduce frame rate
        self.currentFrameRate = MAX(15, self.currentFrameRate - 5);
    } else if (processingTime < targetFrameTime * 0.5) {
        // Processing is fast, we can increase frame rate
        self.currentFrameRate = MIN(self.targetFrameRate, self.currentFrameRate + 2);
    }
}

@end
```

## Error Handling

### Robust Error Management
```objc
// BWVirtualCameraError.h
typedef NS_ENUM(NSInteger, BWVirtualCameraError) {
    BWVirtualCameraErrorNone = 0,
    BWVirtualCameraErrorInitialization,
    BWVirtualCameraErrorDeviceCreation,
    BWVirtualCameraErrorStreamConfiguration,
    BWVirtualCameraErrorFrameDelivery,
    BWVirtualCameraErrorSystemPermissions
};

extern NSString *const BWVirtualCameraErrorDomain;

@interface BWVirtualCameraErrorHandler : NSObject

+ (NSError *)errorWithCode:(BWVirtualCameraError)code
           underlyingError:(NSError *)underlyingError
               description:(NSString *)description;

+ (void)handleCriticalError:(NSError *)error
                   recovery:(void(^)(void))recoveryBlock;

@end

// Error recovery implementation
- (void)handleStreamError:(OSStatus)status {
    NSError *error = [BWVirtualCameraErrorHandler 
        errorWithCode:BWVirtualCameraErrorFrameDelivery
      underlyingError:nil
          description:[NSString stringWithFormat:@"Stream error: %d", (int)status]];
    
    // Attempt recovery
    [BWVirtualCameraErrorHandler handleCriticalError:error recovery:^{
        // Stop and restart stream
        [self stopStreaming];
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 1 * NSEC_PER_SEC), 
                      dispatch_get_main_queue(), ^{
            NSError *restartError;
            if (![self startStreamingWithError:&restartError]) {
                NSLog(@"Failed to restart stream: %@", restartError);
            }
        });
    }];
}
```

## System Integration

### Application Compatibility
```objc
// BWCompatibilityManager.m
@implementation BWCompatibilityManager

+ (NSArray<NSString *> *)knownCompatibleApplications {
    return @[
        @"com.apple.FaceTime",
        @"us.zoom.xos",
        @"com.microsoft.teams",
        @"com.discord.discord",
        @"com.skype.skype",
        @"com.google.Chrome", // Google Meet
        @"com.apple.Safari",  // WebRTC apps
        @"com.reincubate.camo",
        @"com.obsproject.obs-studio"
    ];
}

- (void)testCompatibilityWithApplication:(NSString *)bundleID
                              completion:(void(^)(BOOL compatible, NSError *error))completion {
    
    // Create test virtual camera
    BWVirtualCameraDevice *testDevice = [[BWVirtualCameraDevice alloc] init];
    
    NSError *error;
    if (![testDevice createDeviceWithError:&error]) {
        completion(NO, error);
        return;
    }
    
    // Test if application can discover the device
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 2 * NSEC_PER_SEC), 
                  dispatch_get_global_queue(QOS_CLASS_BACKGROUND, 0), ^{
        
        BOOL found = [self isDeviceVisibleToApplication:bundleID
                                               deviceID:testDevice.deviceID];
        
        [testDevice destroyDevice];
        
        dispatch_async(dispatch_get_main_queue(), ^{
            completion(found, nil);
        });
    });
}

@end
```

### Permission Management
```objc
// Camera access permissions
- (void)requestCameraPermissionWithCompletion:(void(^)(BOOL granted))completion {
    AVAuthorizationStatus status = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    
    switch (status) {
        case AVAuthorizationStatusAuthorized:
            completion(YES);
            break;
            
        case AVAuthorizationStatusNotDetermined:
            [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo
                                     completionHandler:^(BOOL granted) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    completion(granted);
                });
            }];
            break;
            
        case AVAuthorizationStatusDenied:
        case AVAuthorizationStatusRestricted:
            // Guide user to System Preferences
            [self showCameraPermissionAlert];
            completion(NO);
            break;
    }
}
```

## Performance Optimization

### Memory Management
```objc
// Efficient pixel buffer handling
@interface BWPixelBufferPool : NSObject

@property (nonatomic, assign) CVPixelBufferPoolRef bufferPool;

- (instancetype)initWithWidth:(size_t)width
                       height:(size_t)height
                  pixelFormat:(OSType)pixelFormat;

- (CVPixelBufferRef)createPixelBuffer;
- (void)returnPixelBuffer:(CVPixelBufferRef)pixelBuffer;

@end

@implementation BWPixelBufferPool

- (instancetype)initWithWidth:(size_t)width
                       height:(size_t)height
                  pixelFormat:(OSType)pixelFormat {
    if (self = [super init]) {
        NSDictionary *attributes = @{
            (NSString *)kCVPixelBufferPixelFormatTypeKey: @(pixelFormat),
            (NSString *)kCVPixelBufferWidthKey: @(width),
            (NSString *)kCVPixelBufferHeightKey: @(height),
            (NSString *)kCVPixelBufferMetalCompatibilityKey: @YES,
            (NSString *)kCVPixelBufferIOSurfacePropertiesKey: @{}
        };
        
        CVPixelBufferPoolCreate(kCFAllocatorDefault, NULL,
                               (__bridge CFDictionaryRef)attributes,
                               &_bufferPool);
    }
    return self;
}

- (CVPixelBufferRef)createPixelBuffer {
    CVPixelBufferRef pixelBuffer;
    CVReturn status = CVPixelBufferPoolCreatePixelBuffer(kCFAllocatorDefault,
                                                        self.bufferPool,
                                                        &pixelBuffer);
    return (status == kCVReturnSuccess) ? pixelBuffer : NULL;
}

@end
```

## Testing and Validation

### Virtual Camera Testing
```objc
// BWVirtualCameraTests.m
@interface BWVirtualCameraTests : XCTestCase

@property (nonatomic, strong) BWVirtualCameraDevice *testDevice;

@end

@implementation BWVirtualCameraTests

- (void)testDeviceCreation {
    NSError *error;
    BOOL success = [self.testDevice createDeviceWithError:&error];
    
    XCTAssertTrue(success, @"Device creation failed: %@", error);
    XCTAssertNotNil(self.testDevice.deviceID, @"Device ID should not be nil");
}

- (void)testFrameDelivery {
    // Create test pixel buffer
    CVPixelBufferRef pixelBuffer = [self createTestPixelBuffer];
    
    // Start streaming
    NSError *error;
    BOOL started = [self.testDevice startStreamingWithError:&error];
    XCTAssertTrue(started, @"Failed to start streaming: %@", error);
    
    // Send frame
    CMTime timestamp = CMTimeMake(0, 30);
    [self.testDevice sendVideoFrame:pixelBuffer timestamp:timestamp];
    
    // Verify frame was delivered
    XCTestExpectation *frameExpectation = [self expectationWithDescription:@"Frame delivered"];
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 0.1 * NSEC_PER_SEC),
                  dispatch_get_main_queue(), ^{
        [frameExpectation fulfill];
    });
    
    [self waitForExpectationsWithTimeout:1.0 handler:nil];
    
    CVPixelBufferRelease(pixelBuffer);
}

@end
```

## Security Considerations

### Secure Device Management
```objc
// Prevent unauthorized access
- (BOOL)validateClientAccess:(pid_t)clientPID {
    // Get client process information
    NSRunningApplication *client = [NSRunningApplication runningApplicationWithProcessIdentifier:clientPID];
    
    // Check if client is a known safe application
    if ([self.trustedApplications containsObject:client.bundleIdentifier]) {
        return YES;
    }
    
    // Check code signature
    SecCodeRef codeRef;
    OSStatus status = SecCodeCreateWithPID(clientPID, kSecCSDefaultFlags, &codeRef);
    if (status != errSecSuccess) {
        return NO;
    }
    
    // Verify signature
    status = SecCodeCheckValidity(codeRef, kSecCSDefaultFlags, NULL);
    CFRelease(codeRef);
    
    return status == errSecSuccess;
}
```

## Best Practices Summary

1. **Always validate inputs** - Check pixel buffers, format descriptions, and parameters
2. **Use autoreleasepool** for frame processing to manage memory pressure
3. **Implement proper error recovery** - Virtual cameras should be resilient
4. **Test with real applications** - Zoom, Teams, Discord, etc.
5. **Monitor performance** - Frame drops affect user experience
6. **Handle system sleep/wake** - Restart streams after system events
7. **Respect privacy** - Only capture when explicitly requested

---
> Source: [madebyaris/beauty-webcam-mac](https://github.com/madebyaris/beauty-webcam-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
