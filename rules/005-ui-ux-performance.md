# UI, UX & Performance Guidelines

## 1. Styling
- **Engine:** NativeWind v4.
- **Rule:** No magic numbers in `style={{}}`. Use Tailwind classes (e.g., `p-4`, `gap-2`).
- **RTL:** Use logical properties (`text-start`, `ms-2` instead of `ml-2`).

## 2. Animation
- **Library:** Reanimated 3 ONLY.
- **Forbidden:** `Animated` API from React Native.
- **Requirement:** Layout Animations for mounting/unmounting.
- **Example:**
  ```tsx
  <Animated.View entering={FadeInUp} exiting={FadeOutDown}>
  ```

## 3. Lists
- **Mandatory:** FlashList for any list > 5 items.
- **Prop:** `estimatedItemSize` is required.

## 4. Images
- **Library:** expo-image.
- **Policy:** `cachePolicy="memory-disk"`.
- **UX:** Must use blurhash or placeholder transition.

## 5. Loading States
- **Rule:** Every screen must have a Reanimated Shimmer Skeleton for the initial load.
- **Metric:** First Meaningful Paint < 1.2s.
