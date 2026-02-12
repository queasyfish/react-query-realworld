# CLAUDE.md

## Project

**Name:** React Query RealWorld
**Stack:** TypeScript / React 18 / TanStack Query v4 / Axios
**Description:** RealWorld Conduit implementation (Medium clone) using React Query for server state management with optimistic updates and caching.

## Quick Start

```bash
npm install
npm start          # http://localhost:3000
npm test
npm run build
```

## Structure

```
src/
├── pages/              # Route-level components
├── components/         # Reusable UI components
│   ├── article/       # Article-specific (comments, buttons)
│   ├── feed/          # Article feed with pagination
│   ├── header/        # Navigation
│   ├── profile/       # Profile components
│   └── common/        # Layout, footer
├── queries/            # React Query hooks (useQuery, useMutation)
├── repositories/       # API client functions (axios)
├── contexts/           # User authentication context
├── lib/               # Utilities, hooks, types
│   ├── routerMeta.ts  # Route configuration with metadata
│   ├── token.ts       # localStorage JWT management
│   └── hooks/         # Custom hooks (useInputs, useIsLoginContext)
├── constants/          # Query keys, constants
└── Router.tsx         # Dynamic route assignment
```

## Key Files

| File | Purpose |
|------|---------|
| `src/Router.tsx` | Dynamic router with Suspense/ErrorBoundary |
| `src/lib/routerMeta.ts` | Route metadata (auth, visibility) |
| `src/queries/queryClient.ts` | React Query config (suspense mode) |
| `src/repositories/apiClient.ts` | Axios instance with JWT interceptor |
| `src/contexts/UserContextProvider.tsx` | Global auth state |

## Commands

| Command | Purpose |
|---------|---------|
| `npm start` | Start dev server (port 3000) |
| `npm test` | Run Jest tests |
| `npm run build` | Production build |
| `npm run eject` | Eject from CRA |

## Rules

**Do:**
- Use `useQuery` for GET operations
- Use `useMutation` for POST/PUT/DELETE operations
- Put all API calls in `repositories/`
- Use React Query cache for server state (not useState)
- Use Suspense for loading states
- Invalidate queries after mutations for cache updates

**Don't:**
- Call axios directly from components
- Use useState for server data
- Skip the repository layer
- Put business logic in components
- Use `any` type in TypeScript

## Patterns

### Repository Function

```typescript
// Location: src/repositories/articles/articlesRepository.ts
export const getArticles = async ({ isGlobal, selectedTag, page }: getArticlesParam) => {
  return await apiClient({
    method: 'get',
    url: `/articles?limit=${UNIT_PER_PAGE}&offset=${UNIT_PER_PAGE * (page - 1)}`,
  });
};
```

### React Query Hook

```typescript
// Location: src/queries/articles.query.ts
export const useGetArticlesQueries = (isGlobal: boolean, page: number, selectedTag: string) => {
  return useQueries({
    queries: [{
      queryKey: [QUERY_ARTICLES_KEY, isGlobal, selectedTag, page],
      queryFn: () => getArticles({ isGlobal, selectedTag, page }).then(res => res.data),
      staleTime: 20000,
    }],
  });
};
```

### Protected Route

```typescript
// Location: src/components/HOC/ProtectedRoute.tsx
// Redirect to login if route requires auth and user is not logged in
if (routeMeta.isAuth && !isLogin) {
  return <Navigate to="/login" replace />;
}
```

### Axios Interceptor

```typescript
// Location: src/repositories/apiClient.ts
// Automatically attach JWT token to all requests
apiClient.interceptors.request.use((request) => {
  const jwtToken = token.getToken(ACCESS_TOKEN_KEY);
  if (jwtToken) {
    request.headers['Authorization'] = `Token ${jwtToken}`;
  }
  return request;
});
```

## Environment Variables

None required. API endpoint is hardcoded to `https://api.realworld.io/api`.

## Related Docs

- `ARCHITECTURE.md` - Detailed architecture and patterns
- `FEATURES.md` - Feature breakdown with status
