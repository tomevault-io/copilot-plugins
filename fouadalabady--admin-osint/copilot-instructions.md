## admin-osint

> This document serves as the comprehensive API reference for the OSINT Blog headless API system. It's designed to be used by:  - Frontend developers integrating with the blog API - Backend developers maintaining or extending the API - Third-party integrators building external applications - QA engineers testing API functionality - Technical writers creating derived documentation  The API documentation should be consulted when: - Implementing client applications that consume the blog API - Developing new API endpoints or modifying existing ones - Debugging API integration issues - Planning content migration from other systems - Understanding security requirements for API access - Determining appropriate rate limits and caching strategies - Implementing SEO and social media optimization features  This reference is particularly valuable for development teams working on the public-facing components of the OSINT Dashboard project, as it provides a complete specification of available endpoints, request/response formats, authentication requirements, and security considerations. It complements the Project Architecture document by providing det

# OSINT Blog Headless API Documentation

## Overview
This documentation describes the headless blog API system built with Next.js and Supabase. The system provides a complete backend for managing a multilingual blog with support for posts, categories, tags, media, and user management.

## Implementation Status Key
- ✅ **Implemented** - Feature is fully implemented and available
- 🟡 **Partially Implemented** - Basic functionality is available but may need refinement
- 🔄 **In Progress** - Feature is currently being implemented
- 📅 **Planned** - Feature is planned but not yet implemented

## Base URL
```
Production: https://blogosint-c5temew7u-osint2.vercel.app/api
Development: http://localhost:3000/api
```

## Authentication ✅
The API uses JWT tokens for authentication. Include the token in the Authorization header:
```
Authorization: Bearer <your_jwt_token>
```

## GraphQL Integration ✅
The API includes GraphQL support for more flexible querying:
```
GraphQL Endpoint: /api/graphql
```

Example GraphQL query:
```graphql
query GetBlogPosts($limit: Int, $offset: Int, $where: blog_posts_bool_exp, $order_by: [blog_posts_order_by!]) {
  blog_posts(limit: $limit, offset: $offset, where: $where, order_by: $order_by) {
    id
    title
    slug
    excerpt
    featured_image
    created_at
    updated_at
    author_id
  }
  blog_posts_aggregate {
    aggregate {
      count
    }
  }
}
```

## Database Schema

### Tables Structure
1. **blog_posts** ✅
   - Primary blog content storage
   - Supports multilingual content (RTL/LTR)
   - Includes SEO metadata
   - Tracks view counts and engagement

2. **blog_categories** 🟡
   - Hierarchical content organization
   - Supports multilingual categories

3. **blog_tags** 🟡
   - Content classification and search optimization
   - Many-to-many relationship with posts

4. **blog_media** ✅
   - Media asset management
   - Supports images, videos, and documents
   - Includes metadata and alt text

5. **blog_comments** 📅
   - User engagement tracking
   - Moderation support with approval workflow
   - Threaded discussions

6. **users** ✅
   - Author and administrator management
   - Role-based access control

## API Endpoints

### Posts

#### List Posts ✅
```http
GET /api/posts
```

Query Parameters:
- `page` (integer): Page number for pagination
- `limit` (integer): Items per page (default: 10)
- `status` (string): Filter by status ('draft', 'published')
- `category` (uuid): Filter by category ID
- `tag` (uuid): Filter by tag ID
- `author` (uuid): Filter by author ID
- `direction` (string): Filter by content direction ('ltr', 'rtl')
- `featured` (boolean): Filter featured posts
- `search` (string): Search in title and content 🔄

Response:
```json
{
  "data": [{
    "id": "uuid",
    "title": "string",
    "slug": "string",
    "excerpt": "string?",
    "content": "string?",
    "featured_image": "string?",
    "author": {
      "id": "uuid",
      "name": "string"
    },
    "category": {
      "id": "uuid",
      "name": "string"
    },
    "tags": [{
      "id": "uuid",
      "name": "string"
    }],
    "status": "string",
    "is_featured": boolean,
    "view_count": number,
    "published_at": "datetime?",
    "created_at": "datetime",
    "updated_at": "datetime",
    "seo_title": "string?",
    "seo_description": "string?",
    "seo_keywords": "string?",
    "direction": "string"
  }],
  "metadata": {
    "total": number,
    "page": number,
    "limit": number,
    "total_pages": number
  }
}
```

#### Get Single Post ✅
```http
GET /api/posts/{slug}
```

Response: Single post object with full details

#### Create Post ✅
```http
POST /api/posts
```

