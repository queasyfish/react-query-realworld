# FEATURES.md

## Feature Index

| Feature | Key | Status | Path |
|---------|-----|--------|------|
| User Authentication | `user-authentication` | ✅ | `src/pages/SignInPage.tsx`, `src/pages/SignUpPage.tsx` |
| Article Feed | `article-feed` | ✅ | `src/components/feed/` |
| Article CRUD | `article-crud` | ✅ | `src/pages/ArticlePage.tsx`, `src/pages/EditArticlePage.tsx` |
| Comments | `comments` | ✅ | `src/components/article/Comment.tsx` |
| User Profiles | `user-profiles` | ✅ | `src/pages/ProfilePage.tsx` |
| Follow Users | `follow-users` | ✅ | `src/components/profile/FollowButton.tsx` |
| Favorite Articles | `favorite-articles` | ✅ | `src/components/article/` |
| Tag Filtering | `tag-filtering` | ✅ | `src/components/feed/Feed.tsx` |
| User Settings | `user-settings` | ✅ | `src/pages/SettingPage.tsx` |
| Markdown Rendering | `markdown-rendering` | ✅ | `src/pages/ArticlePage.tsx` |

**Status:** ✅ Complete | 🚧 In Progress | 📋 Planned | ⚠️ Deprecated

---

## User Authentication

**Key:** `user-authentication`
**Status:** ✅
**Path:** `src/pages/SignInPage.tsx`, `src/pages/SignUpPage.tsx`

**Capabilities:**
- Email/password login
- User registration
- JWT token management via localStorage
- Automatic token injection in API requests
- Session persistence across page reloads

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/users/login` | Login existing user |
| POST | `/users` | Register new user |
| GET | `/user` | Get current user data |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/repositories/users/usersRepository.ts` | Auth API calls |
| `src/lib/token.ts` | localStorage token wrapper |
| `src/contexts/UserContextProvider.tsx` | Auth context provider |
| `src/lib/hooks/useIsLoginContext.tsx` | Login state hook |
| `src/components/HOC/ProtectedRoute.tsx` | Route protection |

**Depends on:** N/A

---

## Article Feed

**Key:** `article-feed`
**Status:** ✅
**Path:** `src/components/feed/`

**Capabilities:**
- View global article feed (all articles)
- View personal feed (followed authors only)
- Pagination (10 articles per page)
- Filter by tag
- Display article metadata (author, date, favorites count)
- Preview article description

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/articles` | Global feed |
| GET | `/articles/feed` | Personal feed (authenticated) |
| GET | `/articles?tag=<tag>` | Filter by tag |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/components/feed/Feed.tsx` | Feed container with tab switching |
| `src/components/feed/FeedList.tsx` | Article list rendering |
| `src/queries/articles.query.ts` | React Query hooks |
| `src/repositories/articles/articlesRepository.ts` | API calls |

**Depends on:** `user-authentication`, `tag-filtering`, `favorite-articles`

---

## Article CRUD

**Key:** `article-crud`
**Status:** ✅
**Path:** `src/pages/ArticlePage.tsx`, `src/pages/NewArticlePage.tsx`, `src/pages/EditArticlePage.tsx`

**Capabilities:**
- Create new articles with title, description, body, and tags
- Edit existing articles (author only)
- Delete articles (author only)
- View article with rendered markdown
- Article slug generation

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/articles/:slug` | Get single article |
| POST | `/articles` | Create article |
| PUT | `/articles/:slug` | Update article |
| DELETE | `/articles/:slug` | Delete article |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/pages/NewArticlePage.tsx` | Article creation form |
| `src/pages/EditArticlePage.tsx` | Article edit form |
| `src/pages/ArticlePage.tsx` | Article display |
| `src/components/article/ButtonsWIthAccess.tsx` | Edit/delete buttons for author |

**Depends on:** `user-authentication`, `markdown-rendering`

---

## Comments

**Key:** `comments`
**Status:** ✅
**Path:** `src/components/article/Comment.tsx`

