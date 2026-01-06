# Technical Debt Report

> **Generated:** 2026-01-06
> **Codebase:** React Query RealWorld
> **Total Issues Identified:** 31

This document catalogues technical debt and inconsistencies identified during architecture analysis. Issues are categorized by severity and area of concern.

---

## Executive Summary

| Area | Issues | Severity Distribution |
|------|--------|----------------------|
| Components | 10 | 2 High, 7 Medium, 1 Low |
| Query Hooks | 2 | 2 Medium |
| Repository Layer | 3 | 3 Medium |
| Context Management | 2 | 2 Medium |
| Query Invalidation | 3 | 3 Medium |
| Error Handling | 3 | 1 High, 2 Medium |
| Routing | 2 | 2 Medium |
| Configuration | 2 | 2 Low |
| Type Safety | 2 | 2 Medium |
| Missing Patterns | 2 | 2 Medium |

**Priority Breakdown:**
- **High Severity:** 3 issues (require immediate attention)
- **Medium Severity:** 24 issues (should be addressed)
- **Low Severity:** 4 issues (nice to fix)

---

## High Severity Issues

### 1. Misspelled Component Filenames

**Location:**
- `src/components/article/ButtonsWIthAccess.tsx`
- `src/components/article/ButtonsWIthoutAccess.tsx`

**Problem:** "WIth" should be "With" - spelling error propagated to filenames and interface names (`IButtonsWIthAccessProps`, `IButtonsWIthoutAccessProps`).

**Impact:**
- Creates confusion when reading/importing code
- Violates naming conventions
- Difficult to search and refactor

**Fix:** Rename files and interfaces to use correct spelling.

---

### 2. Hardcoded Links Using `<a>` Instead of `<Link>`

**Locations:**
- `src/components/article/Comment.tsx:72,76` - Author links use `href="/"`
- `src/components/feed/Feed.tsx:56` - Uses template filename `href="profile.html"`

**Code Example (Comment.tsx):**
```typescript
<a href="/" className="comment-author">
  <img src={comment.author.image} className="comment-author-img" alt="comment-author" />
</a>
```

**Problem:**
- Links don't navigate to actual profile pages
- Breaks SPA navigation (causes full page refresh)
- Loses router history state

**Fix:** Replace with React Router `<Link>` components:
```typescript
<Link to={`/profile/${comment.author.username}`} className="comment-author">
```

---

### 3. Unsafe Error Object Access

**Locations:**
- `src/pages/SignInPage.tsx:32-35`
- `src/pages/SignUpPage.tsx:30-34`

**Code Example:**
```typescript
.catch((err) => {
  setError({
    email: err.response.data.errors.email,
    password: err.response.data.errors.password,
    emailOrPassword: err.response.data.errors['email or password'],
  });
});
```

**Problem:** Accesses nested error properties without null/undefined checks. Will throw if:
- Network error (no `err.response`)
- Unexpected error format (no `errors` object)
- Missing specific error fields

**Fix:** Use optional chaining and fallbacks:
```typescript
.catch((err) => {
  const errors = err?.response?.data?.errors || {};
  setError({
    email: errors.email || '',
    password: errors.password || '',
    emailOrPassword: errors['email or password'] || '',
  });
});
```

---

## Medium Severity Issues

### Query Hooks

#### 4. Missing Default Mutation Handlers

**Location:** `src/queries/articles.query.ts:51-63`, `src/queries/user.query.ts:12`

**Problem:** All mutation hooks lack default `onSuccess`/`onError` handlers:
- `useCreateArticleMutation()`
- `useUpdateArticleMutation()`
- `useDeleteArticleMutation()`
- `useCreateCommentMutation()`
- `useDeleteCommentMutation()`
- `useFavoriteArticleMutation()`
- `useUnfavoriteArticleMutation()`
- `usePutUserMutation()`

**Impact:** Query invalidation scattered across 8+ component files instead of centralized in hooks.

**Fix:** Add default handlers with invalidation:
```typescript
export const useCreateArticleMutation = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: createArticle,
    onSuccess: () => {
      queryClient.invalidateQueries([QUERY_ARTICLES_KEY]);
    },
  });
};
```

---

### Repository Layer

#### 5. Inconsistent Parameter Handling in `putUser`

**Location:** `src/repositories/users/usersRepository.ts:38-44`

**Problem:** `putUser` receives pre-wrapped `{ user: putUserParam }` while other endpoints unwrap parameters internally.

