# Architecture

## Overview
This is a Next.js application using the App Router pattern.

## Component Diagram
```
┌─────────────────────────────────────┐
│           Next.js App               │
├─────────────────────────────────────┤
│  ┌──────────┐    ┌──────────────┐   │
│  │  Pages   │    │  API Routes  │   │
│  └────┬─────┘    └──────┬───────┘   │
│       │                 │           │
│  ┌────┴─────────────────┴────┐      │
│  │       Components          │      │
│  └────────────┬──────────────┘      │
│               │                     │
│  ┌────────────┴──────────────┐      │
│  │    Services & Utilities   │      │
│  └───────────────────────────┘      │
└─────────────────────────────────────┘
```

## Data Flow
1. User interacts with UI components
2. Components call services/hooks
3. Services make API calls
4. API routes process requests
5. Data returned to components

## Key Design Decisions
- Server-first rendering for performance
- API routes for backend logic
- Zustand for client state
