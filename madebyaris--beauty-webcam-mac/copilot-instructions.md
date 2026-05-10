## ui-appkit

> AppKit UI guidelines and menu bar implementation best practices for BeautyWebcam


# AppKit UI Guidelines

## Menu Bar Application Design

### NSStatusItem Implementation
```objc
// BWMenuBarManager.h
@interface BWMenuBarManager : NSObject

@property (nonatomic, strong) NSStatusItem *statusItem;
@property (nonatomic, strong) NSMenu *statusMenu;
@property (nonatomic, weak) id<BWMenuBarDelegate> delegate;

+ (instancetype)sharedManager;
- (void)setupMenuBar;
- (void)updateStatusWithState:(BWApplicationState)state;

@end

// BWMenuBarManager.m
@implementation BWMenuBarManager

+ (instancetype)sharedManager {
    static BWMenuBarManager *sharedManager = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedManager = [[BWMenuBarManager alloc] init];
    });
    return sharedManager;
}

- (void)setupMenuBar {
    // Create status item with variable width
    self.statusItem = [[NSStatusBar systemStatusBar] 
        statusItemWithLength:NSVariableStatusItemLength];
    
    // Configure status item
    self.statusItem.button.image = [self statusImageForState:BWApplicationStateInactive];
    self.statusItem.button.imagePosition = NSImageOnly;
    self.statusItem.button.target = self;
    self.statusItem.button.action = @selector(statusItemClicked:);
    
    // Create menu
    [self setupStatusMenu];
    
    // Set menu (but don't assign it yet - we'll show it manually)
    // This allows us to handle both left and right clicks
}

- (void)setupStatusMenu {
    self.statusMenu = [[NSMenu alloc] init];
    self.statusMenu.delegate = self;
    
    // Enhancement toggle
    NSMenuItem *toggleItem = [[NSMenuItem alloc] 
        initWithTitle:@"Enhancement Off"
        action:@selector(toggleEnhancement:)
        keyEquivalent:@""];
    toggleItem.target = self;
    toggleItem.tag = BWMenuItemTagToggle;
    [self.statusMenu addItem:toggleItem];
    
    [self.statusMenu addItem:[NSMenuItem separatorItem]];
    
    // Preset menu items
    [self addPresetMenuItems];
    
    [self.statusMenu addItem:[NSMenuItem separatorItem]];
    
    // Settings
    NSMenuItem *settingsItem = [[NSMenuItem alloc] 
        initWithTitle:@"Settings..."
        action:@selector(showSettings:)
        keyEquivalent:@","];
    settingsItem.target = self;
    [self.statusMenu addItem:settingsItem];
    
    // Performance monitor (optional)
    NSMenuItem *performanceItem = [[NSMenuItem alloc] 
        initWithTitle:@"Performance Monitor"
        action:@selector(showPerformanceMonitor:)
        keyEquivalent:@""];
    performanceItem.target = self;
    [self.statusMenu addItem:performanceItem];
    
    [self.statusMenu addItem:[NSMenuItem separatorItem]];
    
    // Help and quit
    NSMenuItem *helpItem = [[NSMenuItem alloc] 
        initWithTitle:@"Help & Support"
        action:@selector(showHelp:)
        keyEquivalent:@""];
    helpItem.target = self;
    [self.statusMenu addItem:helpItem];
    
    NSMenuItem *quitItem = [[NSMenuItem alloc] 
        initWithTitle:@"Quit BeautyWebcam"
        action:@selector(quitApplication:)
        keyEquivalent:@"q"];
    quitItem.target = self;
    [self.statusMenu addItem:quitItem];
}

@end
```

### Status Icon Management
```objc
// Dynamic status icon updates
- (NSImage *)statusImageForState:(BWApplicationState)state {
    NSString *imageName;
    
    switch (state) {
        case BWApplicationStateInactive:
            imageName = @"StatusBarIcon_Inactive";
            break;
        case BWApplicationStateActive:
            imageName = @"StatusBarIcon_Active";
            break;
        case BWApplicationStateProcessing:
            imageName = @"StatusBarIcon_Processing";
            break;
        case BWApplicationStateError:
            imageName = @"StatusBarIcon_Error";
            break;
    }
    
    NSImage *image = [NSImage imageNamed:imageName];
    
    // Configure for menu bar appearance
    image.template = YES; // Adapts to menu bar color scheme
    return image;
}

- (void)updateStatusWithState:(BWApplicationState)state {
    NSImage *image = [self statusImageForState:state];
    
    // Update on main thread
    dispatch_async(dispatch_get_main_queue(), ^{
        self.statusItem.button.image = image;
        
        // Update tooltip
        NSString *tooltip = [self tooltipForState:state];
        self.statusItem.button.toolTip = tooltip;
        
        // Update menu items
        [self updateMenuItemsForState:state];
    });
}

- (void)animateProcessingState {
    // Subtle animation for processing state
    if (self.currentState != BWApplicationStateProcessing) return;
    
    NSArray *frames = @[
        [NSImage imageNamed:@"StatusBarIcon_Processing_1"],
        [NSImage imageNamed:@"StatusBarIcon_Processing_2"],
        [NSImage imageNamed:@"StatusBarIcon_Processing_3"]
    ];
    
    static NSInteger frameIndex = 0;
    self.statusItem.button.image = frames[frameIndex % frames.count];
    frameIndex++;
    
    // Schedule next frame
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 0.5 * NSEC_PER_SEC),
                  dispatch_get_main_queue(), ^{
        [self animateProcessingState];
    });
}
```