**Expected Pattern:**
```typescript
// Other endpoints receive raw params and wrap internally
export const createArticle = ({ title, description, body, tagList }) =>
  apiClient.post('/articles', { article: { title, description, body, tagList } });

// putUser requires caller to pre-wrap
export const putUser = (data: { user: putUserParam }) =>
  apiClient.put('/user', data);
```

**Fix:** Align `putUser` to match other repository functions.

---

#### 6. Complex URL Construction in `getArticles`

**Location:** `src/repositories/articles/articlesRepository.ts:15-22`

**Code:**
```typescript
url: `/articles${isGlobal || username ? '' : '/feed'}?limit=${UNIT_PER_PAGE}&offset=${UNIT_PER_PAGE * (page - 1)}${
  selectedTag ? `&tag=${selectedTag}` : ''
}${username ? `&${isFavorited ? 'favorited' : 'author'}=${username}` : ''}`,
```

**Problem:** Complex ternary logic embedded in URL string is hard to read, test, and maintain.

**Fix:** Use `URLSearchParams`:
```typescript
const params = new URLSearchParams();
params.set('limit', String(UNIT_PER_PAGE));
params.set('offset', String(UNIT_PER_PAGE * (page - 1)));
if (selectedTag) params.set('tag', selectedTag);
if (username) params.set(isFavorited ? 'favorited' : 'author', username);
```

---

#### 7. Inconsistent Response Data Extraction

**Location:** `src/queries/articles.query.ts`

**Variations:**
- Line 22: `getArticles().then((res) => res.data)` - returns full data
- Line 39: `getArticle().then((res) => res.data.article)` - extracts nested
- Line 44: `getComments().then((res) => res.data.comments)` - extracts nested
- Line 27: `getTags().then((res) => res.data.tags)` - extracts nested

**Fix:** Standardize extraction layer (either all in repository or all in query hooks).

---

### Components

#### 8. Weak Type in `useInputs` Hook

**Location:** `src/lib/hooks/useInputs.tsx:3-11`

**Code:**
```typescript
type DefaultType = {
  [key: string]: any;
};

type ReturnTypes = [
  any,
  (event: React.ChangeEvent<HTMLInputElement> | React.ChangeEvent<HTMLTextAreaElement>) => void,
  (value: any) => void,
];
```

**Problem:** Uses `any` type multiple times, destroying TypeScript benefits for all consumers.

**Fix:** Use generics:
```typescript
const useInputs = <T extends Record<string, unknown>>(initialValue: T): [
  T,
  (event: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => void,
  React.Dispatch<React.SetStateAction<T>>,
] => { ... }
```

---

#### 9. Using Array Index as Key

**Location:** `src/components/article/Comment.tsx:67`

**Code:**
```typescript
{comments.map((comment, index) => (
  <div className="card" key={index}>
```

**Problem:** When comments are reordered/deleted, keys become invalid causing rendering issues.

**Fix:** Use unique identifier:
```typescript
<div className="card" key={comment.id}>
```

---

#### 10. Direct API Calls in Pages Instead of Mutations

**Locations:**
- `src/pages/SignInPage.tsx:23-36`
- `src/pages/SignUpPage.tsx:22-35`

**Problem:** Uses `.then()/.catch()` directly on repository functions instead of React Query mutations.

**Impact:**
- No loading state management
- No error retry capability
- Inconsistent with rest of application

**Fix:** Use `useMutation` hook pattern.

---

#### 11. Duplicate Tag Manipulation Logic

**Locations:**
- `src/pages/NewArticlePage.tsx:17-36`
- `src/pages/EditArticlePage.tsx:20-39`

**Problem:** ~20 lines of identical tag management code (`onEnter`, `addTag`, `removeTag`) duplicated.

**Fix:** Extract to custom hook `useArticleTagManager`.

---

#### 12. Loose Props Typing in SettingForm

**Location:** `src/components/SettingForm.tsx:8`

**Code:**
```typescript
interface ISettingFormProps {
  data: { [key: string]: string | number };
}
```

**Fix:** Type as specific interface `IUser`.

---

### Context Management

#### 13. Context Provider Creates New Object Every Render

**Location:** `src/contexts/UserContextProvider.tsx:8,11`

**Code:**
```typescript
<UserContext.Provider value={useIsLoginContext()}>
```

**Problem:** `useIsLoginContext()` called on every render creates new object, breaking memoization in consumers.

