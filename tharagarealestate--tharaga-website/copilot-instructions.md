## tharaga-website

> Row Level Security (RLS) is PostgreSQL's built-in security feature that allows you to control which rows users can access in your database tables. In Supabase, RLS is essential for protecting your data.

# Supabase Row Level Security (RLS) Policies

## Overview
Row Level Security (RLS) is PostgreSQL's built-in security feature that allows you to control which rows users can access in your database tables. In Supabase, RLS is essential for protecting your data.

## RLS Fundamentals

### Golden Rules
1. **Enable RLS on all tables** - Especially tables containing user data
2. **Test policies thoroughly** - Ensure users can only access appropriate data
3. **Use authenticated/anon roles** - Leverage Supabase's built-in roles
4. **Keep policies simple** - Complex policies are harder to debug and maintain
5. **Consider performance** - Policies are applied to every query

### Enable RLS
```sql
-- Enable RLS on a table
ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;

-- Disable RLS (use with caution!)
-- ALTER TABLE table_name DISABLE ROW LEVEL SECURITY;

-- Force RLS even for table owner
ALTER TABLE table_name FORCE ROW LEVEL SECURITY;
```

## Policy Structure

### Basic Syntax
```sql
CREATE POLICY "policy_name"
ON table_name
FOR operation  -- SELECT, INSERT, UPDATE, DELETE, ALL
TO role        -- authenticated, anon, service_role
USING (condition)      -- For SELECT and DELETE
WITH CHECK (condition); -- For INSERT and UPDATE
```

### Policy Components
- **Policy Name**: Descriptive name in quotes
- **Operation**: SELECT, INSERT, UPDATE, DELETE, or ALL
- **Role**: authenticated, anon, service_role
- **USING**: Row-level security check for reading/deleting
- **WITH CHECK**: Row-level security check for creating/updating

## Common Policy Patterns

### User-Owned Resources
Users can only access their own rows:

```sql
-- Read own rows
CREATE POLICY "Users can view own posts"
ON posts
FOR SELECT
TO authenticated
USING (auth.uid() = user_id);

-- Insert own rows
CREATE POLICY "Users can create own posts"
ON posts
FOR INSERT
TO authenticated
WITH CHECK (auth.uid() = user_id);

-- Update own rows
CREATE POLICY "Users can update own posts"
ON posts
FOR UPDATE
TO authenticated
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);

-- Delete own rows
CREATE POLICY "Users can delete own posts"
ON posts
FOR DELETE
TO authenticated
USING (auth.uid() = user_id);
```

### Public Read, Authenticated Write
Anyone can read, only authenticated users can write:

```sql
-- Public read
CREATE POLICY "Anyone can view published posts"
ON posts
FOR SELECT
TO anon, authenticated
USING (status = 'published');

-- Authenticated write
CREATE POLICY "Authenticated users can create posts"
ON posts
FOR INSERT
TO authenticated
WITH CHECK (true);
```

### Role-Based Access
Different access based on user role:

```sql
-- Add role column to profiles
-- user_role: 'admin', 'moderator', 'user'

-- Admins can view all
CREATE POLICY "Admins can view all posts"
ON posts
FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE profiles.user_id = auth.uid()
    AND profiles.user_role = 'admin'
  )
);

-- Moderators can update
CREATE POLICY "Moderators can update posts"
ON posts
FOR UPDATE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE profiles.user_id = auth.uid()
    AND profiles.user_role IN ('admin', 'moderator')
  )
);
```

### Team/Organization Based
Access based on team membership:

```sql
-- Users can view posts in their organization
CREATE POLICY "Users can view organization posts"
ON posts
FOR SELECT
TO authenticated
USING (
  organization_id IN (
    SELECT organization_id
    FROM organization_members
    WHERE user_id = auth.uid()
  )
);

-- Team members can update
CREATE POLICY "Team members can update posts"
ON posts
FOR UPDATE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM team_members
    WHERE team_members.team_id = posts.team_id
    AND team_members.user_id = auth.uid()
  )
)
WITH CHECK (
  EXISTS (
    SELECT 1 FROM team_members
    WHERE team_members.team_id = posts.team_id
    AND team_members.user_id = auth.uid()
  )
);
```

