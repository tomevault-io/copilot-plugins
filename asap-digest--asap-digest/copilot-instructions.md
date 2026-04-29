## asap-digest

> This protocol establishes standardized patterns for implementing AJAX handlers in WordPress plugins. Following these standards ensures security, maintainability, consistency, and proper error handling across all AJAX interactions. Inconsistent AJAX implementations can lead to security vulnerabilities, debugging difficulties, and code fragmentation.

# WordPress AJAX Handler Standardization Protocol v1.0

## 1. Purpose and Importance

This protocol establishes standardized patterns for implementing AJAX handlers in WordPress plugins. Following these standards ensures security, maintainability, consistency, and proper error handling across all AJAX interactions. Inconsistent AJAX implementations can lead to security vulnerabilities, debugging difficulties, and code fragmentation.

## 2. Core AJAX Handler Structure

### 2.1 Handler Registration

```php
/**
 * Register all AJAX handlers for the plugin.
 *
 * @since 1.0.0
 * @return void
 */
function your_plugin_register_ajax_handlers() {
    // Admin-only AJAX handlers
    add_action('wp_ajax_your_plugin_action_name', 'your_plugin_handle_action_name');
    
    // AJAX handlers available to non-logged-in users (use sparingly and with caution)
    add_action('wp_ajax_nopriv_your_plugin_public_action', 'your_plugin_handle_public_action');
    
    // Handlers available to both logged-in and non-logged-in users
    add_action('wp_ajax_your_plugin_public_action', 'your_plugin_handle_public_action');
}
add_action('init', 'your_plugin_register_ajax_handlers');
```

### 2.2 Handler Implementation

```php
/**
 * Handle the your_plugin_action_name AJAX request.
 *
 * @since 1.0.0
 * @return void Dies with JSON response.
 */
function your_plugin_handle_action_name() {
    // 1. Security check: Verify user capabilities
    if (!current_user_can('required_capability')) {
        wp_send_json_error([
            'message' => __('You do not have permission to perform this action.', 'your-plugin'),
            'code'    => 'insufficient_permissions'
        ], 403);
    }
    
    // 2. Security check: Verify nonce
    if (!isset($_POST['nonce']) || !wp_verify_nonce($_POST['nonce'], 'your_plugin_action_nonce')) {
        wp_send_json_error([
            'message' => __('Security verification failed.', 'your-plugin'),
            'code'    => 'invalid_nonce'
        ], 400);
    }
    
    // 3. Parameter validation
    $required_params = ['param1', 'param2'];
    foreach ($required_params as $param) {
        if (!isset($_POST[$param])) {
            wp_send_json_error([
                'message' => sprintf(__('Missing required parameter: %s', 'your-plugin'), $param),
                'code'    => 'missing_parameter'
            ], 400);
        }
    }
    
    // 4. Sanitize input data
    $param1 = sanitize_text_field($_POST['param1']);
    $param2 = absint($_POST['param2']);
    
    // 5. Process the request
    try {
        $result = your_plugin_process_action($param1, $param2);
        
        // 6. Return success response
        wp_send_json_success([
            'message' => __('Action completed successfully.', 'your-plugin'),
            'data'    => $result
        ]);
    } catch (Exception $e) {
        // 7. Handle and log errors
        error_log('AJAX Error: ' . $e->getMessage());
        
        wp_send_json_error([
            'message' => __('An error occurred while processing your request.', 'your-plugin'),
            'code'    => 'processing_error',
            'details' => WP_DEBUG ? $e->getMessage() : null
        ], 500);
    }
}
```

## 3. Frontend AJAX Implementation

### 3.1 Enqueuing Scripts

```php
/**
 * Enqueue scripts and localize AJAX data.
 *
 * @since 1.0.0
 * @return void
 */
function your_plugin_enqueue_scripts() {
    wp_enqueue_script(
        'your-plugin-ajax',
        YOUR_PLUGIN_URL . 'assets/js/ajax.js',
        ['jquery'],
        YOUR_PLUGIN_VERSION,
        true
    );
    
    // Pass AJAX URL and nonces to JavaScript
    wp_localize_script('your-plugin-ajax', 'yourPluginAjax', [
        'ajaxUrl'   => admin_url('admin-ajax.php'),
        'nonces'    => [
            'actionName' => wp_create_nonce('your_plugin_action_nonce'),
            'otherAction' => wp_create_nonce('your_plugin_other_action_nonce')
        ],
        'i18n'      => [
            'error' => __('An error occurred.', 'your-plugin'),
            'success' => __('Action completed successfully.', 'your-plugin')
        ]
    ]);
}
add_action('wp_enqueue_scripts', 'your_plugin_enqueue_scripts');
```

