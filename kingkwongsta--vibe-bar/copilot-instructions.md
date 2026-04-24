## vibe-bar

> This guide covers all server-side implementation details for Vibe Bar, including Supabase database setup, authentication, OpenRouter AI integration, and Next.js Server Actions.

# Vibe Bar - Backend Implementation Guide

## 🗄️ Backend Architecture Overview

This guide covers all server-side implementation details for Vibe Bar, including Supabase database setup, authentication, OpenRouter AI integration, and Next.js Server Actions.

# Vibe Bar - Backend Implementation Guide

## 📊 Current Backend Status

**✅ IMPLEMENTED:**
- FastAPI application with OpenRouter AI integration
- Pydantic models for request/response validation
- CORS middleware for frontend communication
- Environment configuration management
- Recipe generation endpoint working
- Health check and test endpoints

**🚧 PHASE 2 (To Be Implemented):**
- Database integration (Supabase recommended)
- User authentication and authorization
- Recipe saving and management endpoints
- User profile management
- Rate limiting and usage tracking

**📋 FUTURE PHASES:**
- Advanced AI features and model switching
- Recipe sharing and social features
- Performance optimizations and caching
- Production deployment configuration

---

### Current Backend Tech Stack ✅
- **API Framework**: FastAPI (Python)
- **AI Integration**: OpenRouter.ai API
- **Data Validation**: Pydantic models
- **Configuration**: Environment variables with python-dotenv

### Planned Backend Tech Stack (Phase 2)
- **Database**: Supabase PostgreSQL
- **Authentication**: Supabase Auth
- **Server Logic**: FastAPI + Supabase integration
- **File Storage**: Supabase Storage (future)
- **Real-time**: Supabase Realtime (future)

---

## 🎯 Current Backend Implementation

### FastAPI Backend Structure ✅ (IMPLEMENTED)
```
backend/
├── app/
│   ├── main.py                # FastAPI application entry point
│   ├── config.py              # Environment configuration
│   ├── models/                # Pydantic data models
│   │   ├── common.py          # Base response models
│   │   └── cocktail.py        # Cocktail-specific models
│   └── services/              # Business logic services
│       └── openrouter.py      # AI integration service
├── requirements.txt           # Python dependencies
├── test_*.py                 # API testing scripts
└── .env                      # Environment variables
```

### Database Setup (Phase 2 - To Be Implemented)

#### Recommended: Supabase Integration
```
supabase/
├── config.ts              # Supabase client configuration
├── migrations/             # Database migrations
│   ├── 20240101000001_initial_schema.sql
│   ├── 20240101000002_profiles_table.sql
│   ├── 20240101000003_recipes_tables.sql
│   └── 20240101000004_rls_policies.sql
├── seed.sql               # Initial data seeding
└── functions/             # Edge Functions (future)
    └── generate-recipe/
        └── index.ts
```

### Database Schema & Migrations

#### Initial Schema (`migrations/20240101000001_initial_schema.sql`)
```sql
-- Enable necessary extensions
create extension if not exists "uuid-ossp";

-- Create enum types
create type difficulty_level as enum ('easy', 'medium', 'hard');
create type strength_level as enum ('light', 'medium', 'strong', 'none');
create type subscription_tier as enum ('free', 'premium', 'enterprise');
```

#### User Profiles (`migrations/20240101000002_profiles_table.sql`)
```sql
-- User profiles table
create table public.profiles (
  id uuid references auth.users on delete cascade primary key,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null,
  updated_at timestamp with time zone default timezone('utc'::text, now()) not null,
  username text unique,
  full_name text,
  avatar_url text,
  my_bar_inventory jsonb default '[]'::jsonb,
  default_preferences jsonb default '{}'::jsonb,
  subscription_tier subscription_tier default 'free'::subscription_tier,
  usage_count integer default 0,
  last_generation_at timestamp with time zone
);

-- Enable RLS
alter table public.profiles enable row level security;

-- Profiles policies
create policy "Public profiles are viewable by everyone" on public.profiles
  for select using (true);

create policy "Users can insert their own profile" on public.profiles
  for insert with check (auth.uid() = id);

create policy "Users can update their own profile" on public.profiles
  for update using (auth.uid() = id);
```

