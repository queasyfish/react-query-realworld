# React-query-realworld - Claude Code Guidelines

## Project Overview
This project is a modern web application built with Next.js and TypeScript.

## Tech Stack
| Layer | Technology | Notes |
|-------|------------|-------|
| Frontend | Next.js 14 | App Router |
| Styling | Tailwind CSS | v4 |
| Language | TypeScript | Strict mode |
| Database | PostgreSQL | Via Prisma |

## Directory Structure
```
/src
├── app/           # Next.js App Router pages
├── components/    # React components
├── lib/           # Utilities and services
└── styles/        # Global styles
```

## Key Patterns
- Use React Server Components by default
- Client components marked with 'use client'
- API routes in /app/api

## Development Setup
```bash
npm install
npm run dev
```

## Testing
```bash
npm test          # Run tests
npm run test:cov  # With coverage
```

## Common Pitfalls
1. Remember to mark client components with 'use client'
2. Use environment variables for secrets
3. Follow existing code patterns