## Settings Window Design

### Modern Settings Panel
```objc
// BWSettingsWindowController.h
@interface BWSettingsWindowController : NSWindowController

@property (nonatomic, weak) IBOutlet NSTabView *tabView;
@property (nonatomic, weak) IBOutlet NSView *enhancementTabView;
@property (nonatomic, weak) IBOutlet NSView *performanceTabView;
@property (nonatomic, weak) IBOutlet NSView *advancedTabView;

- (void)showSettingsWindow;

@end

// Modern window appearance
- (void)windowDidLoad {
    [super windowDidLoad];
    
    // Configure window appearance
    self.window.titlebarAppearsTransparent = YES;
    self.window.titleVisibility = NSWindowTitleHidden;
    self.window.styleMask |= NSWindowStyleMaskFullSizeContentView;
    
    // Set up toolbar
    [self setupToolbar];
    
    // Configure tabs
    [self setupTabViews];
}

- (void)setupToolbar {
    NSToolbar *toolbar = [[NSToolbar alloc] initWithIdentifier:@"SettingsToolbar"];
    toolbar.delegate = self;
    toolbar.allowsUserCustomization = NO;
    toolbar.displayMode = NSToolbarDisplayModeIconAndLabel;
    
    self.window.toolbar = toolbar;
}
```

### Responsive UI Controls
```objc
// Enhancement controls with real-time preview
@interface BWEnhancementControlsView : NSView

@property (nonatomic, weak) IBOutlet NSSlider *smoothingSlider;
@property (nonatomic, weak) IBOutlet NSSlider *brightnessSlider;
@property (nonatomic, weak) IBOutlet NSSlider *saturationSlider;
@property (nonatomic, weak) IBOutlet NSButton *enableToggle;

@property (nonatomic, weak) id<BWEnhancementDelegate> delegate;

@end

@implementation BWEnhancementControlsView

- (void)awakeFromNib {
    [super awakeFromNib];
    
    // Configure sliders for real-time updates
    [self setupSliders];
    [self setupBindings];
}

- (void)setupSliders {
    // Smoothing slider
    self.smoothingSlider.minValue = 0.0;
    self.smoothingSlider.maxValue = 1.0;
    self.smoothingSlider.doubleValue = 0.3;
    self.smoothingSlider.target = self;
    self.smoothingSlider.action = @selector(smoothingChanged:);
    self.smoothingSlider.continuous = YES; // Real-time updates
    
    // Brightness slider
    self.brightnessSlider.minValue = 0.5;
    self.brightnessSlider.maxValue = 2.0;
    self.brightnessSlider.doubleValue = 1.0;
    self.brightnessSlider.target = self;
    self.brightnessSlider.action = @selector(brightnessChanged:);
    self.brightnessSlider.continuous = YES;
    
    // Add tick marks for better UX
    self.smoothingSlider.numberOfTickMarks = 11;
    self.smoothingSlider.allowsTickMarkValuesOnly = NO;
}

- (IBAction)smoothingChanged:(NSSlider *)sender {
    // Provide immediate feedback
    [self.delegate enhancementParameterChanged:BWEnhancementParameterSmoothing
                                         value:sender.doubleValue];
    
    // Update related UI elements
    [self updateSmoothingLabel:sender.doubleValue];
}

@end
```

## Window Management