#### Recipe Tables (`migrations/20240101000003_recipes_tables.sql`)
```sql
-- Saved recipes table
create table public.saved_recipes (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references public.profiles(id) on delete cascade not null,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null,
  updated_at timestamp with time zone default timezone('utc'::text, now()) not null,
  name text not null,
  recipe_data jsonb not null,
  is_favorite boolean default false,
  tags text[] default array[]::text[],
  notes text,
  rating integer check (rating >= 1 and rating <= 5)
);

-- Recipe generation history
create table public.generation_history (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references public.profiles(id) on delete cascade not null,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null,
  preferences_input jsonb not null,
  generated_recipe jsonb not null,
  generation_time_ms integer,
  model_used text default 'anthropic/claude-3-sonnet',
  user_rating integer check (user_rating >= 1 and user_rating <= 5),
  feedback text
);

-- Usage tracking for rate limiting
create table public.usage_tracking (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references public.profiles(id) on delete cascade not null,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null,
  action_type text not null, -- 'recipe_generation', 'recipe_save', etc.
  metadata jsonb default '{}'::jsonb,
  ip_address inet,
  user_agent text
);

-- Enable RLS on all tables
alter table public.saved_recipes enable row level security;
alter table public.generation_history enable row level security;
alter table public.usage_tracking enable row level security;

-- Create indexes for performance
create index saved_recipes_user_id_idx on public.saved_recipes(user_id);
create index saved_recipes_created_at_idx on public.saved_recipes(created_at desc);
create index saved_recipes_is_favorite_idx on public.saved_recipes(is_favorite) where is_favorite = true;
create index generation_history_user_id_idx on public.generation_history(user_id);
create index generation_history_created_at_idx on public.generation_history(created_at desc);
create index usage_tracking_user_id_created_at_idx on public.usage_tracking(user_id, created_at desc);
```

#### Row Level Security Policies (`migrations/20240101000004_rls_policies.sql`)
```sql
-- Saved recipes policies
create policy "Users can view their own saved recipes" on public.saved_recipes
  for select using (auth.uid() = user_id);

create policy "Users can insert their own saved recipes" on public.saved_recipes
  for insert with check (auth.uid() = user_id);

create policy "Users can update their own saved recipes" on public.saved_recipes
  for update using (auth.uid() = user_id);

create policy "Users can delete their own saved recipes" on public.saved_recipes
  for delete using (auth.uid() = user_id);

-- Generation history policies
create policy "Users can view their own generation history" on public.generation_history
  for select using (auth.uid() = user_id);

create policy "Users can insert their own generation history" on public.generation_history
  for insert with check (auth.uid() = user_id);

create policy "Users can update their own generation history ratings" on public.generation_history
  for update using (auth.uid() = user_id);

-- Usage tracking policies
create policy "Users can view their own usage tracking" on public.usage_tracking
  for select using (auth.uid() = user_id);

create policy "System can insert usage tracking" on public.usage_tracking
  for insert with check (true);
```

### Database Functions & Triggers

#### Profile Management (`migrations/20240101000005_functions.sql`)
```sql
-- Function to handle user profile creation
create or replace function public.handle_new_user()
returns trigger
language plpgsql
security definer set search_path = public
as $$
begin
  insert into public.profiles (id, username, full_name, avatar_url)
  values (
    new.id,
    new.raw_user_meta_data->>'username',
    new.raw_user_meta_data->>'full_name',
    new.raw_user_meta_data->>'avatar_url'
  );
  return new;
end;
$$;

-- Trigger for new user creation
create trigger on_auth_user_created
  after insert on auth.users
  for each row execute procedure public.handle_new_user();

-- Function to update updated_at timestamps
create or replace function public.handle_updated_at()
returns trigger
language plpgsql
as $$
begin
  new.updated_at = timezone('utc'::text, now());
  return new;
end;
$$;

-- Add updated_at triggers
create trigger handle_updated_at before update on public.profiles
  for each row execute procedure public.handle_updated_at();

create trigger handle_updated_at before update on public.saved_recipes
  for each row execute procedure public.handle_updated_at();
```

---

## 🚀 Current FastAPI Implementation

### FastAPI Main Application (`backend/app/main.py`)
```python
from fastapi import FastAPI, HTTPException, Depends, Body
from fastapi.middleware.cors import CORSMiddleware
from datetime import datetime, UTC
import logging

# Import models and services
from app.models import APIResponse, HealthCheck, UserPreferences, CocktailRecipe
from app.services import get_openrouter_service, get_cocktail_service
from app.config import config

app = FastAPI(
    title="Vibe Bar API - AI Cocktail Recipe Generator",
    description="AI-Powered Cocktail Recipe Generation using OpenRouter LLM",
    version="2.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configure for production
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.post("/api/cocktails/generate", response_model=APIResponse)
async def generate_cocktail_recipe(
    preferences: UserPreferences,
    cocktail_service = Depends(get_cocktail_service)
):
    try:
        recipe = await cocktail_service.generate_cocktail_recipe(preferences)
        return APIResponse(
            message="Cocktail recipe generated successfully",
            data=recipe.model_dump()
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### Configuration Management (`backend/app/config.py`)
```python
import os
from typing import Optional
from dotenv import load_dotenv

