## book-quotes

> A **local-first mobile app** for tracking books you own, books you read, favorite quotes, page references, notes, ratings, and your own recorded audio clips.

# Claude.md

## Project: Personal Reading Vault
A **local-first mobile app** for tracking books you own, books you read, favorite quotes, page references, notes, ratings, and your own recorded audio clips.

This app is **for personal use only**.  
It does **not** sync to a server, does **not** use cloud storage, and does **not** require user accounts.

---

# 1. Product Vision

Build a private phone app that feels like a personal digital library:

- A front page that looks like a **bookshelf / library grid** of covers
- Tap a cover to open the book
- Inside each book, save:
  - quotes
  - page numbers
  - optional chapter references
  - notes
  - your own recorded audio clips
  - reading dates
  - rating
- Let the app later suggest **what book to read next from your own library**

This is not a social app, not a marketplace, and not a cloud product.  
It is a **personal reading journal + quote vault + TBR recommender**.

---

# 2. Non-Negotiable Product Constraints

## 2.1 Local-first only
All data stays on-device unless export/import is explicitly added later.

### This means:
- no Supabase
- no Firebase
- no cloud database
- no remote auth
- no remote file storage
- no server APIs required for core usage

## 2.2 Personal use
The app is intended for one user on one phone.

## 2.3 Manual content entry
The app should not depend on external services such as:
- Audible clip importing
- Kindle syncing
- Goodreads syncing
- Apple Books syncing

If those are ever explored, they must be optional and treated as future experiments, not core architecture.

## 2.4 Privacy-first
Since the app contains personal reading history, notes, and audio recordings:
- data should be stored locally
- audio files should remain local
- app should function fully offline
- no analytics required in MVP

---

# 3. Recommended Technical Direction

## 3.1 App type
A **mobile app** for iPhone and Android, with primary use on phone.

## 3.2 Recommended stack
Use **React Native with Expo** for the app shell and UI.

### Why
- fast to build
- excellent for phone-first UI
- easy local media handling
- easier iteration than native Swift/Kotlin for a solo project
- supports local storage and file access

## 3.3 Local persistence
Recommended local storage strategy:

- **SQLite** for structured app data
- **local file storage** for book covers and recorded audio
- lightweight key-value storage for preferences if needed

### Suggested libraries
- `expo-sqlite` or a SQLite wrapper
- `expo-file-system`
- `expo-audio` or current Expo audio recording/playback support
- `expo-image-picker` for cover import
- `expo-router` for navigation
- optional ORM/query layer:
  - Drizzle ORM with SQLite, or
  - direct SQL with a clean repository layer

## 3.4 Architecture style
Use a simple local-first architecture:

- UI layer
- feature/state layer
- repository/data layer
- SQLite database
- local media storage

Avoid overengineering. No microservices nonsense. It is a phone app, not a doomed corporate platform.

---

# 4. Core Product Model

The main objects are:

- **Book**
- **Moment**
- **Tag**
- **Reading Session** (future-friendly if rereads matter)

For MVP, the app can start with `Book` and `Moment`, then add more structured reading history later.

---

# 5. Core User Experience

## 5.1 Library screen
The home screen shows the user's books as a visual shelf/grid of covers.

### Goals
- feel like browsing a personal library
- make books easy to open
- give immediate visual context

### Each book card should show:
- cover
- title
- author
- optional status badge
- optional small indicators:
  - number of moments
  - rating
  - finished date

## 5.2 Book detail screen
When a user taps a book cover, they open the book page.

### Book page includes:
- large cover
- title
- author
- reading status
- date started
- date finished
- rating
- action buttons:
  - add moment
  - record audio
  - edit book

### Content sections
Recommended tabs:
- All
- Quotes
- Audio
- Favorites

## 5.3 Add moment flow
A moment is the core saved item.

A moment may contain:
- quote text
- page number
- chapter
- note
- favorite flag
- optional recorded audio clip
- optional tags

This is better than treating quote and audio as unrelated objects. The real thing being saved is a **moment from the book**.

---

# 6. Local Data Model

## 6.1 Books table

```sql
CREATE TABLE books (
  id TEXT PRIMARY KEY NOT NULL,
  title TEXT NOT NULL,
  author TEXT NOT NULL,
  cover_path TEXT,
  format TEXT,
  status TEXT NOT NULL DEFAULT 'not_started',
  date_started TEXT,
  date_finished TEXT,
  rating REAL,
  priority INTEGER DEFAULT 0,
  genre TEXT,
  description TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);
```

