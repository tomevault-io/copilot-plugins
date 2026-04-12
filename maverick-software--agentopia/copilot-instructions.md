## agentopia

> Complete guide for adding new service integrations to Agentopia with OAuth, API keys, and tool implementation


# Adding New Integrations - Developer Guide

## Overview

This guide walks through the complete process of adding a new service integration to Agentopia. The process involves database configuration, edge function creation, tool registration, and UI integration.

## Integration Types

Agentopia supports three main integration types:

1. **OAuth 2.0**: Services requiring user authorization (Gmail, Microsoft, GitHub)
2. **API Key**: Services using API key authentication (web search, email APIs)
3. **SMTP**: Direct email server connections with credentials

## Step-by-Step Integration Process

### 1. Database Configuration

#### Add Service Provider

First, add the new service to the `service_providers` table:

```sql
-- Example: Adding Slack OAuth integration
INSERT INTO service_providers (
  name,
  display_name,
  provider_type,
  authorization_endpoint,
  token_endpoint,
  scopes_supported,
  configuration_metadata
) VALUES (
  'slack',
  'Slack',
  'oauth',
  'https://slack.com/oauth/v2/authorize',
  'https://slack.com/api/oauth.v2.access',
  '[
    "chat:write",
    "channels:read",
    "groups:read",
    "im:read",
    "mpim:read",
    "users:read"
  ]'::jsonb,
  '{
    "client_id_required": true,
    "pkce_required": false,
    "user_info_endpoint": "https://slack.com/api/users.identity",
    "revoke_endpoint": "https://slack.com/api/auth.revoke",
    "additional_auth_params": {
      "user_scope": "identity.basic,identity.email"
    }
  }'::jsonb
);
```

```sql
-- Example: Adding API key service
INSERT INTO service_providers (
  name,
  display_name,
  provider_type,
  configuration_metadata
) VALUES (
  'openai_api',
  'OpenAI API',
  'api_key',
  '{
    "api_key_header": "Authorization",
    "api_key_prefix": "Bearer ",
    "base_url": "https://api.openai.com/v1",
    "rate_limits": {
      "requests_per_minute": 60,
      "requests_per_hour": 1000
    },
    "supported_models": [
      "gpt-4",
      "gpt-3.5-turbo",
      "text-embedding-ada-002"
    ]
  }'::jsonb
);
```

#### Update Tool Catalog

Add tools to the `tool_catalog` table:

```sql
-- Add Slack tools
INSERT INTO tool_catalog (
  tool_name,
  display_name,
  description,
  category,
  provider_name,
  function_schema,
  is_active
) VALUES 
(
  'slack_send_message',
  'Send Slack Message',
  'Send a message to a Slack channel or direct message',
  'communication',
  'slack',
  '{
    "type": "function",
    "function": {
      "name": "slack_send_message",
      "description": "Send a message to a Slack channel or user",
      "parameters": {
        "type": "object",
        "properties": {
          "channel": {
            "type": "string",
            "description": "Channel ID or name (e.g., #general, @username)"
          },
          "text": {
            "type": "string",
            "description": "Message text to send"
          },
          "blocks": {
            "type": "array",
            "description": "Rich message blocks (optional)",
            "items": {"type": "object"}
          }
        },
        "required": ["channel", "text"]
      }
    }
  }'::jsonb,
  true
),
(
  'slack_list_channels',
  'List Slack Channels',
  'Get a list of channels in the workspace',
  'communication',
  'slack',
  '{
    "type": "function",
    "function": {
      "name": "slack_list_channels",
      "description": "List all channels in the Slack workspace",
      "parameters": {
        "type": "object",
        "properties": {
          "types": {
            "type": "string",
            "description": "Channel types to include (public_channel,private_channel,mpim,im)",
            "default": "public_channel,private_channel"
          },
          "exclude_archived": {
            "type": "boolean",
            "description": "Exclude archived channels",
            "default": true
          }
        }
      }
    }
  }'::jsonb,
  true
);
```

### 2. Create Edge Function

Create a new Supabase Edge Function for the integration:

