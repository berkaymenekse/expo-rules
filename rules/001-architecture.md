# Architecture: Feature-Sliced Design (FSD)

## Folder Structure
We follow a strict FSD implementation.

```bash
src/
├── app/                  # Expo Router (pages & layouts only)
├── processes/            # Complex cross-feature flows (e.g., authentication flow, checkout wizard)
├── features/             # Business logic (e.g., weather, auth, cart)
│   └── [feature-name]/
│       ├── ui/           # Presentational components (Storybook-ready)
│       ├── model/        # Zustand slice, types, selectors
│       ├── api/          # API calls + Zod schemas
│       ├── lib/          # Hooks (useWeatherQuery), Helpers
│       └── index.ts      # THE ONLY PUBLIC ENTRY POINT
├── entities/             # Normalized domain models (user, product, order) - Highly reusable, low logic
├── widgets/              # Composition of multiple features (e.g. Header, Sidebar, NewsFeed)
├── shared/
│   ├── ui/               # Primitives (Button, Input, Skeleton)
│   ├── hooks/            # Generic hooks
│   ├── lib/              # Infrastructure (axios, queryClient, i18n)
│   ├── utils/            # Pure functions
│   └── config/           # Env, Constants
└── assets/
```

## Rules of Engagement

### 1. The Slice Rule
`features` cannot import from other `features`.
*   **Why?** To prevent tight coupling.
*   **Solution:** If Feature A needs Feature B, move logic to `widgets` (composition) or `processes` (flow), or push shared logic down to `entities`.

### 2. The Hierarchy Rule
You can only import "down".
`app` > `processes` > `widgets` > `features` > `entities` > `shared`

### 3. The Public API Rule
Only import from the `index.ts` of a slice.
*   ✅ `import { LoginForm } from '@/features/auth';`
*   ❌ `import { LoginForm } from '@/features/auth/ui/LoginForm';` (Forbidden Deep Import)

### 4. Layer Definitions
*   **Shared:** Reusable code unrelated to business specifics.
*   **Entities:** Business models (User, Product). Just data and simple display.
*   **Features:** User interactions that bring value (AddToCart, LikePost).
*   **Widgets:** Standalone blocks combining entities and features (ProductCard with AddToCart button).
*   **Processes:** Scenarios spanning multiple pages (Registration, Checkout).
*   **App:** Entry point, routing, providers.