load_dotenv()

class Config:
    # Environment
    ENVIRONMENT: str = os.getenv("ENVIRONMENT", "development")
    
    # Frontend Configuration
    FRONTEND_URL: str = os.getenv("FRONTEND_URL", "http://localhost:3000")
    
    # OpenRouter Configuration
    OPENROUTER_API_KEY: Optional[str] = os.getenv("OPENROUTER_API_KEY")
    OPENROUTER_BASE_URL: str = os.getenv("OPENROUTER_BASE_URL", "https://openrouter.ai/api/v1")
    DEFAULT_AI_MODEL: str = os.getenv("DEFAULT_AI_MODEL", "openai/gpt-4o-mini")
    
    # AI Configuration
    AI_TIMEOUT: int = int(os.getenv("AI_TIMEOUT", "30"))
    AI_MAX_RETRIES: int = int(os.getenv("AI_MAX_RETRIES", "3"))
    AI_TEMPERATURE: float = float(os.getenv("AI_TEMPERATURE", "0.7"))
    AI_MAX_TOKENS: int = int(os.getenv("AI_MAX_TOKENS", "1000"))

config = Config()
```

---

## 🔐 Authentication & Authorization (Phase 2 - To Be Implemented)

### Recommended: Supabase Client Configuration

#### Server-side Client (`lib/db/server.ts`)
```typescript
import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { cookies } from 'next/headers'
import type { Database } from '@/types/database'

export function createClient() {
  const cookieStore = cookies()

  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value
        },
        set(name: string, value: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value, ...options })
          } catch (error) {
            // The `set` method was called from a Server Component.
            // This can be ignored if you have middleware refreshing
            // user sessions.
          }
        },
        remove(name: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value: '', ...options })
          } catch (error) {
            // The `delete` method was called from a Server Component.
            // This can be ignored if you have middleware refreshing
            // user sessions.
          }
        },
      },
    }
  )
}
```

#### Client-side Client (`lib/db/client.ts`)
```typescript
import { createBrowserClient } from '@supabase/ssr'
import type { Database } from '@/types/database'