```typescript
// File: supabase/functions/slack-api/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2.39.7';

interface SlackAPIRequest {
  action: string;
  agent_id: string;
  user_id: string;
  params: Record<string, any>;
}

serve(async (req) => {
  try {
    // Parse request
    const { action, agent_id, user_id, params }: SlackAPIRequest = await req.json();
    
    // Initialize Supabase client
    const supabase = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
    );
    
    // Get user's Slack credentials
    const credentials = await getUserSlackCredentials(supabase, user_id, agent_id);
    if (!credentials) {
      throw new Error('Question: Your Slack workspace needs to be connected. Please set up your Slack integration in the settings.');
    }
    
    // Execute the requested action
    let result;
    switch (action) {
      case 'send_message':
        result = await sendSlackMessage(credentials.access_token, params);
        break;
      case 'list_channels':
        result = await listSlackChannels(credentials.access_token, params);
        break;
      case 'create_channel':
        result = await createSlackChannel(credentials.access_token, params);
        break;
      default:
        throw new Error(`Question: What Slack action would you like me to perform? Available actions: send_message, list_channels, create_channel.`);
    }
    
    return new Response(JSON.stringify({
      success: true,
      data: result
    }), {
      headers: { 'Content-Type': 'application/json' }
    });
    
  } catch (error) {
    console.error('Slack API error:', error);
    
    return new Response(JSON.stringify({
      success: false,
      error: error.message
    }), {
      status: 200, // Return 200 to prevent retry on client errors
      headers: { 'Content-Type': 'application/json' }
    });
  }
});

async function getUserSlackCredentials(supabase: any, userId: string, agentId: string) {
  // Get the user's Slack connection with agent permissions
  const { data: permission } = await supabase
    .from('agent_integration_permissions')
    .select(`
      allowed_scopes,
      user_integration_credentials!inner(
        vault_access_token_id,
        service_providers!inner(name)
      )
    `)
    .eq('agent_id', agentId)
    .eq('user_integration_credentials.service_providers.name', 'slack')
    .eq('is_active', true)
    .single();

  if (!permission) return null;

  // Decrypt the access token
  const { data: accessToken } = await supabase.rpc('vault_decrypt', {
    vault_id: permission.user_integration_credentials.vault_access_token_id
  });

  return {
    access_token: accessToken,
    scopes: permission.allowed_scopes
  };
}

async function sendSlackMessage(accessToken: string, params: any) {
  // Validate required parameters
  if (!params.channel) {
    throw new Error('Question: Which Slack channel should I send the message to? Please provide a channel name or ID.');
  }
  
  if (!params.text) {
    throw new Error('Question: What message would you like me to send to Slack?');
  }
  
  // Call Slack API
  const response = await fetch('https://slack.com/api/chat.postMessage', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      channel: params.channel,
      text: params.text,
      blocks: params.blocks
    })
  });
  
  const result = await response.json();
  
  if (!result.ok) {
    // Convert Slack errors to LLM-friendly messages
    if (result.error === 'channel_not_found') {
      throw new Error(`Question: The channel '${params.channel}' was not found. Please check the channel name or try a different channel.`);
    }
    if (result.error === 'not_in_channel') {
      throw new Error(`Question: I don't have access to the channel '${params.channel}'. Please invite me to the channel or use a different channel.`);
    }
    throw new Error(`Slack API error: ${result.error}`);
  }
  
  return {
    message: 'Message sent successfully to Slack',
    channel: result.channel,
    timestamp: result.ts,
    text: params.text
  };
}

async function listSlackChannels(accessToken: string, params: any) {
  const response = await fetch('https://slack.com/api/conversations.list', {
    method: 'GET',
    headers: {
      'Authorization': `Bearer ${accessToken}`
    }
  });
  
  const result = await response.json();
  
  if (!result.ok) {
    throw new Error(`Failed to list channels: ${result.error}`);
  }
  
  // Filter and format channels
  const channels = result.channels
    .filter((channel: any) => {
      if (params.exclude_archived && channel.is_archived) return false;
      return true;
    })
    .map((channel: any) => ({
      id: channel.id,
      name: channel.name,
      is_private: channel.is_private,
      member_count: channel.num_members,
      purpose: channel.purpose?.value || ''
    }));
  
  return {
    channels,
    total_count: channels.length
  };
}
```

### 3. Update Universal Tool Executor

Add the new integration to the routing map:

```typescript
// File: supabase/functions/chat/function_calling/universal-tool-executor.ts

