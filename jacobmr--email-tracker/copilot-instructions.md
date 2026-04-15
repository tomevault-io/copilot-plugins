## email-tracker

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md - Email Tracker

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

Email Tracker is a comprehensive email tracking and campaign management system that provides pixel tracking, link click tracking, and analytics for email campaigns. The system consists of a Node.js backend API, a web dashboard, and a browser extension for Gmail integration.

## Architecture

### System Components
- **Backend API**: Node.js/Express server for tracking and analytics
- **Web Dashboard**: HTML/CSS/JS frontend for campaign management
- **Browser Extension**: Chrome extension for Gmail integration
- **Database**: PostgreSQL for tracking data storage
- **Infrastructure**: Cloudflare Zero Trust with home server deployment

### Key Technologies
- **Backend**: Node.js 18+, Express.js, PostgreSQL
- **Frontend**: Vanilla HTML/CSS/JavaScript
- **Extension**: Chrome Extension Manifest V3
- **Deployment**: PM2 process manager, Cloudflare tunnels
- **Security**: Helmet.js, CORS, rate limiting
- **Database**: PostgreSQL with migrations

## Development Commands

### Backend Development
```bash
cd email-tracker/backend

# Install dependencies
npm install

# Development server with auto-reload
npm run dev

# Production server
npm run start:prod

# Database setup
npm run setup:db
npm run migrate

# Production database setup
npm run setup:db:prod
npm run migrate:prod
```

### Extension Development
```bash
cd email-tracker/extension

# Load unpacked extension in Chrome
# 1. Open chrome://extensions/
# 2. Enable Developer mode
# 3. Click "Load unpacked"
# 4. Select extension directory

# Package extension
zip -r email-tracker-extension.zip . -x "*.DS_Store"
```

### Frontend Development
```bash
# Serve frontend files
cd email-tracker/frontend

# Files served by backend at:
# http://localhost:3001/dashboard
# http://localhost:3001/extension
```

## Deployment

### Home Server Deployment (Primary)
```bash
# Automated deployment via GitHub Actions
git push origin main

# Manual deployment
ssh jacob@172.28.108.247
cd /var/www/projects/email-tracker
git pull origin main
cd backend
npm install
pm2 restart email-tracker
```

### Server Configuration
- **Server**: jacob-casa-o (172.28.108.247)
- **Application Directory**: `/var/www/projects/email-tracker`
- **Process Manager**: PM2
- **Port**: 3001
- **Domain**: email-tracker.brasilito.org (via Cloudflare tunnel)

### PM2 Configuration (ecosystem.config.js)
```javascript
module.exports = {
  apps: [{
    name: 'email-tracker',
    script: 'src/index.js',
    instances: 1,
    autorestart: true,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'production',
      PORT: 3001,
      CORS_ORIGIN: 'https://email-tracker.brasilito.org',
      TRUST_PROXY: true
    }
  }]
};
```

### GitHub Actions Deployment
- **Workflow**: Automated deployment on push to main
- **SSH Access**: Uses GitHub secrets for SSH key
- **Secrets Required**:
  - `HOME_SERVER_SSH_KEY`: Private SSH key
  - `HOME_SERVER_HOST`: 172.28.108.247
  - `HOME_SERVER_USER`: jacob

## Environment Configuration

### Required Environment Variables
```bash
# Server Configuration
NODE_ENV=production
PORT=3001
CORS_ORIGIN=https://email-tracker.brasilito.org
TRUST_PROXY=true

# Database Configuration
DATABASE_URL=postgresql://user:password@localhost:5432/email_tracker
PGUSER=email_tracker_user
PGPASSWORD=your_secure_password
PGDATABASE=email_tracker
PGHOST=localhost
PGPORT=5432

# Security Configuration
JWT_SECRET=your_jwt_secret
API_KEY=your_api_key

# Tracking Configuration
TRACKING_DOMAIN=email-tracker.brasilito.org
PIXEL_ENDPOINT=/api/track/pixel
LINK_ENDPOINT=/api/track/link
```

## Database Schema

### Core Tables
```sql
-- Email tracking records
CREATE TABLE email_opens (
    id SERIAL PRIMARY KEY,
    email_id VARCHAR(255) NOT NULL,
    recipient_email VARCHAR(255),
    ip_address INET,
    user_agent TEXT,
    opened_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    campaign_id VARCHAR(255)
);

-- Link click tracking
CREATE TABLE link_clicks (
    id SERIAL PRIMARY KEY,
    email_id VARCHAR(255) NOT NULL,
    link_url TEXT NOT NULL,
    recipient_email VARCHAR(255),
    ip_address INET,
    user_agent TEXT,
    clicked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    campaign_id VARCHAR(255)
);

-- Campaign management
CREATE TABLE campaigns (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Migration System
```bash
# Run database migrations
npm run migrate

