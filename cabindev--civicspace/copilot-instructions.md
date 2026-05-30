## civicspace

> CivicSpace : พื้นที่พลเมืองร่วมหาทางออกปัญหาแอลกอฮอล์ เป็นศูนย์ข้อมูลและพื้นที่สำหรับเจ้าหน้าที่ในการทำงานร่วมกันหาทางออกปัญหาแอลกอฮอล์ รวบรวมข้อมูล บทความ และโครงการต่างๆ โดยใช้ API จาก CivicSpace API สำหรับแสดงเนื้อหา

# CivicSpace Project - Claude Context

## Project Overview
CivicSpace : พื้นที่พลเมืองร่วมหาทางออกปัญหาแอลกอฮอล์ เป็นศูนย์ข้อมูลและพื้นที่สำหรับเจ้าหน้าที่ในการทำงานร่วมกันหาทางออกปัญหาแอลกอฮอล์ รวบรวมข้อมูล บทความ และโครงการต่างๆ โดยใช้ API จาก CivicSpace API สำหรับแสดงเนื้อหา

## Technical Stack
- **Frontend**: Next.js 14 with React 18, TypeScript
- **UI Libraries**: Tailwind CSS, Lucide React Icons
- **Database**: MySQL with Prisma ORM (auth only)
- **Authentication**: NextAuth.js with Prisma adapter
- **External Data**: CivicSpace API integration
- **Styling**: Custom CSS with yellow color scheme
- **Design**: Clean, minimal design focused on small fonts and readability

## Project Structure
```
app/
├── api/                    # API routes (proxy to external API)
│   ├── auth/              # Authentication endpoints
│   ├── post/              # Posts proxy API with pagination
│   ├── categories/        # Categories proxy API
│   ├── videos/            # Videos proxy API with pagination
│   └── surveys/           # Surveys proxy API with pagination
├── dashboard/             # Admin dashboard for staff
│   ├── page.tsx           # Dashboard with API statistics
│   ├── posts/             # Posts management
│   ├── categories/        # Categories management
│   └── surveys/           # Surveys management
├── post/                  # All posts page with pagination
│   └── page.tsx           # Display all posts (24 per page)
├── videos/                # All videos page with pagination
│   └── page.tsx           # Display all videos (24 per page)
├── components/            # Reusable components
│   ├── Navbar.tsx         # Navigation bar
│   ├── Footer.tsx         # Footer component
│   ├── Loading.tsx        # Loading states
│   └── SurveyCard.tsx     # Survey card component
├── auth/                  # Authentication pages
├── page.tsx               # Homepage with latest content
├── post/[slug]/           # Individual post detail pages
├── layout.tsx             # Root layout
└── globals.css            # Global styles with yellow theme

lib/
└── api.ts                 # API library with interfaces

prisma/
├── schema.prisma          # Minimal schema (User + Role only)
└── migrations/            # Database migrations
```

## Key Features
1. **หน้าแรก** - แสดงบทความล่าสุด 12 รายการ, วิดีโอล่าสุด 8 รายการ, และแบบสำรวจล่าสุด 3 รายการ
2. **หน้าบทความทั้งหมด** (/post) - แสดงบทความทั้งหมดแบบ Masonry grid พร้อม pagination (24 รายการต่อหน้า)
3. **หน้าวิดีโอทั้งหมด** (/videos) - แสดงวิดีโอทั้งหมดแบบ grid พร้อม pagination (24 รายการต่อหน้า)
4. **แดชบอร์ด** - สถิติและข้อมูลภาพรวมสำหรับเจ้าหน้าที่ (ไม่มี mock data)
5. **ระบบผู้ใช้** - การจัดการผู้ใช้และสิทธิ์ (NextAuth.js)
6. **API Integration** - เชื่อมต่อกับ CivicSpace API ผ่าน proxy routes (ไม่มี mock data)
7. **Surveys Management** - จัดการและดาวน์โหลดแบบสำรวจ
8. **Clean Design** - ดีไซน์เรียบง่าย เน้นตัวหนังสือขนาดเล็ก สีเหลือง
9. **เฉพาะเจ้าหน้าที่** - ระบบสำหรับเจ้าหน้าที่เท่านั้น

## Database Configuration
- **Development**: `mysql://root:root@localhost:3306/civicspace`
- **Authentication**: NextAuth with custom secret

## Development Commands
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

## External API Integration

### CivicSpace API
Base URL: `https://civicspace-gqdcg0dxgjbqe8as.southeastasia-01.azurewebsites.net/api/v1`

**Posts Endpoints:**
- `GET /posts/` - All posts with pagination
- `GET /posts/latest/?limit=N` - Latest posts
- `GET /posts/popular/?limit=N` - Popular posts by view count
- `GET /posts/{slug}/` - Single post details
- `GET /categories/{slug}/posts/` - Posts by category

