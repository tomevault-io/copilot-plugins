## spacr-full

> Backend development rules for AdonisJS 6


# Backend Development Rules

## Tech Stack

- **AdonisJS 6** (TypeScript, ESM)
- **Lucid ORM** with PostgreSQL
- **VineJS** for validation
- **Redis** for caching and job queues
- **Puppeteer** for web scraping

## Project Structure

```
apps/backend/
├── app/
│   ├── controllers/    # HTTP controllers
│   ├── services/       # Business logic services
│   ├── models/         # Lucid ORM models
│   ├── validators/     # VineJS validators
│   ├── dto/            # Data transfer objects
│   ├── middleware/     # HTTP middleware
│   ├── lib/            # Utility libraries
│   └── exceptions/     # Custom exceptions
├── config/             # Configuration files
├── database/
│   ├── migrations/     # Database migrations
│   └── seeders/        # Database seeders
├── start/
│   ├── routes.ts       # Route definitions
│   ├── kernel.ts       # Middleware registration
│   └── env.ts          # Environment validation
└── bin/
    └── server.ts       # Server entry point
```

## Path Aliases

Use `#` prefix for internal imports:
```typescript
import ApodService from '#services/apod.service'
import Rover from '#models/rover'
import { getValidator } from '#validators/get_validators'
```

## Controller Pattern

Controllers should be thin and delegate to services:

```typescript
import type { HttpContext } from '@adonisjs/core/http'
import RoverImagesService from '#services/rover-images.service'

export default class RoverImagesController {
  async getImages({ request, response }: HttpContext) {
    const params = request.qs()
    const service = new RoverImagesService()
    const images = await service.getImages(params)
    return response.json(images)
  }
}
```

### Route Registration

Use lazy imports for controllers:
```typescript
const RoverController = () => import('#controllers/rover.controller')

router.group(() => {
  router.get('/rovers', [RoverController, 'getRovers'])
}).prefix('/api/v1')
```

## Service Pattern

Services contain business logic:

```typescript
import env from '#start/env'

export default class ApodService {
  async getApod() {
    const response = await fetch(
      `https://api.nasa.gov/planetary/apod?api_key=${env.get('NASA_API_KEY')}`
    )
    return response.json()
  }
}
```

## Model Pattern

Lucid models with decorators:

```typescript
import { DateTime } from 'luxon'
import { BaseModel, column, belongsTo } from '@adonisjs/lucid/orm'
import type { BelongsTo } from '@adonisjs/lucid/types/relations'
import Rover from './rover.js'

export default class RoverImage extends BaseModel {
  @column({ isPrimary: true })
  declare id: number

  @column()
  declare roverId: number

  @column()
  declare imgSrc: string

  @column.dateTime({ autoCreate: true })
  declare createdAt: DateTime

  @belongsTo(() => Rover)
  declare rover: BelongsTo<typeof Rover>
}
```

## Validation with VineJS

```typescript
import vine from '@vinejs/vine'

export const getRoverImagesValidator = vine.compile(
  vine.object({
    page: vine.number().optional(),
    limit: vine.number().max(100).optional(),
    rover: vine.string().optional(),
    camera: vine.string().optional(),
  })
)
```

## Database Migrations

```typescript
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'rovers'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id')
      table.string('name').notNullable()
      table.timestamp('created_at')
      table.timestamp('updated_at')
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

## Environment Variables

Access via `env.get()`:
```typescript
import env from '#start/env'

const apiKey = env.get('NASA_API_KEY')
const dbUrl = env.get('DATABASE_URL')
```

## Error Handling

Use proper HTTP status codes:
```typescript
return response.status(404).json({ error: 'Resource not found' })
return response.status(400).json({ error: 'Invalid parameters' })
return response.status(500).json({ error: 'Internal server error' })
```

## Web Scraping

Use Puppeteer for scraping external sites:
```typescript
import puppeteer from 'puppeteer'

async function scrapeImages(url: string) {
  const browser = await puppeteer.launch({ headless: true })
  const page = await browser.newPage()
  await page.goto(url)
  // ... scraping logic
  await browser.close()
}
```

## Caching with Redis

Use ioredis for caching:
```typescript
import Redis from 'ioredis'

const redis = new Redis(env.get('REDIS_URL'))

// Cache pattern
const cached = await redis.get(cacheKey)
if (cached) return JSON.parse(cached)

const data = await fetchData()
await redis.set(cacheKey, JSON.stringify(data), 'EX', 3600)
return data
```

## Testing

Use Japa for testing:
```typescript
import { test } from '@japa/runner'

test.group('RoverImagesController', () => {
  test('returns paginated images', async ({ client }) => {
    const response = await client.get('/api/v1/rover-image')
    response.assertStatus(200)
    response.assertBodyContains({ data: [] })
  })
})
```

## Common Commands

```bash
node ace serve --hmr      # Development server
node ace build            # Production build
node ace migration:run    # Run migrations
node ace migration:fresh  # Fresh migrations
node ace db:seed          # Run seeders
node ace make:controller  # Generate controller
node ace make:model       # Generate model
node ace make:migration   # Generate migration
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/antoine-gmnz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/antoine-gmnz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