export function createClient() {
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

### Authentication Middleware (`middleware.ts`)
```typescript
import { createMiddlewareClient } from '@supabase/ssr'
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export async function middleware(req: NextRequest) {
  const res = NextResponse.next()
  const supabase = createMiddlewareClient({ req, res })

  // Refresh session if expired - required for Server Components
  const { data: { session } } = await supabase.auth.getSession()

  // Protect dashboard routes
  if (req.nextUrl.pathname.startsWith('/dashboard')) {
    if (!session) {
      const redirectUrl = req.nextUrl.clone()
      redirectUrl.pathname = '/login'
      redirectUrl.searchParams.set('redirectedFrom', req.nextUrl.pathname)
      return NextResponse.redirect(redirectUrl)
    }
  }

  // Protect generation routes for authenticated users only
  if (req.nextUrl.pathname.startsWith('/generate')) {
    if (!session) {
      const redirectUrl = req.nextUrl.clone()
      redirectUrl.pathname = '/login'
      redirectUrl.searchParams.set('redirectedFrom', req.nextUrl.pathname)
      return NextResponse.redirect(redirectUrl)
    }

    // Check rate limiting
    const rateLimitResult = await checkRateLimit(session.user.id)
    if (!rateLimitResult.allowed) {
      const redirectUrl = req.nextUrl.clone()
      redirectUrl.pathname = '/dashboard'
      redirectUrl.searchParams.set('error', 'rate_limit_exceeded')
      return NextResponse.redirect(redirectUrl)
    }
  }

  // Redirect authenticated users away from auth pages
  if ((req.nextUrl.pathname.startsWith('/login') || 
       req.nextUrl.pathname.startsWith('/signup')) && session) {
    return NextResponse.redirect(new URL('/dashboard', req.url))
  }

  return res
}

async function checkRateLimit(userId: string): Promise<{ allowed: boolean; remaining: number }> {
  // Implement rate limiting logic here
  // Check user's subscription tier and usage count
  return { allowed: true, remaining: 10 }
}

export const config = {
  matcher: [
    '/dashboard/:path*',
    '/generate/:path*',
    '/login',
    '/signup',
    '/((?!_next/static|_next/image|favicon.ico).*)',
  ]
}
```

### Authentication Server Actions (`lib/actions/auth-actions.ts`)
```typescript
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'
import { createClient } from '@/lib/db/server'
import { z } from 'zod'

const signUpSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  username: z.string().min(3, 'Username must be at least 3 characters'),
  fullName: z.string().min(1, 'Full name is required'),
})

const signInSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(1, 'Password is required'),
})

export async function signUpAction(formData: FormData) {
  const supabase = createClient()

  const data = {
    email: formData.get('email') as string,
    password: formData.get('password') as string,
    username: formData.get('username') as string,
    fullName: formData.get('fullName') as string,
  }

  const validatedData = signUpSchema.parse(data)

  const { error } = await supabase.auth.signUp({
    email: validatedData.email,
    password: validatedData.password,
    options: {
      data: {
        username: validatedData.username,
        full_name: validatedData.fullName,
      },
    },
  })

  if (error) {
    throw new Error(error.message)
  }

  revalidatePath('/', 'layout')
  redirect('/dashboard?message=Check your email to confirm your account')
}

export async function signInAction(formData: FormData) {
  const supabase = createClient()

  const data = {
    email: formData.get('email') as string,
    password: formData.get('password') as string,
  }

  const validatedData = signInSchema.parse(data)

  const { error } = await supabase.auth.signInWithPassword({
    email: validatedData.email,
    password: validatedData.password,
  })

  if (error) {
    throw new Error(error.message)
  }

  revalidatePath('/', 'layout')
  redirect('/dashboard')
}

export async function signOutAction() {
  const supabase = createClient()

  const { error } = await supabase.auth.signOut()

  if (error) {
    throw new Error(error.message)
  }

  revalidatePath('/', 'layout')
  redirect('/')
}

export async function resetPasswordAction(formData: FormData) {
  const supabase = createClient()
  const email = formData.get('email') as string

  if (!email) {
    throw new Error('Email is required')
  }

  const { error } = await supabase.auth.resetPasswordForEmail(email, {
    redirectTo: `${process.env.NEXT_PUBLIC_SITE_URL}/auth/reset-password`,
  })

  if (error) {
    throw new Error(error.message)
  }

  return { message: 'Password reset email sent' }
}
```

---

## 🤖 OpenRouter AI Integration

### OpenRouter Service (`lib/services/openrouter.ts`)
```typescript
import type { RecipePreferences, Recipe } from '@/types/recipe'

const OPENROUTER_API_KEY = process.env.OPENROUTER_API_KEY!
const OPENROUTER_BASE_URL = 'https://openrouter.ai/api/v1'

interface OpenRouterResponse {
  choices: Array<{
    message: {
      content: string
    }
  }>
  usage: {
    prompt_tokens: number
    completion_tokens: number
    total_tokens: number
  }
}

export async function callOpenRouterAPI(
  preferences: RecipePreferences,
  model: string = 'anthropic/claude-3-sonnet'
): Promise<{ recipe: Recipe; usage: any }> {
  const prompt = buildRecipePrompt(preferences)
  
  try {
    const response = await fetch(`${OPENROUTER_BASE_URL}/chat/completions`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${OPENROUTER_API_KEY}`,
        'Content-Type': 'application/json',
        'HTTP-Referer': process.env.NEXT_PUBLIC_SITE_URL || 'http://localhost:3000',
        'X-Title': 'Vibe Bar - AI Cocktail Generator',
      },
      body: JSON.stringify({
        model,
        messages: [
          {
            role: 'system',
            content: SYSTEM_PROMPT,
          },
          {
            role: 'user',
            content: prompt,
          },
        ],
        temperature: 0.7,
        max_tokens: 1200,
        top_p: 0.9,
      }),
    })

    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}))
      throw new Error(`OpenRouter API error: ${response.status} - ${errorData.error?.message || 'Unknown error'}`)
    }

    const data: OpenRouterResponse = await response.json()
    const content = data.choices[0]?.message?.content

    if (!content) {
      throw new Error('No recipe content generated')
    }

    // Parse and validate the JSON response
    const recipe = parseRecipeResponse(content)
    
    return {
      recipe: validateRecipeResponse(recipe),
      usage: data.usage,
    }
  } catch (error) {
    console.error('OpenRouter API call failed:', error)
    throw new Error(
      error instanceof Error 
        ? `Recipe generation failed: ${error.message}`
        : 'Recipe generation failed: Unknown error'
    )
  }
}

