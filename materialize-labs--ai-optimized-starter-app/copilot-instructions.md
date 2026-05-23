## storage

> Follow these rules when working on file storage.

# Supabase Storage Architecture

This document outlines our approach to file storage using Supabase Storage. These guidelines ensure consistent, secure, and efficient file handling across the application.

## Core Principles

1. **Security First**: Enforce proper access controls and permissions
2. **Structured Organization**: Maintain consistent file paths and bucket structures
3. **Performance Optimization**: Implement efficient upload/download strategies
4. **Error Resilience**: Handle failures gracefully at all points

## Configuration Best Practices

- Use environment variables for all bucket names
  ```ts
  // In .env.local
  NEXT_PUBLIC_SUPABASE_PROFILE_BUCKET="profile-images"
  
  // In code
  const profileBucket = process.env.NEXT_PUBLIC_SUPABASE_PROFILE_BUCKET
  ```
- Never hardcode bucket names in application code
- Set appropriate file size limits and validate file types
- Use `upsert: true` only when intentionally replacing files
- Always include proper content-type headers when uploading

## Bucket Organization

### Naming & Structure

- Use kebab-case for bucket names: `profile-images`, `document-files`
- Create purpose-specific buckets rather than general-purpose ones
- Document each bucket's purpose and access pattern

### Security Policies

- Make buckets private by default
- Implement Row Level Security (RLS) for user-specific access
- Example RLS policy for user-owned files:
  ```sql
  CREATE POLICY "Users can only access their own files"
  ON storage.objects
  FOR ALL
  USING (auth.uid()::text = (storage.foldername(name))[1]);
  ```

> **Note**: For production deployment, provide SQL scripts for required RLS policies to your DevOps team.

## File Path Conventions

Organize files using consistent path structures:

```
{bucket}/{userId}/{purpose}/{filename}
```

Examples:
- User avatars: `profile-images/user_123/avatar/profile.jpg`
- Documents with versioning: `documents/user_123/contracts/2024-02-13-contract.pdf`

This structure:
- Enables effective RLS policies
- Simplifies permission management
- Provides natural organization

## Implementation Patterns

### Action Structure

- Store storage actions in `actions/storage/` directory
- Name files descriptively: `profile-storage.ts`
- Follow the standardized `ActionState<T>` return pattern

### Example Storage Action

```ts
"use server"

import { createClientComponentClient } from "@supabase/auth-helpers-nextjs"
import { ActionState } from "@/types"

export async function uploadFile(
  bucket: string,
  path: string,
  file: File
): Promise<ActionState<{ path: string }>> {
  try {
    const supabase = createClientComponentClient()
    
    const { data, error } = await supabase.storage
      .from(bucket)
      .upload(path, file, {
        upsert: false,
        contentType: file.type
      })

    if (error) throw error

    return {
      isSuccess: true,
      message: "File uploaded successfully",
      data: { path: data.path }
    }
  } catch (error) {
    console.error("Error uploading file:", error)

    return { isSuccess: false, message: "Failed to upload file" }
  }
}
```

## File Operations

### Upload Implementation

Always validate files before upload:

```ts
const MAX_FILE_SIZE = 10 * 1024 * 1024 // 10MB
const ALLOWED_TYPES = ["image/jpeg", "image/png", "image/webp"]

function validateFile(file: File): boolean {
  if (file.size > MAX_FILE_SIZE) {
    throw new Error("File size exceeds limit")
  }
  
  if (!ALLOWED_TYPES.includes(file.type)) {
    throw new Error("File type not allowed")
  }
  
  return true
}
```

Key upload considerations:
- Generate unique filenames to prevent collisions
- Validate file types using MIME type checking
- Implement client-side size restrictions
- Set appropriate content-type headers

### Download Strategies

- Use signed URLs for private files with appropriate expiration
- Implement proper error handling for missing files
- Consider caching strategies for frequently accessed files

### Deletion Guidelines

- Verify ownership before deletion
- Implement soft deletes when appropriate
- Clean up related database records
- Handle bulk operations with care
- Remove all related versions/transforms

## Security Considerations

### Access Controls

- Generate short-lived signed URLs for private content
- Configure appropriate CORS policies
- Never expose internal file paths to clients
- Validate user permissions before operations

### Client-Side Implementation

```tsx
"use client"

import { useState } from "react"
import { uploadFile } from "@/actions/storage/profile"

export function AvatarUploader() {
  const [isUploading, setIsUploading] = useState(false)
  
  const handleUpload = async (e: React.ChangeEvent<HTMLInputElement>) => {
    if (!e.target.files?.[0]) return
    
    try {
      setIsUploading(true)
      const file = e.target.files[0]
      
      // Validate file
      if (file.size > 5 * 1024 * 1024) {
        throw new Error("File too large (max 5MB)")
      }
      
      if (!["image/jpeg", "image/png"].includes(file.type)) {
        throw new Error("Only JPEG and PNG allowed")
      }
      
      // Generate path with user ID and timestamp
      const userId = "current-user-id" // Get from auth context
      const filename = `${Date.now()}-${file.name}`
      const path = `${userId}/avatars/${filename}`
      
      // Upload file
      const result = await uploadFile("profile-images", path, file)
      
      if (!result.isSuccess) {
        throw new Error(result.message)
      }
      
      // Handle successful upload
      console.log("Uploaded to:", result.data.path)
    } catch (error) {
      console.error("Upload failed:", error)
    } finally {
      setIsUploading(false)
    }
  }
  
  return (
    <div>
      <input 
        type="file" 
        accept="image/jpeg,image/png" 
        onChange={handleUpload}
        disabled={isUploading}
      />
      {isUploading && <p>Uploading...</p>}
    </div>
  )
}
```

## Performance Optimization

- Implement progressive uploads for large files
- Use batch operations for handling multiple files
- Clean up temporary and failed uploads
- Consider image compression for large image files
- Configure appropriate caching headers

---
> Source: [materialize-labs/ai-optimized-starter-app](https://github.com/materialize-labs/ai-optimized-starter-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
