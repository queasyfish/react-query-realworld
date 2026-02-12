# FEATURES.md

## Feature Index

| Feature | Key | Status | Path |
|---------|-----|--------|------|
| User Authentication | `user-authentication` | ✅ | `src/pages/SignInPage.tsx`, `src/pages/SignUpPage.tsx` |
| Article Feed | `article-feed` | ✅ | `src/components/feed/` |
| Article Management | `article-management` | ✅ | `src/pages/NewArticlePage.tsx`, `src/pages/EditArticlePage.tsx` |
| Article View | `article-view` | ✅ | `src/pages/ArticlePage.tsx` |
| Comments | `comments` | ✅ | `src/components/article/Comment.tsx` |
| User Profiles | `user-profiles` | ✅ | `src/pages/ProfilePage.tsx` |
| Follow Users | `follow-users` | ✅ | `src/components/profile/FollowButton.tsx` |
| Favorite Articles | `favorite-articles` | ✅ | `src/repositories/articles/articlesRepository.ts` |
| Tag System | `tag-system` | ✅ | `src/repositories/tags/tagsRepository.ts` |
| User Settings | `user-settings` | ✅ | `src/pages/SettingPage.tsx` |

**Status:** ✅ Complete | 🚧 In Progress | 📋 Planned | ⚠️ Deprecated

---

## User Authentication

**Key:** `user-authentication`
**Status:** ✅
**Path:** `src/pages/SignInPage.tsx`, `src/pages/SignUpPage.tsx`

**Capabilities:**
- Email/password login
- User registration with validation
- JWT token storage in localStorage
- Automatic token injection via Axios interceptors
- Protected route handling
- Session persistence across page reloads

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/users/login` | Authenticate user |
| POST | `/users` | Register new user |
| GET | `/user` | Get current user |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/pages/SignInPage.tsx` | Login form and logic |
| `src/pages/SignUpPage.tsx` | Registration form |
| `src/repositories/apiClient.ts` | JWT token interceptor |
| `src/lib/token.ts` | localStorage token management |
| `src/components/HOC/ProtectedRoute.tsx` | Route protection wrapper |

**Depends on:** `user-settings`

---

## Article Feed

**Key:** `article-feed`
**Status:** ✅
**Path:** `src/components/feed/`, `src/pages/HomePage.tsx`