const SYSTEM_PROMPT = `You are a professional mixologist AI assistant that creates unique, creative cocktail recipes. 

Guidelines:
- Create original, innovative cocktail recipes based on user preferences
- Ensure all measurements are precise and realistic
- Provide clear, step-by-step instructions
- Consider flavor balance and complementary ingredients
- Be creative with cocktail names that reflect the ingredients or theme
- Include appropriate garnish suggestions
- Recommend suitable glassware
- Provide accurate difficulty and time estimates

Always respond with valid JSON in the exact format specified by the user.`

function buildRecipePrompt(preferences: RecipePreferences): string {
  const {
    ingredients,
    flavorProfile,
    baseSpirit,
    strength,
    occasion,
    dietaryRestrictions
  } = preferences

  return `Create a unique cocktail recipe based on these preferences:

AVAILABLE INGREDIENTS: ${ingredients.length > 0 ? ingredients.join(', ') : 'No specific ingredients provided'}

FLAVOR PREFERENCES: ${flavorProfile.length > 0 ? flavorProfile.join(', ') : 'No specific flavor preferences'}

PREFERRED BASE SPIRITS: ${baseSpirit.length > 0 ? baseSpirit.join(', ') : 'Any spirit'}

STRENGTH PREFERENCE: ${strength}

${occasion ? `OCCASION: ${occasion}` : ''}

${dietaryRestrictions.length > 0 ? `DIETARY RESTRICTIONS: ${dietaryRestrictions.join(', ')}` : ''}

Create a creative, unique cocktail recipe that incorporates these preferences. If specific ingredients are provided, try to use at least some of them. Be creative and original while ensuring the cocktail is balanced and delicious.

Respond with valid JSON in this EXACT format:
{
  "name": "Creative Cocktail Name",
  "description": "Brief description of the cocktail and its flavor profile",
  "ingredients": [
    {"name": "Ingredient Name", "amount": "2", "unit": "oz"},
    {"name": "Another Ingredient", "amount": "0.5", "unit": "oz"},
    {"name": "Garnish Component", "amount": "1", "unit": "piece"}
  ],
  "instructions": [
    "Step 1: Detailed instruction",
    "Step 2: Another detailed instruction",
    "Step 3: Final instruction and serving"
  ],
  "garnish": "Detailed garnish description",
  "glassware": "Specific glass type (e.g., 'Coupe glass', 'Rocks glass', 'Highball glass')",
  "estimatedTime": 5,
  "difficulty": "easy"
}`
}

function parseRecipeResponse(content: string): any {
  // Try to extract JSON from the response, handling potential markdown formatting
  const jsonMatch = content.match(/\{[\s\S]*\}/)
  if (!jsonMatch) {
    throw new Error('No valid JSON found in response')
  }

  try {
    return JSON.parse(jsonMatch[0])
  } catch (error) {
    throw new Error('Invalid JSON format in recipe response')
  }
}

function validateRecipeResponse(recipe: any): Recipe {
  // Validate required fields
  const requiredFields = ['name', 'ingredients', 'instructions', 'glassware', 'difficulty']
  for (const field of requiredFields) {
    if (!recipe[field]) {
      throw new Error(`Missing required field: ${field}`)
    }
  }

  // Validate ingredients array
  if (!Array.isArray(recipe.ingredients) || recipe.ingredients.length === 0) {
    throw new Error('Ingredients must be a non-empty array')
  }

  // Validate each ingredient
  recipe.ingredients.forEach((ingredient: any, index: number) => {
    if (!ingredient.name || !ingredient.amount || !ingredient.unit) {
      throw new Error(`Invalid ingredient at index ${index}: missing name, amount, or unit`)
    }
  })

  // Validate instructions
  if (!Array.isArray(recipe.instructions) || recipe.instructions.length === 0) {
    throw new Error('Instructions must be a non-empty array')
  }

  // Validate difficulty
  const validDifficulties = ['easy', 'medium', 'hard']
  if (!validDifficulties.includes(recipe.difficulty)) {
    recipe.difficulty = 'medium' // Default fallback
  }

  // Ensure estimatedTime is a number
  if (typeof recipe.estimatedTime !== 'number') {
    recipe.estimatedTime = 5 // Default fallback
  }

  return {
    ...recipe,
    id: crypto.randomUUID(),
    created_at: new Date().toISOString(),
  } as Recipe
}

// Rate limiting and cost tracking
export async function trackAPIUsage(
  userId: string,
  model: string,
  usage: any,
  cost: number
) {
  const supabase = createClient()
  
  await supabase
    .from('usage_tracking')
    .insert({
      user_id: userId,
      action_type: 'recipe_generation',
      metadata: {
        model,
        tokens: usage,
        estimated_cost: cost,
      },
    })
}
```

---

## 🎯 Server Actions Implementation

### Recipe Server Actions (`lib/actions/recipe-actions.ts`)
```typescript
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'
import { z } from 'zod'
import { createClient } from '@/lib/db/server'
import { callOpenRouterAPI, trackAPIUsage } from '@/lib/services/openrouter'
import { recipePreferencesSchema } from '@/lib/validations'
import type { Recipe } from '@/types/recipe'

