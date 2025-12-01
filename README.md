ENGINEERING_GUIDELINES.md – 2025/Q4 Sürümü (Ultra Katı, Gerçek Üretim Seviyesi)
Tek kaynak, sıfır tolerans, her satırı uygulanmak zorunda.1. Mimari Felsefe & İlkeler60 FPS her koşulda garanti – Jank = bug  
Zero runtime surprise – Tip güvenliği + Zod parse ile %100  
Feature-Sliced + Domain-Driven katı sınırlar  
Scale-out ready – 50+ feature, 20+ geliştirici, monorepo bile destekler  
Observability first – Her hata, her yavaşlık izlenir

2. 2025 Sonu – Kesin Teknoloji YığınıKatman
Teknoloji
Versiyon & Notlar
Framework
Expo SDK
≥ 53 (EAS Build + EAS Update + Expo Dev Client zorunlu)
Language
TypeScript
strict: true + @tsconfig/strictest + noImplicitOverride
Navigation
Expo Router v3
Typed routes, parallel routes, slot-based layout, deep linking contract
Server State
TanStack Query v5
+ React Query Devtools + queryClient persist + offline mutations
Client State
Zustand v5
immer + devtools + persist + slices pattern
Styling
NativeWind v4.0+
Tailwind v3.4+ syntax, automatic RTL flip, dark mode tokens
Animations
Reanimated 3.10+
Sadece worklet + layout animations, Gesture Handler v2.18+
i18n
i18next v24 + react-i18next + expo-localization
JSON flat (max depth 2), lazy load, missing key → Sentry + dev console error
Validation
Zod v3.24+
Runtime + zod-to-ts ile compile-time + automatic OpenAPI sync
Forms
React Hook Form v8 + zodResolver
Uncontrolled + zero re-renders + dirtyFields tracking
Persistence
MMKV v3 + expo-secure-store
Ayarlar → MMKV, token/refresh → SecureStore
Images
expo-image v1.5+
memory-disk + prefetch + blurhash + priority loading
Logging & Monitoring
Sentry 8 + @sentry
/react-native + custom logger
Performance tracing, breadcrumbs, release health, user feedback
Testing
Vitest + RTL + MSW2 + Detox + Maestro
Unit 97 %+, Component 94 %+, E2E kritik yollar %100
Bundle Analysis
expo-bundle-optimize + webpack-bundle-analyzer
Her PR’de bundle boyutu regrese olmaz

3. Klasör Yapısı – 2025 Standardı (Katı)

src/
├── app/                     # Expo Router (_layout, +not-found, (auth), (app) vb.)
├── processes/              # Çok feature’ı birleştiren iş akışları (onboarding, checkout)
├── features/
│   └── weather/
│       ├── ui/              # Sadece presentational + Storybook
│       ├── model/           # store slice + types + selectors
│       ├── api/             # weatherApi.ts + schemas
│       ├── lib/             # hook’lar (useWeatherQuery, useWeatherUnit)
│       └── index.ts         # Tek giriş noktası – başka hiçbir şey import edilemez
├── entities/
│   └── user/                # Domain entity’ler (normalleştirilmiş store’lar burada)
├── widgets/                # Birden çok feature’ı birleştiren büyük bloklar
├── shared/
│   ├── ui/                  # Atomik tasarım sistemi (Button, Input, Card, Skeleton)
│   ├── hooks/               # useDebounce, useTheme, useRTL, useOnline
│   ├── lib/                 # queryClient, i18n, logger, axiosInstance, zod schemas
│   ├── utils/               # cn, formatters, validators
│   ├── config/              # tailwind.config.js, theme tokens, feature flags
│   └── constants/
├── services/
│   ├── api/                 # axios + interceptors + refresh logic
│   └── monitoring/
└── assets/

