## kernelint

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Development**: `npm run dev` - Starts Next.js development server with Turbopack
- **Build**: `npm run build` - Builds the production application with Turbopack
- **Start**: `npm start` - Starts the production server
- **Lint**: `npm run lint` - Runs ESLint (configured to ignore during builds)

## Deployment Commands

- **Docker Build**: `docker-compose build` - Builds the Docker container
- **Docker Development**: `docker-compose up -d kernel-site` - Runs application in Docker
- **Docker Production**: `docker-compose --profile production up -d` - Runs with Nginx reverse proxy
- **Quick Deploy**: `./deploy.sh` - Automated deployment script

## Project Architecture

This is a Next.js 15 application for Kernel International Limited (KIL), a consultancy company website.

### Core Structure

- **App Router**: Uses Next.js App Router with TypeScript
- **Features**: Feature-based architecture under `/features` directory
- **Components**: Organized by feature with barrel exports from `index.ts` files
- **Fonts**: Uses Google Fonts (Inter, Poppins, JetBrains Mono) configured in layout
- **Styling**: Tailwind CSS with custom CSS variables and design system

### Key Dependencies

- **UI**: Lucide React icons, class-variance-authority for component variants
- **Styling**: Tailwind CSS v4, tailwind-merge, clsx for conditional classes
- **Framework**: Next.js 15 with React 19

### Architecture Patterns

**Feature-Based Organization**: Each feature (e.g., `landing`) contains:
- Components in `/components` directory
- Data in `/data` directory
- Types in `/types` directory
- Main feature export from `index.ts`

**Component Pattern**: Components use:
- TypeScript interfaces for props
- Lucide React for icons
- Color-coded variants (rose, blue, green, purple)
- Animation delays for staggered effects

**Data Structure**: Static data is centralized in `landing-data.ts` with typed interfaces covering:
- Hero statistics and services
- Company information (mission, vision, established dates)
- Contact details and team members
- Service categories and expertise areas

### Build Configuration

- TypeScript and ESLint errors ignored during builds (configured in next.config.ts)
- Path aliases: `@/*` maps to project root
- Turbopack enabled for faster development and builds
- Standalone output configured for Docker deployment

### Docker Configuration

- Multi-stage Dockerfile for optimized production builds
- Docker Compose with development and production profiles
- Nginx reverse proxy for production with SSL support
- Watchtower for automatic container updates in production
- Health checks configured for container monitoring

### Content Management

Company content is managed through structured data objects in `/features/landing/data/landing-data.ts`. This includes hero stats, services, team information, and contact details that populate the landing page sections.

### Company Technical Capabilities

Kernel International Limited positions itself as an "End-to-End AI Development Partner" with expertise across:

**Core Technology Areas:**
- Data Analytics & Data Science (R, Python, Dask, Google Analytics, Power BI)
- Cloud AI Platforms (IBM Watson, Cortana Intelligence)
- Deep Learning (TensorFlow, Keras, MLlib, FastAI, Transformers, spaCy, PyTorch)
- Generative AI (LangChain, OpenAI)
- DevOps & MLOps (Docker, Kubernetes, Lov, Azure ML)
- Chatbot Development (LangChain, OpenAI)
- AI Model Optimization (3D modeling, optimization tools)
- Backend Development (PHP, Django, JavaScript, Node.js)
- Frontend Development (Next.js, Svelte, Vue.js, Angular)

**Platform Partnerships:**
- ScalableMinds
- ProtexAI
- V7 Darwin
- Lightly
- Heartex
- Segments.ai
- Kili Technology
- CVAT
- Supervisely

This technical stack information should be reflected in the services and capabilities sections of the website content.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krnltech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