export async function generateRecipeAction(formData: FormData) {
  const startTime = Date.now()
  
  // Get authenticated user
  const supabase = createClient()
  const { data: { user }, error: authError } = await supabase.auth.getUser()
  
  if (authError || !user) {
    redirect('/login')
  }

  // Check user's rate limits
  const canGenerate = await checkGenerationLimits(user.id)
  if (!canGenerate.allowed) {
    throw new Error(`Generation limit exceeded. ${canGenerate.message}`)
  }

  // Validate input data
  const preferences = {
    ingredients: formData.getAll('ingredients') as string[],
    flavorProfile: formData.getAll('flavorProfile') as string[],
    baseSpirit: formData.getAll('baseSpirit') as string[],
    strength: formData.get('strength') as string,
    occasion: formData.get('occasion') as string || undefined,
    dietaryRestrictions: formData.getAll('dietaryRestrictions') as string[],
  }

  const validatedPreferences = recipePreferencesSchema.parse(preferences)

  try {
    // Generate recipe using OpenRouter API
    const { recipe, usage } = await callOpenRouterAPI(validatedPreferences)
    
    const generationTime = Date.now() - startTime

    // Save generation to history
    const { data: savedGeneration, error: saveError } = await supabase
      .from('generation_history')
      .insert({
        user_id: user.id,
        preferences_input: validatedPreferences,
        generated_recipe: recipe,
        generation_time_ms: generationTime,
        model_used: 'anthropic/claude-3-sonnet',
      })
      .select()
      .single()

    if (saveError) {
      console.error('Failed to save generation:', saveError)
      // Continue anyway - don't fail the whole operation
    }

    // Track API usage for cost monitoring
    await trackAPIUsage(user.id, 'anthropic/claude-3-sonnet', usage, 0.01)

    // Update user's usage count
    await supabase
      .from('profiles')
      .update({ 
        usage_count: supabase.rpc('increment_usage_count'),
        last_generation_at: new Date().toISOString(),
      })
      .eq('id', user.id)

    // Redirect to result page
    const resultId = savedGeneration?.id || 'temp'
    redirect(`/generate/result/${resultId}`)
    
  } catch (error) {
    console.error('Recipe generation failed:', error)
    
    // Log the error for monitoring
    await supabase
      .from('usage_tracking')
      .insert({
        user_id: user.id,
        action_type: 'recipe_generation_failed',
        metadata: {
          error: error instanceof Error ? error.message : 'Unknown error',
          preferences: validatedPreferences,
        },
      })

    throw new Error('Failed to generate recipe. Please try again.')
  }
}

export async function saveRecipeAction(recipe: Recipe) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    return { success: false, error: 'Not authenticated' }
  }

  // Check if recipe is already saved
  const { data: existingRecipe } = await supabase
    .from('saved_recipes')
    .select('id')
    .eq('user_id', user.id)
    .eq('name', recipe.name)
    .single()

  if (existingRecipe) {
    return { success: false, error: 'Recipe already saved' }
  }

  const { data, error } = await supabase
    .from('saved_recipes')
    .insert({
      user_id: user.id,
      recipe_data: recipe,
      name: recipe.name,
      is_favorite: false,
    })
    .select()
    .single()

  if (error) {
    console.error('Failed to save recipe:', error)
    return { success: false, error: 'Failed to save recipe' }
  }

  // Track the save action
  await supabase
    .from('usage_tracking')
    .insert({
      user_id: user.id,
      action_type: 'recipe_save',
      metadata: { recipe_id: data.id },
    })

  revalidatePath('/dashboard/recipes')
  
  return { success: true, data }
}

export async function toggleFavoriteAction(recipeId: string) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    return { success: false, error: 'Not authenticated' }
  }

  // Get current favorite status
  const { data: recipe } = await supabase
    .from('saved_recipes')
    .select('is_favorite')
    .eq('id', recipeId)
    .eq('user_id', user.id)
    .single()

  if (!recipe) {
    return { success: false, error: 'Recipe not found' }
  }

  // Toggle favorite status
  const { error } = await supabase
    .from('saved_recipes')
    .update({ is_favorite: !recipe.is_favorite })
    .eq('id', recipeId)
    .eq('user_id', user.id)

  if (error) {
    console.error('Failed to update favorite status:', error)
    return { success: false, error: 'Failed to update favorite status' }
  }

  revalidatePath('/dashboard/recipes')
  
  return { success: true }
}

