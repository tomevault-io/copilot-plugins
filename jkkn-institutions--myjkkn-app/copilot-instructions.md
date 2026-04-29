## myjkkn-app

> description: Supabase SQL & Schema Rules:

---
description: Supabase SQL & Schema Rules:
globs: lib/supabase/schema/**/*.sql, **/supabase/**/*.sql, **/migrations/**/*.sql
alwaysApply: false
---
Scope: lib/supabase/schema/**/*.sql, **/supabase/**/*.sql, **/migrations/**/*.sql

# Supabase SQL & Schema Rules

## Database Schema Organization
- Organize table definitions in separate SQL files
- Group related tables in the same file when appropriate
- Include comments for complex fields or constraints
- Follow consistent naming conventions for tables and fields
- Use proper data types and constraints

## Table Naming Conventions
- Use plural, snake_case table names (e.g., `users`, `product_categories`)
- Use singular, snake_case column names (e.g., `first_name`, `created_at`)
- Use descriptive names that reflect the entity and its attributes
- Prefix junction/linking tables with both entity names (e.g., `users_products`)
- Use consistent suffixes for similar columns across tables

## Table Creation Pattern
```sql
-- lib/supabase/schema/entities.sql
-- Entities table for storing core entity data
CREATE TABLE IF NOT EXISTS public.entities (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL CHECK (char_length(name) >= 2),
  code TEXT NOT NULL CHECK (char_length(code) >= 2 AND char_length(code) <= 50),
  description TEXT,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
  updated_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
  
  -- Constraints
  CONSTRAINT entities_code_unique UNIQUE (code)
);

-- Add comment for table documentation
COMMENT ON TABLE public.entities IS 'Stores entity information with validation constraints';

-- Add RLS policies
ALTER TABLE public.entities ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "Allow public read access" 
  ON public.entities
  FOR SELECT USING (is_active = TRUE);

CREATE POLICY "Allow authenticated users to create entities" 
  ON public.entities
  FOR INSERT
  TO authenticated
  WITH CHECK (TRUE);

CREATE POLICY "Allow users to update their own entities" 
  ON public.entities
  FOR UPDATE
  TO authenticated
  USING (auth.uid() = created_by)
  WITH CHECK (auth.uid() = created_by);

-- Add created_by field for user ownership
ALTER TABLE public.entities ADD COLUMN created_by UUID REFERENCES auth.users(id);

-- Create index for common query patterns
CREATE INDEX entities_name_idx ON public.entities (name);
CREATE INDEX entities_created_by_idx ON public.entities (created_by);

-- Create updated_at trigger
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_entities_updated_at
BEFORE UPDATE ON public.entities
FOR EACH ROW
EXECUTE PROCEDURE update_modified_column();
```

## Row Level Security
- Always implement Row Level Security (RLS) for all tables
- Create explicit policies for each operation type (SELECT, INSERT, UPDATE, DELETE)
- Use auth.uid() for user-specific policies
- Document policy logic with clear comments
- Test policies thoroughly to ensure proper access control

## Foreign Key Relationships
- Use UUID for primary and foreign keys consistently
- Implement proper ON DELETE/UPDATE behaviors (CASCADE, RESTRICT, etc.)
- Create indexes for foreign key columns
- Use junction tables for many-to-many relationships
- Name foreign keys consistently (e.g., `entity_id` for a reference to `entities.id`)

## Junction Table Pattern
```sql
-- Many-to-many relationship between entities and categories
CREATE TABLE IF NOT EXISTS public.entities_categories (
  entity_id UUID REFERENCES public.entities(id) ON DELETE CASCADE,
  category_id UUID REFERENCES public.categories(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
  
  -- Composite primary key
  PRIMARY KEY (entity_id, category_id)
);

-- Add RLS policies for junction table
ALTER TABLE public.entities_categories ENABLE ROW LEVEL SECURITY;

-- Create policy for read access
CREATE POLICY "Allow authenticated read access" 
  ON public.entities_categories
  FOR SELECT
  TO authenticated
  USING (TRUE);
```

## Indexes and Performance
- Create indexes for commonly queried columns
- Use composite indexes for multi-column queries
- Avoid over-indexing (consider write performance)
- Add GIN indexes for text search columns
- Document indexing strategy with comments

## Functions and Triggers
- Create reusable database functions for common operations
- Use triggers for maintaining audit fields (updated_at, etc.)
- Document complex functions with detailed comments
- Ensure triggers are efficient and focused
- Test performance impact of triggers on write operations

## Function Pattern
```sql
-- Search function for entities with pagination
CREATE OR REPLACE FUNCTION search_entities(
  search_term TEXT,
  page_number INTEGER DEFAULT 1,
  page_size INTEGER DEFAULT 10
)
RETURNS TABLE (
  id UUID,
  name TEXT,
  code TEXT,
  description TEXT,
  is_active BOOLEAN,
  created_at TIMESTAMPTZ,
  total_count BIGINT
) AS $$
BEGIN
  RETURN QUERY
  WITH matching_entities AS (
    SELECT e.*
    FROM public.entities e
    WHERE 
      e.is_active = TRUE AND
      (
        e.name ILIKE '%' || search_term || '%' OR
        e.code ILIKE '%' || search_term || '%' OR
        e.description ILIKE '%' || search_term || '%'
      )
  )
  SELECT
    e.id,
    e.name,
    e.code,
    e.description,
    e.is_active,
    e.created_at,
    COUNT(*) OVER() AS total_count
  FROM matching_entities e
  ORDER BY e.created_at DESC
  LIMIT page_size
  OFFSET (page_number - 1) * page_size;
END;
$$ LANGUAGE plpgsql;
```

## Data Migration
- Use numbered migration files for sequential application
- Include both up and down migrations
- Make migrations idempotent where possible
- Test migrations thoroughly before applying to production
- Document complex migrations with detailed comments

## Migration Pattern
```sql
-- migration_20240311_01_add_status_field.sql
-- Add status field to entities table

-- Up Migration
ALTER TABLE public.entities ADD COLUMN IF NOT EXISTS status TEXT DEFAULT 'active';
CREATE INDEX IF NOT EXISTS entities_status_idx ON public.entities (status);

-- Update RLS policies to consider status
DROP POLICY IF EXISTS "Allow public read access" ON public.entities;
CREATE POLICY "Allow public read access" 
  ON public.entities
  FOR SELECT USING (is_active = TRUE AND status = 'active');

-- Down Migration
/*
DROP POLICY IF EXISTS "Allow public read access" ON public.entities;
CREATE POLICY "Allow public read access" 
  ON public.entities
  FOR SELECT USING (is_active = TRUE);

DROP INDEX IF EXISTS entities_status_idx;
ALTER TABLE public.entities DROP COLUMN IF EXISTS status;
*/
```

## Types and Enums
- Use PostgreSQL enums for constrained string values
- Create custom composite types when appropriate
- Document enum values with comments
- Consider future extensibility when creating enums

## Enum Pattern
```sql
-- Create enum for entity status
CREATE TYPE entity_status AS ENUM ('draft', 'active', 'archived', 'deleted');

-- Alter table to use the enum
ALTER TABLE public.entities 
  ALTER COLUMN status TYPE entity_status 
  USING status::entity_status;

-- Set default value for the enum column
ALTER TABLE public.entities 
  ALTER COLUMN status SET DEFAULT 'draft';
```

## Security and Permissions
- Grant minimum necessary permissions to database roles
- Use RLS instead of role-based permissions when possible
- Audit sensitive data access
- Encrypt sensitive columns when necessary
- Document security considerations for each table

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JKKN-Institutions) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
