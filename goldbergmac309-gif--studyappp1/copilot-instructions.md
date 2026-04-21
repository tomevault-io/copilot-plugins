## studyappp1

> **Document Status:** CANONICAL REALITY. This is your "Single Source of Truth." It is a precise map of the codebase's target state. All code you generate MUST conform perfectly to the structures and contracts defined herein.


#  & ARCHITECTURAL STATE v3.0 - Synapse OS

**Document Status:** CANONICAL REALITY. This is your "Single Source of Truth." It is a precise map of the codebase's target state. All code you generate MUST conform perfectly to the structures and contracts defined herein.

**Current Focus: Phase 1, Sprint 1 - The Identity Core**

## 1. Target Directory Structure (for `core-service`)

After you complete the scaffolding task, this is the exact directory structure you must adhere to within `apps/core-service/src/`. You are **not permitted** to create new top-level directories under `src/` without explicit instruction.

apps/core-service/src/
├── app.module.ts
├── main.ts
├── prisma/
│ ├── schema.prisma # <--- DATABASE SCHEMA SSOT
│ ├── prisma.module.ts # DI Module for Prisma
│ └── prisma.service.ts # Injectable Prisma Client Service
├── config/
│ ├── config.module.ts
│ └── configuration.ts # Centralized Configuration
├── users/
│ ├── users.module.ts
│ ├── users.controller.ts
│ ├── users.service.ts
│ └── dto/
│ └── create-user.dto.ts
└── auth/
├── auth.module.ts
├── auth.controller.ts
├── auth.service.ts
├── guards/
│ └── jwt-auth.guard.ts
└── strategies/
└── jwt.strategy.ts


## 2. Data Contracts (The Law of Definition-First)

These are the non-negotiable shapes of our data.

### 2.1 Database Schema (`prisma/schema.prisma`)
This is the ONLY definition of our database state.

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../../../node_modules/.prisma/client"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  password  String // Will be a hash
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

// NOTE: Subject and Document models will be added in later sprints.
// We are focused ONLY on the User model for Sprint 1.


2.2 Shared DTOs (Data Transfer Objects)
For this sprint, all DTOs will live inside their respective modules for simplicity (e.g., users/dto/). They MUST be class-based to support class-validator.
users/dto/create-user.dto.ts:

import { IsEmail, IsNotEmpty, IsString, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsEmail({}, { message: 'A valid email is required.' })
  @IsNotEmpty()
  email: string;

  @IsString()
  @IsNotEmpty()
  @MinLength(8, { message: 'Password must be at least 8 characters long.' })
  password: string;
}

2.3 API Response Contracts (To Be Enforced by Tests)
Login Response:

export interface LoginResponse {
  accessToken: string;
  user: {
    id: string;
    email: string;
  };
}


3. API Contract (The Endpoints You Will Build)
Service: core-service
POST /auth/signup
Request Body: CreateUserDto
Success (201) Response Body: LoginResponse
Failure (400) Response: On validation error (e.g., weak password, invalid email).
Failure (409) Response: If email already exists.
POST /auth/login
Request Body: { email, password }
Success (200) Response Body: LoginResponse
Failure (401) Response: On invalid credentials.
4. Canonical Implementations (Core Architectural Patterns)
These are specific implementations that are non-negotiable to prevent architectural drift.
Centralized Configuration (config/configuration.ts):


Centralized Configuration (config/configuration.ts):
import { registerAs } from '@nestjs/config';

export default registerAs('app', () => ({
  jwtSecret: process.env.JWT_SECRET,
}));


Injectable Prisma Service (prisma/prisma.service.ts):
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goldbergmac309-gif) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
