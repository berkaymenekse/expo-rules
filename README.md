# ENGINEERING_GUIDELINES.md  
**2025/Q4 – Ultra Katı Üretim Standardı**  
Tek kaynak. Sıfır tolerans. Her satır zorunlu.

## Mimari Felsefe
- 60 FPS her koşulda → **Jank = bug**
- Zero runtime surprise → Tip + Zod ile %100 runtime güvenliği
- Feature-Sliced + Domain-Driven, katı sınırlar
- 50+ feature, 20+ geliştirici, monorepo-ready
- Observability first → her hata ve yavaşlık izlenir

## Teknoloji Yığını (2025 Sonu)

| Katman              | Teknoloji                              | Notlar                                                                 |
|---------------------|----------------------------------------|------------------------------------------------------------------------|
| Framework           | Expo SDK                               | ≥ 53 · EAS Build + Update + Dev Client zorunlu                          |
| Dil                 | TypeScript                             | `strictest` + `noImplicitOverride`                                     |
| Navigation          | Expo Router v3                         | Typed routes, parallel routes, slots                                   |
| Server State        | TanStack Query v5                      | Persist + offline mutations + devtools                                 |
| Client State        | Zustand v5                             | immer + devtools + persist + slices                                    |
| Stil                | NativeWind v4+                         | Tailwind 3.4+, otomatik RTL flip                                       |
| Animasyon           | Reanimated 3.10+                       | Sadece worklet · Gesture Handler 2.18+                                 |
| i18n                | i18next v24                            | Flat JSON, lazy load, missing key → Sentry                             |
| Validasyon          | Zod v3.24+                             | Runtime + compile-time (`zod-to-ts`)                                   |
| Form                | React Hook Form v8 + zodResolver       | Uncontrolled, zero re-render                                           |
| Depolama            | MMKV + SecureStore                     | Ayarlar → MMKV · Token → SecureStore                                   |
| Resim               | expo-image                             | memory-disk cache + blurhash + prefetch                                |
| İzleme              | Sentry 8                               | Performance tracing + breadcrumbs + release health                     |
| Test                | Vitest + RTL + MSW + Detox/Maestro    | Unit ≥97%, Component ≥94%, E2E kritik yollar %100                      |

## Klasör Yapısı (2025 Katı Standardı)

```bash
src/
├── app/                  # Expo Router layout & sayfalar
├── processes/            # onboarding, checkout gibi akışlar
├── features/
│   └── weather/
│       ├── ui/           # Sadece presentational (Storybook-ready)
│       ├── model/        # Zustand slice + types + selectors
│       ├── api/          # API + Zod schema
│       ├── lib/          # useWeatherQuery, useWeatherUnit
│       └── index.ts      # TEK DIŞARIYA AÇILAN DOSYA
├── entities/             # Normalleştirilmiş domain modelleri (user, product…)
├── widgets/              # Birden çok feature’ı birleştiren büyük bloklar
├── shared/
│   ├── ui/               # Button, Card, Input, Skeleton…
│   ├── hooks/
│   ├── lib/              # queryClient, i18n, logger, axios
│   ├── utils/
│   └── config/
├── services/             # api interceptors, monitoring
└── assets/

Import kuralları katıdır (ESLint no-restricted-imports ile zorlanır):Feature’lar birbirinin iç dosyalarını asla import edemez → sadece index.ts
Döngüsel bağımlılık → CI’da madge ile anında fail

Zero toleranceKesin Kurallar (Zero Tolerance)1. i18n – Hardcoded string = derleme hatasıtsx

// Doğru
t('login.title')
t('user.welcome', { name })

// Yasak → ESLint hatası
'Giriş Yap'

2. Veri & Validasyon – Raw data asla kullanılmazts

const WeatherSchema = z.object({
  temp_c: z.number(),
  observed_at: z.coerce.date(),
}).transform(d => ({
  tempC: d.temp_c,
  observedAt: d.observed_at,
}));

export const weatherApi = {
  getCurrent: (city: string) =>
    api.get(`/weather/${city}`).then(r => WeatherSchema.parse(r.data))
};

3. Query Client (2025 ayarları)ts

staleTime: 5 * 60_000,
gcTime: 30 * 60_000,
placeholderData: keepPreviousData,
refetchOnWindowFocus: false

4. Animasyon → Sadece Reanimated 3tsx

const style = useAnimatedStyle(() => ({
  opacity: withTiming(visible.value ? 1 : 0),
  transform: [{ translateX: withSpring(panX.value) }]
}));

Animated API tamamen yasak · Layout animation zorunlu5. Zustand 2025 Patternts

export const useWeatherStore = create<WeatherState & WeatherActions>()(
  persist(devtools(immer(set => ({
    unit: 'C',
    toggleUnit: () => set(s => { s.unit = s.unit === 'C' ? 'F' : 'C' })
  })))), {
    name: 'weather-v2',
    version: 2,
    migrate: s => WeatherPrefsSchema.parse(s),
    getStorage: () => MMKV
  })
);

Performans & UX ZorunluluklarıHer ekran → ilk render’da Reanimated shimmer skeleton
Liste → <FlashList estimatedItemSize={…} /> zorunlu
Resim → cachePolicy="memory-disk" + priority
İlk anlamlı paint < 1.2 sn (EAS metrics izlenir)

CI/CD Kapıları (Geçemezsen merge yok)Lint + Prettier
TypeScript strict
Vitest coverage ≥ 97%
Bundle boyutu regresyonu < 2%
Madge circular deps
Sentry source maps
EAS Update (staging) → manuel prod

Hızlı Kopyala-Yapıştır Şablonlartsx

// RTL-safe satır
<View className="flex-row justify-between items-center px-5 py-4">
  <Text className="text-start">{t('from')}</Text>
  <Text className="text-end font-semibold">{fromCity}</Text>
</View>

tsx

// Mükemmel query
export const useWeatherQuery = (city: string) => useQuery({
  queryKey: ['weather', city] as const,
  queryFn: () => weatherApi.getCurrent(city),
  select: d => ({
    temp: `${d.tempC}°${unit}`,
    condition: t(`weather.${d.condition.toLowerCase()}`),
    time: new Intl.DateTimeFormat(locale).format(d.observedAt)
  }),
  enabled: !!city,
  staleTime: 10 * 60_000
});