Import Sınırları (ESLint no-restricted-imports ile zorunlu):features/* birbirini iç klasörlerden asla import edemez → sadece ../index.ts
shared/ hariç hiçbir yer başka bir domain’e doğrudan erişemez
Döngüsel bağımlılık → CI’da madge ile fail

4. Kesin Kurallar (Zero Tolerance)4.1 i18ntsx

// Doğru
t('login.title')
t('welcome_user', { name }))

// Yasak
'Giriş Yap' // ← derleme hatası (eslint rule)

Tüm translation dosyaları src/shared/lib/i18n/locales/{lng}.json
Maksimum nesting 2 seviye
Eksik key → DEV’de kırmızı console.error + otomatik Sentry issue

4.2 Veri Akışı & Validasyonts

// Her API yanıtı parse edilir, asla raw data kullanılmaz
const GetWeatherResponse = z.object({
  temp_c: z.number(),
  condition: z.string(),
  observed_at: z.coerce.date(),
}).transform(d => ({
  tempC: d.temp_c,
  condition: d.condition,
  observedAt: d.observed_at,
}));

export const weatherApi = {
  getCurrent: (city: string) => 
    api.get(`/weather/${city}`).then(r => GetWeatherResponse.parse(r.data))
};

Hata tipleri: ApiError extends Error → subclasses: ValidationError, AuthError, NetworkError, ServerError
Tüm hatalar errorHandler(err) ile merkezi işlenir → toast + Sentry

4.3 Query Client – 2025 Ayarlarıts

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60_000,
      gcTime: 30 * 60_000,
      retry: (failureCount, error) => isNetworkError(error) && failureCount < 3,
      retryDelay: attempt => Math.min(1000 * 2 ** attempt, 30_000),
      refetchOnWindowFocus: false,
      refetchOnReconnect: true,
      placeholderData: keepPreviousData,
      suspense: false,
    },
    mutations: {
      onError: (err) => errorHandler(err),
    },
  },
});

4.4 Animations – Sadece Reanimated 3tsx

const animatedStyle = useAnimatedStyle(() => ({
  opacity: withTiming(isVisible.value ? 1 : 0, { duration: 300 }),
  transform: [{ translateX: withSpring(panX.value) }],
}));

Animated API tamamen yasak (eslint rule)
Layout animation → Layout component + entering/exiting key’ler zorunlu

4.5 Zustand – Modern Pattern 2025ts

// features/weather/model/store.ts
export const useWeatherStore = create<WeatherState & WeatherActions>()(
  persist(
    devtools(
      immer((set) => ({
        unit: 'C',
        favoriteCities: [],
        toggleUnit: () => set(s => { s.unit = s.unit === 'C' ? 'F' : 'C' }),
        addFavorite: city => set(s => { s.favoriteCities.push(city) }),
      }))
    ),
    {
      name: 'weather-v2',
      version: 2,
      migrate: (persisted, version) => WeatherPrefsSchema.parse(persisted),
      getStorage: () => MMKV,
      partialize: state => ({ unit: state.unit, favoriteCities: state.favoriteCities }),
    }
  )
);

5. Performans & UX ZorunluluklarıHer ekran ilk render’da skeleton gösterir (Reanimated shimmer)
Listeler → <FlashList estimatedItemSize> zorunlu
Image → <Image cachePolicy="memory-disk" priority="high" />
Font preloading splash screen’de biter
İlk anlamlı paint < 1.2 saniye (EAS metrics izlenir)

6. CI/CD & Kalite Kapılarıyaml

- Lint (ESLint + Prettier)
- TypeScript strict check
- Vitest + coverage ≥ 97%
- Bundle size regression check (< 2% artış)
- Madge circular dependency
- Sentry source maps upload
- EAS Update (staging) → manuel prod promotion

7. Hızlı Şablonlar (Kopyala-Yapıştır)tsx

// RTL-safe flex row
<View className="flex-row justify-between items-center px-5 py-4">
  <Text className="text-start text-base">{t('from')}</Text>
  <Text className="text-end font-medium">{fromCity}</Text>
</View>

tsx

// Typed, validated, translated query
export const useWeatherQuery = (city: string) => useQuery({
  queryKey: ['weather', city] as const,
  queryFn: () => weatherApi.getCurrent(city),
  select: data => ({
    temp: `${data.tempC}°${unit}`,
    condition: t(`weather.${data.condition.toLowerCase()}`),
    time: new Intl.DateTimeFormat(locale).format(data.observedAt),
  }),
  enabled: !!city,
  staleTime: 10 * 60_1000,
});