### Proper Window Lifecycle
```objc
// BWWindowManager.h
@interface BWWindowManager : NSObject

@property (nonatomic, strong) BWSettingsWindowController *settingsController;
@property (nonatomic, strong) BWPerformanceWindowController *performanceController;

+ (instancetype)sharedManager;
- (void)showSettingsWindow;
- (void)showPerformanceMonitor;
- (void)closeAllWindows;

@end

@implementation BWWindowManager

- (void)showSettingsWindow {
    if (!self.settingsController) {
        self.settingsController = [[BWSettingsWindowController alloc] 
            initWithWindowNibName:@"BWSettingsWindow"];
    }
    
    [self.settingsController showWindow:nil];
    [self.settingsController.window makeKeyAndOrderFront:nil];
    
    // Bring app to front if needed
    [NSApp activateIgnoringOtherApps:YES];
}

- (void)windowWillClose:(NSNotification *)notification {
    NSWindow *closingWindow = notification.object;
    
    if (closingWindow == self.settingsController.window) {
        // Save settings before closing
        [[BWSettingsManager sharedManager] saveSettings];
        
        // Don't release controller immediately - user might reopen
        // Let it be released naturally by ARC when no longer needed
    }
}

@end
```

## Accessibility Support

### VoiceOver and Accessibility
```objc
// Proper accessibility implementation
- (void)setupAccessibility {
    // Status bar button
    self.statusItem.button.accessibilityLabel = @"BeautyWebcam";
    self.statusItem.button.accessibilityHelp = @"Click to open BeautyWebcam menu";
    
    // Sliders
    self.smoothingSlider.accessibilityLabel = @"Skin Smoothing";
    self.smoothingSlider.accessibilityHelp = @"Adjust the intensity of skin smoothing effect";
    
    // Enable accessibility for custom views
    self.enhancementControlsView.accessibilityElement = YES;
    self.enhancementControlsView.accessibilityRole = NSAccessibilityGroupRole;
    self.enhancementControlsView.accessibilityLabel = @"Enhancement Controls";
}

// Dynamic accessibility updates
- (void)updateAccessibilityForSlider:(NSSlider *)slider {
    NSString *valueDescription = [NSString stringWithFormat:@"%.0f percent", 
                                 slider.doubleValue * 100];
    slider.accessibilityValue = valueDescription;
}
```

## Dark Mode Support

### Adaptive UI Elements
```objc
// Color management for dark mode
@interface BWColorManager : NSObject

+ (NSColor *)primaryAccentColor;
+ (NSColor *)secondaryTextColor;
+ (NSColor *)backgroundColorForView:(NSView *)view;

@end

@implementation BWColorManager

+ (NSColor *)primaryAccentColor {
    if (@available(macOS 10.14, *)) {
        return [NSColor controlAccentColor];
    } else {
        return [NSColor systemBlueColor];
    }
}

+ (NSColor *)backgroundColorForView:(NSView *)view {
    if (@available(macOS 10.14, *)) {
        // Use semantic colors that adapt to appearance
        return [NSColor controlBackgroundColor];
    } else {
        return [NSColor windowBackgroundColor];
    }
}

@end

// Respond to appearance changes
- (void)viewDidChangeEffectiveAppearance {
    [super viewDidChangeEffectiveAppearance];
    
    // Update colors for new appearance
    [self updateColorsForCurrentAppearance];
    
    // Update custom drawing
    [self setNeedsDisplay:YES];
}

- (void)updateColorsForCurrentAppearance {
    self.backgroundColor = [BWColorManager backgroundColorForView:self];
    
    // Update any custom UI elements
    for (NSView *subview in self.subviews) {
        if ([subview respondsToSelector:@selector(updateColorsForCurrentAppearance)]) {
            [(id)subview updateColorsForCurrentAppearance];
        }
    }
}
```

## Animation and Transitions

### Smooth UI Transitions
```objc
// Smooth window transitions
- (void)showWindowWithAnimation:(NSWindow *)window {
    // Start with window scaled down and transparent
    window.alphaValue = 0.0;
    NSRect frame = window.frame;
    NSRect startFrame = NSInsetRect(frame, frame.size.width * 0.1, frame.size.height * 0.1);
    [window setFrame:startFrame display:NO];
    
    [window makeKeyAndOrderFront:nil];
    
    // Animate to final state
    [NSAnimationContext runAnimationGroup:^(NSAnimationContext *context) {
        context.duration = 0.3;
        context.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut];
        
        [[window animator] setAlphaValue:1.0];
        [[window animator] setFrame:frame display:YES];
    }];
}

// Smooth value transitions for sliders
- (void)animateSliderToValue:(double)targetValue {
    double currentValue = self.smoothingSlider.doubleValue;
    
    [NSAnimationContext runAnimationGroup:^(NSAnimationContext *context) {
        context.duration = 0.2;
        [[self.smoothingSlider animator] setDoubleValue:targetValue];
    }];
}
```

## Performance Monitoring UI