### 3.2 JavaScript AJAX Implementation

```javascript
/**
 * Handle AJAX requests in JavaScript.
 */
(function($) {
    'use strict';
    
    /**
     * Perform an AJAX action.
     *
     * @param {string} action The AJAX action name without prefix.
     * @param {Object} data The data to send with the request.
     * @param {Function} callback Callback function on success.
     * @param {Function} errorCallback Callback function on error.
     */
    function doAjaxAction(action, data, callback, errorCallback) {
        // Ensure nonce is included if available
        const nonceName = action + 'Name'; // Match the nonce name pattern
        const nonce = yourPluginAjax.nonces[nonceName];
        
        // Prepare request data
        const requestData = {
            action: 'your_plugin_' + action,
            nonce: nonce,
            ...data
        };
        
        // Perform AJAX request
        $.ajax({
            url: yourPluginAjax.ajaxUrl,
            type: 'POST',
            data: requestData,
            success: function(response) {
                if (response.success) {
                    if (typeof callback === 'function') {
                        callback(response.data);
                    }
                } else {
                    const errorMessage = response.data && response.data.message 
                        ? response.data.message 
                        : yourPluginAjax.i18n.error;
                    
                    if (typeof errorCallback === 'function') {
                        errorCallback(errorMessage, response.data);
                    } else {
                        console.error('AJAX Error:', errorMessage);
                    }
                }
            },
            error: function(xhr, status, error) {
                const errorMessage = xhr.responseJSON && xhr.responseJSON.data && xhr.responseJSON.data.message
                    ? xhr.responseJSON.data.message
                    : yourPluginAjax.i18n.error;
                
                if (typeof errorCallback === 'function') {
                    errorCallback(errorMessage, { xhr, status, error });
                } else {
                    console.error('AJAX Request Failed:', errorMessage);
                }
            }
        });
    }
    
    // Example usage
    $('#your-action-button').on('click', function(e) {
        e.preventDefault();
        
        const data = {
            param1: $('#param1-input').val(),
            param2: $('#param2-input').val()
        };
        
        doAjaxAction('action_name', data, 
            function(responseData) {
                // Success callback
                console.log('Success:', responseData);
            },
            function(errorMessage, errorData) {
                // Error callback
                console.error('Error:', errorMessage, errorData);
            }
        );
    });
    
})(jQuery);
```

## 4. Class-Based AJAX Implementation

### 4.1 AJAX Handler Class

