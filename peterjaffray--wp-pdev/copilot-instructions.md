## wp-pdev

> Choice-UFT Plugin Debugging and Troubleshooting


# Choice-UFT Plugin Debugging Guide

## Plugin Architecture

### Core Files
- **Main Plugin**: `/home/r11/dev/choice-uft/choice-universal-form-tracker.php`
- **Admin Interface**: `/home/r11/dev/choice-uft/includes/class-cuft-admin.php`
- **Form Detector**: `/home/r11/dev/choice-uft/includes/class-cuft-form-detector.php`
- **Logger**: `/home/r11/dev/choice-uft/includes/class-cuft-logger.php`

### Framework Integrations
```
includes/forms/
├── class-cuft-avada-forms.php
├── class-cuft-cf7-forms.php
├── class-cuft-elementor-forms.php
├── class-cuft-gravity-forms.php
└── class-cuft-ninja-forms.php

assets/forms/
├── cuft-avada-forms.js
├── cuft-cf7-forms.js
├── cuft-elementor-forms.js
├── cuft-gravity-forms.js
└── cuft-ninja-forms.js
```

## Current Known Issues

### Fatal Error: WP_Upgrader_Skin
**Problem**: Class 'WP_Upgrader_Skin' not found in class-cuft-admin.php:14
**Solution**: Add conditional loading:
```php
if (!class_exists('WP_Upgrader_Skin')) {
    require_once(ABSPATH . 'wp-admin/includes/class-wp-upgrader-skin.php');
}
```

### Forms Not Tracking
**Common Causes**:
1. JavaScript files not loading
2. Success state detection failing
3. Timing issues with AJAX submissions
4. Framework version incompatibilities

## Debug Mode Activation

### Enable Console Logging
```javascript
// In browser console
localStorage.setItem('cuft_debug', 'true');
localStorage.setItem('cuft_console_logging', 'true');

// Reload page to apply
location.reload();
```

### PHP Debug Logging
```php
// In wp-config.php (already enabled)
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
```

### Plugin Debug Settings
```bash
# Enable via WP-CLI
docker exec wp-pdev-cli wp option update cuft_debug_enabled 1
docker exec wp-pdev-cli wp option update cuft_console_logging_enabled 1
```

## Debugging Workflow

### 1. Check Framework Detection
```php
// Add to test page or console
<?php
$frameworks = CUFT_Form_Detector::get_framework_status();
foreach ($frameworks as $fw) {
    echo $fw['name'] . ': ' . ($fw['detected'] ? 'DETECTED' : 'NOT FOUND') . "\n";
}
?>
```

### 2. Verify Script Loading
```javascript
// Check in browser console
console.log('UTM Utils:', typeof window.cuftUtmUtils);
console.log('DataLayer:', typeof window.dataLayer);

// Check loaded scripts
Array.from(document.scripts)
    .filter(s => s.src.includes('cuft'))
    .map(s => s.src);
```

### 3. Monitor Form Submission Flow

#### PHP Side (Server Hooks)
```php
// Add to functions.php for testing
add_action('wpcf7_mail_sent', function($form) {
    error_log('[CUFT Debug] CF7 Form Submitted: ' . $form->id());
});

add_action('gform_after_submission', function($entry, $form) {
    error_log('[CUFT Debug] GF Form Submitted: ' . $form['id']);
}, 10, 2);

add_action('ninja_forms_after_submission', function($form_data) {
    error_log('[CUFT Debug] NF Form Submitted: ' . $form_data['form_id']);
});
```

#### JavaScript Side (Client Events)
```javascript
// Monitor all form submissions
document.addEventListener('submit', function(e) {
    console.log('[Form Submit]', e.target);
}, true);

// Monitor AJAX requests
(function() {
    const originalOpen = XMLHttpRequest.prototype.open;
    XMLHttpRequest.prototype.open = function() {
        this.addEventListener('load', function() {
            if (this.responseURL.includes('admin-ajax.php') ||
                this.responseURL.includes('wp-json')) {
                console.log('[AJAX Response]', this.responseURL, this.response);
            }
        });
        originalOpen.apply(this, arguments);
    };
})();
```