### Relationship-Based Access
Access based on relationships:

```sql
-- Users can view posts from people they follow
CREATE POLICY "Users can view posts from followed users"
ON posts
FOR SELECT
TO authenticated
USING (
  user_id IN (
    SELECT followed_user_id
    FROM follows
    WHERE follower_user_id = auth.uid()
  )
  OR user_id = auth.uid() -- Also own posts
);
```

### Time-Based Access
Access based on timestamps:

```sql
-- Users can only edit posts within 1 hour of creation
CREATE POLICY "Users can edit posts within 1 hour"
ON posts
FOR UPDATE
TO authenticated
USING (
  auth.uid() = user_id
  AND created_at > NOW() - INTERVAL '1 hour'
);

-- Scheduled content access
CREATE POLICY "View published and scheduled posts"
ON posts
FOR SELECT
TO authenticated
USING (
  (status = 'published' AND published_at <= NOW())
  OR (user_id = auth.uid())
);
```

### Hierarchical Access
Parent-child relationship access:

```sql
-- Users can view comments on posts they can view
CREATE POLICY "Users can view comments on accessible posts"
ON comments
FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM posts
    WHERE posts.id = comments.post_id
    AND (
      posts.status = 'published'
      OR posts.user_id = auth.uid()
    )
  )
);
```

## Advanced Patterns

### Using JWT Claims
Access custom claims from JWT token:

```sql
-- Access JWT claim
CREATE POLICY "Users with verified email can post"
ON posts
FOR INSERT
TO authenticated
WITH CHECK (
  (auth.jwt() -> 'email_verified')::boolean = true
);

-- Access user metadata
CREATE POLICY "Premium users can upload videos"
ON videos
FOR INSERT
TO authenticated
WITH CHECK (
  (auth.jwt() -> 'user_metadata' ->> 'subscription_tier') = 'premium'
);
```

### Combining Multiple Conditions
Complex access logic:

```sql
CREATE POLICY "Complex access policy"
ON posts
FOR SELECT
TO authenticated
USING (
  -- Own posts
  auth.uid() = user_id
  -- OR published posts
  OR status = 'published'
  -- OR shared with user
  OR EXISTS (
    SELECT 1 FROM post_shares
    WHERE post_shares.post_id = posts.id
    AND post_shares.shared_with_user_id = auth.uid()
  )
  -- OR user is team member
  OR EXISTS (
    SELECT 1 FROM team_members
    WHERE team_members.team_id = posts.team_id
    AND team_members.user_id = auth.uid()
  )
);
```

### Security Definer Functions
Bypass RLS for specific operations:

```sql
-- Create function that bypasses RLS
CREATE OR REPLACE FUNCTION get_post_stats(post_id_param UUID)
RETURNS JSON
LANGUAGE plpgsql
SECURITY DEFINER -- Function runs with definer's privileges
AS $$
BEGIN
  RETURN (
    SELECT json_build_object(
      'views', COUNT(DISTINCT user_id),
      'likes', COUNT(*) FILTER (WHERE action = 'like')
    )
    FROM post_interactions
    WHERE post_id = post_id_param
  );
END;
$$;

-- Grant execute permission
GRANT EXECUTE ON FUNCTION get_post_stats TO authenticated;
```

## Policy Management

### Dropping Policies
```sql
-- Drop a specific policy
DROP POLICY IF EXISTS "policy_name" ON table_name;

-- Drop all policies on a table (careful!)
-- SELECT 'DROP POLICY IF EXISTS "' || policyname || '" ON ' || tablename || ';'
-- FROM pg_policies
-- WHERE tablename = 'table_name';
```

