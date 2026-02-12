# ARCHITECTURE.md

## Overview

**Type:** web-app
**Pattern:** Repository + React Query (Server State Management)
**Description:** RealWorld Conduit implementation using React 18, TanStack Query v4, and TypeScript. Follows a layered architecture with clear separation between UI, data fetching, and API communication.

## Critical Rules

**Queries only fetch data, Mutations only modify data.**
- Use `useQuery` for GET operations
- Use `useMutation` for POST/PUT/DELETE operations

**Repository layer handles all API calls.**
- Never call `axios` directly from components
- All API calls go through `repositories/`

**React Query manages server state.**
- Don't use useState for server data
- Use query keys for cache management
- Leverage suspense mode for loading states

**Never put business logic in components.**
- Components render UI and handle user interaction
- Queries/mutations handle data operations
- Repositories handle HTTP communication

## Layers

| Layer | Location | Responsibility |
|-------|----------|----------------|
| Pages | `src/pages/` | Route-level components, compose features |
| Components | `src/components/` | Reusable UI components |
| Queries | `src/queries/` | React Query hooks (useQuery, useMutation) |
| Repositories | `src/repositories/` | API client functions (axios calls) |
| Contexts | `src/contexts/` | Global state (user authentication) |
| Router | `src/Router.tsx` | Route configuration with metadata |
| Lib | `src/lib/` | Utilities, hooks, types |

## Data Flow

```
User Action → Component → Query/Mutation Hook → Repository → API
                                    ↓
                            React Query Cache
                                    ↓
                            Component Re-render
```

**Example: Favoriting an Article**
```
1. User clicks favorite button
2. Component calls useFavoriteArticleMutation()
3. Mutation calls favoriteArticle() from repository
4. Repository makes POST to /articles/{slug}/favorite
5. React Query invalidates article cache
6. Component re-renders with updated favorite count
```

## Key Patterns

### Repository Pattern

```typescript
// Location: src/repositories/articles/articlesRepository.ts
export const getArticles = async ({ isGlobal, selectedTag, page }: getArticlesParam) => {
  return await apiClient({
    method: 'get',
    url: `/articles?limit=${UNIT_PER_PAGE}&offset=${UNIT_PER_PAGE * (page - 1)}`,
  });
};
```

**Purpose:** Centralize all API calls, make them reusable and testable.

### React Query Hooks

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

export const useFavoriteArticleMutation = () => useMutation(favoriteArticle);
```

**Purpose:** Wrap repository calls with React Query for caching, loading states, and error handling.

### Dynamic Router with Metadata

```typescript
// Location: src/lib/routerMeta.ts
const routerMeta: RouterMetaType = {
  HomePage: {
    name: 'Home',
    path: '/',
    isShow: true,
    isCommon: true,
  },
  SettingPage: {
    name: 'Setting',
    path: '/settings',
    isShow: true,
    isAuth: true, // Requires authentication
  },
};

// Location: src/Router.tsx
const assignRouter = Object.keys(routerMeta).map((componentKey: string) => {
  const props: IRouterMeta = routerMeta[componentKey];
  return {
    Component: lazyImport(componentKey),
    props,
  };
});
```

**Purpose:** Configure routes with metadata (auth requirements, navigation visibility) in one place.

### Protected Routes

```typescript
// Location: src/components/HOC/ProtectedRoute.tsx
const ProtectedRoute = ({ path, children }: IProtectedRoute) => {
  const { isLogin } = useContext(UserContext);
  const routeMeta = routerMeta[path];

  if (routeMeta.isAuth && !isLogin) {
    return <Navigate to="/login" replace />;
  }
  return <>{children}</>;
};
```

**Purpose:** Restrict access to authenticated routes, redirect to login if needed.

### Axios Interceptors

```typescript
// Location: src/repositories/apiClient.ts
apiClient.interceptors.request.use((request) => {
  const jwtToken = token.getToken(ACCESS_TOKEN_KEY);
  if (jwtToken) {
    request.headers['Authorization'] = `Token ${jwtToken}`;
  }
  return request;
});

apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response.status === 429) {
      token.removeToken('ACCESS_TOKEN_KEY');
      window.location.reload();
    }
    return Promise.reject(error);
  }
);
```

**Purpose:** Automatically attach JWT token to requests, handle 429 rate limits by forcing re-login.

### Suspense + Error Boundaries

```typescript
// Location: src/Router.tsx
<ErrorBoundary onReset={reset} fallbackRender={({ resetErrorBoundary }) => (
  <ErrorFallback resetErrorBoundary={resetErrorBoundary} />
)}>
  <Suspense fallback={<LoadingFallback />}>
    <Component />
  </Suspense>
</ErrorBoundary>
```

**Purpose:** Handle loading and error states declaratively with React 18 features.

## External Services

| Service | Purpose | Config |
|---------|---------|--------|
| RealWorld API | Backend for articles, users, comments | `https://api.realworld.io/api` |

## React Query Configuration

**Location:** `src/queries/queryClient.ts`

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      suspense: true,  // Use Suspense for loading states
      retry: false,    // Don't retry failed queries
    },
  },
});
```

## Authentication Flow

```
1. User submits login form (SignInPage)
   ↓
2. POST /users/login via usersRepository
   ↓
3. Store JWT token in localStorage (token.ts)
   ↓
4. Update UserContext with isLogin state
   ↓
5. Axios interceptor attaches token to all requests
   ↓
6. Protected routes now accessible
```

## State Management

| State Type | Solution | Location |
|------------|----------|----------|
| Server state | React Query | `src/queries/` |
| Auth state | Context API | `src/contexts/UserContextProvider.tsx` |
| Local UI state | useState | Component level |
| Form state | Custom hook | `src/lib/hooks/useInputs.tsx` |

## Component Organization

```
src/components/
├── article/          # Article-specific components
│   ├── ButtonSelector.tsx
│   ├── ButtonsWIthAccess.tsx  # Author actions (edit/delete)
│   ├── ButtonsWIthoutAccess.tsx  # Reader actions (favorite/follow)
│   └── Comment.tsx
├── feed/             # Article feed components
│   ├── Feed.tsx
│   └── FeedList.tsx
├── header/           # Navigation components
│   ├── Header.tsx
│   ├── NavItem.tsx
│   └── ProfileItem.tsx
├── profile/          # Profile-specific components
│   └── FollowButton.tsx
├── common/           # Shared layout components
│   ├── Footer.tsx
│   └── Layout.tsx
└── HOC/              # Higher-order components
    └── ProtectedRoute.tsx
```

## Type Safety

**Location:** `src/interfaces/main.d.ts`

All API response types and component props are defined with TypeScript interfaces. Repository param types are defined alongside repository functions.

**Example:**
```typescript
// src/repositories/articles/articlesRepository.param.ts
export type getArticlesParam = {
  isGlobal?: boolean;
  selectedTag?: string;
  page: number;
  username?: string;
  isFavorited?: boolean;
};
```
