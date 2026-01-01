# CLAUDE.md

This file provides guidance for AI agents (Claude, Copilot, etc.) working on this codebase.

## Project Overview

This is a **React Query RealWorld** application - a Medium-clone social blogging platform built with React 18, TypeScript, and TanStack Query. It implements the [RealWorld](https://github.com/gothinkster/realworld) specification.

## Quick Start

```bash
# Install dependencies
npm install

# Start development server
npm start

# Run tests
npm test

# Production build
npm run build
```

## Key Technologies

- **React 18** with functional components and hooks
- **TanStack Query (React Query) 4** for server state management
- **React Router 6** for routing
- **Axios** for HTTP requests
- **TypeScript** with strict mode
- **Craco** for CRA customization

## Architecture Overview

```
src/
├── pages/          # Route-level components
├── components/     # Reusable UI components
├── queries/        # React Query hooks
├── repositories/   # API layer (Axios calls)
├── contexts/       # React Context providers
├── lib/            # Utilities and custom hooks
├── constants/      # Application constants
└── interfaces/     # TypeScript type definitions
```

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed architecture documentation.

## Features & Capabilities

### Authentication
- JWT token-based auth stored in localStorage
- Sign up / Sign in pages
- Protected routes via `ProtectedRoute` HOC
- User context for auth state (`UserContextProvider`)

### Articles
- CRUD operations (create, read, update, delete)
- Markdown rendering with `react-markdown`
- Tag-based filtering
- Pagination
- Favorite/unfavorite functionality

### Comments
- Create and delete comments on articles
- Display comment list with author info

### User Profiles
- View user profiles
- Follow/unfollow users
- View user's articles and favorited articles
- Edit profile settings

### Feeds
- Global feed (all articles)
- User feed (followed authors)
- Tag-filtered feeds

## Code Patterns & Standards

### Adding a New Query

```typescript
// 1. Define repository function in repositories/
export const getNewData = (params: GetNewDataParams) =>
  apiClient.get('/endpoint', { params });

// 2. Create query hook in queries/
export const useGetNewDataQuery = (params: GetNewDataParams) => {
  return useQuery({
    queryKey: [QUERY_KEY, params],
    queryFn: () => getNewData(params).then(res => res.data),
    staleTime: 1000 * 20,
  });
};
```

### Adding a New Mutation

```typescript
export const useCreateNewDataMutation = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: CreateNewDataParams) => createNewData(data),
    onSuccess: () => {
      queryClient.invalidateQueries([QUERY_KEY]);
    },
  });
};
```

### Adding a New Page

1. Create page component in `src/pages/`
2. Add route metadata in `src/lib/routerMeta.ts`
3. Add lazy import in `src/Router.tsx`
4. Route is automatically rendered

### Adding a New Component

```typescript
// src/components/feature/NewComponent.tsx
interface NewComponentProps {
  data: IDataType;
  onAction: () => void;
}

function NewComponent({ data, onAction }: NewComponentProps) {
  return (
    // JSX
  );
}

export default NewComponent;
```

## Import Conventions

Use the `@/` alias for all src imports:

```typescript
// Good
import { useGetArticleQuery } from '@/queries/articles.query';
import Feed from '@/components/feed/Feed';

// Avoid
import { useGetArticleQuery } from '../../queries/articles.query';
```

## Query Keys

Always use constants from `constants/query.constant.ts`:

```typescript
import { QUERY_ARTICLES_KEY } from '@/constants/query.constant';

useQuery({
  queryKey: [QUERY_ARTICLES_KEY, params],
  // ...
});
```

## API Integration

### Base URL
`https://api.realworld.io/api`

### Authentication
Token is automatically attached via Axios interceptor when present in localStorage.

### Error Handling
- 429 errors trigger token clear and page reload
- Other errors bubble to error boundary

## Testing

Tests are located in `__tests__/` directory. Run with:

```bash
npm test
```

Currently tests cover utility functions. When adding tests:

```typescript
// __tests__/NewTest.ts
import { myFunction } from '@/lib/utils/myFunction';

describe('myFunction', () => {
  it('should do something', () => {
    expect(myFunction(input)).toBe(expected);
  });
});
```

## Styling

This project uses Bootstrap-based CSS classes matching the RealWorld spec. Classes include:
- `container`, `row`, `col-md-*` for layout
- `btn`, `btn-primary`, `btn-outline-*` for buttons
- `form-control` for inputs

## TypeScript Types

Domain types are defined in `src/interfaces/main.d.ts`:

```typescript
interface IArticle {
  slug: string;
  title: string;
  description: string;
  body: string;
  tagList: string[];
  createdAt: string;
  updatedAt: string;
  favorited: boolean;
  favoritesCount: number;
  author: IAuthor;
}
```

## Common Tasks

### Add authentication to a route

In `src/lib/routerMeta.ts`, set:
```typescript
{
  isAuth: true,       // Only show in nav when logged in
  isProtected: true,  // Wrap with ProtectedRoute
}
```

### Invalidate cache after mutation

```typescript
const queryClient = useQueryClient();
queryClient.invalidateQueries([QUERY_KEY]);
```

### Access current user

```typescript
const { data: user } = useGetUserQuery();
```

### Check login status

```typescript
const { isLogin } = useContext(UserContext);
// or
const { isLogin } = useIsLoginContext();
```

## File Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Page | `*Page.tsx` | `ArticlePage.tsx` |
| Component | PascalCase | `FeedList.tsx` |
| Query hook | `*.query.ts` | `articles.query.ts` |
| Repository | `*Repository.ts` | `articlesRepository.ts` |
| Constants | `*.constant.ts` | `query.constant.ts` |
| Utility | camelCase | `convertToDate.ts` |
| Hook | `use*.ts(x)` | `useInputs.tsx` |

## Do's and Don'ts

### Do
- Use React Query for all server state
- Use TypeScript interfaces for all props
- Use the `@/` import alias
- Add query keys to constants file
- Use existing patterns when adding features
- Keep components focused and reusable

### Don't
- Don't store server data in React state (use React Query)
- Don't make API calls directly in components (use queries)
- Don't skip TypeScript types
- Don't hardcode API URLs (use apiClient)
- Don't add inline styles (use CSS classes)

## Debugging

### React Query DevTools
Install and add to App.tsx for query debugging:
```typescript
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
```

### Common Issues

**Token not being sent**: Check `apiClient.ts` interceptor and localStorage

**Stale data**: Check query invalidation after mutations

**Route not rendering**: Verify `routerMeta.ts` configuration

## Contributing Checklist

- [ ] TypeScript types defined for new interfaces
- [ ] Query keys added to constants
- [ ] Repository functions for new endpoints
- [ ] Query hooks wrapping repository calls
- [ ] Components receive data via props
- [ ] Error states handled
- [ ] Loading states handled (Suspense)
- [ ] ESLint/Prettier passing
