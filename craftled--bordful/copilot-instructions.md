## airtable-integration

> Bordful uses Airtable as its backend database, providing a no-code solution for managing job listings with a user-friendly interface.

# Airtable Integration

Bordful uses Airtable as its backend database, providing a no-code solution for managing job listings with a user-friendly interface.

## Database Structure

### Jobs Table
The main table storing all job listings with these key fields:
- **Title** - Job title/position name
- **Company** - Company name
- **Description** - Rich text job description
- **Status** - Publication status (Published, Draft, Archived)
- **Location** - Job location (Remote, City, Country)
- **Type** - Employment type (Full-time, Part-time, Contract, Freelance)
- **Level** - Experience level (Junior, Mid-level, Senior, Lead)
- **Language** - Programming languages/technologies
- **Salary Min/Max** - Salary range
- **Currency** - Salary currency
- **Apply URL** - External application link
- **Company Logo** - Company logo image
- **Posted Date** - Publication date
- **Slug** - URL-friendly identifier

## Data Access Patterns

### Environment Variables
Required Airtable credentials in `.env`:
```env
AIRTABLE_ACCESS_TOKEN=pat_your_token_here
AIRTABLE_BASE_ID=app_your_base_id_here
AIRTABLE_TABLE_NAME=Jobs
```

### Data Fetching
Jobs are fetched using the Airtable API with:
- Filtering by status (only published jobs)
- Sorting by posted date (newest first)
- Field selection for performance
- Pagination for large datasets

## Database Operations

### Read Operations
- **getJobs()** - Fetch all published jobs
- **getJob(id)** - Fetch single job by ID
- **testConnection()** - Test Airtable connection
- Additional filtering and transformation happens in the application layer

### Data Transformation
Raw Airtable data is transformed to:
- Normalize field names and types
- Generate slugs for URLs
- Format dates and currencies
- Process rich text descriptions
- Optimize images and attachments

## Content Management

### Admin Interface
Airtable provides the admin interface for:
- Adding new job listings
- Editing existing jobs
- Managing job status (publish/draft/archive)
- Bulk operations and data import
- User permissions and collaboration

### Content Workflow
1. **Draft Creation** - Jobs start as drafts in Airtable
2. **Content Review** - Team reviews job details
3. **Publication** - Status changed to "Published"
4. **Site Update** - ISR regenerates pages automatically
5. **Archival** - Old jobs moved to "Archived" status

## Data Validation

### Required Fields
- Title (must be present)
- Company (must be present)
- Description (must be present)
- Status (must be valid option)

### Optional Fields
- All other fields are optional
- Default values applied where appropriate
- Graceful handling of missing data

## Performance Optimization

### Caching Strategy
- Static generation at build time
- ISR for dynamic updates
- Client-side caching for filters
- CDN caching for assets

### Data Fetching Optimization
- Selective field fetching
- Batch operations where possible
- Connection pooling
- Rate limit handling

## SEO Integration

### Structured Data
Jobs automatically generate:
- schema.org JobPosting markup
- Open Graph meta tags
- Twitter Card data
- Rich snippets for search engines

### URL Generation
- SEO-friendly slugs from job titles
- Canonical URLs for job pages
- Sitemap generation from job data
- RSS feeds for job updates

## Error Handling

### API Errors
- Graceful degradation for API failures
- Retry logic for transient errors
- Fallback content for missing data
- User-friendly error messages

### Data Validation
- Type checking for all fields
- Sanitization of user input
- Validation of required fields
- Error logging and monitoring

## Development Workflow

### Local Development
- Use Airtable's development environment
- Test data separate from production
- API key management for team members
- Local caching for faster development

### Deployment
- Environment-specific base IDs
- Production API keys
- Data migration strategies
- Backup and recovery procedures

## Best Practices

### Data Management
- Consistent field naming conventions
- Regular data cleanup and archival
- Backup strategies for critical data
- Documentation of schema changes

### Security
- Secure API key storage
- Access control and permissions
- Data privacy compliance
- Regular security audits

---
> Source: [craftled/bordful](https://github.com/craftled/bordful) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
