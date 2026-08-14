## unibook

> UniBook is a university-focused social platform inspired by Reddit. Users can join communities, post content, and follow each other. Built as an incremental personal/student project.

# UniBook — Project Knowledge

## What is this

UniBook is a university-focused social platform inspired by Reddit. Users can join communities, post content, and follow each other. Built as an incremental personal/student project.

## Architecture

```
UniBook/
├── monorepo/          # Go backend (API + router + domain logic)
│   ├── wb_api/        # main.go — server entry point, DI wiring
│   ├── wb_router/     # HTTP routing (Gorilla Mux)
│   │   └── routes/    # One file per domain: user, post, community, login, comment
│   ├── v1/handlers/   # HTTP handler layer (thin, delegates to service)
│   ├── users/         # User domain: model, service, repository
│   ├── post/          # Post domain: model, service, repository (includes like logic)
│   ├── community/     # Community domain: model, service, repository
│   ├── comment/       # Comment domain: model, service, repository
│   ├── tag/           # Tag domain: model, service, repository
│   └── util/          # JWT auth, middleware, response helpers
└── wb-front/          # React + TypeScript frontend
    └── src/
        ├── pages/     # Login, Register, Index (redirect)
        ├── components/ # Feed, PostContainer, Communities, CommunityContainer, Layout, Header, SideMenu
        └── utils/     # auth.ts (JWT storage + axios setup)
```

## Tech Stack

**Backend**
- Go
- Gorilla Mux (routing)
- JWT (authentication via `util/authentication`)
- MySQL

**Frontend**
- React 18 + TypeScript
- React Router v6
- Axios (HTTP, token set as default header)
- Ant Design (UI components)
- SCSS (styling)
- React Toastify (notifications)
- Vite (build tool)

## Auth Flow

1. `POST /login` → returns JWT
2. Frontend stores token in `localStorage` (remember me) or `sessionStorage`
3. `setAuthToken()` in `utils/auth.ts` sets `axios.defaults.headers.common['Authorization']`
4. All protected routes use `m.IsAuth()` middleware that validates the JWT
5. A 401 response triggers the Axios interceptor in `App.tsx` → clears token → redirects to `/login`
6. `PrivateRoute` component guards all authenticated frontend routes

## Backend Conventions

- Handlers are **interfaces** (`UserHandler`, `PostHandler`, `CommunityHandler`) — concrete types are unexported structs wired via `New*Handler()` constructors
- All responses go through `util/response`: `response.JSON(w, status, data)` and `response.Erro(w, status, err)`
- `authentication.ExtractUserId(r)` reads the userId from the JWT inside any handler
- Routes are declared as `[]Route` slices, assembled in `routes.Config()`, auth routes wrapped with `m.IsAuth()`

## Frontend Conventions

- API base URL comes from `import.meta.env.VITE_API_URL`
- Token helpers are all in `wb-front/src/utils/auth.ts`
- Components export from `wb-front/src/components/index.tsx`
- Pages live under `wb-front/src/pages/` and are registered in `App.tsx`

## What Is and Isn't Done

## Database Schema (MySQL)

Canonical DDL in `migrations/schema.sql`. Incremental files: 001–005.

| Table | Key columns |
|-------|-------------|
| users | id, name varchar(100), nick varchar(50), email varchar(255), password varchar(255), image_url |
| followers | user_id, follower_id (composite PK) |
| community | id, name varchar(100), description text, image_url |
| community_followers | user_id BIGINT UNSIGNED, community_id BIGINT UNSIGNED |
| posts | id, user_id, community_id, title, body, image_url, likes INT DEFAULT 0, updated_at |
| tags | id, name UNIQUE, created_by (nullable FK→users) |
| post_tags | post_id, tag_id (composite PK, cascade delete) |
| post_likes | post_id, user_id (composite PK) — one row per like, prevents duplicates |
| post_comments | id AI, post_id, user_id, body TEXT |

`posts.likes` is a denormalized counter kept in sync by LikePost/UnlikePost transactions. `post_likes` is the deduplication source of truth.

## API Routes