**Fix:** Memoize the value:
```typescript
const contextValue = useMemo(() => ({
  isLogin,
  setIsLogin,
}), [isLogin]);
```

---

### Query Invalidation

#### 14. Scattered Query Invalidation Logic

**Locations:** Invalidation code spread across 8+ files:
- `src/pages/NewArticlePage.tsx:47`
- `src/pages/EditArticlePage.tsx:50`
- `src/components/feed/Feed.tsx:34,45`
- `src/components/article/Comment.tsx:28,39`
- `src/components/article/ButtonsWIthoutAccess.tsx:25,37,53,64`
- `src/components/article/ButtonsWIthAccess.tsx:20`
- `src/components/SettingForm.tsx:30`
- `src/components/profile/FollowButton.tsx:23,35`

**Fix:** Centralize in mutation `onSuccess` callbacks in query hooks.

---

#### 15. Incomplete Query Invalidation

**Example:** `NewArticlePage` invalidates only `QUERY_ARTICLES_KEY` after article creation, should also invalidate user profile articles.

---

### Error Handling

#### 16. Missing Error Handlers in Comment Component

**Location:** `src/components/article/Comment.tsx:20-31,34-43`

**Problem:** No `onError` handler for failed comment creation/deletion. User gets no feedback.

---

### Routing

#### 17. Hardcoded Path Strings Instead of Constants

**Locations:**
- `src/pages/HomePage.tsx:32,42,62`
- `src/components/common/Footer.tsx:7`
- `src/components/header/Header.tsx:14`

**Fix:** Use `routerMeta.HomePage.path` constants.

---

#### 18. Router State Instead of URL Params

**Locations:**
- `src/pages/ArticlePage.tsx:14`
- `src/pages/ProfilePage.tsx:8`

**Problem:** Relies on `useLocation().state` instead of `useParams()`. State is lost on refresh/bookmark.

---

### Type Safety

#### 19. Incorrect Boolean Type Definitions

**Location:** `src/interfaces/main.d.ts:9,15`

**Code:**
```typescript
favorited: true;   // Should be: favorited: boolean;
following: true;   // Should be: following: boolean;
```

---

### Missing Patterns

#### 20. Suspense/ErrorBoundary Order

**Location:** `src/Router.tsx:35-42`

**Problem:** ErrorBoundary wraps component AFTER Suspense. Should wrap Suspense for better error capture.

---

## Low Severity Issues

### 21. Inconsistent Props Interface Naming

**Locations:**
- `src/components/profile/FollowButton.tsx:7` - `IFollowButton` (missing `Props` suffix)
- `src/components/HOC/ProtectedRoute.tsx:6` - `IProtectedRoute` (missing `Props` suffix)

---

### 22. Same staleTime for All Queries

**Locations:** All queries use hardcoded `20000ms` without considering data volatility.

**Recommendation:** Use constants like `STALE_TIME_TAGS=60000`, `STALE_TIME_ARTICLES=20000`.

---

### 23. Misspelled/Inconsistent Constant Filenames

**Issues:**
- `src/constants/token.contant.ts` - "contant" instead of "constant"
- `src/constants/units.constants.ts` - plural "constants" vs singular elsewhere

---

### 24. Unused Variables in onSuccess Callbacks

**Locations:** Multiple files use `onSuccess: (_) => { ... }` - response destructured but unused.

---

## Recommended Refactoring Order

### Phase 1: Critical Fixes (High Severity)
1. Fix misspelled filenames (`ButtonsWIth...`)
2. Replace hardcoded `<a>` tags with `<Link>` components
3. Add null-checking to error handlers

### Phase 2: Architecture Alignment (Medium Severity)
1. Centralize mutation handlers with invalidation
2. Standardize repository parameter handling
3. Fix type definitions in `main.d.ts`
4. Extract duplicate tag management logic
5. Convert direct API calls to mutations in auth pages

### Phase 3: Consistency & Polish (Low Severity)
1. Rename constant files consistently
2. Add Props suffix to remaining interfaces
3. Configure different staleTime values
4. Clean up unused variables

---

## Technical Debt Metrics

| Metric | Value |
|--------|-------|
| Files with Issues | ~25 |
| Duplicated Code Blocks | 1 (~20 lines) |
| Type Safety Violations | 4 |
| Naming Convention Violations | 5 |
| Missing Error Handlers | 3 |
| Hardcoded Values | 8+ |