```php
/**
 * Class for handling AJAX requests.
 *
 * @since 1.0.0
 */
class Your_Plugin_Ajax_Handler {
    
    /**
     * Initialize the class and set up hooks.
     *
     * @since 1.0.0
     */
    public function __construct() {
        // Register AJAX handlers
        add_action('init', [$this, 'register_ajax_handlers']);
    }
    
    /**
     * Register all AJAX handlers.
     *
     * @since 1.0.0
     * @return void
     */
    public function register_ajax_handlers() {
        // Admin-only handlers
        add_action('wp_ajax_your_plugin_get_data', [$this, 'handle_get_data']);
        add_action('wp_ajax_your_plugin_save_settings', [$this, 'handle_save_settings']);
        
        // Public and admin handlers
        add_action('wp_ajax_your_plugin_public_action', [$this, 'handle_public_action']);
        add_action('wp_ajax_nopriv_your_plugin_public_action', [$this, 'handle_public_action']);
    }
    
    /**
     * Verify request security.
     *
     * @since 1.0.0
     * @param string $action     The nonce action.
     * @param string $capability The capability required.
     * @return void Dies with error if verification fails.
     */
    private function verify_request($action, $capability = 'manage_options') {
        // Check user capabilities
        if ($capability && !current_user_can($capability)) {
            wp_send_json_error([
                'message' => __('You do not have permission to perform this action.', 'your-plugin'),
                'code'    => 'insufficient_permissions'
            ], 403);
        }
        
        // Verify nonce
        if (!isset($_REQUEST['nonce']) || !wp_verify_nonce($_REQUEST['nonce'], $action)) {
            wp_send_json_error([
                'message' => __('Security verification failed.', 'your-plugin'),
                'code'    => 'invalid_nonce'
            ], 400);
        }
    }
    
    /**
     * Handle get_data AJAX request.
     *
     * @since 1.0.0
     * @return void Dies with JSON response.
     */
    public function handle_get_data() {
        $this->verify_request('your_plugin_get_data_nonce');
        
        // Process request
        try {
            $data = $this->get_data_from_source();
            wp_send_json_success($data);
        } catch (Exception $e) {
            $this->handle_error($e, 'get_data_error');
        }
    }
    
    /**
     * Handle save_settings AJAX request.
     *
     * @since 1.0.0
     * @return void Dies with JSON response.
     */
    public function handle_save_settings() {
        $this->verify_request('your_plugin_save_settings_nonce');
        
        // Validate required parameters
        $required = ['setting_name', 'setting_value'];
        foreach ($required as $param) {
            if (!isset($_POST[$param])) {
                wp_send_json_error([
                    'message' => sprintf(__('Missing required parameter: %s', 'your-plugin'), $param),
                    'code'    => 'missing_parameter'
                ], 400);
            }
        }
        
        // Sanitize input
        $setting_name = sanitize_key($_POST['setting_name']);
        $setting_value = sanitize_text_field($_POST['setting_value']);
        
        // Process request
        try {
            $result = $this->save_setting($setting_name, $setting_value);
            wp_send_json_success([
                'message' => __('Settings saved successfully.', 'your-plugin'),
                'data'    => $result
            ]);
        } catch (Exception $e) {
            $this->handle_error($e, 'save_settings_error');
        }
    }
    
    /**
     * Handle public_action AJAX request.
     *
     * @since 1.0.0
     * @return void Dies with JSON response.
     */
    public function handle_public_action() {
        $this->verify_request('your_plugin_public_action_nonce', false);
        
        // Process public request
        try {
            $result = $this->process_public_action();
            wp_send_json_success($result);
        } catch (Exception $e) {
            $this->handle_error($e, 'public_action_error');
        }
    }
    
    /**
     * Handle errors in a standardized way.
     *
     * @since 1.0.0
     * @param Exception $exception The exception that occurred.
     * @param string    $code      Error code for tracking.
     * @return void Dies with error JSON.
     */
    private function handle_error($exception, $code = 'error') {
        // Log the error
        error_log('Your Plugin AJAX Error: ' . $exception->getMessage());
        
        // Send error response
        wp_send_json_error([
            'message' => __('An error occurred while processing your request.', 'your-plugin'),
            'code'    => $code,
            'details' => WP_DEBUG ? $exception->getMessage() : null
        ], 500);
    }
    
    /**
     * Example method to get data from a source.
     *
     * @since 1.0.0
     * @return array Data from the source.
     */
    private function get_data_from_source() {
        // Implementation...
        return ['example' => 'data'];
    }
    
    /**
     * Example method to save a setting.
     *
     * @since 1.0.0
     * @param string $name  Setting name.
     * @param mixed  $value Setting value.
     * @return bool True on success.
     */
    private function save_setting($name, $value) {
        // Implementation...
        return true;
    }
    
    /**
     * Example method for public action.
     *
     * @since 1.0.0
     * @return array Result of the action.
     */
    private function process_public_action() {
        // Implementation...
        return ['status' => 'success'];
    }
}

// Initialize the AJAX handler
new Your_Plugin_Ajax_Handler();
```

## 5. Best Practices and Security Guidelines

### 5.1 Security Checklist

1. **Always verify user capabilities** before processing any admin AJAX requests.
2. **Always verify nonces** to prevent CSRF attacks.
3. **Sanitize all input data** using appropriate sanitization functions.
4. **Validate required parameters** early in the handler.
5. **Limit AJAX handlers for non-logged-in users** (`wp_ajax_nopriv_`) to only what's absolutely necessary.
6. **Use specific capabilities** rather than just checking if the user is an admin.
7. **Apply WordPress security functions** like `sanitize_text_field()`, `absint()`, etc.
8. **Avoid exposing sensitive data** in AJAX responses.
9. **Implement rate limiting** for public AJAX endpoints.
10. **Set appropriate HTTP status codes** in error responses.

### 5.2 Standardized Response Format

```php
// Success response format
wp_send_json_success([
    'message' => __('Action completed successfully.', 'your-plugin'),
    'data'    => $result_data,  // The actual data being returned
    'meta'    => [              // Optional metadata
        'timestamp' => current_time('timestamp'),
        'version'   => YOUR_PLUGIN_VERSION
    ]
]);

// Error response format
wp_send_json_error([
    'message' => __('An error occurred.', 'your-plugin'),  // User-friendly message
    'code'    => 'error_code',                             // Machine-readable code
    'details' => $error_details,                           // Detailed error info (debug only)
    'meta'    => [                                         // Optional metadata
        'timestamp' => current_time('timestamp'),
        'version'   => YOUR_PLUGIN_VERSION
    ]
], $http_status_code);
```