Request Body:
```json
{
  "title": "string",
  "content": "string",
  "excerpt": "string?",
  "featured_image": "string?",
  "category_id": "uuid?",
  "tag_ids": "uuid[]?",
  "status": "draft|published",
  "is_featured": "boolean?",
  "published_at": "datetime?",
  "seo_title": "string?",
  "seo_description": "string?",
  "seo_keywords": "string?",
  "direction": "ltr|rtl"
}
```

#### Update Post ✅
```http
PUT /api/posts/{id}
```

Request Body: Same as Create Post

#### Delete Post ✅
```http
DELETE /api/posts/{id}
```

### Categories

#### List Categories 🟡
```http
GET /api/categories
```

GraphQL equivalent (✅ Implemented):
```graphql
query GET_CATEGORIES {
  blog_categories {
    id
    name
    slug
    description
  }
}
```

#### Get Posts by Category 🟡
```http
GET /api/categories/{slug}/posts
```

GraphQL equivalent (✅ Implemented):
```graphql
query GET_POSTS_BY_CATEGORY($categoryId: uuid!, $limit: Int, $offset: Int) {
  blog_posts(where: {category_id: {_eq: $categoryId}}, limit: $limit, offset: $offset) {
    id
    title
    slug
    excerpt
    # other fields
  }
}
```

#### Create Category 🟡
```http
POST /api/categories
```

Request Body:
```json
{
  "name": "string",
  "slug": "string",
  "description": "string?",
  "parent_id": "uuid?",
  "direction": "ltr|rtl"
}
```

### Tags

#### List Tags 🟡
```http
GET /api/tags
```

GraphQL equivalent (✅ Implemented):
```graphql
query GET_TAGS {
  blog_tags {
    id
    name
    slug
  }
}
```

#### Create Tag 🟡
```http
POST /api/tags
```

Request Body:
```json
{
  "name": "string",
  "slug": "string"
}
```

### Media

#### Upload Media ✅
```http
POST /api/blog/media
```
Content-Type: multipart/form-data

Form Fields:
- `file`: Media file
- `alt_text`: Alternative text
- `caption`: Media caption

Response:
```json
{
  "media": {
    "id": "uuid",
    "fileName": "string",
    "fileUrl": "string",
    "fileType": "string",
    "fileSize": number,
    "createdAt": "datetime",
    "updatedAt": "datetime"
  }
}
```

#### List Media ✅
```http
GET /api/blog/media
```

Query Parameters:
- `type`: Filter by media type
- `limit`: Items per page
- `offset`: Pagination offset

### Comments

#### List Comments 📅
```http
GET /api/posts/{post_id}/comments
```

#### Add Comment 📅
```http
POST /api/posts/{post_id}/comments
```

Request Body:
```json
{
  "content": "string",
  "parent_id": "uuid?",
  "author_name": "string?",
  "author_email": "string?"
}
```

## Comment System with Approval Workflow 📅

The comment system includes a moderation workflow to ensure content quality and prevent spam.

### Comment States
- **Pending**: Initial state for all new comments
- **Approved**: Comments that have been reviewed and approved
- **Rejected**: Comments that have been reviewed and rejected
- **Spam**: Comments automatically flagged as spam

### Comment Approval Cycle
1. User submits a comment
2. Comment is saved with "pending" status
3. Notification is sent to moderators
4. Moderator reviews and takes action:
   - Approve → Comment becomes visible
   - Reject → Comment remains hidden
   - Mark as spam → Comment is flagged for analysis

### Moderator Endpoints 📅

#### List Pending Comments
```http
GET /api/moderation/comments?status=pending
```

#### Approve Comment
```http
POST /api/moderation/comments/{id}/approve
```

#### Reject Comment
```http
POST /api/moderation/comments/{id}/reject
```

#### Mark Comment as Spam
```http
POST /api/moderation/comments/{id}/spam
```

### Comment Notification System 📅
- Email notifications for moderators when new comments are submitted
- Webhook notifications for approved comments
- Batch moderation capabilities for high-volume sites

## SEO and Social Media Optimization 🟡

### SEO Features

#### Meta Tags ✅
The API provides endpoints to manage SEO metadata for blog posts:

```http
POST/PUT /api/posts
```

SEO-specific fields:
```json
{
  "seo_title": "string",
  "seo_description": "string",
  "seo_keywords": "string",
  "canonical_url": "string?"
}
```

#### Structured Data (Schema.org) 🔄
Implemented for:
- BlogPosting
- Article
- Person (author)

