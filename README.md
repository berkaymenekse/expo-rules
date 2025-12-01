İşte **2025/Q4 Ultra Katı Üretim Standardı** için nihai, birleştirilmiş ve tüm iyileştirmeleri içeren `ENGINEERING_GUIDELINES.md` dosyası.

Bu dosya, projenin **Tek Gerçeklik Kaynağıdır (Single Source of Truth)**.

***

# ENGINEERING_GUIDELINES.md

**Versiyon:** 2025.Q4  
**Statü:** Ultra Katı (Non-negotiable)  
**Motto:** "Tek kaynak. Sıfır tolerans. Her satır tip güvenli."

## 1. Mimari Felsefe & Prensipler

Bu projede "çalışıyorsa dokunma" yoktur; "standarda uymuyorsa rewrite edilir" vardır.

1.  **60 FPS & Zero Jank:** `Jank = Bug`. UI thread bloklanamaz.
2.  **Zero Runtime Surprise:** `any` yasak. API yanıtları Zod ile parse edilmeden UI'a inemez. Environment değişkenleri tip güvenlidir.
3.  **Feature-Sliced Design (FSD):** Katmanlar arası sınırlar `eslint-plugin-boundaries` ile korunur.
4.  **Erişilebilirlik (a11y):** Erişilebilirlik bir özellik değil, **zorunluluktur**.
5.  **Observability First:** Her hata, her yavaşlık Sentry'de görünür olmalıdır.

## 2. Teknoloji Yığını (2025 Standartları)

| Katman | Teknoloji | Versiyon / Kural |
| :--- | :--- | :--- |
| **Framework** | Expo SDK | ≥ 52 · EAS Build + Dev Client Zorunlu |
| **Dil** | TypeScript | `strictest` + `noImplicitOverride` |
| **Navigation** | Expo Router v3+ | **Typed Routes** zorunlu (String path yasak) |
| **Server State** | TanStack Query v5 | Persist + GC Time ayarlı + Factory Pattern |
| **Client State** | Zustand v5 | `immer` + `devtools` + **Selectors** zorunlu |
| **Stil** | NativeWind v4+ | Tailwind 3.4+ · Magic Number (px) yasak |
| **Animasyon** | Reanimated 3.10+ | Sadece Worklet · Layout Animation |
| **i18n** | i18next v24 | **Typed Keys** · Lazy Load · Missing Key Log |
| **Validasyon** | Zod v3.24+ | Runtime (`z.parse`) + Compile-time (`z.infer`) |
| **Form** | React Hook Form | Uncontrolled · `zodResolver` |
| **Depolama** | MMKV / SecureStore | Hızlı veri MMKV · Tokenlar SecureStore |
| **Resim** | expo-image | `memory-disk` cache + BlurHash + Transition |
| **Test** | Vitest + Maestro | Unit (Logic), Integration (Flow), E2E (Critical) |

## 3. Klasör Yapısı (FSD Strict)

```bash
src/
├── app/                  # Expo Router (Sadece sayfa tanımları, logic yok)
├── processes/            # Karmaşık akışlar (Auth, Checkout, Onboarding)
├── features/             # İşlevsel özellikler
│   └── weather/
│       ├── ui/           # Presentational Components (State bilmez)
│       ├── model/        # Zustand slice + Selectors (Logic burada)
│       ├── api/          # API Calls + Zod Schemas + Transformers
│       ├── lib/          # Hooks (useWeatherQuery)
│       └── index.ts      # PUBLIC API (Sadece buradan export edilir)
├── entities/             # Domain Modelleri (User, Product - Business logic)
├── widgets/              # Feature'ları birleştiren bloklar (DashboardCard)
├── shared/               # Proje genelindeki ortak yapılar
│   ├── ui/               # Atomik UI (Button, Input, Skeleton)
│   ├── hooks/            # useDebounce, useAppState
│   ├── lib/              # axios, queryClient, i18n, logger
│   └── config/           # env.ts (Validated), constants
└── assets/
```