### 5.3 Error Handling and Logging

```php
/**
 * Handle and log AJAX errors.
 *
 * @param Exception $e     The exception that occurred.
 * @param string    $context The context where the error occurred.
 * @return array Error data.
 */
function your_plugin_handle_ajax_error($e, $context = 'ajax') {
    // Log the error
    error_log(sprintf(
        '[%s] Your Plugin AJAX Error (%s): %s in %s on line %s',
        current_time('mysql'),
        $context,
        $e->getMessage(),
        $e->getFile(),
        $e->getLine()
    ));
    
    // Return error data
    return [
        'message' => __('An error occurred while processing your request.', 'your-plugin'),
        'code'    => 'ajax_error',
        'context' => $context,
        'details' => WP_DEBUG ? [
            'message' => $e->getMessage(),
            'file'    => basename($e->getFile()),
            'line'    => $e->getLine()
        ] : null
    ];
}
```

## 6. AJAX Handler Organization

### 6.1 File Structure

```
your-plugin/
├── includes/
│   ├── ajax/
│   │   ├── class-ajax-handler.php       # Main AJAX handler class
│   │   ├── admin/                       # Admin-only AJAX handlers
│   │   │   ├── class-admin-ajax.php     # Admin AJAX handler class
│   │   │   ├── class-settings-ajax.php  # Settings-specific AJAX
│   │   │   └── class-data-ajax.php      # Data management AJAX
│   │   └── public/                      # Public AJAX handlers
│   │       └── class-public-ajax.php    # Public AJAX handler class
│   └── class-ajax-manager.php           # Central AJAX registration
├── assets/
│   └── js/
│       ├── admin/
│       │   ├── admin-ajax.js            # Admin AJAX JavaScript
│       │   └── settings-ajax.js         # Settings AJAX JavaScript
│       └── public/
│           └── public-ajax.js           # Public AJAX JavaScript
└── your-plugin.php                      # Main plugin file
```

### 6.2 Central Registration

```php
/**
 * Central AJAX manager for registering all AJAX handlers.
 */
class Your_Plugin_Ajax_Manager {
    
    /**
     * Initialize the manager.
     */
    public function __construct() {
        // Load handler classes
        $this->load_handlers();
        
        // Register all handlers
        add_action('init', [$this, 'register_handlers']);
    }
    
    /**
     * Load all AJAX handler classes.
     */
    private function load_handlers() {
        // Admin handlers
        require_once plugin_dir_path(dirname(__FILE__)) . 'ajax/admin/class-admin-ajax.php';
        require_once plugin_dir_path(dirname(__FILE__)) . 'ajax/admin/class-settings-ajax.php';
        require_once plugin_dir_path(dirname(__FILE__)) . 'ajax/admin/class-data-ajax.php';
        
        // Public handlers
        require_once plugin_dir_path(dirname(__FILE__)) . 'ajax/public/class-public-ajax.php';
    }
    
    /**
     * Register all AJAX handlers.
     */
    public function register_handlers() {
        // Initialize handler classes
        new Your_Plugin_Admin_Ajax();
        new Your_Plugin_Settings_Ajax();
        new Your_Plugin_Data_Ajax();
        new Your_Plugin_Public_Ajax();
    }
}

// Initialize the AJAX manager
new Your_Plugin_Ajax_Manager();
```

## 7. Verification Checklist

- [ ] AJAX handlers follow the standardized naming convention
- [ ] All handlers verify user capabilities and nonces
- [ ] Input data is properly sanitized
- [ ] Required parameters are validated
- [ ] Error handling is consistent
- [ ] Responses follow the standardized format
- [ ] JavaScript properly handles success and error responses
- [ ] Nonce names are consistent between PHP and JavaScript
- [ ] AJAX handlers are properly organized in the file structure
- [ ] Documentation is complete with PHPDoc comments

## 8. Common Anti-Patterns to Avoid

1. **Using different nonce names** in PHP and JavaScript
2. **Inconsistent error handling** across different AJAX handlers
3. **Missing capability checks** for admin-only AJAX actions
4. **Hardcoded text** instead of using translation functions
5. **Improper sanitization** of input data
6. **Direct database queries** without proper preparation
7. **Not checking for required parameters** before processing
8. **Exposing sensitive information** in responses
9. **Not using wp_send_json_* functions** for consistent responses
10. **Scattering AJAX handlers** throughout the codebase instead of centralizing them

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ASAP-Digest) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
