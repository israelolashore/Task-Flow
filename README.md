# TaskFlow

> **Organize your work. Own your progress.**

TaskFlow is a full-stack task management web application. Users can create an account, log in, and manage their personal tasks with full CRUD operations, search, filtering, sorting, pagination, and a real-time statistics dashboard. Each user's tasks are completely private — enforced at the database level.

Built as a portfolio project based on the [roadmap.sh Todo List API project](https://roadmap.sh/projects/todo-list-api).

---

## Tech Stack

| Layer        | Technology                                            |
| ------------ | ----------------------------------------------------- |
| Frontend     | React, TypeScript, Vite, Tailwind CSS, React Router   |
| Icons        | Lucide React                                          |
| Validation   | Zod                                                   |
| Backend      | Supabase (managed PostgreSQL + Auth + REST data API)  |
| Database     | PostgreSQL (via Supabase)                             |
| Auth         | Supabase Auth (JWT sessions, bcrypt password hashing) |
| Security     | Row Level Security (RLS), per-user data isolation     |

---

## Features

- **Authentication** — register, log in, log out (email + password)
- **Task CRUD** — create, read, update, delete tasks
- **Task fields** — title, description, status, priority, due date
- **Statuses** — Pending, In Progress, Completed
- **Priorities** — Low, Medium, High
- **Search** — full-text search across title and description
- **Filtering** — filter by status and priority
- **Sorting** — sort by created date, due date, priority, or title (asc/desc)
- **Pagination** — paginated task list
- **Dashboard** — real-time stats (total, completed, pending, in-progress), recent tasks, today's tasks, high-priority tasks, completion rate
- **Completed tasks view** — review, reopen, or delete completed tasks
- **Profile** — view and edit your name
- **Settings** — preferences and account management
- **UX** — loading states, empty states, error states, success toasts, delete confirmation dialogs, form validation, fully responsive

---

## Getting Started

### Prerequisites

- Node.js 18+
- npm

### Installation

```bash
npm install
```

### Environment Variables

The project uses Supabase. The following variables are pre-configured in `.env`:

```env
VITE_SUPABASE_URL=your_supabase_project_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

These are exposed to the frontend via the `VITE_` prefix.

### Running the Project

```bash
npm run dev
```

The app will be available at the URL shown in your terminal.

### Building

```bash
npm run build
```

---

## Database Schema

### profiles

| Column      | Type         | Description                              |
| ----------- | ------------ | ---------------------------------------- |
| id          | uuid (PK)    | Matches `auth.users.id`                  |
| name        | text         | Display name                             |
| email       | text (unique)| User email (mirrors auth)               |
| created_at | timestamptz  | Account creation timestamp               |

A database trigger (`on_auth_user_created`) automatically creates a profile row when a user registers via Supabase Auth.

### todos

| Column        | Type         | Description                                    |
| ------------- | ------------ | ---------------------------------------------- |
| id            | uuid (PK)    | Auto-generated                                 |
| user_id       | uuid (FK)    | Owner → `auth.users.id`, defaults to `auth.uid()` |
| title         | text         | Task title                                     |
| description   | text         | Task description (defaults to '')             |
| status        | text         | `pending` \| `in_progress` \| `completed`      |
| priority      | text         | `low` \| `medium` \| `high`                    |
| due_date      | date         | Optional due date                              |
| created_at    | timestamptz  | Creation timestamp                             |
| updated_at    | timestamptz  | Auto-maintained by trigger                     |
| completed_at  | timestamptz  | Set when completed, cleared when reopened      |

### Security (Row Level Security)

All tables have RLS enabled. Policies enforce that authenticated users can only access their own data:

- **profiles**: SELECT and UPDATE on `auth.uid() = id`
- **todos**: full CRUD on `auth.uid() = user_id`

The `user_id` column defaults to `auth.uid()`, so the frontend never needs to supply a user ID — it is determined server-side from the authenticated session. A user can never read or write another user's tasks.

---

## API (REST Data API)

The frontend communicates with Supabase's auto-generated REST data API (PostgREST). All requests are authenticated via the user's JWT session token.

### Auth Endpoints

| Method | Endpoint                | Description              |
| ------ | ----------------------- | ------------------------ |
| POST   | `/auth/v1/signup`       | Register a new user      |
| POST   | `/auth/v1/token`        | Login (get session)     |
| GET    | `/auth/v1/user`         | Get current user         |
| POST   | `/auth/v1/logout`       | Sign out                 |

### Todo Endpoints

| Method | Endpoint                          | Description                          |
| ------ | --------------------------------- | ------------------------------------ |
| GET    | `/rest/v1/todos`                  | List todos (with query params)      |
| POST   | `/rest/v1/todos`                  | Create a todo                       |
| GET    | `/rest/v1/todos?id=eq.<uuid>`     | Get a single todo                   |
| PATCH  | `/rest/v1/todos?id=eq.<uuid>`     | Update a todo                       |
| DELETE | `/rest/v1/todos?id=eq.<uuid>`     | Delete a todo                       |

### Query Parameters

```
GET /rest/v1/todos?status=eq.completed
GET /rest/v1/todos?priority=eq.high
GET /rest/v1/todos?title=ilike.*project*
GET /rest/v1/todos?limit=10&offset=0
```

---

## Project Structure

```
src/
├── components/
│   ├── layout/
│   │   ├── AppLayout.tsx       # Sidebar + content shell
│   │   └── ProtectedRoute.tsx  # Auth guard
│   ├── tasks/
│   │   ├── TaskCard.tsx
│   │   ├── TaskFormModal.tsx
│   │   └── ConfirmDeleteModal.tsx
│   └── ui/
│       ├── Button.tsx
│       ├── Feedback.tsx        # Card, Badge, Spinner
│       ├── Input.tsx           # Input, Textarea, Select
│       ├── Modal.tsx
│       └── States.tsx           # ErrorState, EmptyState
├── context/
│   ├── AuthContext.tsx
│   └── ToastContext.tsx
├── lib/
│   ├── supabase.ts             # Supabase client
│   ├── todos.ts               # Todo data access
│   ├── stats.ts               # Dashboard stats
│   ├── validation.ts           # Zod schemas
│   └── format.ts               # Formatting helpers
├── pages/
│   ├── LandingPage.tsx
│   ├── LoginPage.tsx
│   ├── RegisterPage.tsx
│   ├── DashboardPage.tsx
│   ├── TasksPage.tsx
│   ├── CompletedPage.tsx
│   ├── ProfilePage.tsx
│   └── SettingsPage.tsx
├── types/
│   └── index.ts
├── App.tsx                     # Router
└── main.tsx                    # Entry point
```

---

## Scripts

| Command              | Description                     |
| -------------------- | ------------------------------- |
| `npm run dev`        | Start the dev server           |
| `npm run build`      | Build for production            |
| `npm run typecheck`  | Run TypeScript type checking    |
| `npm run lint`       | Run ESLint                      |

---

## License

MIT — free to use as a portfolio project.
