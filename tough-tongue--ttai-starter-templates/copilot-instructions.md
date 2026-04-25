## ttai-starter-templates

> Tough Tongue AI provides a conversational AI platform for practicing and improving communication skills. This guide contains key elements for developers to integrate Tough Tongue AI into their websites or applications.

# Tough Tongue AI - Developer Integration Guide

## Overview

Tough Tongue AI provides a conversational AI platform for practicing and improving communication skills. This guide contains key elements for developers to integrate Tough Tongue AI into their websites or applications.

## API Authentication

- Create an API token from the [API Keys page](mdc:https:/app.toughtongueai.com/developer?tab=api-keys)
- Include the token as a bearer token in all API requests:
  ```
  Authorization: Bearer YOUR_API_TOKEN
  ```

## API Endpoints

### 1. Creating Scenarios

**Endpoint:** `https://api.toughtongueai.com/api/public/scenarios`  
**Method:** POST

```json
{
  "name": "Customer Support Training",
  "description": "Internal description for your team",
  "user_friendly_description": "Practice handling difficult customer inquiries",
  "ai_instructions": "You are a frustrated customer with a billing issue...",
  "is_public": true,
  "is_recording": true
}
```

**Required Fields:**
- `name`: The name of your scenario
- `description`: Internal description for your reference
- `ai_instructions`: Instructions for the AI on how to behave

**Optional Fields:**
- `user_friendly_description`: Description shown to users
- `pdf_context`: Additional context in PDF format (base64 encoded)
- `rubrik`: Evaluation criteria
- `is_public`: Whether the scenario is publicly accessible
- `passcode`: Optional passcode protection
- `is_recording`: Whether to record conversations
- `restrict_analysis`: Whether to restrict analysis features

### 2. Listing Scenarios

**Endpoint:** `https://api.toughtongueai.com/api/public/scenarios`  
**Method:** GET

### 3. Listing Sessions for a Scenario

**Endpoint:** `https://api.toughtongueai.com/api/public/sessions?scenario_id=SCENARIO_ID&user_email=optional@email.com`  
**Method:** GET

### 4. Getting Session Details

**Endpoint:** `https://api.toughtongueai.com/api/public/sessions/SESSION_ID`  
**Method:** GET

### 5. Analyzing Session Data

**Endpoint:** `https://api.toughtongueai.com/api/public/sessions/analyze`  
**Method:** POST

```json
{
  "session_id": "SESSION_ID"
}
```

### 6. Listing Featured Scenarios

**Endpoint:** `https://api.toughtongueai.com/api/public/featured-scenarios`  
**Method:** GET

### 7. Listing Purchases

**Endpoint:** `https://api.toughtongueai.com/api/public/purchases`  
**Method:** GET

## Embedding Scenarios in Your Application

### Basic Embed

```html
<iframe
  src="https://app.toughtongueai.com/embed/basic/SCENARIO_ID"
  width="100%"
  height="600px"
  frameborder="0"
  allow="microphone"
></iframe>
```

### Full Embed

```html
<iframe
  src="https://app.toughtongueai.com/embed/SCENARIO_ID"
  width="100%"
  height="800px"
  frameborder="0"
  allow="microphone"
></iframe>
```

### Minimal Embed

```html
<iframe
  src="https://app.toughtongueai.com/embed/minimal/SCENARIO_ID"
  width="100%"
  height="300px"
  frameborder="0"
  allow="microphone"
></iframe>
```

### Pre-populating User Information

Add URL parameters to skip the login screen:

```html
<iframe
  src="https://app.toughtongueai.com/embed/SCENARIO_ID?userName=John%20Doe&userEmail=john@example.com"
  width="100%"
  height="800px"
  frameborder="0"
  allow="microphone"
></iframe>
```

### Customization Options

- `name`: Custom conversation name (e.g., `?name=Support%20Training`)
- `color`: Accent color (e.g., `?color=violet-500`)
- `showPulse`: Toggle pulse animation (e.g., `?showPulse=false`)
- `background`: Background color (e.g., `?background=black`)
- `hidePoweredBy`: Hide branding (e.g., `?hidePoweredBy=true`)

## Handling Iframe Events

Add this code to the page that hosts your iframe to listen for events:

```javascript
window.addEventListener('message', (event) => {
  // Optional: verify the origin for security
  // if (event.origin !== 'https://app.toughtongueai.com') return;

  const data = event.data;

  if (data && data.event) {
    switch (data.event) {
      case 'onStart':
        console.log('Session started:', data);
        // Handle session start: initialize analytics, etc.
        break;

      case 'onStop':
        console.log('Session stopped:', data);
        // Handle session end: redirect to results, etc.
        break;
    }
  }
});
```

Event payload structure:
```javascript
{
  event: 'onStart' | 'onStop' | 'onTerminated',  // Event type
  sessionId: string,                           // Unique session identifier
  timestamp: number                            // Unix timestamp in milliseconds
}
```

## Use Cases

Tough Tongue AI can be integrated for:
- Interview preparation
- Sales training
- Customer service role play
- Leadership development
- Performance review practice
- Conflict resolution training

## Limitations

- Conversation Duration: Limited to 10-12 minutes per session (based on token consumption)
- Use Chrome browser for best experience. Recording has issues on Safari.

## Support and Resources

- API Playground: [https://app.toughtongueai.com/api-playground](mdc:https:/app.toughtongueai.com/api-playground)
- Developer Community: [https://discord.com/invite/jfq2wVAP](mdc:https:/discord.com/invite/jfq2wVAP)
- Support Email: [help@getarchieai.com](mdc:mailto:help@getarchieai.com)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tough-tongue) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