### Real-time Performance Display
```objc
// BWPerformanceView.h
@interface BWPerformanceView : NSView

@property (nonatomic, assign) double cpuUsage;
@property (nonatomic, assign) double memoryUsage;
@property (nonatomic, assign) double frameRate;

- (void)updateWithPerformanceData:(BWPerformanceData *)data;

@end

@implementation BWPerformanceView

- (void)updateWithPerformanceData:(BWPerformanceData *)data {
    // Update properties
    self.cpuUsage = data.cpuUsage;
    self.memoryUsage = data.memoryUsage;
    self.frameRate = data.frameRate;
    
    // Trigger redraw
    dispatch_async(dispatch_get_main_queue(), ^{
        [self setNeedsDisplay:YES];
    });
}

- (void)drawRect:(NSRect)dirtyRect {
    [super drawRect:dirtyRect];
    
    // Draw performance graphs
    [self drawCPUGraph];
    [self drawMemoryGraph];
    [self drawFrameRateIndicator];
}

- (void)drawCPUGraph {
    // Simple bar graph for CPU usage
    NSRect graphRect = NSMakeRect(20, 20, 200, 100);
    
    // Background
    [[NSColor controlBackgroundColor] setFill];
    NSRectFill(graphRect);
    
    // CPU usage bar
    NSRect usageRect = NSMakeRect(graphRect.origin.x,
                                 graphRect.origin.y,
                                 graphRect.size.width * (self.cpuUsage / 100.0),
                                 graphRect.size.height);
    
    // Color based on usage level
    NSColor *usageColor;
    if (self.cpuUsage < 50) {
        usageColor = [NSColor systemGreenColor];
    } else if (self.cpuUsage < 80) {
        usageColor = [NSColor systemYellowColor];
    } else {
        usageColor = [NSColor systemRedColor];
    }
    
    [usageColor setFill];
    NSRectFill(usageRect);
    
    // Border
    [[NSColor controlColor] setStroke];
    NSFrameRect(graphRect);
    
    // Label
    NSString *label = [NSString stringWithFormat:@"CPU: %.1f%%", self.cpuUsage];
    NSDictionary *attributes = @{
        NSFontAttributeName: [NSFont systemFontOfSize:12],
        NSForegroundColorAttributeName: [NSColor labelColor]
    };
    
    [label drawAtPoint:NSMakePoint(graphRect.origin.x, graphRect.origin.y - 20)
        withAttributes:attributes];
}

@end
```

## User Preferences

### Settings Persistence
```objc
// BWSettingsManager.h
@interface BWSettingsManager : NSObject

@property (nonatomic, assign) BOOL enhancementEnabled;
@property (nonatomic, assign) double smoothingIntensity;
@property (nonatomic, assign) double brightnessAdjustment;
@property (nonatomic, assign) double saturationBoost;
@property (nonatomic, assign) BWProcessingQuality processingQuality;

+ (instancetype)sharedManager;
- (void)saveSettings;
- (void)loadSettings;
- (void)resetToDefaults;

@end

@implementation BWSettingsManager

- (void)saveSettings {
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    
    [defaults setBool:self.enhancementEnabled forKey:@"BWEnhancementEnabled"];
    [defaults setDouble:self.smoothingIntensity forKey:@"BWSmoothingIntensity"];
    [defaults setDouble:self.brightnessAdjustment forKey:@"BWBrightnessAdjustment"];
    [defaults setDouble:self.saturationBoost forKey:@"BWSaturationBoost"];
    [defaults setInteger:self.processingQuality forKey:@"BWProcessingQuality"];
    
    [defaults synchronize];
}

- (void)loadSettings {
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    
    // Register defaults first
    [self registerDefaults];
    
    self.enhancementEnabled = [defaults boolForKey:@"BWEnhancementEnabled"];
    self.smoothingIntensity = [defaults doubleForKey:@"BWSmoothingIntensity"];
    self.brightnessAdjustment = [defaults doubleForKey:@"BWBrightnessAdjustment"];
    self.saturationBoost = [defaults doubleForKey:@"BWSaturationBoost"];
    self.processingQuality = [defaults integerForKey:@"BWProcessingQuality"];
}

@end
```

## Best Practices Summary

1. **Use NSStatusItem** properly for menu bar integration
2. **Implement proper accessibility** for all UI elements
3. **Support dark mode** with semantic colors
4. **Provide real-time feedback** for user interactions
5. **Use smooth animations** for state transitions
6. **Persist user preferences** with NSUserDefaults
7. **Handle window lifecycle** properly
8. **Test with VoiceOver** and other accessibility tools
9. **Follow macOS HIG** for consistent user experience
10. **Optimize drawing** for smooth performance

---
> Source: [madebyaris/beauty-webcam-mac](https://github.com/madebyaris/beauty-webcam-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