### Notes
- `cover_path` is a local file path, not a remote URL
- `rating` can support half-stars, e.g. `4.5`
- `status` values:
  - `not_started`
  - `currently_reading`
  - `finished`
  - `paused`
  - `abandoned`

## 6.2 Moments table

```sql
CREATE TABLE moments (
  id TEXT PRIMARY KEY NOT NULL,
  book_id TEXT NOT NULL,
  quote_text TEXT,
  page_number INTEGER,
  chapter TEXT,
  note_text TEXT,
  audio_path TEXT,
  audio_duration_seconds INTEGER,
  audio_reference_timestamp_seconds INTEGER,
  is_favorite INTEGER NOT NULL DEFAULT 0,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (book_id) REFERENCES books(id) ON DELETE CASCADE
);
```

### Notes
- `audio_path` is a local file path
- `audio_reference_timestamp_seconds` is optional and can represent where the passage occurs in the audiobook
- a moment does not need both quote and audio, but it may contain both

## 6.3 Tags table

```sql
CREATE TABLE tags (
  id TEXT PRIMARY KEY NOT NULL,
  name TEXT NOT NULL UNIQUE,
  type TEXT NOT NULL DEFAULT 'custom'
);
```

## 6.4 Moment tags join table

```sql
CREATE TABLE moment_tags (
  moment_id TEXT NOT NULL,
  tag_id TEXT NOT NULL,
  PRIMARY KEY (moment_id, tag_id),
  FOREIGN KEY (moment_id) REFERENCES moments(id) ON DELETE CASCADE,
  FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);
```

## 6.5 Optional future: reading sessions

```sql
CREATE TABLE reading_sessions (
  id TEXT PRIMARY KEY NOT NULL,
  book_id TEXT NOT NULL,
  date_started TEXT,
  date_finished TEXT,
  format TEXT,
  notes TEXT,
  created_at TEXT NOT NULL,
  FOREIGN KEY (book_id) REFERENCES books(id) ON DELETE CASCADE
);
```

### Why later
This supports rereads without complicating MVP.

---

# 7. File Storage Strategy

## 7.1 Covers
Store imported covers in the app's local documents directory.

Example:
- `/documents/covers/{bookId}.jpg`

## 7.2 Audio recordings
Store recordings in a dedicated local folder.

Example:
- `/documents/audio/{momentId}.m4a`

## 7.3 Deletion rules
When deleting a book:
- delete all related moments
- delete related audio files
- optionally delete the cover file if not reused

When deleting a moment:
- delete its audio file if present

## 7.4 Backup compatibility
Even though the app is local-only, data should be structured so export/import is possible later.

That means:
- stable IDs
- clean file references
- no hidden magic state

---

# 8. Screen Map

## 8.1 MVP screens
1. Splash / app load
2. Library
3. Add Book
4. Edit Book
5. Book Detail
6. Add Moment
7. Edit Moment
8. Record / Preview Audio
9. Settings

## 8.2 Future screens
1. Search
2. Tags / filters
3. Reading stats
4. Pick my next book
5. Export / import
6. Backup management
7. Themes / appearance

---

# 9. Implementation Phases

The app should be built in phases from smallest useful product to richer personal features.

---

# Phase 1: Foundation MVP

## Goal
Deliver a functioning local app where the user can:
- add books
- see them in a cover grid
- open a book
- save text moments
- record local audio moments
- track reading dates and rating

## Scope

### 1. Project setup
- create Expo app
- set up navigation
- set up local SQLite database
- set up local file storage utilities
- define TypeScript models
- create repository layer

### 2. Library screen
- grid view of books
- add book button
- empty state
- local cover display

### 3. Add/Edit Book
Fields:
- title
- author
- cover image
- format
- status
- date started
- date finished
- rating

### 4. Book detail page
Display:
- cover
- title
- author
- status
- date started
- date finished
- rating
- list of moments

### 5. Add/Edit Moment
Fields:
- quote text
- page number
- chapter
- note
- favorite toggle

### 6. Local audio recording
- record audio clip
- save locally
- attach to a moment
- playback inside book detail / moment detail

### 7. Data persistence
- all reads/writes stored locally
- app works offline with no network dependency

