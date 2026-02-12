# ARCHITECTURE.md

## Overview

**Type:** web-app
**Pattern:** Layered architecture with Repository pattern and React Query for state management

## Critical Rules

**Repositories only handle:**
- HTTP requests via Axios
- Request/response transformation
- API endpoint construction

**Queries only handle:**
- React Query hook definitions
- Cache configuration (staleTime, etc.)
- Query key management

**Components only handle:**
- UI rendering
- User interaction
- Calling query/mutation hooks

**Never put API calls in components or business logic in repositories.**

## Layers

| Layer | Location | Responsibility |
|-------|----------|----------------|
| Pages | `src/pages/` | Top-level route components |
| Components | `src/components/` | Reusable UI elements |
| Queries | `src/queries/` | React Query hooks (useQuery, useMutation) |
| Repositories | `src/repositories/` | HTTP API calls via Axios |
| Contexts | `src/contexts/` | Global state (user auth context) |
| Utils | `src/lib/` | Helpers, custom hooks, route config |

## Data Flow

```
User Action → Component → React Query Hook → Repository → API
                 ↑                                         ↓
                 └─────── Cache Update ←─────────────────┘
```

### Example: Fetching an Article

```
ArticlePage.tsx
  → useGetArticleQueries(slug)
    → queries/articles.query.ts
      → repositories/articlesRepository.getArticle({ slug })
        → apiClient.get('/articles/:slug')
          → RealWorld API
```

### Example: Creating a Comment

```
Comment.tsx
  → useCreateCommentMutation()
    → mutate({ slug, body })
      → repositories/articlesRepository.createComment({ slug, body })
        → apiClient.post('/articles/:slug/comments', { comment: { body } })
```

## Key Patterns

### Repository Pattern

All API calls abstracted into repository functions:

```typescript
// repositories/articles/articlesRepository.ts
export const getArticles = async ({ isGlobal, selectedTag, page }: Params) => {
  return await apiClient({
    method: 'get',
    url: `/articles${isGlobal ? '' : '/feed'}?limit=10&offset=${page * 10}...`,
  });
};
```

### React Query Integration

Repositories wrapped in React Query hooks:

```typescript
// queries/articles.query.ts
export const useGetArticlesQueries = (isGlobal: boolean, page: number, selectedTag: string) => {
  return useQueries({
    queries: [
      {
        queryKey: [QUERY_ARTICLES_KEY, isGlobal, selectedTag, page],
        queryFn: () => getArticles({ isGlobal, selectedTag, page }).then(res => res.data),
        staleTime: 20000,
      },
    ],
  });
};
```

### Axios Interceptors

JWT token automatically injected into requests:

```typescript
// repositories/apiClient.ts
apiClient.interceptors.request.use((request) => {
  const jwtToken = token.getToken(ACCESS_TOKEN_KEY);
  if (jwtToken) {
    request.headers['Authorization'] = `Token ${jwtToken}`;
  }
  return request;
});
```

### Dynamic Routing

Routes defined in metadata and dynamically loaded:

```typescript
// lib/routerMeta.ts
const routerMeta = {
  HomePage: { path: '/', isShow: true, isCommon: true },
  ArticlePage: { path: '/article/:slug', isShow: false },
  // ...
};

// Router.tsx - lazy loads pages dynamically
const lazyImport = (pageName: string) => lazy(() => import(`@/pages/${pageName}`));
```

## Database

**N/A** - This is a frontend-only application. Data stored in external RealWorld API.

## External Services

| Service | Purpose | Config |
|---------|---------|--------|
| RealWorld API | Backend for articles, users, auth | `https://api.realworld.io/api` |

## Authentication Flow

```
1. User submits login form
   ↓
2. postLogin({ email, password }) → API
   ↓
3. API returns JWT token
   ↓
4. Token stored in localStorage via token.ts
   ↓
5. Axios interceptor adds token to all requests
   ↓
6. Protected routes check token presence
```

## State Management

| State Type | Solution |
|------------|----------|
| Server state | TanStack Query (React Query) |
| Auth state | React Context + localStorage |
| Form state | Local component state (useState) + custom useInputs hook |
| Route state | React Router v6 |

## Error Handling

- **Error Boundaries** - Catch rendering errors per route
- **React Query** - Automatic retry and error states
- **Axios Interceptors** - Log errors, handle 429 rate limits