// Add to TOOL_ROUTING_MAP
'slack_': {
  edgeFunction: 'slack-api',
  actionMapping: (toolName: string) => {
    const actionMap: Record<string, string> = {
      'slack_send_message': 'send_message',
      'slack_list_channels': 'list_channels',
      'slack_create_channel': 'create_channel',
      'slack_get_user_info': 'get_user_info'
    };
    return actionMap[toolName] || 'send_message';
  },
  parameterMapping: (params: Record<string, any>, context: any) => {
    const actionMap: Record<string, string> = {
      'slack_send_message': 'send_message',
      'slack_list_channels': 'list_channels',
      'slack_create_channel': 'create_channel',
      'slack_get_user_info': 'get_user_info'
    };
    
    const action = actionMap[context.toolName] || 'send_message';
    
    return {
      action: action,
      agent_id: context.agentId,
      user_id: context.userId,
      params: params
    };
  }
}
```

### 4. Frontend Integration Setup

#### Add OAuth Flow (for OAuth integrations)

```typescript
// File: src/integrations/slack/SlackSetupModal.tsx
import React, { useState } from 'react';
import { Button } from '@/components/ui/button';
import { supabase } from '@/lib/supabase';
import { useAuth } from '@/contexts/AuthContext';

export function SlackSetupModal({ isOpen, onClose, onSuccess }) {
  const { user } = useAuth();
  const [loading, setLoading] = useState(false);
  
  const handleConnect = async () => {
    setLoading(true);
    
    try {
      // Get Slack OAuth URL from backend
      const { data, error } = await supabase.functions.invoke('oauth-handler', {
        body: {
          provider: 'slack',
          action: 'get_auth_url',
          user_id: user.id,
          redirect_uri: `${window.location.origin}/integrations/callback`
        }
      });
      
      if (error) throw error;
      
      // Redirect to Slack OAuth
      window.location.href = data.auth_url;
      
    } catch (error) {
      console.error('Slack OAuth error:', error);
      toast.error('Failed to initiate Slack connection');
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <Modal isOpen={isOpen} onClose={onClose}>
      <div className="p-6">
        <h2 className="text-xl font-semibold mb-4">Connect Slack Workspace</h2>
        
        <p className="text-gray-600 mb-6">
          Connect your Slack workspace to enable your agents to send messages, 
          create channels, and interact with your team.
        </p>
        
        <div className="mb-6">
          <h3 className="font-medium mb-2">Permissions Requested:</h3>
          <ul className="text-sm text-gray-600 space-y-1">
            <li>• Send messages to channels and users</li>
            <li>• Read channel and user information</li>
            <li>• Create and manage channels</li>
          </ul>
        </div>
        
        <div className="flex justify-end space-x-3">
          <Button variant="outline" onClick={onClose}>
            Cancel
          </Button>
          <Button onClick={handleConnect} disabled={loading}>
            {loading ? 'Connecting...' : 'Connect Slack'}
          </Button>
        </div>
      </div>
    </Modal>
  );
}
```

#### Add API Key Setup (for API key integrations)

```typescript
// File: src/integrations/openai/OpenAISetupModal.tsx
export function OpenAISetupModal({ isOpen, onClose, onSuccess }) {
  const [apiKey, setApiKey] = useState('');
  const [loading, setLoading] = useState(false);
  
  const handleSave = async () => {
    if (!apiKey.trim()) {
      toast.error('Please enter your OpenAI API key');
      return;
    }
    
    setLoading(true);
    
    try {
      // Test the API key first
      const testResponse = await fetch('https://api.openai.com/v1/models', {
        headers: {
          'Authorization': `Bearer ${apiKey}`,
        }
      });
      
      if (!testResponse.ok) {
        throw new Error('Invalid API key or insufficient permissions');
      }
      
      // Save the API key
      const { error } = await supabase.functions.invoke('integration-setup', {
        body: {
          provider: 'openai_api',
          credential_type: 'api_key',
          api_key: apiKey,
          connection_name: 'OpenAI API'
        }
      });
      
      if (error) throw error;
      
      toast.success('OpenAI API key saved successfully!');
      onSuccess();
      onClose();
      
    } catch (error) {
      console.error('OpenAI setup error:', error);
      toast.error(error.message || 'Failed to save API key');
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <Modal isOpen={isOpen} onClose={onClose}>
      <div className="p-6">
        <h2 className="text-xl font-semibold mb-4">Add OpenAI API Key</h2>
        
        <div className="mb-4">
          <Label htmlFor="api-key">API Key</Label>
          <Input
            id="api-key"
            type="password"
            placeholder="sk-..."
            value={apiKey}
            onChange={(e) => setApiKey(e.target.value)}
          />
          <p className="text-xs text-gray-500 mt-1">
            Get your API key from <a href="https://platform.openai.com/api-keys" target="_blank" className="text-blue-600">OpenAI Platform</a>
          </p>
        </div>
        
        <div className="flex justify-end space-x-3">
          <Button variant="outline" onClick={onClose}>
            Cancel
          </Button>
          <Button onClick={handleSave} disabled={loading || !apiKey.trim()}>
            {loading ? 'Testing...' : 'Save API Key'}
          </Button>
        </div>
      </div>
    </Modal>
  );
}
```

### 5. Integration Registration

Add the integration to the integrations registry:

```typescript
// File: src/integrations/index.ts
export const AVAILABLE_INTEGRATIONS = [
  // ... existing integrations
  {
    id: 'slack',
    name: 'Slack',
    description: 'Send messages and manage channels in your Slack workspace',
    category: 'communication',
    provider_type: 'oauth',
    icon: SlackIcon,
    setupComponent: SlackSetupModal,
    tools: [
      'slack_send_message',
      'slack_list_channels',
      'slack_create_channel'
    ],
    scopes: [
      'chat:write',
      'channels:read',
      'groups:read'
    ]
  },
  {
    id: 'openai_api',
    name: 'OpenAI API',
    description: 'Access GPT models and other OpenAI services',
    category: 'ai',
    provider_type: 'api_key',
    icon: OpenAIIcon,
    setupComponent: OpenAISetupModal,
    tools: [
      'openai_chat_completion',
      'openai_embeddings',
      'openai_image_generation'
    ]
  }
];
```

## Testing Your Integration

### 1. Database Testing

```sql
-- Test service provider creation
SELECT * FROM service_providers WHERE name = 'slack';

-- Test tool catalog entries
SELECT * FROM tool_catalog WHERE provider_name = 'slack';
```

### 2. Edge Function Testing

Create a test script:

```typescript
// File: scripts/test-slack-integration.ts
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

async function testSlackIntegration() {
  try {
    const { data, error } = await supabase.functions.invoke('slack-api', {
      body: {
        action: 'send_message',
        agent_id: 'test-agent-id',
        user_id: 'test-user-id',
        params: {
          channel: '#general',
          text: 'Test message from Agentopia'
        }
      }
    });
    
    if (error) throw error;
    
    console.log('Slack integration test successful:', data);
  } catch (error) {
    console.error('Slack integration test failed:', error);
  }
}

testSlackIntegration();
```

### 3. Frontend Testing

```typescript
// Test the setup modal
import { SlackSetupModal } from '@/integrations/slack/SlackSetupModal';

function TestPage() {
  const [showModal, setShowModal] = useState(false);
  
  return (
    <div>
      <Button onClick={() => setShowModal(true)}>
        Test Slack Setup
      </Button>
      
      <SlackSetupModal 
        isOpen={showModal}
        onClose={() => setShowModal(false)}
        onSuccess={() => console.log('Setup successful')}
      />
    </div>
  );
}
```

### 4. Tool Execution Testing

```typescript
// Test tool execution through Universal Tool Executor
const testToolExecution = async () => {
  const context = {
    agentId: 'test-agent-id',
    userId: 'test-user-id',
    toolName: 'slack_send_message',
    parameters: {
      channel: '#general',
      text: 'Hello from Agentopia!'
    },
    supabase
  };
  
  const result = await UniversalToolExecutor.executeTool(context);
  console.log('Tool execution result:', result);
};
```

## Best Practices

### 1. Error Handling

Always provide LLM-friendly error messages:

```typescript
// Good: Interactive error message
if (!params.channel) {
  throw new Error('Question: Which Slack channel should I send the message to? Please provide a channel name or ID.');
}

// Bad: Technical error message
if (!params.channel) {
  throw new Error('ValidationError: channel parameter is required');
}
```

### 2. Parameter Validation

Validate all required parameters with helpful messages:

```typescript
function validateSlackMessage(params: any) {
  const errors: string[] = [];
  
  if (!params.channel) {
    errors.push('Question: Which Slack channel should I send the message to?');
  }
  
  if (!params.text) {
    errors.push('Question: What message would you like me to send?');
  }
  
  if (params.text && params.text.length > 4000) {
    errors.push('Question: Your message is too long. Slack messages must be under 4000 characters. Could you shorten it?');
  }
  
  if (errors.length > 0) {
    throw new Error(errors.join(' '));
  }
}
```

### 3. Security

Always use Vault for credential storage:

```typescript
// Store credentials securely
const vaultId = await supabase.rpc('create_vault_secret', {
  p_secret: accessToken,
  p_name: `slack_token_${userId}`,
  p_description: `Slack access token for user ${userId}`
});

// Store only vault ID in database
await supabase.from('user_integration_credentials').insert({
  user_id: userId,
  service_provider_id: slackProviderId,
  vault_access_token_id: vaultId,
  credential_type: 'oauth'
});
```

### 4. Performance

Implement caching for frequently accessed data:

```typescript
// Cache channel lists for 5 minutes
const channelCache = new Map<string, { channels: any[], expires: number }>();

async function getChannelsWithCache(accessToken: string): Promise<any[]> {
  const cacheKey = `channels_${accessToken.slice(-8)}`;
  const cached = channelCache.get(cacheKey);
  
  if (cached && cached.expires > Date.now()) {
    return cached.channels;
  }
  
  const channels = await listSlackChannels(accessToken, {});
  channelCache.set(cacheKey, {
    channels: channels.channels,
    expires: Date.now() + (5 * 60 * 1000) // 5 minutes
  });
  
  return channels.channels;
}
```

## Deployment Checklist

- [ ] Service provider added to database
- [ ] Tool catalog entries created
- [ ] Edge function deployed and tested
- [ ] Universal Tool Executor routing configured
- [ ] Frontend setup modal implemented
- [ ] Integration added to registry
- [ ] Documentation updated
- [ ] Tests written and passing
- [ ] Security review completed
- [ ] Performance benchmarked

## Common Issues and Solutions

### 1. **OAuth Callback Issues**

```typescript
// Ensure redirect URI matches exactly
const redirectUri = `${window.location.origin}/integrations/callback`;

// Handle OAuth errors gracefully
if (error === 'access_denied') {
  throw new Error('User denied access. Please try connecting again if you want to use this integration.');
}
```

### 2. **API Rate Limiting**

```typescript
// Implement retry with exponential backoff
async function callAPIWithRetry(url: string, options: any, maxRetries = 3): Promise<Response> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    const response = await fetch(url, options);
    
    if (response.status === 429) {
      const retryAfter = parseInt(response.headers.get('Retry-After') || '60');
      const delay = Math.min(retryAfter * 1000, Math.pow(2, attempt) * 1000);
      
      if (attempt === maxRetries) {
        throw new Error(`Question: The ${serviceName} service is currently rate limited. Please try again in a few minutes.`);
      }
      
      await new Promise(resolve => setTimeout(resolve, delay));
      continue;
    }
    
    return response;
  }
  
  throw new Error('Max retry attempts reached');
}
```

### 3. **Credential Refresh**

```typescript
// Implement automatic token refresh for OAuth
async function refreshOAuthToken(refreshToken: string, providerConfig: any) {
  const response = await fetch(providerConfig.token_endpoint, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'refresh_token',
      refresh_token: refreshToken,
      client_id: providerConfig.client_id
    })
  });
  
  if (!response.ok) {
    throw new Error('Question: Your connection has expired. Please reconnect your account in the integration settings.');
  }
  
  return await response.json();
}
```

## Related Documentation

- **[Integration Architecture](../02_integrations/integration_architecture.mdc)** - Overall integration system design
- **[Service Providers Schema](../01_database/service_providers_schema.mdc)** - Database configuration
- **[Universal Tool Executor](../06_backend_services/universal_tool_executor.mdc)** - Tool routing system
- **[OAuth Flow Protocol](../02_integrations/oauth_flow_protocol.mdc)** - OAuth implementation details
- **[Tool Retry System](../03_tools/tool_retry_system.mdc)** - Error handling patterns

---

**Last Updated**: September 17, 2025  
**Integration Support**: OAuth 2.0, API Keys, SMTP  
**Current Integrations**: 15+ providers with 50+ tools

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maverick-software) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