# Migration files in backend/migrations/
# 001_initial_schema.sql
# 002_add_link_clicks_table.sql
# 003_add_ip_address_to_emails.sql
```

## API Endpoints

### Tracking Endpoints
- **GET** `/api/track/pixel/:emailId`: Email open tracking pixel
- **GET** `/api/track/link/:emailId`: Link click tracking and redirect
- **POST** `/api/track/event`: Custom event tracking

### Analytics Endpoints
- **GET** `/api/analytics/campaign/:campaignId`: Campaign analytics
- **GET** `/api/analytics/email/:emailId`: Email-specific analytics
- **GET** `/api/analytics/summary`: Overall tracking summary

### Management Endpoints
- **GET** `/api/health`: Health check endpoint
- **GET** `/dashboard`: Web dashboard interface
- **GET** `/extension`: Extension setup page

### Frontend Routes
- **GET** `/`: Main landing page
- **GET** `/dashboard`: Analytics dashboard
- **GET** `/extension`: Extension download and setup

## Cloudflare Infrastructure

### Zero Trust Setup
```yaml
# /etc/cloudflared/config.yml
tunnel: dev-server
credentials-file: /etc/cloudflared/credentials.json

ingress:
  - hostname: email-tracker.brasilito.org
    service: http://localhost:3001
  - service: http_status:404
```

### Cloudflare Access Policy
- **Application**: email-tracker.brasilito.org
- **Policy**: Allow jacob@reider.us
- **Authentication**: Email-based authentication

### DNS Configuration
- **Type**: CNAME
- **Name**: email-tracker
- **Content**: Cloudflare tunnel endpoint
- **Proxy**: Enabled (orange cloud)

## Browser Extension

### Extension Structure
```
extension/
├── manifest.json          # Extension manifest
├── popup.html            # Extension popup interface
├── js/
│   ├── background.js     # Background service worker
│   ├── content.js        # Gmail content script
│   └── popup.js          # Popup logic
└── images/               # Extension icons
```

### Key Features
- **Gmail Integration**: Inject tracking pixels and links
- **Campaign Management**: Create and manage campaigns
- **Real-time Analytics**: View tracking data in popup
- **Link Wrapping**: Automatic link tracking setup

### Installation
1. Download extension package
2. Open Chrome Extensions (chrome://extensions/)
3. Enable Developer mode
4. Load unpacked extension
5. Pin extension to toolbar

## Monitoring and Analytics

### Health Monitoring
```bash
# Service status
pm2 status email-tracker

# Application health
curl https://email-tracker.brasilito.org/api/health

# System logs
pm2 logs email-tracker

# Cloudflare tunnel status
sudo systemctl status cloudflared
```

### Analytics Features
- **Email Open Tracking**: Pixel-based open detection
- **Link Click Tracking**: Click-through rate monitoring
- **Geographic Analytics**: IP-based location tracking
- **Device Analytics**: User agent analysis
- **Campaign Performance**: ROI and engagement metrics

### Performance Monitoring
- **Memory Usage**: PM2 restart at 1GB memory limit
- **Response Time**: API endpoint performance tracking
- **Error Rates**: Error logging and monitoring
- **Database Performance**: Query optimization and indexing

## Security and Privacy

### Security Measures
- **Helmet.js**: Security headers and protection
- **CORS**: Cross-origin request protection
- **Rate Limiting**: API abuse prevention
- **Input Validation**: SQL injection prevention
- **Authentication**: Secure access control

### Privacy Considerations
- **IP Address Handling**: GDPR-compliant IP storage
- **Data Retention**: Configurable data retention policies
- **Anonymization**: Personal data anonymization options
- **Consent Management**: Email tracking consent handling

## Development Workflow

1. **Local Development**: Use Node.js development server
2. **Extension Testing**: Load unpacked extension in Chrome
3. **Database Changes**: Create migration files
4. **Testing**: Manual testing of tracking functionality
5. **Deployment**: Push to GitHub for automatic deployment
6. **Monitoring**: Check PM2 logs and health endpoints

## Troubleshooting

### Common Issues
- **Port Conflicts**: Ensure port 3001 is available
- **Database Connection**: Check PostgreSQL connectivity
- **Cloudflare Tunnel**: Verify tunnel configuration
- **Extension Permissions**: Check Chrome extension permissions

### Debugging Commands
```bash
# Check running processes
sudo netstat -tulpn | grep :3001

# PM2 debugging
pm2 status
pm2 logs email-tracker --lines 100

# Database connectivity
psql -U email_tracker_user -d email_tracker -h localhost

# Cloudflare tunnel
cloudflared tunnel info dev-server
sudo journalctl -u cloudflared -f
```

### Recovery Procedures
```bash
# Restart services
pm2 restart email-tracker
sudo systemctl restart cloudflared

# Database recovery
npm run setup:db:prod
npm run migrate:prod

# Complete redeployment
cd /var/www/projects/email-tracker
git pull origin main
cd backend
npm install
pm2 restart email-tracker
```

## AI Development Notes

- **Real Email Integration**: Work with actual email systems, not mock data
- **Privacy Compliance**: Implement GDPR and privacy-compliant tracking
- **Production Infrastructure**: Use actual Cloudflare and server infrastructure
- **Browser Extension**: Real Chrome extension development
- **Database Persistence**: Use actual PostgreSQL database
- **Analytics**: Real tracking data and analytics
- **Security**: Implement proper security measures for production use

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jacobmr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jacobmr)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