export async function deleteRecipeAction(recipeId: string) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    return { success: false, error: 'Not authenticated' }
  }

  const { error } = await supabase
    .from('saved_recipes')
    .delete()
    .eq('id', recipeId)
    .eq('user_id', user.id)

  if (error) {
    console.error('Failed to delete recipe:', error)
    return { success: false, error: 'Failed to delete recipe' }
  }

  revalidatePath('/dashboard/recipes')
  
  return { success: true }
}

async function checkGenerationLimits(userId: string): Promise<{ allowed: boolean; message: string }> {
  const supabase = createClient()
  
  // Get user's subscription tier and current usage
  const { data: profile } = await supabase
    .from('profiles')
    .select('subscription_tier, usage_count, last_generation_at')
    .eq('id', userId)
    .single()

  if (!profile) {
    return { allowed: false, message: 'Profile not found' }
  }

  // Define limits based on subscription tier
  const limits = {
    free: { daily: 5, monthly: 50 },
    premium: { daily: 50, monthly: 1000 },
    enterprise: { daily: Infinity, monthly: Infinity },
  }

  const userLimits = limits[profile.subscription_tier as keyof typeof limits]

  // Check daily limit
  const today = new Date().toISOString().split('T')[0]
  const { count: dailyUsage } = await supabase
    .from('generation_history')
    .select('*', { count: 'exact', head: true })
    .eq('user_id', userId)
    .gte('created_at', `${today}T00:00:00.000Z`)
    .lt('created_at', `${today}T23:59:59.999Z`)

  if ((dailyUsage || 0) >= userLimits.daily) {
    return { 
      allowed: false, 
      message: `Daily generation limit of ${userLimits.daily} recipes reached. Upgrade your plan for more generations.` 
    }
  }

  return { allowed: true, message: 'Generation allowed' }
}
```

### Profile Management Actions (`lib/actions/profile-actions.ts`)
```typescript
'use server'

import { revalidatePath } from 'next/cache'
import { z } from 'zod'
import { createClient } from '@/lib/db/server'

const updateProfileSchema = z.object({
  username: z.string().min(3).max(20).optional(),
  fullName: z.string().min(1).max(100).optional(),
  myBarInventory: z.array(z.string()).optional(),
  defaultPreferences: z.object({
    flavorProfile: z.array(z.string()).optional(),
    baseSpirit: z.array(z.string()).optional(),
    strength: z.enum(['light', 'medium', 'strong', 'none']).optional(),
    dietaryRestrictions: z.array(z.string()).optional(),
  }).optional(),
})

export async function updateProfileAction(formData: FormData) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    throw new Error('Not authenticated')
  }

  const data = {
    username: formData.get('username') as string || undefined,
    fullName: formData.get('fullName') as string || undefined,
    myBarInventory: formData.getAll('myBarInventory') as string[] || undefined,
  }

  // Handle default preferences JSON
  const defaultPreferencesStr = formData.get('defaultPreferences') as string
  if (defaultPreferencesStr) {
    try {
      data.defaultPreferences = JSON.parse(defaultPreferencesStr)
    } catch (error) {
      throw new Error('Invalid default preferences format')
    }
  }

  const validatedData = updateProfileSchema.parse(data)

  const updateData: any = {}
  if (validatedData.username) updateData.username = validatedData.username
  if (validatedData.fullName) updateData.full_name = validatedData.fullName
  if (validatedData.myBarInventory) updateData.my_bar_inventory = validatedData.myBarInventory
  if (validatedData.defaultPreferences) updateData.default_preferences = validatedData.defaultPreferences

  const { error } = await supabase
    .from('profiles')
    .update(updateData)
    .eq('id', user.id)

  if (error) {
    console.error('Failed to update profile:', error)
    throw new Error('Failed to update profile')
  }

  revalidatePath('/dashboard/profile')
  
  return { success: true }
}

export async function getProfileAction() {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    throw new Error('Not authenticated')
  }

  const { data: profile, error } = await supabase
    .from('profiles')
    .select('*')
    .eq('id', user.id)
    .single()

  if (error) {
    console.error('Failed to get profile:', error)
    throw new Error('Failed to get profile')
  }

  return profile
}