| Method | Path | Auth | Handler |
|--------|------|------|---------|
| POST | /login | no | Login |
| POST | /users | no | Register |
| GET | /users?user= | yes | SearchUsers |
| PUT | /users/{userId} | yes | UpdateUser |
| DELETE | /users/{userId} | yes | DeleteUser |
| POST | /users/{userId}/follow | yes | FollowUser |
| POST | /users/{userId}/unfollow | yes | UnfollowUser |
| GET | /users/{userId}/followers | yes | GetFollowers |
| PUT | /users/{userId}/update-password | yes | UpdatePassword |
| POST | /post | yes | CreatePost |
| GET | /post/c/{communityId} | yes | GetCommunityPosts |
| GET | /post/{userId} | yes | GetUserPosts |
| PUT | /post/{id} | yes | UpdatePost (stub) |
| DELETE | /post/{id} | yes | DeletePost (stub) |
| POST | /post/{id}/like | yes | LikePost |
| DELETE | /post/{id}/like | yes | UnlikePost |
| POST | /post/{postId}/comments | yes | CreateComment |
| GET | /post/{postId}/comments | yes | GetComments |
| GET | /c/all | yes | ListCommunities |
| GET | /c/joined | yes | JoinedCommunities |
| GET | /c/{id} | yes | GetCommunity |
| POST | /c/{communityId}/follow | yes | FollowCommunity |
| POST | /c/{communityId}/unfollow | yes | UnfollowCommunity |
| POST | /tags | yes | CreateTag |
| GET | /tags?name= | yes | ListTags (supports partial search) |

LikePost/UnlikePost return `{"likes": N}` (updated count). Already-liked returns 400.

## What Is and Isn't Done

### Working end-to-end
- Login / Register
- List all communities, view/join/leave a community
- View posts from a community with tag filtering
- Create post with tags (modal in CommunityPage)
- Like / Unlike a post (backend only — frontend buttons still dead)
- Post comments (backend only — frontend not wired)

### Backend ready, frontend missing
- Like/Unlike buttons in PostContainer → `POST/DELETE /post/{id}/like`
- Comment section in PostContainer → `POST/GET /post/{postId}/comments`
- Profile page (`/profile`) — `GET /users?user=` + `PUT /users/{userId}`
- Change password page (`/change-password`) — `PUT /users/{userId}/update-password`
- View user posts — `GET /post/{userId}` (belongs on profile page)
- Search users — `GET /users?user=` (Header search bar not wired)
- Follow / Unfollow user — `POST /users/{userId}/follow|unfollow`
- View followers — `GET /users/{userId}/followers`

### Backend handler exists but route not registered yet
- `Following()` — needs a GET route added to `user_routes.go`
- `CreateCommunity()` — needs a POST route added to `community_routes.go`

### Both backend and frontend TODO (handler stubs)
- Update post / Delete post
- Search posts by title
- Get community by name / ID
- Delete community
- Get community members
- Feed/timeline (posts from followed communities/users)
- Popular posts / New posts

### Dead UI elements to wire up
- PostContainer: Like button → `POST/DELETE /post/{id}/like`
- PostContainer: Comment button → expand comment section
- CommunityContainer: member count not fetched
- Header search bar

flowchart TD
    Client(["Front-end React / TypeScript"])

    subgraph main["Inicialização — main.go"]
        direction LR
        CFG["config.Load()"]
        DBC["dbconfig.Connect()\n→ *sql.DB"]
        DI["Injeção de Dependência\nNew*Repository → New*Service → New*Handler"]
        CFG --> DBC --> DI
    end

    subgraph server["Servidor Go — Gorilla Mux"]
        direction TB

        CORS["Middleware CORS\ncors.EnableCORS()"]
        LOG["Middleware Logger\nm.Logger()"]

        subgraph auth["Middleware de Autenticação"]
            REQ{"RequireAuth?"}
            VAL["authentication.ValidateToken()\nJWT HS256 — extrai userId"]
        end

        subgraph routes["Roteamento — wb_router/routes"]
            direction LR
            R1["POST /login"]
            R2["/users/..."]
            R3["/post/..."]
            R4["/c/..."]
            R5["/tag/..."]
        end

        subgraph handlers["Camada Handler — v1/handlers (interfaces Go)"]
            H["Parse body · ExtractUserId(r)\nDelega ao Service · response.JSON()"]
        end

        subgraph services["Camada Service"]
            S["Validações de domínio\nRegras de negócio\nOrquestra Repository"]
        end

        subgraph repos["Camada Repository"]
            R["Prepared Statements SQL\nMapeamento para structs Go"]
        end
    end

    DB[("MySQL\nusers · post · community\ncommunity_followers · tags")]

    Client -->|"HTTP Request\nAuthorization: Bearer token"| CORS
    CORS --> LOG
    LOG --> REQ
    REQ -->|"sim"| VAL
    VAL -->|"Token inválido\n← 401 Unauthorized"| Client
    VAL -->|"Token válido"| routes
    REQ -->|"não"| routes
    routes --> H
    H --> S
    S --> R
    R <-->|"Query / Result"| DB
    H -->|"JSON Response"| Client

    main -.->|"wires"| server

---
> Source: [Mat-develop/UniBook](https://github.com/Mat-develop/UniBook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