**Capabilities:**
- View comments on articles
- Add comments (authenticated users)
- Delete own comments
- Display commenter profile info

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/articles/:slug/comments` | Get all comments |
| POST | `/articles/:slug/comments` | Add comment |
| DELETE | `/articles/:slug/comments/:id` | Delete comment |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/components/article/Comment.tsx` | Comment list and form |
| `src/queries/articles.query.ts` | Comment mutations |

**Depends on:** `user-authentication`, `article-crud`

---

## User Profiles

**Key:** `user-profiles`
**Status:** ✅
**Path:** `src/pages/ProfilePage.tsx`

**Capabilities:**
- View user profile with bio and avatar
- List user's articles
- List user's favorited articles
- Tab switching between "My Articles" and "Favorited Articles"

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/profiles/:username` | Get user profile |
| GET | `/articles?author=:username` | Get user's articles |
| GET | `/articles?favorited=:username` | Get favorited articles |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/pages/ProfilePage.tsx` | Profile page with tabs |
| `src/components/Profile.tsx` | Profile header component |
| `src/repositories/profiles/profileRepository.ts` | Profile API calls |

**Depends on:** `article-feed`, `follow-users`, `favorite-articles`

---

## Follow Users

**Key:** `follow-users`
**Status:** ✅
**Path:** `src/components/profile/FollowButton.tsx`

**Capabilities:**
- Follow other users
- Unfollow users
- See follow status on profiles
- Following count display

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/profiles/:username/follow` | Follow user |
| DELETE | `/profiles/:username/follow` | Unfollow user |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/components/profile/FollowButton.tsx` | Follow/unfollow button |
| `src/queries/profiles.query.ts` | Follow mutations |
| `src/repositories/profiles/profileRepository.ts` | Follow API calls |

**Depends on:** `user-authentication`, `user-profiles`

---

## Favorite Articles

**Key:** `favorite-articles`
**Status:** ✅
**Path:** `src/components/article/ButtonsWIthoutAccess.tsx`

**Capabilities:**
- Mark articles as favorite
- Unfavorite articles
- View favorite count
- Filter articles by favorited status on profile

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/articles/:slug/favorite` | Favorite article |
| DELETE | `/articles/:slug/favorite` | Unfavorite article |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/components/article/ButtonsWIthoutAccess.tsx` | Favorite button |
| `src/queries/articles.query.ts` | Favorite mutations |
| `src/repositories/articles/articlesRepository.ts` | Favorite API calls |

**Depends on:** `user-authentication`, `article-crud`

---

## Tag Filtering

**Key:** `tag-filtering`
**Status:** ✅
**Path:** `src/components/feed/Feed.tsx`

**Capabilities:**
- View popular tags
- Filter articles by tag
- Click tag to filter feed
- Clear tag filter

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/tags` | Get all tags |
| GET | `/articles?tag=<tag>` | Filter articles by tag |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/components/feed/Feed.tsx` | Tag list and filtering |
| `src/repositories/tags/tagsRepository.ts` | Tags API calls |
| `src/queries/articles.query.ts` | Query with tag param |

**Depends on:** `article-feed`

---

## User Settings

**Key:** `user-settings`
**Status:** ✅
**Path:** `src/pages/SettingPage.tsx`

**Capabilities:**
- Update profile image URL
- Update username
- Update bio
- Update email
- Change password
- Logout

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| PUT | `/user` | Update user settings |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/pages/SettingPage.tsx` | Settings page |
| `src/components/SettingForm.tsx` | Settings form component |
| `src/repositories/users/usersRepository.ts` | Update user API call |

**Depends on:** `user-authentication`

---

## Markdown Rendering

**Key:** `markdown-rendering`
**Status:** ✅
**Path:** `src/pages/ArticlePage.tsx`

**Capabilities:**
- Render markdown article bodies to HTML
- Support GitHub Flavored Markdown (GFM)
- Safe HTML rendering
- Code syntax support

**Key Files:**
| File | Purpose |
|------|---------|
| `src/pages/ArticlePage.tsx` | Uses react-markdown |
| Package: `react-markdown` | Markdown parser |
| Package: `remark-gfm` | GFM plugin |

**Depends on:** `article-crud`
