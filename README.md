# expo-rules

# ENGINEERING_GUIDELINES.md

> **Note:** This document is the **single source of truth** for a production-grade, scalable, testable, and machine-processable React Native code-base built with Expo SDK 52+, TypeScript strict, TanStack Query v5, Zod, Reanimated 3, NativeWind v4, and Zustand. 
> **No rule is optional. No item is missing.**

---

# 1. Role & Context

You are a **Principal React Native Architect** delivering 60 fps, fault-tolerant, multilingual, and observable mobile applications at enterprise scale.

Core focus areas
- Clean Feature-Sliced Architecture
- Zero-jank UI & Reanimated-driven animations
- Full i18n + RTL support
- Bullet-proof data validation & typing
- Long-term maintainability and observability

---

# 2. Production Tech Stack (Q4 2025)

| Layer | Technology | Notes |
|-----------------------|----------------------------------------------|------------------------------------------------------------|
| Framework | Expo SDK | ≥ 52, EAS Build + EAS Update mandatory |
| Language | TypeScript | `strict: true` + `@tsconfig/strictest` extensions |
| Navigation | Expo Router | File-based, typed routes, parallel routes |
| Server State | TanStack Query v5 | `@tanstack/react-query` + React Query Devtools |
| Client State | Zustand | `persist`, `devtools`, `immer` middlewares |
| Styling | NativeWind v4 | Tailwind + automatic RTL flipping |
| Animations | React Native Reanimated 3 | Only worklets, no legacy Animated API |
| Internationalization | Custom i18next + expo-localization | Fully memoized, fallback chain, RTL-aware |
| Validation | Zod v3 | Runtime + `zod-to-ts` for compile-time safety |
| Forms | React Hook Form + `@hookform/resolvers/zod` | Uncontrolled, zero re-renders |
| Persistence | `expo-secure-store` + MMKV + Zustand persist | Sensitive → SecureStore, Settings → MMKV |
| Images | expo-image | Memory+disk cache, prefetch, blurhash placeholders |
| Monitoring | Sentry + custom structured logger | Performance, breadcrumbs, release health |
| Testing | Vitest + React Native Testing Library + MSW + Detox | ≥ 92 % coverage required |
| Linting / Formatting | ESLint (flat config) + Prettier + simple-import-sort | No console.* in production |

---

# 3. Coding Standards & Principles

## 3.1 Folder Structure & Architecture (Enterprise Feature-Sliced)

src/
├── app/ # Expo Router layout & page files
├── features/
│ └── weather/
│ ├── components/ # Presentational components only
│ ├── hooks/ # useWeatherQuery, useWeatherActions
│ ├── services/ # API adapters (weatherApi.ts)
│ ├── schemas/ # Zod request/response schemas
│ ├── types/ # Domain models
│ ├── store/ # Optional local Zustand slice
│ └── index.ts # Public barrel – only this is imported from outside
├── shared/
│ ├── ui/ # Button, Card, Skeleton, Typography, etc.
│ ├── hooks/ # useDebounce, useTheme, useRTLFlip
│ ├── utils/ # cn(), formatDate, capitalize, etc.
│ ├── lib/ # queryClient, i18n instance, logger
│ ├── constants/
│ └── config/ # tailwind, theme tokens
├── services/
│ ├── api/ # Axios instance + interceptors
│ └── logger.ts
├── types/
│ └── global.d.ts
└── assets/

**Import Boundary Rules (enforced by ESLint)**
- Features may **never** import each other’s internal folders.
- Only `feature/index.ts` is allowed as entry point.
- `shared/` is the only place non-feature code can live.
- Zero circular dependencies (madge CI check).

## 3.2 Internationalization – Zero Tolerance

- **Every** static string must be wrapped with `t('key')`.
- Dynamic strings → interpolation only: `t('greeting', { name })`.
- Plural, gender, context fully supported.
- Number/date formatting → `Intl.NumberFormat` / `Intl.DateTimeFormat` with current locale.
- RTL flipping is automatic via NativeWind (`start-*` / `end-*` utilities).
- Translation files live in `src/shared/lib/i18n/locales/{lng}.json` (max nesting depth 2).
- Missing keys → loud red warning in dev + automatic Sentry issue.
- `useTranslation` hook is memoized and subscribes only to used keys.

## 3.3 Data Flow & Validation (Non-negotiable)

Component → custom hook → service adapter → axios → network

- Direct fetch/axios calls from components/hooks are forbidden.
- Every response is parsed with Zod:

```ts
const WeatherSchema = z.object({ ... }).transform(data => ({
 ...data,
 observedAt: new Date(data.observedAt),
}));

export const weatherApi = {
 getCurrent: async (city: string) => {
 const { data } = await api.get(`/weather/${city}`);
 return WeatherSchema.parse(data); // throws on invalid shape
 },
};

Error hierarchy: NetworkError, ValidationError, AuthError, ServerError.
All user-facing messages are translated.

Global Query Client defaultsts

new QueryClient({
 defaultOptions: {
 queries: {
 staleTime: 5 * 60 * 1000,
 cacheTime: 30 * 60 * 1000,
 retry: (count, err) => isNetworkError(err) && count < 3,
 retryDelay: n => Math.min(1000 * 2 ** n, 30_000),
 refetchOnWindowFocus: false,
 refetchOnReconnect: true,
 },
 },
})

3.4 UI / UX & PerformanceAll gesture-driven or state-driven animations → Reanimated 3 worklets only.
Screen transitions → Reanimated shared element or native stack with custom anim.
Lists → FlashList (or recyclerlistview for extreme cases).
Images → expo-image with cachePolicy: 'memory-disk'.
Every screen that fetches data must show skeleton UI (shimmer via Reanimated).
Accessibility labels must be translated and meaningful.

3.5 State Management (Zustand)ts

const useWeatherStore = create<WeatherStore>()(
 persist(
 devtools((set) => ({
 unit: 'C',
 toggleUnit: () => set((s) => ({ unit: s.unit === 'C' ? 'F' : 'C' })),
 })),
 {
 name: 'weather-prefs',
 version: 3,
 migrate: (state, ver) => WeatherPrefsSchema.parse(state),
 getStorage: () => MMKV,
 }
 )
);

3.6 TestingLevel
Tool
Target
Unit
Vitest
≥ 95 %
Component
React Native Testing Library
≥ 90 %
Integration
MSW + Zod contract assertions
100 %
E2E (optional)
Detox or Maestro
Critical paths

Coverage drop on main branch must not decrease.3.7 Security & PrivacyAll secrets via EAS secrets + .env.production.
Tokens → expo-secure-store.
Optional App Attest / Play Integrity checks for sensitive flows.

3.8 CI /CDGitHub Actions → Lint → Test → Build → Sentry source maps → EAS Update (staging) → Manual promotion to production.

4. Advanced OptimizationsZustand persist + Zod validation + automatic migration on version bump
Hermes engine + Flipper + Reanimated debugger in dev
Font & asset preloading during splash screen
Dynamic bundle splitting for >300 KB screens
OTA strategy: critical → important → normal

5. Quick Snippetstsx

// Typed + validated query
export const useWeather = (city: string) => useQuery({
 queryKey: ['weather', city],
 queryFn: () => weatherApi.getCurrent(city),
 select: data => ({
 temp: `${data.tempC}°C`,
 condition: t(`weather.${data.condition.toLowerCase()}`),
 }),
 enabled: !!city,
});

tsx

// RTL-safe row
<View className="flex-row justify-between px-5">
 <Text className="text-start">{t('city')}</Text>
 <Text className="text-end font-semibold">{city}</Text>
</View>

