## dropemail-net

> Build a modern, clean email header analyzer that parses .eml files and raw email headers to display security, routing, and authentication information similar to MXToolbox.

# Email Analyzer App - Cursor Rules

## Project Overview
Build a modern, clean email header analyzer that parses .eml files and raw email headers to display security, routing, and authentication information similar to MXToolbox.

## Tech Stack
- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS v4
- **UI Components**: shadcn/ui
- **Email Parsing**: mailparser, addressparser
- **Icons**: lucide-react
- **State Management**: React hooks (useState, useReducer)

## Core Functionality

### Input Methods
1. **File Upload**: Accept .eml files via drag-and-drop or file picker
2. **Text Input**: Large textarea for pasting raw email headers
3. Support parsing both full emails and header-only content

### Analysis & Display
Parse and display the following information in organized sections:

#### 1. Summary Card
- Subject line
- From/To addresses
- Date/Time (with timezone)
- Message-ID
- Overall security score/status indicator

#### 2. Authentication Results
- **SPF**: Pass/Fail/Neutral with domain
- **DKIM**: Signature verification status with selector
- **DMARC**: Policy and result
- **ARC**: Authentication chain (if present)
Display with color-coded badges (green=pass, red=fail, yellow=neutral)

#### 3. Routing Information
- Full "Received" chain in chronological order
- Display: Server name, IP address, timestamp, delay between hops
- Visualize the email journey with a timeline or flowchart
- Highlight any suspicious hops or delays

#### 4. Security Analysis
- Spam score (if available in headers like X-Spam-Score)
- Suspicious patterns detection
- IP geolocation for sender
- Blacklist checking status
- TLS/SSL encryption indicators

#### 5. Technical Details (Collapsible)
- All parsed headers in organized key-value pairs
- MIME structure
- Content-Type information
- Attachments list (name, size, type)
- Raw header view (syntax highlighted)

## Design Principles

### Visual Design
- **Clean & Modern**: Minimal design, ample whitespace
- **Card-based Layout**: Group related information in cards
- **Color System**: 
  - Green for verified/secure
  - Red for failed/suspicious
  - Yellow for warnings
  - Gray for neutral/info
- **Responsive**: Mobile-first design, works on all screen sizes
- **Dark Mode**: Support system preference with toggle

### UX Guidelines
- **Progressive Disclosure**: Show summary first, details on demand
- **Copy to Clipboard**: Add copy buttons for IPs, domains, headers
- **Export Options**: Allow exporting analysis as JSON or PDF
- **Loading States**: Show parsing progress for large files
- **Error Handling**: Clear error messages for invalid input
- **Tooltips**: Explain technical terms on hover

## Code Structure

### File Organization
```
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   └── api/
│       └── analyze/
├── components/
│   ├── ui/              # shadcn components
│   ├── EmailInput.tsx   # Upload & paste interface
│   ├── AnalysisView.tsx # Main results display
│   ├── SummaryCard.tsx
│   ├── AuthResults.tsx
│   ├── RoutingChain.tsx
│   ├── SecurityPanel.tsx
│   └── TechnicalDetails.tsx
├── lib/
│   ├── email-parser.ts  # Core parsing logic
│   ├── header-analyzer.ts
│   ├── security-checker.ts
│   └── utils.ts
└── types/
    └── email.ts         # TypeScript interfaces
```

### Type Definitions
```typescript
interface EmailAnalysis {
  summary: {
    subject: string;
    from: EmailAddress;
    to: EmailAddress[];
    date: Date;
    messageId: string;
  };
  authentication: {
    spf: AuthResult;
    dkim: AuthResult;
    dmarc: AuthResult;
    arc?: AuthResult;
  };
  routing: ReceivedHeader[];
  security: SecurityAnalysis;
  headers: Record<string, string>;
  attachments?: Attachment[];
}
```

## Parsing Requirements

### Header Parsing
- Extract all standard headers (From, To, Subject, Date, etc.)
- Parse complex headers like Received, Authentication-Results
- Handle multi-line headers correctly
- Decode encoded subjects (=?UTF-8?B?...)
- Parse email addresses with display names

### Authentication Parsing
- Extract SPF results from Received-SPF or Authentication-Results
- Parse DKIM-Signature and verification results
- Extract DMARC policy and results
- Handle multiple authentication headers

### Routing Analysis
- Parse all "Received" headers
- Extract: from/by servers, IPs, timestamps, protocols
- Calculate time delays between hops
- Identify the originating server

## Features to Implement

### Phase 1 (MVP)
- [ ] File upload (.eml)
- [ ] Header paste input
- [ ] Basic header parsing
- [ ] Summary display
- [ ] Authentication results
- [ ] Routing chain visualization

### Phase 2
- [ ] Security analysis
- [ ] IP geolocation
- [ ] Copy to clipboard
- [ ] Export results
- [ ] Dark mode
- [ ] Mobile responsive

### Phase 3
- [ ] Batch processing
- [ ] History/saved analyses
- [ ] Advanced filtering
- [ ] API endpoint
- [ ] Comparison tool

## Best Practices

### Code Quality
- Use TypeScript strict mode
- Implement proper error boundaries
- Write unit tests for parsing logic
- Use Zod for runtime validation
- Follow React best practices (hooks rules, memo when needed)

### Performance
- Parse files in Web Worker for large .eml files
- Lazy load heavy components
- Optimize re-renders with useMemo/useCallback
- Implement virtualization for long lists

### Security
- Sanitize all displayed content (prevent XSS)
- Don't send email content to external servers
- Process everything client-side
- Warn users about privacy when analyzing sensitive emails

### Accessibility
- Semantic HTML
- ARIA labels for interactive elements
- Keyboard navigation support
- Screen reader friendly
- Sufficient color contrast

## Example Component Pattern

```typescript
'use client';

import { useState } from 'react';
import { Card } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';

export function AuthResults({ auth }: { auth: Authentication }) {
  return (
    <Card className="p-6">
      <h2 className="text-xl font-semibold mb-4">Authentication</h2>
      <div className="space-y-3">
        <div className="flex items-center justify-between">
          <span className="font-medium">SPF</span>
          <Badge variant={auth.spf.status === 'pass' ? 'success' : 'destructive'}>
            {auth.spf.status.toUpperCase()}
          </Badge>
        </div>
        {/* Additional auth results */}
      </div>
    </Card>
  );
}
```

## Testing Strategy
- Unit tests for parsing functions
- Integration tests for file upload/processing
- E2E tests for critical user flows
- Test with various email formats (.eml, raw headers)
- Test edge cases (malformed headers, encoding issues)

## Documentation
- README with setup instructions
- Inline code comments for complex logic
- JSDoc for public functions
- User guide for analyzing emails
- Examples of supported formats

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/soulox) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