## Deliverable
A usable personal app that already solves the main problem:
saving books, quotes, pages, ratings, dates, and audio clips locally.

## Out of scope for Phase 1
- search
- tags
- recommendations
- export/import
- statistics
- reread support
- OCR
- external integrations

---

# Phase 2: Quality of Life Features

## Goal
Make the app easier to browse and maintain.

## Scope

### 1. Search
Allow searching by:
- title
- author
- quote text
- note text

### 2. Sorting and filtering
Sort books by:
- title
- author
- date added
- rating
- status
- recently finished

Filter by:
- not started
- currently reading
- finished
- paused
- abandoned
- favorites only

### 3. Favorites
- mark moments as favorite
- dedicated favorites filter or tab

### 4. Better library metadata
Show optional small indicators on book cards:
- rating
- moment count
- status

### 5. Edit and delete flows
- edit moments
- delete moments
- delete books safely
- cleanup local files on delete

### 6. Better audio UX
- preview recording before save
- replace recording
- remove recording from moment

## Deliverable
A polished local reading vault that is comfortable to use regularly.

---

# Phase 3: Tagging and Classification

## Goal
Let the user organize books and moments semantically.

## Scope

### 1. Tags for moments
Examples:
- heartbreaking
- witchy
- rage
- romantic
- lyrical
- devastating

### 2. Optional book tags
Examples:
- gothic
- fantasy
- literary
- dark romance
- cozy
- horror

### 3. Tag filters
- filter by one or more tags
- browse moments by tag

### 4. Suggested tag UX
Keep this manual first:
- choose existing tags
- add new custom tags

No AI. No fake intelligence pretending to know your soul from three highlights.

## Deliverable
A categorised personal archive that is easier to browse by feeling, genre, or theme.

---

# Phase 4: Reading Journal Depth

## Goal
Evolve the app from book storage into a fuller reading journal.

## Scope

### 1. Reading progress fields
Optional additions:
- current page
- total pages
- reading percentage

### 2. Reading sessions / rereads
Introduce `reading_sessions` table if needed.

Use this only once the user wants:
- reread history
- multiple start/finish cycles
- separate audiobook and physical reads

### 3. More nuanced book metadata
Optional fields:
- source (owned, borrowed, library, gift)
- format (paperback, hardcover, ebook, audiobook)
- series name
- series order
- personal review

### 4. Reading timeline
Show:
- books finished by month
- recent reads
- currently reading
- paused books

## Deliverable
A richer personal reading log with long-term usefulness.

---

# Phase 5: Recommendation Engine v1

## Goal
Recommend the next book to read from the user's own library.

## Principle
Do **not** begin with machine learning.
Use a transparent rules-based system.

The app should explain **why** it recommended something.

## Inputs available
- books owned
- unread books
- finished books
- ratings
- genre
- tags
- priority
- recent reading history

## Candidate pool
Recommend from books where status is:
- `not_started`
- `paused`

Exclude:
- `finished`
- `abandoned` by default

## Scoring model
Example v1 scoring:

- `+3` if genre matches books rated 4.0+
- `+2` if book tags match highly rated books
- `+2` if author matches a previously loved author
- `+2` if priority is high
- `+1` if format matches recent preference
- `+1` if shorter than recent long reads
- `-2` if the book has been repeatedly skipped or deprioritised

## Recommendation modes
- Best match
- Based on favorites
- Mood pick
- Short read
- Neglected book
- Surprise me

## UI
Create a dedicated screen or card section:
- "What should I read next?"

Each recommendation should display:
- cover
- title
- author
- reason text

Example reason text:
- "Matches genres you've rated highly"
- "High-priority unread book"
- "Similar tone to recent favorites"

## Deliverable
A useful local recommendation feature based entirely on the user's own data.

---

# Phase 6: Export / Import / Backup

## Goal
Prevent the app from becoming a fragile little shrine that dies with one broken phone.

## Scope

### 1. Export data
Export a portable backup bundle containing:
- SQLite DB or JSON export
- covers
- audio files
- metadata manifest

### 2. Import data
Restore from a previous export bundle.

### 3. Share/export options
Optional exports:
- CSV of books
- Markdown of favorite quotes
- JSON backup

### 4. Backup reminders
Optional local reminder banner:
- "You have not exported a backup in 90 days"

