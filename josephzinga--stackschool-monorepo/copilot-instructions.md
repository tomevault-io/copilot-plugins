## stackschool-monorepo

> You are working in a Node.js / TypeScript backend using:

# School Timetable Generation – Prisma + date-fns

## Context
You are working in a Node.js / TypeScript backend using:
- Prisma ORM
- PostgreSQL
- date-fns
- An existing Prisma Client instance (`prisma`)

The goal is to generate school timetables (lessons) for a school using:
- schoolId
- classId
- teacherId
- subjectId

The lesson name MUST be derived from the subject name (`subject.name`).

---

## Important Naming Rule

When creating a Lesson:
- Use `subject.name` as the lesson title
- Do NOT hardcode lesson names

Example:
```ts
title: subject.name
```

---

## Data Relationship Reference (NO SQL IN CODE)

```sql
Class -> ClassSubjects -> Subject
                     -> Teacher
```

This SQL is for understanding only and must not be implemented in code.

---

## Prisma Lesson Model Recommendation

It is RECOMMENDED to add a `title` column to the Lesson model.

```prisma
model Lesson {
  id        String   @id @default(cuid())
  title     String
  day       Day
  startTime DateTime
  endTime   DateTime

  schoolId  String
  classId   String
  teacherId String
  subjectId String

  school    School  @relation(fields: [schoolId], references: [id])
  class     Class   @relation(fields: [classId], references: [id])
  teacher   Teacher @relation(fields: [teacherId], references: [id])
  subject   Subject @relation(fields: [subjectId], references: [id])

  @@index([schoolId, day])
}
```

---

## Constraints

- Lessons must be created per `schoolId`
- A teacher cannot teach two lessons at the same time
- A class cannot have two lessons at the same time
- Lesson duration: 1 hour
- Time range: 08:00 → 16:00
- Days: MONDAY → FRIDAY

---

## date-fns Requirements

Use ONLY date-fns for time manipulation:
- setHours
- setMinutes
- addHours
- isBefore

---

## Seed Example

```ts
await prisma.lesson.create({
  data: {
    title: subject.name,
    schoolId,
    classId,
    subjectId,
    teacherId,
    day: 'MONDAY',
    startTime,
    endTime,
  },
});
```

---

## What NOT to Do

- No raw SQL
- No hardcoded dates
- No redefining Prisma client
- No fake IDs

---
> Source: [Josephzinga/stackschool-monorepo](https://github.com/Josephzinga/stackschool-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