Example structured data returned by API:
```json
{
  "structured_data": {
    "@context": "https://schema.org",
    "@type": "BlogPosting",
    "headline": "Post Title",
    "author": {
      "@type": "Person",
      "name": "Author Name"
    },
    "datePublished": "2025-01-01T00:00:00Z",
    "dateModified": "2025-01-02T00:00:00Z",
    "image": "https://example.com/image.jpg",
    "publisher": {
      "@type": "Organization",
      "name": "OSINT Dashboard",
      "logo": {
        "@type": "ImageObject",
        "url": "https://example.com/logo.jpg"
      }
    }
  }
}
```

#### SEO Optimization Endpoints 📅
```http
GET /api/seo/analysis/{post_id}
```

Response:
```json
{
  "score": 85,
  "recommendations": [
    "Add more headings to improve structure",
    "Increase keyword density in the first paragraph"
  ],
  "keyword_analysis": {
    "primary_keyword": {
      "term": "string",
      "count": number,
      "density": number
    },
    "secondary_keywords": []
  }
}
```

### Social Media Optimization 🟡

#### Open Graph Tags ✅
The API automatically generates Open Graph metadata for shared content:

```json
{
  "og_metadata": {
    "og:title": "string",
    "og:description": "string",
    "og:image": "string",
    "og:url": "string",
    "og:type": "article"
  }
}
```

#### Twitter Card Tags ✅
Twitter-specific card metadata:

```json
{
  "twitter_metadata": {
    "twitter:card": "summary_large_image",
    "twitter:site": "@osint_dashboard",
    "twitter:title": "string",
    "twitter:description": "string",
    "twitter:image": "string"
  }
}
```

#### Social Media Preview API 📅
```http
GET /api/social/preview/{post_id}?platform=facebook|twitter|linkedin
```

Returns social media preview data for specified platform.

## Search Implementation 🔄

The search functionality is provided through multiple approaches:

### Full-Text Search (GraphQL) 🔄
```graphql
query SEARCH_POSTS($searchTerm: String!, $limit: Int) {
  blog_posts(
    where: {_or: [
      {title: {_ilike: $searchTerm}}, 
      {content: {_ilike: $searchTerm}}
    ]}, 
    limit: $limit
  ) {
    id
    title
    slug
    excerpt
  }
}
```

### API Endpoint Search 🔄
```http
GET /api/search?q=search_term
```

Search capabilities:
- Title and content search
- Category and tag filtering
- Author search
- Date range filtering

## Error Handling ✅

All endpoints return standard HTTP status codes:
- 200: Success
- 201: Created
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 500: Server Error

Error Response Format:
```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": {}
  }
}
```

## Rate Limiting 🟡
- 1000 requests per hour per IP
- 100 requests per minute per IP for media uploads

## Caching 🟡
- GET requests are cached for 5 minutes
- Cache can be bypassed with `Cache-Control: no-cache` header

## Security Considerations ✅
1. All endpoints require authentication except:
   - GET /api/posts (published only)
   - GET /api/posts/{slug} (published only)
   - GET /api/categories
   - GET /api/tags

2. Role-Based Access:
   - ADMIN: Full access
   - EDITOR: Can manage posts, categories, tags
   - AUTHOR: Can manage own posts
   - CONTRIBUTOR: Can create draft posts

3. Content Security:
   - Media files are scanned for malware
   - File size limits: 10MB for images, 50MB for videos
   - Supported image formats: JPG, PNG, WebP, GIF
   - Supported video formats: MP4, WebM
   - Supported document formats: PDF, DOCX

## Example Usage

### JavaScript/TypeScript (with GraphQL) ✅
```typescript
// Using Apollo Client
import { gql, useQuery } from '@apollo/client';

const GET_BLOG_POSTS = gql`
  query GetBlogPosts($limit: Int, $offset: Int) {
    blog_posts(limit: $limit, offset: $offset, order_by: {created_at: desc_nulls_last}) {
      id
      title
      slug
      excerpt
    }
    blog_posts_aggregate {
      aggregate {
        count
      }
    }
  }
`;

function BlogList() {
  const { loading, error, data } = useQuery(GET_BLOG_POSTS, {
    variables: { limit: 10, offset: 0 }
  });

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <h1>Blog Posts</h1>
      <ul>
        {data.blog_posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
      <p>Total posts: {data.blog_posts_aggregate.aggregate.count}</p>
    </div>
  );
}
```