## Deliverable
A local-only app with user-controlled portability.

---

# Phase 7: UI and Aesthetic Polish

## Goal
Make the app feel beautiful and personal.

## Scope

### 1. Library presentation improvements
Possible views:
- shelf grid
- compact grid
- list view

### 2. Themes
Examples:
- dark library
- parchment
- moonlit
- minimalist black

### 3. Book detail polish
- better typography for quotes
- card designs for moments
- better audio player layout

### 4. Small delight features
- reading streak visual
- "finished this month"
- subtle transitions
- pinned favorites

## Deliverable
An app that feels emotionally nice to open, not just functional.

---

# Phase 8: Smart Features for Later

## Goal
Optional advanced features once the core app is already excellent.

## Possible features

### 1. Smarter recommendations
Use richer heuristics from:
- ratings
- tags
- genre clusters
- reading pace
- time since purchase/addition

### 2. Quote rediscovery
Show:
- On this day
- Random favorite quote
- Rediscover neglected highlights

### 3. OCR experiments
Possible but non-core:
- scan a page photo
- extract text locally if feasible

This should only be explored if legal, UX, and device performance make sense.

### 4. Local semantic search
Potential advanced feature:
- embedding-based search done fully on-device

This is absolutely not an MVP feature.

### 5. Widget support
Potential home screen widgets:
- current read
- random quote
- next recommendation

## Deliverable
Optional advanced personal features without betraying the local-first model.

---

# 10. Explicitly Rejected for Core Build

These are not part of the planned architecture:

- cloud sync
- online accounts
- remote auth
- public sharing
- Goodreads clone features
- social feed
- public profiles
- external clip importing from Audible
- automatic scraping from ebook services
- complex AI features in early phases

If a feature requires turning the app into a server product, it is outside current scope.

---

# 11. Suggested Folder Structure

```text
src/
  app/
    (routes and screens)
  components/
  features/
    books/
    moments/
    recommendations/
    tags/
    settings/
  db/
    schema/
    migrations/
    repositories/
  lib/
    audio/
    files/
    dates/
    search/
  hooks/
  types/
  constants/
```

---

# 12. Domain Models

## Book
```ts
type BookStatus =
  | 'not_started'
  | 'currently_reading'
  | 'finished'
  | 'paused'
  | 'abandoned';

type Book = {
  id: string;
  title: string;
  author: string;
  coverPath?: string | null;
  format?: string | null;
  status: BookStatus;
  dateStarted?: string | null;
  dateFinished?: string | null;
  rating?: number | null;
  priority: number;
  genre?: string | null;
  description?: string | null;
  createdAt: string;
  updatedAt: string;
};
```

## Moment
```ts
type Moment = {
  id: string;
  bookId: string;
  quoteText?: string | null;
  pageNumber?: number | null;
  chapter?: string | null;
  noteText?: string | null;
  audioPath?: string | null;
  audioDurationSeconds?: number | null;
  audioReferenceTimestampSeconds?: number | null;
  isFavorite: boolean;
  createdAt: string;
  updatedAt: string;
};
```

---

# 13. Suggested MVP Acceptance Criteria

The MVP is complete when the user can:

1. install and open the app
2. add a book with title, author, and cover
3. see books displayed as covers in a library grid
4. open a book detail page
5. add a moment with quote text and page number
6. add a note to that moment
7. record an audio clip locally
8. attach audio to a moment
9. set started and finished dates for a book
10. rate the book
11. close and reopen the app without losing any data
12. use the app fully offline

If those twelve things work smoothly, the MVP is real.

---

# 14. Build Order Summary

## MVP
- app shell
- SQLite
- book CRUD
- library grid
- book detail
- moment CRUD
- local audio recording
- reading dates
- rating

## Next
- search
- filters
- favorites
- deletion cleanup
- better playback UX

## Then
- tags
- reading journal depth
- rereads
- recommendation engine

## Then
- export/import
- polish
- advanced local-only smart features

---

# 15. Final Product Statement

This app is a **local-first personal reading vault**.

It exists to let one user:
- store their books
- track reading dates and ratings
- save favorite quotes and pages
- record personal audio passages
- browse everything in a beautiful library-like interface
- eventually get recommendations on what to read next from their own shelves

No cloud. No accounts. No sync dependency. No platform lock-in.  
Just your library, your reading history, your moments, on your phone.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marSanGal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