### Bağımlılık Kuralları (CI'da Fail Verir)
*   **Yasak:** `features` birbirini import edemez. (İletişim `app` veya `widgets` üzerinden).
*   **Yasak:** `shared` katmanı `features` veya `entities` import edemez.
*   **Yasak:** Üst katman (`app`) alt katmanlardan her şeyi alabilir, tersi yasaktır.
*   **Araç:** `madge` (döngüsel bağımlılık) ve `eslint-plugin-boundaries` (katman ihlali).

## 4. Sıfır Tolerans Kuralları (Zero Tolerance)

### 4.1. Tip Güvenliği: "Tanrı Modu" (God-Tier Type Safety)
Sadece props değil, environment ve i18n de tip güvenli olmalıdır.

**A. Environment Variables:**
`.env` dosyasından string okumak yasaktır. Uygulama boot olurken validate edilmelidir.

```typescript
// src/shared/config/env.ts
import { z } from 'zod';

const EnvSchema = z.object({
  API_URL: z.string().url(),
  SENTRY_DSN: z.string(),
  ENABLE_ANALYTICS: z.coerce.boolean(),
});

// Validasyon başarısızsa uygulama çöker (Fail Fast)
export const ENV = EnvSchema.parse(process.env); 
```

**B. Typed i18n:**
Hardcoded string yasaktır. `t` fonksiyonu sadece tanımlı key'leri kabul eder.

```tsx
// YANLIŞ (Derleme Hatası)
<Text>Giriş Yap</Text>
t('giris.yap') // Typo varsa derlenmez

// DOĞRU
t('auth.login.button_label')
```

### 4.2. Veri & Validasyon (Zod veya Hiç)
Backend'den gelen veri asla doğrudan UI'a basılamaz. `unknown` gelir, `z.parse` ile `Typed` olur.

```typescript
// 1. Şema Tanımı (Backend response)
const UserResponseSchema = z.object({
  user_id: z.string(),
  full_name: z.string(),
  created_at: z.coerce.date(), // String -> Date dönüşümü
});

// 2. Transform (Frontend model)
const UserSchema = UserResponseSchema.transform((u) => ({
  id: u.user_id,
  name: u.full_name,
  joinedAt: u.created_at,
}));

// 3. Kullanım
export const fetchUser = async (id: string) => {
  const res = await api.get(`/users/${id}`);
  return UserSchema.parse(res.data); // Veri bozuksa burada patlar, UI bozulmaz
};
```

### 4.3. State Yönetimi (Zustand & Query)

**A. React Query (Server State) - 2025 Ayarları:**
```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 dk boyunca fetch etme
      gcTime: 30 * 60 * 1000,   // 30 dk cache'te tut
      refetchOnWindowFocus: false,
      retry: 1,
    },
  },
});
```

**B. Zustand (Client State) - Selector Zorunluluğu:**
Tüm store'u hook olarak çağırmak (`const store = useStore()`) **performans suçudur**.

```tsx
// YANLIŞ: Her update'te re-render
const { user } = useAuthStore();

// DOĞRU: Sadece ilgili alan değişince render (Atomic Selector)
const userName = useAuthStore(useShallow((s) => s.user?.name));
const toggleTheme = useAuthStore((s) => s.actions.toggleTheme);
```

### 4.4. UI, Animasyon & Performans

*   **Animasyon:** `Animated` API yasak. Sadece `react-native-reanimated` worklet'leri.
*   **Listeler:** `FlatList` yerine `<FlashList estimatedItemSize={...} />` zorunlu.
*   **Resimler:** `expo-image` ile `cachePolicy="memory-disk"` ve `transition={200}`.
*   **Skeleton:** Her ekranın ilk render'ı `isLoading` state'inde **Skeleton** göstermelidir. Spinner yasak.
*   **Magic Numbers:** `padding: 13` yasak. `p-3` veya `gap-4` (Tailwind) kullanılmalı.

