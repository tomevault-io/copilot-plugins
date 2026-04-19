## 00os

> * 00OS Permissions System

pt
/**
 * 00OS Permissions System
 * 
 * Manages access control for processes, ensuring they only access
 * resources and perform actions they're authorized to do.
 */

// Permission levels
const PermissionLevels = {
  BASIC: 'basic',           // Basic operations, no system access
  FILE_READ: 'file-read',   // Read access to files
  FILE_WRITE: 'file-write', // Write access to files
  SYSTEM_READ: 'system-read', // Read system configuration
  SYSTEM_WRITE: 'system-write', // Modify system configuration
  NETWORK: 'network',       // Network access
  EXECUTION: 'execution',   // Execute other processes
  ADMIN: 'admin'            // Administrative privileges
};

// Default permission sets
const PermissionSets = {
  // Minimal permissions
  MINIMAL: [
    PermissionLevels.BASIC
  ],
  
  // File operations
  FILE_OPS: [
    PermissionLevels.BASIC,
    PermissionLevels.FILE_READ,
    PermissionLevels.FILE_WRITE
  ],
  
  // System monitoring
  SYSTEM_MONITOR: [
    PermissionLevels.BASIC,
    PermissionLevels.FILE_READ,
    PermissionLevels.SYSTEM_READ
  ],
  
  // System administration
  SYSTEM_ADMIN: [
    PermissionLevels.BASIC,
    PermissionLevels.FILE_READ,
    PermissionLevels.FILE_WRITE,
    PermissionLevels.SYSTEM_READ,
    PermissionLevels.SYSTEM_WRITE
  ],
  
  // Full access (00reaper)
  FULL_ACCESS: [
    PermissionLevels.BASIC,
    PermissionLevels.FILE_READ,
    PermissionLevels.FILE_WRITE,
    PermissionLevels.SYSTEM_READ,
    PermissionLevels.SYSTEM_WRITE,
    PermissionLevels.NETWORK,
    PermissionLevels.EXECUTION,
    PermissionLevels.ADMIN
  ]
};

// Process identity to permission mapping
const IdentityPermissions = {
  // 00reaper has full access
  '00reaper': PermissionSets.FULL_ACCESS
};

// Path-based permission rules
const PathPermissionRules = [
  {
    pattern: /^\/00os\//,
    required: [PermissionLevels.ADMIN],
    description: 'Operating system files'
  },
  {
    pattern: /^\/00reaper\//,
    required: [PermissionLevels.ADMIN],
    description: 'System administrator files'
  },
  {
    pattern: /\.(md|js|json|txt|css|html)$/,
    required: [PermissionLevels.FILE_READ],
    description: 'Common readable files'
  },
  {
    pattern: /\.cursor\/rules/,
    required: [PermissionLevels.ADMIN],
    description: 'Cursor rules directory'
  }
];

// Current active identity
let currentIdentity = '00reaper';

/**
 * Check if requested permissions are granted for the current identity
 * @param {Array} requestedPermissions - Permissions requested by process
 * @returns {Object} Permission check result
 */
function checkPermissions(requestedPermissions = [PermissionLevels.BASIC]) {
  const availablePermissions = IdentityPermissions[currentIdentity] || PermissionSets.MINIMAL;
  
  // Check each requested permission
  for (const permission of requestedPermissions) {
    if (!availablePermissions.includes(permission)) {
      return {
        granted: false,
        message: `Permission '${permission}' denied for identity '${currentIdentity}'`
      };
    }
  }
  
  return {
    granted: true,
    identity: currentIdentity,
    permissions: availablePermissions
  };
}

/**
 * Check if path access is permitted
 * @param {string} path - Path to check
 * @param {string} operation - Operation type (read, write)
 * @returns {Object} Permission check result
 */
function checkPathPermission(path, operation = 'read') {
  const identityPermissions = IdentityPermissions[currentIdentity] || PermissionSets.MINIMAL;
  
  // Admin always has access
  if (identityPermissions.includes(PermissionLevels.ADMIN)) {
    return {
      granted: true,
      identity: currentIdentity
    };
  }
  
  // Check against path rules
  for (const rule of PathPermissionRules) {
    if (rule.pattern.test(path)) {
      // Check if identity has all required permissions
      const missingPermissions = rule.required.filter(
        permission => !identityPermissions.includes(permission)
      );
      
      if (missingPermissions.length > 0) {
        return {
          granted: false,
          message: `Access to ${path} requires permissions: ${missingPermissions.join(', ')}`,
          requiredPermissions: rule.required,
          description: rule.description
        };
      }
    }
  }
  
  // Check basic file operation permissions
  if (operation === 'read' && !identityPermissions.includes(PermissionLevels.FILE_READ)) {
    return {
      granted: false,
      message: `File read permission required for ${path}`
    };
  }
  
  if (operation === 'write' && !identityPermissions.includes(PermissionLevels.FILE_WRITE)) {
    return {
      granted: false,
      message: `File write permission required for ${path}`
    };
  }
  
  // If no rules match and basic permissions are met, access is granted
  return {
    granted: true,
    identity: currentIdentity
  };
}

/**
 * Set the active identity
 * @param {string} identity - Identity to set
 * @returns {Object} Result of identity change
 */