**Capabilities:**
- Global feed (all articles)
- Personal feed (followed users' articles)
- Tag-based filtering
- Pagination (10 articles per page)
- Real-time favorite count updates
- Optimistic UI updates via React Query

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/articles` | Get global articles |
| GET | `/articles/feed` | Get personal feed |
| GET | `/articles?tag={tag}` | Filter by tag |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/pages/HomePage.tsx` | Main feed page with toggle |
| `src/components/feed/Feed.tsx` | Feed container |
| `src/components/feed/FeedList.tsx` | Article list with pagination |
| `src/queries/articles.query.ts` | React Query hooks |

**Depends on:** `user-authentication`, `tag-system`, `favorite-articles`

---

## Article Management

**Key:** `article-management`
**Status:** ✅
**Path:** `src/pages/NewArticlePage.tsx`, `src/pages/EditArticlePage.tsx`

**Capabilities:**
- Create new articles with title, description, body, and tags
- Edit existing articles (author only)
- Delete articles (author only)
- Markdown support via react-markdown
- Tag management (add/remove)
- Form validation

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/articles` | Create article |
| PUT | `/articles/{slug}` | Update article |
| DELETE | `/articles/{slug}` | Delete article |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/pages/NewArticlePage.tsx` | Article creation form |
| `src/pages/EditArticlePage.tsx` | Article editing form |
| `src/repositories/articles/articlesRepository.ts` | API calls |
| `src/queries/articles.query.ts` | Mutation hooks |

**Depends on:** `user-authentication`, `tag-system`

---

## Article View

**Key:** `article-view`
**Status:** ✅
**Path:** `src/pages/ArticlePage.tsx`

**Capabilities:**
- Full article display with markdown rendering
- Author information and profile link
- Publication date
- Favorite/unfavorite button
- Follow/unfollow author button
- Edit/delete buttons (author only)
- Comment section
- Article metadata (tags, author, date)

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/articles/{slug}` | Get single article |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/pages/ArticlePage.tsx` | Article detail page |
| `src/components/article/ButtonSelector.tsx` | Conditional action buttons |
| `src/components/article/ButtonsWIthAccess.tsx` | Edit/delete for authors |
| `src/components/article/ButtonsWIthoutAccess.tsx` | Favorite/follow for readers |

**Depends on:** `user-authentication`, `comments`, `favorite-articles`, `follow-users`

---

## Comments

**Key:** `comments`
**Status:** ✅
**Path:** `src/components/article/Comment.tsx`

**Capabilities:**
- Add comments to articles
- Delete own comments
- View all comments on article
- Comment author information
- Comment timestamps

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/articles/{slug}/comments` | Get all comments |
| POST | `/articles/{slug}/comments` | Add comment |
| DELETE | `/articles/{slug}/comments/{id}` | Delete comment |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/components/article/Comment.tsx` | Comment component |
| `src/repositories/articles/articlesRepository.ts` | Comment API calls |
| `src/queries/articles.query.ts` | Comment mutations |

**Depends on:** `user-authentication`, `article-view`

---

## User Profiles

**Key:** `user-profiles`
**Status:** ✅
**Path:** `src/pages/ProfilePage.tsx`, `src/components/Profile.tsx`

**Capabilities:**
- View user profile information (bio, image)
- View user's published articles
- View user's favorited articles
- Tab navigation between articles and favorites
- Follow/unfollow from profile page

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/profiles/{username}` | Get profile data |
| GET | `/articles?author={username}` | Get user's articles |
| GET | `/articles?favorited={username}` | Get favorited articles |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/pages/ProfilePage.tsx` | Profile page with tabs |
| `src/components/Profile.tsx` | Profile header component |
| `src/repositories/profiles/profileRepository.ts` | Profile API |
| `src/queries/profiles.query.ts` | Profile queries |

**Depends on:** `user-authentication`, `follow-users`, `article-feed`

---

## Follow Users

**Key:** `follow-users`
**Status:** ✅
**Path:** `src/components/profile/FollowButton.tsx`

**Capabilities:**
- Follow users from profile page
- Follow users from article page
- Unfollow users
- Real-time follow count updates
- Optimistic UI updates

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/profiles/{username}/follow` | Follow user |
| DELETE | `/profiles/{username}/follow` | Unfollow user |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/components/profile/FollowButton.tsx` | Follow button component |
| `src/repositories/profiles/profileRepository.ts` | Follow API calls |
| `src/queries/profiles.query.ts` | Follow mutations |

**Depends on:** `user-authentication`

---

## Favorite Articles

**Key:** `favorite-articles`
**Status:** ✅
**Path:** `src/repositories/articles/articlesRepository.ts`

**Capabilities:**
- Favorite articles
- Unfavorite articles
- View favorite count
- Optimistic UI updates
- Favorite state persistence

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/articles/{slug}/favorite` | Favorite article |
| DELETE | `/articles/{slug}/favorite` | Unfavorite article |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/repositories/articles/articlesRepository.ts` | Favorite API calls |
| `src/queries/articles.query.ts` | Favorite mutations |

**Depends on:** `user-authentication`

---

## Tag System

**Key:** `tag-system`
**Status:** ✅
**Path:** `src/repositories/tags/tagsRepository.ts`

**Capabilities:**
- Display popular tags on home page
- Filter articles by tag
- Add tags to articles
- Tag-based article discovery

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/tags` | Get all popular tags |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/repositories/tags/tagsRepository.ts` | Tags API |
| `src/queries/articles.query.ts` | Tag queries combined with articles |

**Depends on:** None

---

## User Settings

**Key:** `user-settings`
**Status:** ✅
**Path:** `src/pages/SettingPage.tsx`, `src/components/SettingForm.tsx`

**Capabilities:**
- Update profile image URL
- Update username
- Update bio
- Update email
- Change password
- Logout functionality

**API Endpoints:**
| Method | Path | Purpose |
|--------|------|---------|
| PUT | `/user` | Update user profile |

**Key Files:**
| File | Purpose |
|------|---------|
| `src/pages/SettingPage.tsx` | Settings page container |
| `src/components/SettingForm.tsx` | Settings form with validation |
| `src/repositories/users/usersRepository.ts` | User update API |
| `src/queries/user.query.ts` | User mutations |

**Depends on:** `user-authentication`