**Interaction to Next Paint (INP):**
Ağır işlemler `onPress` içinde UI'ı donduramaz.
```typescript
// Ağır işlem varsa
onPress={() => {
  // 1. Önce UI update (Optimistic)
  setClicked(true);
  // 2. Sonra Logic
  InteractionManager.runAfterInteractions(() => {
    heavyCalculation();
  });
}}
```

### 4.5. Erişilebilirlik (Accessibility - a11y)
Erişilebilirlik testi geçmeyen PR merge edilemez.

*   Her `Pressable` / `TouchableOpacity`:
    *   `accessibilityRole="button"` (veya uygun rol)
    *   `accessibilityLabel="..."` (i18n key)
*   Renk kontrastı WCAG AA standardında olmalı.

## 5. Hata Yönetimi (Granular Error Boundaries)

Tüm uygulama tek bir hata ile beyaz ekrana düşmemeli.
*   **Global Error Boundary:** App crash durumları için.
*   **Feature Error Boundary:** Örn: "Hava Durumu Widget'ı" hata verirse, sadece o kutu içinde "Yüklenemedi" yazmalı, uygulamanın kalanı çalışmalı.

## 6. Test Stratejisi

Testler "mış gibi" değil, gerçek senaryoları kapsamalıdır.

| Tür | Hedef | Araç | Kapsam |
| :--- | :--- | :--- | :--- |
| **Unit** | %95+ | Vitest | Hooks, Utils, Helpers (Logic) |
| **Component** | %50+ | RNTL | Karmaşık UI state'leri (Simple UI test edilmez) |
| **Integration** | %80+ | MSW | Feature akışları (Mock API ile) |
| **E2E** | %100 | Maestro | Kritik Yollar (Login, Checkout, Onboarding) |

## 7. CI/CD Kapıları (Merge Yasakları)

Aşağıdaki maddelerden **biri bile** sağlanmazsa CI build'i `fail` eder:

1.  🛑 **Lint & Prettier:** Hata veya warning olmamalı.
2.  🛑 **Type Check:** `tsc` hatasız tamamlanmalı.
3.  🛑 **Circular Deps:** `madge` döngüsel bağımlılık bulmamalı.
4.  🛑 **Test Coverage:** Belirlenen oranların altında kalmamalı.
5.  🛑 **Bundle Size:** Regresyon <%2 olmalı.
6.  🛑 **EAS Update:** Staging kanalına otomatik update çıkılmalı.

## 8. Hızlı Kopyala-Yapıştır Şablonlar

### Feature Query Hook Şablonu
```typescript
export const useWeatherQuery = (city: string) => {
  const { t } = useTranslation();
  return useQuery({
    queryKey: ['weather', city] as const,
    queryFn: () => weatherApi.getCurrent(city),
    select: (data) => ({
      // UI için formatlama burada yapılır, Component'te değil
      displayTemp: `${Math.round(data.temp)}°`,
      conditionText: t(`weather.conditions.${data.conditionCode}`),
    }),
    enabled: !!city && city.length > 2,
    staleTime: 10 * 60 * 1000, // 10 dakika
  });
};
```

### UI Component Şablonu (RTL & a11y Safe)
```tsx
<Pressable 
  className="flex-row items-center justify-between p-4 bg-card rounded-xl active:opacity-80"
  onPress={handlePress}
  accessibilityRole="button"
  accessibilityLabel={t('actions.open_details', { item: title })}
>
  <View className="flex-1 gap-1">
    <Text className="text-base font-semibold text-foreground text-start">
      {title}
    </Text>
    <Text className="text-sm text-muted-foreground text-start">
      {subtitle}
    </Text>
  </View>
  <ChevronRight className="text-muted-foreground" size={20} />
</Pressable>
```