export async function updateBarInventoryAction(ingredients: string[]) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    return { success: false, error: 'Not authenticated' }
  }

  const { error } = await supabase
    .from('profiles')
    .update({ my_bar_inventory: ingredients })
    .eq('id', user.id)

  if (error) {
    console.error('Failed to update bar inventory:', error)
    return { success: false, error: 'Failed to update bar inventory' }
  }

  revalidatePath('/dashboard/bar')
  
  return { success: true }
}
```

---

## 🔧 Environment Variables & Configuration

### Required Environment Variables (`.env.local`)
```env
# Supabase Configuration
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# OpenRouter API
OPENROUTER_API_KEY=your-openrouter-api-key

# Application URLs
NEXT_PUBLIC_SITE_URL=http://localhost:3000

# Optional: Analytics and Monitoring
VERCEL_ANALYTICS_ID=your-analytics-id
SENTRY_DSN=your-sentry-dsn
```

### Supabase Configuration (`supabase/config.ts`)
```typescript
export const supabaseConfig = {
  url: process.env.NEXT_PUBLIC_SUPABASE_URL!,
  anonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
  serviceRoleKey: process.env.SUPABASE_SERVICE_ROLE_KEY!,
}

// Rate limiting configuration
export const rateLimits = {
  free: {
    dailyGenerations: 5,
    monthlyGenerations: 50,
  },
  premium: {
    dailyGenerations: 50,
    monthlyGenerations: 1000,
  },
  enterprise: {
    dailyGenerations: Infinity,
    monthlyGenerations: Infinity,
  },
}

// OpenRouter model configurations
export const openRouterModels = {
  default: 'anthropic/claude-3-sonnet',
  premium: 'anthropic/claude-3-opus',
  fast: 'anthropic/claude-3-haiku',
}
```

---

## 🚀 Database Setup Commands

### Local Development Setup
```bash
# Initialize Supabase project
npx supabase init

# Start local Supabase stack
npx supabase start

# Apply migrations
npx supabase db reset

# Generate TypeScript types
npx supabase gen types typescript --local > types/database.ts

# Stop local Supabase
npx supabase stop
```

### Production Deployment
```bash
# Link to production project
npx supabase link --project-ref your-project-ref

# Push migrations to production
npx supabase db push

# Generate production types
npx supabase gen types typescript --project-id your-project-id > types/database.ts
```

### Database Seeding (`supabase/seed.sql`)
```sql
-- Insert sample data for development
INSERT INTO public.profiles (id, username, full_name) 
SELECT 
  uuid_generate_v4(), 
  'demo_user', 
  'Demo User'
WHERE NOT EXISTS (SELECT 1 FROM public.profiles WHERE username = 'demo_user');

-- Insert common ingredients for autocomplete
CREATE TABLE IF NOT EXISTS public.common_ingredients (
  id uuid default uuid_generate_v4() primary key,
  name text not null unique,
  category text,
  is_spirit boolean default false
);

INSERT INTO public.common_ingredients (name, category, is_spirit) VALUES
-- Spirits
('Vodka', 'Spirits', true),
('Gin', 'Spirits', true),
('Rum', 'Spirits', true),
('Tequila', 'Spirits', true),
('Whiskey', 'Spirits', true),
('Bourbon', 'Spirits', true),
-- Mixers
('Lime juice', 'Citrus', false),
('Lemon juice', 'Citrus', false),
('Simple syrup', 'Sweeteners', false),
('Club soda', 'Mixers', false),
('Tonic water', 'Mixers', false),
-- Liqueurs
('Triple sec', 'Liqueurs', false),
('Cointreau', 'Liqueurs', false),
('Amaretto', 'Liqueurs', false)
ON CONFLICT (name) DO NOTHING;
```

---

## 📊 Monitoring & Analytics

### Error Tracking
```typescript
// lib/monitoring.ts
import { createClient } from '@/lib/db/server'

export async function logError(
  error: Error,
  context: {
    userId?: string
    action: string
    metadata?: Record<string, any>
  }
) {
  const supabase = createClient()
  
  await supabase
    .from('usage_tracking')
    .insert({
      user_id: context.userId,
      action_type: `error_${context.action}`,
      metadata: {
        error_message: error.message,
        error_stack: error.stack,
        ...context.metadata,
      },
    })
}

export async function trackMetric(
  metric: string,
  value: number,
  userId?: string,
  metadata?: Record<string, any>
) {
  const supabase = createClient()
  
  await supabase
    .from('usage_tracking')
    .insert({
      user_id: userId,
      action_type: `metric_${metric}`,
      metadata: {
        value,
        ...metadata,
      },
    })
}
```

This backend implementation guide provides the complete server-side foundation for Vibe Bar, including database setup, authentication, AI integration, and server actions using modern Next.js 15 patterns. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kingkwongsta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
