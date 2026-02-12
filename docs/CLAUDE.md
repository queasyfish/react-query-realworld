# CLAUDE.md

## Project

**Name:** react-query-realworld
**Stack:** TypeScript / React 18 / TanStack Query v4
**Description:** RealWorld Conduit implementation demonstrating React Query best practices with authentication, CRUD operations, and advanced caching patterns.

## Quick Start

```bash
npm install
npm start          # http://localhost:3000
npm test          # Run Jest tests
npm run build     # Production build
```

## Structure

```
src/
├── pages/            # Route-level components
├── components/       # Reusable UI components
│   ├── article/      # Article-specific components
│   ├── feed/         # Feed components
│   ├── header/       # Navigation components
│   ├── profile/      # Profile components
│   ├── common/       # Shared layout components
│   └── HOC/          # Higher-order components
├── queries/          # React Query hooks (useQuery, useMutation)
├── repositories/     # API client functions (Axios)
│   ├── articles/     # Article CRUD operations
│   ├── profiles/     # Profile operations
│   ├── tags/         # Tag operations
│   ├── users/        # User/auth operations
│   └── apiClient.ts  # Axios instance with interceptors
├── contexts/         # Context API (UserContext)
├── lib/              # Utilities, hooks, types
│   ├── hooks/        # Custom hooks (useInputs, useIsLoginContext)
│   ├── utils/        # Helper functions
│   ├── routerMeta.ts # Route configuration with metadata
│   └── token.ts      # localStorage JWT manager
├── constants/        # Query keys, units, token constants
├── interfaces/       # TypeScript types
├── Router.tsx        # Dynamic router with suspense/error boundaries
└── App.tsx           # Root component with QueryClientProvider
```

## Key Files

| File | Purpose |
|------|---------|
| `src/Router.tsx` | Dynamic route assignment with lazy loading |
| `src/lib/routerMeta.ts` | Route metadata (auth, navigation visibility) |
| `src/queries/queryClient.ts` | React Query config (suspense mode) |
| `src/repositories/apiClient.ts` | Axios instance with JWT interceptor |
| `src/contexts/UserContextProvider.tsx` | Global auth state |
| `src/queries/articles.query.ts` | Article query/mutation hooks |

## Commands

| Command | Purpose |
|---------|---------|
| `npm start` | Start dev server |
| `npm test` | Run Jest tests |
| `npm run build` | Production build |
| `npm run eject` | Eject CRA config (not recommended) |

## Rules

**Do:**
- Use React Query for all server state (never useState for API data)
- Put API calls in `repositories/`, never call axios directly from components
- Use `useQuery` for GET, `useMutation` for POST/PUT/DELETE
- Define query keys in `constants/query.constant.ts`
- Use Suspense boundaries for loading states
- Leverage optimistic updates for better UX
- Follow repository pattern: Component → Query Hook → Repository → API

**Don't:**
- Call axios directly from components or pages
- Mix server state (React Query) with local state (useState)
- Skip query keys - they're critical for caching
- Put business logic in components
- Forget to invalidate queries after mutations
- Use `any` type - this project uses strict TypeScript

## Patterns

### Repository Pattern
```typescript
// Location: src/repositories/articles/articlesRepository.ts
export const getArticles = async ({ isGlobal, selectedTag, page }: getArticlesParam) => {
  const offset = UNIT_PER_PAGE * (page - 1);
  return await apiClient({
    method: 'get',
    url: `/articles?limit=${UNIT_PER_PAGE}&offset=${offset}`,
  });
};
```

### Query Hooks
```typescript
// Location: src/queries/articles.query.ts
export const useGetArticlesQueries = (isGlobal: boolean, page: number, selectedTag: string) => {
  return useQueries({
    queries: [
      {
        queryKey: [QUERY_ARTICLES_KEY, isGlobal, selectedTag, page],
        queryFn: () => getArticles({ isGlobal, selectedTag, page }).then((res) => res.data),
        staleTime: 20000,
      },
    ],
  });
};
```

### Protected Routes
```typescript
// Location: src/components/HOC/ProtectedRoute.tsx
// Routes with routeMeta.isAuth = true redirect to /login if not authenticated
const ProtectedRoute = ({ path, children }: IProtectedRoute) => {
  const { isLogin } = useContext(UserContext);
  const routeMeta = routerMeta[path];
  if (routeMeta.isAuth && !isLogin) {
    return <Navigate to="/login" replace />;
  }
  return <>{children}</>;
};
```

### Dynamic Router with Metadata
```typescript
// Location: src/lib/routerMeta.ts
// Define routes once with auth requirements, navigation visibility
const routerMeta: RouterMetaType = {
  SettingPage: {
    name: 'Setting',
    path: '/settings',
    isShow: true,      // Show in navigation
    isAuth: true,      // Requires authentication
    icon: 'ion-gear-a',
  },
};
```

## Environment Variables

This project uses the public RealWorld API and requires no environment variables.

| Variable | Required | Description |
|----------|----------|-------------|
| None | - | API endpoint hardcoded to `https://api.realworld.io/api` |

## Authentication

- JWT token stored in localStorage via `src/lib/token.ts`
- Axios interceptor automatically attaches `Authorization: Token {jwt}` header
- 429 rate limit errors trigger automatic token removal and page reload
- UserContext provides global `isLogin` state
- Protected routes redirect to `/login` when unauthenticated

## React Query Configuration

```typescript
// Location: src/queries/queryClient.ts
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      suspense: true,  // Use React 18 Suspense
      retry: false,    // Don't retry failed queries
    },
  },
});
```

## Related Docs

- `ARCHITECTURE.md` - Layered architecture, data flow patterns
- `FEATURES.md` - Feature inventory with keys for Confluence sync
- `TECHNICAL_DEBT.md` - Known issues and improvement opportunities