### 4. DataLayer Monitoring
```javascript
// Intercept all dataLayer pushes
(function() {
    window.dataLayer = window.dataLayer || [];
    const originalPush = window.dataLayer.push;
    window.dataLayer.push = function() {
        console.group('[DataLayer Push]');
        console.log('Event:', arguments[0]);
        console.trace('Stack trace');
        console.groupEnd();
        return originalPush.apply(this, arguments);
    };
})();
```

## Common Fixes

### Fix: Avada Forms Not Detecting Success
```javascript
// Add additional success selectors
const successSelectors = [
    '.fusion-form-response-success',
    '.fusion-alert.success',
    '.fusion-form-success-message',
    '[data-status="sent"]'
];
```

### Fix: Elementor Forms Timing Issue
```javascript
// Add delay for Elementor's async handling
jQuery(document).on('submit_success', function(event, response) {
    setTimeout(function() {
        // Process form data
    }, 100);
});
```

### Fix: CF7 Multiple Events
```javascript
// Prevent duplicate tracking
let tracked = {};
document.addEventListener('wpcf7mailsent', function(event) {
    const formId = event.detail.contactFormId;
    if (!tracked[formId]) {
        tracked[formId] = true;
        // Track submission
        setTimeout(() => delete tracked[formId], 5000);
    }
});
```

## Testing Tools

### Quick Test Script
```bash
# Create test script
cat > /tmp/test-cuft.php << 'EOF'
<?php
require_once('/var/www/html/wp-load.php');

echo "Framework Detection:\n";
$frameworks = CUFT_Form_Detector::get_detected_frameworks();
print_r($frameworks);

echo "\nPlugin Settings:\n";
echo "GTM ID: " . get_option('cuft_gtm_id') . "\n";
echo "Debug: " . get_option('cuft_debug_enabled') . "\n";
echo "Generate Lead: " . get_option('cuft_generate_lead_enabled') . "\n";
EOF

# Run test
docker exec wp-pdev-wordpress php /tmp/test-cuft.php
```

### Browser Bookmarklets

#### Enable Debug Mode
```javascript
javascript:(function(){localStorage.setItem('cuft_debug','true');localStorage.setItem('cuft_console_logging','true');alert('CUFT Debug Enabled - Reload Page');})();
```

#### Show DataLayer
```javascript
javascript:(function(){console.table(window.dataLayer);})();
```

#### Test Form Submit
```javascript
javascript:(function(){window.dataLayer=window.dataLayer||[];window.dataLayer.push({event:'test_form_submit',test:true,timestamp:new Date().toISOString()});console.log('Test event pushed');})();
```

## Log Analysis

### View Recent Errors
```bash
docker exec wp-pdev-wordpress grep -i "error\|fatal\|warning" /var/www/html/wp-content/debug.log | grep -i cuft | tail -20
```

### Track Form Submissions
```bash
docker exec wp-pdev-wordpress tail -f /var/www/html/wp-content/debug.log | grep -E "form_submit|CUFT|dataLayer"
```

### Export Debug Log
```bash
docker exec wp-pdev-wordpress cat /var/www/html/wp-content/debug.log > ~/cuft-debug-$(date +%Y%m%d-%H%M%S).log
```

## Performance Monitoring

### Check Script Load Time
```javascript
performance.getEntriesByType('resource')
    .filter(r => r.name.includes('cuft'))
    .map(r => ({
        file: r.name.split('/').pop(),
        duration: r.duration + 'ms',
        size: (r.transferSize / 1024).toFixed(2) + 'KB'
    }));
```

### Monitor Memory Usage
```javascript
if (performance.memory) {
    console.log('Memory Usage:', {
        used: (performance.memory.usedJSHeapSize / 1048576).toFixed(2) + ' MB',
        total: (performance.memory.totalJSHeapSize / 1048576).toFixed(2) + ' MB'
    });
}
```

@/home/r11/dev/choice-uft/includes/class-cuft-logger.php
@/home/r11/dev/choice-uft/includes/class-cuft-form-detector.php
@/home/r11/dev/choice-uft/choice-universal-form-tracker.php

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peterjaffray)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/peterjaffray)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
