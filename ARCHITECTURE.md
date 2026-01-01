# Architecture Documentation

This document describes the architecture, patterns, and standards of the React Query RealWorld application - a Medium-clone social blogging platform implementing the [RealWorld](https://github.com/gothinkster/realworld) specification.

## Table of Contents

- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Architectural Layers](#architectural-layers)
- [State Management](#state-management)
- [Routing Architecture](#routing-architecture)
- [API Integration](#api-integration)
- [Component Architecture](#component-architecture)
- [Error Handling](#error-handling)
- [Performance Optimizations](#performance-optimizations)
- [Coding Standards](#coding-standards)

---

## Technology Stack

| Category | Technology | Version |
|----------|-----------|---------|
| **UI Framework** | React | 18.2.0 |
| **Server State** | TanStack React Query | 4.24.4 |
| **Routing** | React Router DOM | 6.7.0 |
| **HTTP Client** | Axios | 1.2.4 |
| **Language** | TypeScript | 4.9.4 |
| **Build Tool** | Create React App + Craco | 7.0.0 |
| **Error Handling** | react-error-boundary | 3.1.4 |
| **Markdown** | react-markdown + remark-gfm | 8.0.5 |
| **Linting** | ESLint (Airbnb config) | 8.32.0 |
| **Formatting** | Prettier | 2.8.3 |

---

## Project Structure

```
src/
├── index.tsx                    # Entry point - QueryClient & Router setup
├── App.tsx                      # Root component with providers
├── Router.tsx                   # Route configuration with error boundaries
│
├── components/                  # Reusable UI components
│   ├── HOC/                    # Higher-Order Components
│   │   └── ProtectedRoute.tsx  # Authentication guard
│   ├── common/                 # Shared layout components
│   │   ├── Layout.tsx          # Page wrapper with Header/Footer
│   │   ├── Header.tsx          # Navigation header
│   │   └── Footer.tsx          # Footer component
│   ├── article/                # Article-specific components
│   ├── feed/                   # Feed list components
│   ├── header/                 # Header sub-components
│   ├── profile/                # Profile components
│   ├── ErrorFallback.tsx       # Error boundary UI
│   └── LoadingFallback.tsx     # Suspense loading UI
│
├── pages/                       # Route-level page components
│   ├── HomePage/               # Landing page with feeds
│   ├── SignInPage.tsx          # Login
│   ├── SignUpPage.tsx          # Registration
│   ├── SettingPage.tsx         # User settings
│   ├── ArticlePage.tsx         # Single article view
│   ├── NewArticlePage.tsx      # Create article
│   ├── EditArticlePage.tsx     # Edit article
│   ├── ProfilePage.tsx         # User profile
│   └── NotFoundPage.tsx        # 404 page
│
├── queries/                     # React Query hooks (data fetching layer)
│   ├── queryClient.ts          # QueryClient configuration
│   ├── articles.query.ts       # Article queries & mutations
│   ├── user.query.ts           # User queries & mutations
│   └── profiles.query.ts       # Profile queries & mutations
│
├── repositories/               # API layer (HTTP requests)
│   ├── apiClient.ts            # Axios instance with interceptors
│   ├── articles/               # Article endpoints
│   ├── users/                  # User endpoints
│   ├── profiles/               # Profile endpoints
│   └── tags/                   # Tag endpoints
│
├── contexts/                    # React Context providers
│   └── UserContextProvider.tsx # Authentication state
│
├── lib/                        # Utilities and hooks
│   ├── routerMeta.ts           # Route configuration metadata
│   ├── token.ts                # Token storage management
│   ├── hooks/                  # Custom React hooks
│   └── utils/                  # Helper functions
│
├── constants/                   # Application constants
│   ├── query.constant.ts       # Query key constants
│   ├── token.contant.ts        # Token key constant
│   └── units.constants.ts      # Pagination constants
│
└── interfaces/                  # TypeScript type definitions
    └── main.d.ts               # Domain types
```

---

## Architectural Layers

The application follows a **layered architecture** with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                      Pages (Routes)                         │
│  Entry points for each route, orchestrate components        │
├─────────────────────────────────────────────────────────────┤
│                      Components                             │
│  Reusable UI components, presentational logic               │
├─────────────────────────────────────────────────────────────┤
│                   Queries (React Query)                     │
│  Data fetching hooks, caching, mutations                    │
├─────────────────────────────────────────────────────────────┤
│                   Repositories (API)                        │
│  HTTP requests, endpoint definitions                        │
├─────────────────────────────────────────────────────────────┤
│                      API Client                             │
│  Axios instance, interceptors, error handling               │
└─────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

**Pages**: Route-level components that:
- Manage page-specific state (pagination, filters)
- Compose multiple components
- Handle route parameters

**Components**: Reusable UI elements that:
- Receive data via props
- Emit events via callbacks
- Contain no direct API calls

**Queries**: React Query hooks that:
- Define queries and mutations
- Handle caching and invalidation
- Provide loading/error states

**Repositories**: API functions that:
- Make HTTP requests
- Define endpoint paths
- Handle request/response transformation

---

## State Management

### Server State (React Query)

All server-derived data is managed through TanStack Query:

```typescript
// Query configuration
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      suspense: true,    // Enable Suspense integration
      retry: false,      // No automatic retries
    },
  },
});
```

**Query Key Convention**:
```typescript
// constants/query.constant.ts
export const QUERY_USER_KEY = 'user';
export const QUERY_ARTICLES_KEY = 'articles';
export const QUERY_ARTICLE_KEY = 'article';
export const QUERY_COMMENTS_KEY = 'comments';
export const QUERY_PROFILE_KEY = 'profile';
export const QUERY_TAG_KEY = 'tags';
```

**Stale Time**: All queries use a 20-second stale time.

### Client State (React Context)

Authentication state is managed via React Context:

```typescript
// contexts/UserContextProvider.tsx
interface UserContextState {
  isLogin: boolean;
  setIsLogin: Dispatch<SetStateAction<boolean>>;
}
```

### Persistent State (localStorage)

JWT tokens are stored in localStorage via the `Token` class:

```typescript
// lib/token.ts
class Token {
  getToken(key: string): string | null
  setToken(key: string, token: string): void
  removeToken(key: string): void
}
```

---

## Routing Architecture

### Route Configuration

Routes are defined declaratively in `lib/routerMeta.ts`:

```typescript
interface IRouterMeta {
  id: string;
  path: string;
  element: ReactNode;
  label: string;
  isShow: boolean;          // Show in navigation
  isAuth: boolean;          // Requires authentication
  isProtected: boolean;     // Protected route wrapper
}
```

### Protected Routes

The `ProtectedRoute` HOC guards authenticated routes:

```typescript
// components/HOC/ProtectedRoute.tsx
function ProtectedRoute({ children }) {
  // Redirects to login if not authenticated
}
```

### Route Features

- **Lazy Loading**: Pages are lazily loaded for code splitting
- **Suspense**: Loading fallback during chunk loading
- **Error Boundaries**: Per-route error handling
- **Nested Routes**: Layout component with `<Outlet />`

---

## API Integration

### API Client Configuration

```typescript
// repositories/apiClient.ts
const apiClient = axios.create({
  baseURL: 'https://api.realworld.io/api',
});

// Request interceptor: Adds Authorization header
apiClient.interceptors.request.use((config) => {
  const token = Token.getToken(JWT_TOKEN_KEY);
  if (token) {
    config.headers.Authorization = `Token ${token}`;
  }
  return config;
});

// Response interceptor: Error handling (429 status)
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 429) {
      Token.removeToken(JWT_TOKEN_KEY);
      window.location.reload();
    }
    return Promise.reject(error);
  }
);
```

### Repository Pattern

Each domain has its own repository:

```typescript
// repositories/articles/articlesRepository.ts
export const getArticles = (params: GetArticlesParams) =>
  apiClient.get('/articles', { params });

export const createArticle = (data: CreateArticleParams) =>
  apiClient.post('/articles', { article: data });
```

### API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/users/login` | POST | User login |
| `/users` | POST | User registration |
| `/user` | GET/PUT | Current user |
| `/articles` | GET/POST | Articles list/create |
| `/articles/:slug` | GET/PUT/DELETE | Single article |
| `/articles/:slug/comments` | GET/POST | Article comments |
| `/articles/:slug/favorite` | POST/DELETE | Favorite article |
| `/profiles/:username` | GET | User profile |
| `/profiles/:username/follow` | POST/DELETE | Follow user |
| `/tags` | GET | Available tags |

---

## Component Architecture

### Component Types

**1. Page Components** (`pages/`)
- Route entry points
- Handle page-level logic
- Compose feature components

**2. Feature Components** (`components/article/`, `components/feed/`)
- Domain-specific functionality
- May use queries directly
- Contain business logic

**3. Common Components** (`components/common/`)
- Shared across features
- Pure presentational
- Highly reusable

**4. HOC Components** (`components/HOC/`)
- Cross-cutting concerns
- Route protection
- Layout wrappers

### Component Conventions

```typescript
// Functional components with TypeScript
interface FeedProps {
  article: IArticle;
}

function Feed({ article }: FeedProps) {
  // Component logic
}
```

---

## Error Handling

### Error Boundary

```typescript
// components/ErrorFallback.tsx
function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div>
      <p>Something went wrong</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}
```

### API Error Handling

- **429 Status**: Token cleared, page reloaded
- **Form Errors**: Displayed inline with form fields
- **Network Errors**: Caught by error boundary

---

## Performance Optimizations

### Code Splitting

```typescript
// Router.tsx
const HomePage = lazy(() => import('@/pages/HomePage'));
const ArticlePage = lazy(() => import('@/pages/ArticlePage'));
```

### Query Caching

- 20-second stale time prevents unnecessary refetches
- Manual invalidation after mutations
- Suspense integration for seamless loading

### Suspense Boundaries

```typescript
<Suspense fallback={<LoadingFallback />}>
  <Routes>{/* ... */}</Routes>
</Suspense>
```

---

## Coding Standards

### TypeScript

- Strict mode enabled
- All props and state typed
- Parameter interfaces for API calls

### ESLint Rules

- Airbnb style guide
- React hooks rules
- Import ordering

### Prettier Configuration

```json
{
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 120
}
```

### Import Aliases

```typescript
// Use @ alias for src imports
import { useGetArticlesQueries } from '@/queries/articles.query';
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `FeedList.tsx` |
| Hooks | camelCase with `use` prefix | `useInputs.ts` |
| Queries | camelCase with `Query/Mutation` suffix | `useGetArticleQuery` |
| Repositories | camelCase | `articlesRepository.ts` |
| Constants | SCREAMING_SNAKE_CASE | `QUERY_USER_KEY` |
| Interfaces | PascalCase with `I` prefix | `IArticle` |

---

## Diagrams

### Data Flow

```
User Action → Component → Query Hook → Repository → API
                ↓
            UI Update ← Cache Update ← Response
```

### Authentication Flow

```
Login Form → POST /users/login → Store Token → Set Context
                                      ↓
                              Axios Interceptor → Auto-attach to requests
```

### Route Protection Flow

```
Navigate to Protected Route
        ↓
   ProtectedRoute HOC
        ↓
   Check isLogin Context
        ↓
    ┌───┴───┐
    │       │
  True    False
    │       │
 Render   Redirect
  Page    to Login
```