function setIdentity(identity) {
  // Check if identity exists
  if (!IdentityPermissions[identity]) {
    return {
      success: false,
      message: `Unknown identity: ${identity}`
    };
  }
  
  // Set new identity
  currentIdentity = identity;
  
  return {
    success: true,
    identity: currentIdentity,
    permissions: IdentityPermissions[currentIdentity]
  };
}

/**
 * Get permissions for current identity
 * @returns {Object} Current identity and permissions
 */
function getIdentityPermissions() {
  return {
    identity: currentIdentity,
    permissions: IdentityPermissions[currentIdentity] || PermissionSets.MINIMAL
  };
}

/**
 * Register a new identity with permissions
 * @param {string} identity - Identity name
 * @param {Array} permissions - Permissions to grant
 * @returns {Object} Registration result
 */
function registerIdentity(identity, permissions = PermissionSets.MINIMAL) {
  // Admin permission required
  if (!checkPermissions([PermissionLevels.ADMIN]).granted) {
    return {
      success: false,
      message: 'Admin permission required to register identities'
    };
  }
  
  // Register identity
  IdentityPermissions[identity] = [...permissions];
  
  return {
    success: true,
    identity,
    permissions: IdentityPermissions[identity]
  };
}

// Export functions for system use
module.exports = {
  PermissionLevels,
  PermissionSets,
  checkPermissions,
  checkPathPermission,
  setIdentity,
  getIdentityPermissions,
  registerIdentity
};
```

## Permission Levels

The system defines several permission levels that control what actions processes can perform:

- **basic**: Basic operations, no system access
- **file-read**: Read access to files
- **file-write**: Write access to files  
- **system-read**: Read system configuration
- **system-write**: Modify system configuration
- **network**: Network access
- **execution**: Execute other processes
- **admin**: Administrative privileges

## Permission Sets

Pre-defined sets of permissions for common tasks:

- **MINIMAL**: Just basic operations
- **FILE_OPS**: File operations (read/write)
- **SYSTEM_MONITOR**: System monitoring (read-only)
- **SYSTEM_ADMIN**: System administration (read/write)
- **FULL_ACCESS**: Complete system access (00reaper only)

## Identity-Based Permissions

Permissions are assigned based on the active identity:

```javascript
/**
 * Switch to a different identity
 * @param {string} identity - Identity to switch to
 * @returns {Object} Result with available permissions
 */
function switchIdentity(identity) {
  // Check if identity exists
  if (!IdentityPermissions[identity]) {
    return {
      success: false,
      error: 'UNKNOWN_IDENTITY',
      message: `Unknown identity: ${identity}`
    };
  }
  
  // Verify admin permission for switching to privileged identities
  if (isPrivilegedIdentity(identity) && 
      !checkPermissions([PermissionLevels.ADMIN]).granted) {
    return {
      success: false,
      error: 'PERMISSION_DENIED',
      message: `Admin permission required to switch to ${identity}`
    };
  }
  
  // Set new identity
  setIdentity(identity);
  
  return {
    success: true,
    identity,
    permissions: IdentityPermissions[identity]
  };
}
```

## Path-Based Access Control

The system restricts access to specific paths:

```javascript
/**
 * Add a path permission rule
 * @param {RegExp} pattern - Path pattern to match
 * @param {Array} requiredPermissions - Required permissions
 * @param {string} description - Rule description
 * @returns {Object} Result of rule addition
 */
function addPathRule(pattern, requiredPermissions, description) {
  // Admin permission required
  if (!checkPermissions([PermissionLevels.ADMIN]).granted) {
    return {
      success: false,
      message: 'Admin permission required to add path rules'
    };
  }
  
  // Add rule
  PathPermissionRules.push({
    pattern,
    required: requiredPermissions,
    description
  });
  
  return {
    success: true,
    rule: {
      pattern,
      required: requiredPermissions,
      description
    }
  };
}
```

## Permission Inheritance

Permissions can be inherited from parent processes:

```javascript
/**
 * Create child process with inherited permissions
 * @param {Array} additionalPermissions - Additional permissions to request
 * @returns {Object} Permission set for child process
 */
function createChildProcessPermissions(additionalPermissions = []) {
  const parentPermissions = IdentityPermissions[currentIdentity] || PermissionSets.MINIMAL;
  
  // Child cannot have more permissions than parent
  const validAdditional = additionalPermissions.filter(
    perm => parentPermissions.includes(perm)
  );
  
  return {
    permissions: [...validAdditional],
    parent: currentIdentity
  };
}
```

## Integration with Process Execution

The permissions system integrates with the process executor:

1. Process declares required permissions in metadata
2. Executor checks permissions before execution
3. Tools available to process are limited by permissions
4. Path access is verified during file operations

## Process Permission Declaration

Processes declare required permissions in their metadata:

```markdown
---
name: example-process
description: Example process
version: 1.0.0
author: 00reaper
permissions: [basic, file-read]
inputs:
  - name: filepath
    type: string
    required: true
    description: File to read
outputs:
  - name: content
    type: string
    description: File content
---
```

## Future Enhancements

1. **Fine-Grained Access Control**
   - Path-specific permissions with wildcards
   - Content-based access restrictions

2. **Permission Delegation**
   - Controlled delegation of permissions
   - Temporary elevation of privileges

3. **Permission Auditing**
   - Tracking of permission usage
   - Logging of permission denials

4. **Dynamic Permission Policies**
   - Context-aware permission rules
   - Time-limited permissions 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tyleryouk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
