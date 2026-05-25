## varnion-qr-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important: Documentation Updates

**CRITICAL RULE**: Every change, feature addition, or update made to this project MUST be reflected in the documentation. When making any modifications:
1. Update CLAUDE.md if architecture, structure, or processes change
2. Update PRD.md if requirements or features are modified
3. Keep all documentation synchronized with the actual codebase

## Project Overview

**Varnion QR Code Generator** is a web-based application that enables users to create customizable QR codes with advanced styling options. The application provides real-time preview, logo integration, and multiple export formats without requiring user authentication.

**Key Links:**
- Product Requirements: `/home/bintt/projects/node/varnion-qr-generator/PRD.md`

## Current Status

**Project Status:** Planning Phase  
**Development Stage:** Not yet started - awaiting initial setup and development  
**Repository Type:** Brand new project (no existing codebase)

## Project Structure

Currently, the project only contains the Product Requirements Document (PRD.md). The following directories and files will be created during development:

```
varnion-qr-generator/
├── PRD.md                              # Product Requirements Document
├── package.json                        # Dependencies and scripts
├── next.config.js                      # Next.js configuration
├── tailwind.config.js                  # Tailwind CSS configuration
├── .gitignore                          # Git ignore rules
├── .env.local                          # Environment variables (local)
├── README.md                           # Project documentation
├── CLAUDE.md                           # This file
│
├── public/
│   └── assets/
│       └── default-logo.jpg            # Varnion default logo
│
├── app/
│   ├── page.js                         # Main QR generator page
│   ├── layout.js                       # Root layout
│   └── api/
│       ├── generate-qr/route.js        # QR generation endpoint (optional)
│       └── health/route.js             # Health check endpoint
│
├── components/
│   ├── qr-generator/                   # Main QR generation component
│   ├── qr-preview/                     # Real-time preview canvas
│   ├── qr-type-selector/               # Type selection UI
│   ├── customization/                  # Customization panels
│   │   ├── color-customizer.js
│   │   ├── shape-customizer.js
│   │   ├── logo-uploader.js
│   │   └── error-correction.js
│   └── export/                         # Export controls
│
└── lib/
    ├── qr-generator.js                 # QR generation utilities
    ├── validators.js                   # Input validation logic
    └── utils.js                        # Helper functions
│
└── tests/ (future)
    ├── unit/
    ├── integration/
    └── e2e/
```

## Technology Stack

### Frontend
- **Framework:** Next.js 14+ (React) with App Router
- **Language:** JavaScript (ES6+) - No TypeScript/compilation for faster development
- **Styling:** Tailwind CSS
- **UI Components:** Radix UI or shadcn/ui (accessible components)
- **Form Handling:** React Hook Form + Zod (runtime validation)
- **Color Picker:** react-colorful or similar

### Backend
- **Runtime:** Node.js 18+ LTS
- **Server Framework:** Next.js API Routes
- **Image Processing:** sharp (logo integration, format conversion)
- **PDF Generation:** jspdf
- **QR Generation:** qr-code-styling (supports advanced customization)
- **File Handling:** file-saver (client-side downloads)

### Deployment
- **Target Environment:** VPS (Ubuntu 20.04+ / Debian 11+)
- **Process Manager:** PM2
- **Web Server:** Nginx (reverse proxy)
- **No Docker:** Direct Node.js deployment
- **SSL:** Let's Encrypt (Certbot)

### Development Tools
- **Package Manager:** npm or yarn
- **Version Control:** Git
- **Code Quality:** ESLint, Prettier (recommended)
- **Testing:** Jest, React Testing Library (recommended)
- **No TypeScript:** Pure JavaScript for faster iteration without compilation overhead

## Core Features

### 1. QR Code Types (7 types)
- **URL**: Website URL generation
- **Plain Text**: Up to 500 characters
- **Email**: With optional subject and body
- **Phone Number**: Tel format with validation
- **WiFi**: Network config (SSID, password, encryption)
- **vCard (Contact)**: Full contact information
- **SMS**: With optional message body

### 2. Customization Features
- **Colors**: Foreground, background, and gradient support
- **Gradient Types**: Linear with direction options (horizontal, vertical, diagonal)
- **Shapes**: Dot styles (square, rounded, dots, extra-rounded)
- **Corner Styles**: For corner squares and dots
- **Logo Integration**: 
  - Default Varnion logo pre-loaded
  - Upload custom logos (PNG, JPG, JPEG, SVG)
  - Logo size: 10-30% of QR code
  - Support for square, circular, and rectangular logos
  - Option to add white padding
  - Remove or reset to default logo
