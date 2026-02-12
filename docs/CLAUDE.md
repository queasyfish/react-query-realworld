# CLAUDE.md

## Project

**Name:** React Query RealWorld
**Stack:** TypeScript / React / TanStack Query (React Query) / Axios
**Description:** Medium.com clone (Conduit) demonstrating React Query patterns for CRUD, auth, and data fetching.

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
├── pages/            # Page components (Home, Article, Profile, etc.)
├── components/       # Reusable UI components
│   ├── article/      # Article-specific components
│   ├── feed/         # Article feed components
│   ├── header/       # Navigation header
│   ├── profile/      # Profile components
│   └── common/       # Shared components (Layout, Footer)
├── queries/          # React Query hooks
├── repositories/     # API client functions
│   ├── apiClient.ts  # Axios instance with interceptors
│   ├── articles/     # Article API calls
│   ├── profiles/     # Profile API calls
│   ├── users/        # User/auth API calls
│   └── tags/         # Tags API calls
├── contexts/         # React Context providers
├── lib/              # Utilities and custom hooks
│   ├── routerMeta.ts # Route configuration
│   ├── token.ts      # localStorage wrapper
│   ├── hooks/        # Custom hooks (useInputs, useIsLoginContext)
│   └── utils/        # Helper functions
├── constants/        # App constants
└── interfaces/       # TypeScript type definitions
```

## Key Files

| File | Purpose |
|------|---------|
| `src/App.tsx` | Root component with UserContextProvider |
| `src/Router.tsx` | Dynamic routing with lazy loading and error boundaries |
| `src/repositories/apiClient.ts` | Axios instance with JWT auth interceptor |
| `src/queries/articles.query.ts` | React Query hooks for articles |
| `src/lib/routerMeta.ts` | Centralized route metadata |

## Commands

| Command | Purpose |
|---------|---------|
| `npm start` | Start dev server (port 3000) |
| `npm test` | Run Jest tests |
| `npm run build` | Production build |
| `npm run eject` | Eject from Create React App |

## Rules

**Do:**
- Keep API calls in `repositories/` directory
- Wrap API calls with React Query hooks in `queries/`
- Use path aliases (`@/` points to `src/`)
- Store JWT in localStorage via `token.ts` utility
- Use lazy loading for page components

**Don't:**
- Call API directly from components
- Store auth token manually (use `lib/token.ts`)
- Mix data fetching logic in components
- Bypass the axios interceptor for auth

## Patterns

### React Query Mutation

```typescript
// queries/articles.query.ts
export const useCreateArticleMutation = () => useMutation(createArticle);

// In component
const { mutate } = useCreateArticleMutation();
mutate({ title, description, body, tagList });
```

### Repository Pattern

```typescript
// repositories/articles/articlesRepository.ts
export const getArticle = async ({ slug }: getArticleParam) => {
  return await apiClient({
    method: 'get',
    url: `/articles/${slug}`,
  });
};
```

### Route Protection

```typescript
// ProtectedRoute HOC checks authentication
<ProtectedRoute path={props.path}>
  <Component />
</ProtectedRoute>
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `NODE_ENV` | No | Environment mode (development/production) |

**Note:** API endpoint hardcoded to `https://api.realworld.io/api` in `apiClient.ts`.

## Related Docs

- `ARCHITECTURE.md` - Layer separation and data flow
- `FEATURES.md` - Feature breakdown and status