### REST API Example ✅
```typescript
const fetchPosts = async () => {
  const response = await fetch('https://your-api.com/api/posts', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });
  return response.json();
};

const createPost = async (postData) => {
  const response = await fetch('https://your-api.com/api/posts', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(postData)
  });
  return response.json();
};
```

### cURL
```bash
# List posts
curl -X GET "https://your-api.com/api/posts" \
  -H "Authorization: Bearer YOUR_TOKEN"

# Create post
curl -X POST "https://your-api.com/api/posts" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"New Post","content":"Content here"}'
```

## Webhooks 📅
The API supports webhooks for the following events:
- Post published
- Comment added
- Media uploaded

Webhook payload format:
```json
{
  "event": "string",
  "timestamp": "datetime",
  "data": {}
}
```

## Migration and Backup 🔄
- Daily automated backups
- Backup retention: 30 days
- Migration endpoints for WordPress import/export

## Support and Contact
For API support and questions:
- Email: api-support@yourdomain.com
- Documentation Updates: Check GitHub repository
- Status Page: status.yourdomain.com 

## Implementation Roadmap

### Q2 2025
- Complete comment system with approval workflow
- Enhanced search functionality with filters
- Webhook integration for content changes
- Social media preview API with validation tools

### Q3 2025
- Analytics integration for content performance
- Social media sharing enhancements
- Email newsletter integration
- Advanced SEO analysis tools

### Q4 2025
- AI-assisted content recommendations
- Enhanced media management with image editing
- Advanced comment moderation with AI spam detection
- Automated social media posting integration

## Best Practices Checklist

### SEO Best Practices
- [ ] **Content Optimization** ✅
  - [x] Each post targets a single, specific keyword/topic
  - [x] Content structured with proper HTML headings
  - [x] Descriptive alt text for all images
  - [x] Internal linking between related posts
  - [ ] Keyword density analysis tools
  
- [ ] **Technical SEO** 🟡
  - [x] Proper schema.org structured data implementation
  - [x] SEO-friendly URL slugs
  - [x] Mobile-responsive content
  - [ ] XML sitemap generation
  - [ ] Canonical URL support
  
- [ ] **Performance** 🟡
  - [x] Image optimization
  - [x] Content caching
  - [ ] Core Web Vitals monitoring

### Social Media Optimization Best Practices
- [ ] **Open Graph Tags** ✅
  - [x] og:title (up to 95 characters, no brand name)
  - [x] og:description (compelling summary)
  - [x] og:image (high-quality featured image)
  - [x] og:url (canonical URL)
  - [x] og:type (set to "article" for blog posts)
  
- [ ] **Twitter Card Tags** ✅
  - [x] twitter:card (using summary_large_image for posts)
  - [x] twitter:site (organization Twitter handle)
  - [x] twitter:title (can include brand name)
  - [x] twitter:description (same as og:description)
  - [x] twitter:image (properly sized image)
  
- [ ] **Social Preview Tools** 📅
  - [ ] Preview generator for all major platforms
  - [ ] Social card validation tools
  - [ ] A/B testing for social media headlines

### Content Management Best Practices
- [ ] **Multilingual Support** 🟡
  - [x] RTL/LTR content direction
  - [x] Language-specific metadata
  - [ ] Hreflang tag support
  - [ ] Translation management workflow
  
- [ ] **Media Management** ✅
  - [x] Proper alt text storage
  - [x] Media organization by type
  - [x] Image sizing for different placements
  - [x] Media security scanning

- [ ] **Publishing Workflow** 🟡
  - [x] Draft/published states
  - [x] Scheduled publishing
  - [ ] Editorial calendar
  - [ ] Content approval process
  - [ ] Content version history

## Document Purpose & Reference Usage

This document serves as the comprehensive API reference for the OSINT Blog headless API system. It's designed to be used by:

- Frontend developers integrating with the blog API
- Backend developers maintaining or extending the API
- Third-party integrators building external applications
- QA engineers testing API functionality
- Technical writers creating derived documentation

The API documentation should be consulted when:
- Implementing client applications that consume the blog API
- Developing new API endpoints or modifying existing ones
- Debugging API integration issues
- Planning content migration from other systems
- Understanding security requirements for API access
- Determining appropriate rate limits and caching strategies
- Implementing SEO and social media optimization features

This reference is particularly valuable for development teams working on the public-facing components of the OSINT Dashboard project, as it provides a complete specification of available endpoints, request/response formats, authentication requirements, and security considerations. It complements the Project Architecture document by providing detailed API-specific information that's essential for proper integration and usage of the headless blog system. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fouadalabady) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