### Modifying Policies
You cannot modify policies directly - drop and recreate:

```sql
-- Drop old policy
DROP POLICY IF EXISTS "old_policy_name" ON table_name;

-- Create new policy
CREATE POLICY "new_policy_name"
ON table_name
FOR SELECT
TO authenticated
USING (new_condition);
```

### Viewing Existing Policies
```sql
-- View all policies on a table
SELECT *
FROM pg_policies
WHERE tablename = 'table_name';

-- View policy details
SELECT
  schemaname,
  tablename,
  policyname,
  permissive,
  roles,
  cmd,
  qual,
  with_check
FROM pg_policies
WHERE tablename = 'table_name';
```

## Performance Considerations

### Index Supporting Columns
```sql
-- Add indexes for columns used in policies
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_status ON posts(status);
CREATE INDEX idx_team_members_user_id ON team_members(user_id);
```

### Avoid Expensive Subqueries
```sql
-- BAD: Multiple subquery checks
CREATE POLICY "slow_policy"
ON posts
FOR SELECT
USING (
  user_id = auth.uid()
  OR user_id IN (SELECT ... complex subquery ...)
  OR EXISTS (SELECT ... another complex subquery ...)
);

-- BETTER: Use joins or simpler checks when possible
-- Consider denormalizing data if necessary
```

### Use Materialized Views
For complex access patterns, consider materialized views:

```sql
CREATE MATERIALIZED VIEW user_accessible_posts AS
SELECT DISTINCT
  p.id AS post_id,
  um.user_id
FROM posts p
CROSS JOIN organization_members um
WHERE p.organization_id = um.organization_id;

-- Refresh periodically
REFRESH MATERIALIZED VIEW user_accessible_posts;

-- Use in policy
CREATE POLICY "Users can view accessible posts"
ON posts
FOR SELECT
TO authenticated
USING (
  id IN (
    SELECT post_id
    FROM user_accessible_posts
    WHERE user_id = auth.uid()
  )
);
```

## Testing RLS Policies

### Test as Different Users
```sql
-- Set role to test
SET ROLE authenticated;
SET request.jwt.claims = '{"sub": "user-id-here"}';

-- Run test query
SELECT * FROM posts;

-- Reset
RESET ROLE;
```

### Verify Policy Coverage
```sql
-- Check which tables have RLS enabled
SELECT
  schemaname,
  tablename,
  rowsecurity
FROM pg_tables
WHERE schemaname = 'public';

-- Check which tables have policies
SELECT DISTINCT tablename
FROM pg_policies
WHERE schemaname = 'public';
```

## Common Patterns for Supabase Tables

### Profiles Table
```sql
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Users can view all profiles
CREATE POLICY "Profiles are viewable by everyone"
ON profiles FOR SELECT
TO authenticated
USING (true);

-- Users can update own profile
CREATE POLICY "Users can update own profile"
ON profiles FOR UPDATE
TO authenticated
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);
```

### Private User Data
```sql
ALTER TABLE user_settings ENABLE ROW LEVEL SECURITY;

-- Only users can access their own settings
CREATE POLICY "Users can access own settings"
ON user_settings
FOR ALL
TO authenticated
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);
```

## Security Best Practices

### Checklist
- [ ] RLS is enabled on all public-facing tables
- [ ] Policies are tested with different user roles
- [ ] No data leaks through related tables
- [ ] Policies are as simple as possible
- [ ] Indexes support policy conditions
- [ ] Service role usage is minimized
- [ ] Anonymous access is restricted appropriately
- [ ] Policy names are descriptive

### Avoid
- Disabling RLS without good reason
- Using service_role for client operations
- Overly complex policy logic
- Missing policies on related tables
- Forgetting to test edge cases
- Exposing sensitive data through functions
- Not considering performance impact

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tharagarealestate) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