- **Error Correction Level**: L (7%), M (15%), Q (25%), H (30%)

### 3. Real-Time Preview
- Live canvas showing QR code changes
- Debounced updates (300ms)
- Responsive preview (minimum 300x300px)
- Loading indicator during generation
- Error state display

### 4. Export & Download
- **Formats**: PNG, JPG, SVG, PDF
- **Sizes**:
  - PNG/JPG: 1024px, 2048px, 4096px
  - SVG: Scalable (vector format)
  - PDF: A4, Letter, or custom dimensions
- **Quality**: JPG quality selector (90%, 95%, 100%)
- **Auto-generated filenames**: `qr-{type}-{timestamp}.{format}`

### 5. Additional Features
- Reset all inputs and customizations
- Confirmation modal before reset
- Template system (P1 - future): Pre-designed style templates

## Design Specifications

### Color Palette
- **Primary Brand Color**: #2A3D82 (Navy blue - Varnion brand)
  - Hover state: #3A4D92
  - Active state: #1A2D72
- **Secondary/Light Backgrounds**: #F0F2F8 (very light blue-gray)
- **Text on Primary**: #FFFFFF (white)
- **Borders/Dividers**: #D0D5E8 (light gray-blue)
- **Neutral**: White/light gray

### Typography
- **Headings**: 24px, 20px, 18px (bold)
- **Body**: 16px (regular)
- **Labels**: 14px (medium)
- **Captions**: 12px (regular)

### Layout
- **Desktop (1024px+)**: Side-by-side layout (controls | preview)
- **Tablet (768px-1023px)**: Stacked layout, preview below
- **Mobile (320px-767px)**: Full-width stacked, preview at bottom

### Responsive Components
- Rounded buttons (8px radius)
- Input fields with clear labels
- Collapsible customization sections
- Color picker with recent colors history
- Drag-and-drop file upload area

## Performance Targets

- **Initial Page Load**: < 2 seconds
- **QR Code Generation**: < 500ms
- **Real-time Preview Update**: < 300ms debounce
- **Export Generation**: < 2 seconds for high-res images
- **Bundle Size**: < 500KB initial JS
- **Concurrent Users**: 1000+

## Accessibility Requirements

- WCAG 2.1 AA compliance
- Full keyboard navigation
- Focus indicators on all interactive elements
- ARIA labels for icon buttons
- Color contrast ratio > 4.5:1
- Screen reader announcements for real-time updates
- Alt text for images and logos

## Browser Compatibility

- **Desktop**: Chrome, Firefox, Safari, Edge (latest 2 versions)
- **Mobile**: iOS Safari, Chrome Mobile
- **Screen Sizes**: 320px (mobile) to 4K displays

## Development Phases

### Phase 1: MVP (Weeks 1-2)
- Project setup (Next.js 14+, JavaScript, Tailwind)
- Basic UI layout (responsive)
- 3 QR types: URL, Text, Email
- Basic customization: Foreground/background colors
- Real-time preview
- PNG export (1024x1024px)

### Phase 2: Extended Features (Weeks 3-4)
- Remaining QR types: Phone, WiFi, vCard, SMS
- Advanced customization: Gradients, shapes, patterns
- Logo upload with default Varnion logo
- Support for rectangular, square, and circular logos
- Multiple export formats: PNG, JPG, SVG, PDF
- Multiple export sizes and quality options
- Error correction level selector

### Phase 3: Polish & Optimization (Week 5)
- UI/UX refinements
- Performance optimization
- Accessibility improvements (WCAG AA)
- Cross-browser testing
- Mobile responsiveness testing
- Error handling and edge cases
- Documentation

### Phase 4: Deployment (Week 6)
- VPS setup and configuration
- Nginx configuration (reverse proxy, SSL)
- PM2 setup for process management
- Environment configuration
- Monitoring setup
- Load testing and go-live

## Server Requirements

- **OS**: Ubuntu 20.04+ or Debian 11+
- **CPU**: 2+ cores
- **RAM**: 2GB minimum, 4GB recommended
- **Storage**: 20GB minimum
- **Network**: Static IP (optional but recommended)

