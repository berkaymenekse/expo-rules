# Technology Stack

| Layer | Technology | Version Requirement | Notes |
| :--- | :--- | :--- | :--- |
| **Framework** | Expo SDK | ≥ 52 | EAS Build/Update mandatory. No bare workflow unless authorized. |
| **Navigation** | Expo Router | v3+ | Use Typed Routes. No React Navigation legacy usage. |
| **Language** | TypeScript | 5.x | `strictest: true`, `noImplicitOverride: true`. |
| **Server State** | TanStack Query | v5 | Use `useSuspenseQuery` for read, `useMutation` for write. Persist enabled. |
| **Client State** | Zustand | v5 | Immer middleware + Devtools + Persist (MMKV). |
| **Styling** | NativeWind | v4 | Tailwind 3.4+ syntax. No `StyleSheet.create`. |
| **Animation** | Reanimated | 3.10+ | Worklets only. No JS-driver animations. |
| **Validation** | Zod | v3.24+ | Runtime + Compile-time inference. |
| **Storage** | MMKV | Latest | Default storage engine. Async Storage is banned. |
| **Images** | Expo Image | Latest | `cachePolicy="memory-disk"` required. |
| **Icons** | Lucide React Native | Latest | Use consistent stroke width. |
| **Forms** | React Hook Form | Latest | Integrate with Zod resolver. |
| **Dates** | date-fns | v3+ | No Moment.js. |