**Categories Endpoints:**
- `GET /categories/` - All categories with post counts
- `GET /categories/{slug}/` - Category details

**Videos Endpoints:**
- `GET /videos/?page=X&page_size=N` - All videos with pagination (returns {count, next, previous, results})
- `GET /videos/latest/?limit=N` - Latest videos (returns array)

**Surveys Endpoints (NEW):**
- `GET /surveys/` - All surveys with pagination
- `GET /surveys/latest/?limit=N` - Latest surveys
- `GET /surveys/popular/?limit=N` - Popular surveys by view count
- `GET /surveys/{slug}/` - Single survey details (increments view_count)
- `GET /categories/{slug}/surveys/` - Surveys by category

**Survey Response Structure:**
```json
{
  "id": 1,
  "title": "แบบสำรวจ...",
  "slug": "survey-slug",
  "description": "คำอธิบาย",
  "author": "email@example.com",
  "category": {
    "id": 10,
    "name": "หมวดหมู่",
    "slug": "category-slug"
  },
  "survey_file_url": "https://.../file.docx",
  "is_published": true,
  "survey_date": "2025-10-08",
  "response_count": 0,
  "view_count": 0,
  "created_at": "2025-10-08T11:44:48+07:00",
  "updated_at": "2025-10-08T11:44:48+07:00",
  "published_at": "2025-10-08T11:44:48+07:00"
}
```

## Color Scheme
Yellow-based theme with careful attention to readability:
```css
--primary: #f59e0b;        /* Main yellow */
--secondary: #92400e;      /* Dark brown */
--accent: #fbbf24;         /* Light yellow */
--background: #fffbeb;     /* Cream background */
--foreground: #78350f;     /* Dark text */
```

## Design Principles
1. **Small font sizes** - 14px base, clean typography
2. **Minimal interface** - Focus on content
3. **Staff-focused** - Built for internal use by officials
4. **Data-driven** - Real statistics from external API
5. **Responsive** - Mobile-friendly design

## Context for AI Assistant
When working with this codebase:
1. **Language**: Content is primarily in Thai, maintain Thai language for user-facing text
2. **Database**: Use Prisma only for authentication (User model)
3. **Authentication**: NextAuth.js is configured for user management
4. **External Data**: Fetch all content from CivicSpace API - **NO MOCK DATA**
5. **API Proxy**: Use internal API routes (`/api/surveys`, `/api/posts`, `/api/videos`) instead of calling external API directly
6. **Error Handling**: All API routes return 500 errors when external API fails - **NO FALLBACK MOCK DATA**
7. **UI Consistency**: Follow clean, minimal design patterns with Tailwind CSS
8. **Icons**: Use Lucide React icons only
9. **Color Scheme**: Stick to yellow-based theme for consistency (#f59e0b)
10. **Target Audience**: Government officials and staff members only
11. **Surveys**: Files are downloadable via `window.open()` to external blob storage URLs
12. **Pagination**: Posts and Videos pages show 24 items per page with proper pagination UI

## Recent Updates (December 2025)
### Latest Changes
- ✅ **Removed ALL mock data** from API routes and dashboard pages
- ✅ Created dedicated **/post** page with pagination (24 items/page, Masonry layout)
- ✅ Created dedicated **/videos** page with pagination (24 items/page, Grid layout)
- ✅ Fixed videos API to support both homepage (latest) and pagination modes
- ✅ Homepage now has "ดูทั้งหมด" buttons that redirect to dedicated pages
- ✅ Proper error handling (500 status) when external API fails

### Previous Updates (October 2025)
- ✅ Added Surveys Management feature
- ✅ Created `/api/surveys` proxy route with pagination support
- ✅ Built `SurveyCard` component (compact variant)
- ✅ Dashboard surveys page with statistics
- ✅ Homepage displays latest 3 surveys
- ✅ Download functionality for survey files (.docx)
- ✅ Updated terminology: "หมวดหมู่" → "ประเด็น", "บทความ" → "โพสต์"
- ✅ Removed survey response count from dashboard (not tracked in system)

## Important Notes
1. **NO MOCK DATA**: The entire application uses only real data from CivicSpace API
2. **Error Handling**: When API fails, return proper HTTP 500 errors instead of mock data
3. **Pagination**:
   - Homepage: Shows limited recent items (12 posts, 8 videos, 3 surveys)
   - Dedicated pages: Show 24 items per page with numbered pagination
4. **API Response Formats**:
   - Posts: `{count: number, results: Post[]}`
   - Videos: `{count: number, next: string, previous: string, results: Video[]}`
   - Surveys: `Survey[]` (array directly)
5. **Navigation**: Use Next.js Link component for client-side navigation between pages

---
> Source: [cabindev/civicspace](https://github.com/cabindev/civicspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