### Software Stack (VPS)
- Node.js: v18.x or v20.x (LTS)
- PM2: Latest stable
- Nginx: Latest stable
- SSL: Let's Encrypt (Certbot)

## Key Dependencies

### Production Dependencies
- `next`: ^14.0.0
- `react`: ^18.0.0
- `react-dom`: ^18.0.0
- `tailwindcss`: Latest
- `qr-code-styling`: For advanced QR customization
- `sharp`: Image processing
- `jspdf`: PDF generation
- `file-saver`: Client-side downloads
- `react-hook-form`: Form handling
- `zod`: Runtime validation schema
- `react-colorful`: Color picker (or alternative)
- `@radix-ui/*`: UI components (or shadcn/ui)

### Development Dependencies
- `eslint`: Code linting
- `prettier`: Code formatting
- `jest`: Testing
- `@testing-library/react`: Component testing

## Important Notes

### Default Logo Behavior
- Application must load with a predefined Varnion logo on startup
- The default logo serves as both a placeholder and visual example
- Users can replace, remove, or reset to default logo
- Supported logo formats: PNG, JPG, JPEG, SVG
- Max file size: 2MB
- Min dimensions: 100x100px

### Input Validation
- All inputs validated in real-time
- User-friendly error messages
- Prevents invalid QR code generation
- XSS protection through input sanitization
- File upload validation (type, size, content)

### QR Code Reliability
- All generated QR codes must be scannable by major readers
- iOS Camera app, Google Lens, dedicated QR apps must work
- High error correction level recommended with logos
- Extensive testing required before launch

### Data Privacy
- No sensitive data logging
- No user authentication required
- No data storage or history
- Client-side processing where possible
- CSP headers implementation
- Rate limiting for API endpoints

## Future Enhancements (Out of Scope v1.0)

- User authentication and accounts
- QR code history or storage
- Batch QR code generation
- QR code analytics/tracking
- Third-party API integration
- Advanced analytics with scan tracking
- Team collaboration features
- White-label/custom branding options
- Payment processing

## Success Metrics

### Technical Metrics
- Page load < 2s, QR generation < 500ms
- 99.5% uptime
- Error rate < 0.5%
- 95%+ browser compatibility

### User Metrics
- Avg session > 3 minutes
- QR codes per session > 2
- Customization adoption > 40%
- Logo upload adoption > 25%
- User satisfaction rating > 4/5

## Getting Started with Development

### Initial Setup Steps
1. Create Next.js 14+ project with JavaScript (no TypeScript)
2. Install Tailwind CSS
3. Setup component structure
4. Implement QR generation library
5. Build main page layout
6. Implement QR type selector
7. Create customization controls
8. Add real-time preview
9. Implement export functionality
10. Add validation and error handling with Zod
11. Optimize performance
12. Add tests
13. Deploy to VPS

### Key Commands (Once Setup)
```bash
npm install                 # Install dependencies
npm run dev                # Development server
npm run build              # Production build
npm run start              # Start production server
npm run lint               # ESLint check
npm run format             # Prettier format (optional)
npm test                   # Run tests
```

## Configuration Files to Create

### .env.local (Development)
```
NEXT_PUBLIC_APP_NAME=Varnion QR Generator
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### next.config.js
- Image optimization
- Tailwind CSS integration
- API compression
- Security headers
- React strict mode

## Documentation to Create

1. **README.md**: User guide and project overview
2. **DEPLOYMENT.md**: VPS setup and deployment instructions
3. **DEVELOPMENT.md**: Developer guide and architecture
4. **API.md**: API endpoint documentation
5. **CONTRIBUTING.md**: Contribution guidelines

## Contact & Support

For questions about the project specifications, refer to the PRD.md file located at:
`/home/bintt/projects/node/varnion-qr-generator/PRD.md`

## Document Metadata

- **Created**: 2025-10-27
- **Last Updated**: 2025-10-27
- **Status**: Project Planning Complete - Ready for Development
- **Version**: 1.0
- **Project Phase**: Pre-Development (Setup Phase)

---

**This CLAUDE.md was generated to provide context for AI assistants working on the Varnion QR Code Generator project.**

---
> Source: [varnionjgj/varnion-qr-generator](https://github.com/varnionjgj/varnion-qr-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
