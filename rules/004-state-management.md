# State Management Guidelines

## 1. TanStack Query (Server State)
- **Config:**
  ```typescript
  staleTime: 5 * 60_000, // 5 minutes
  gcTime: 30 * 60_000,   // 30 minutes
  placeholderData: keepPreviousData,
  ```
- **Usage:** Always wrap in custom hooks (e.g., `useWeatherQuery`). **Never** use `useQuery` directly in UI components.
- **Rule:** If data comes from an API, it is Server State. Do not store it in Zustand.

## 2. Zustand (Client State)
- **Pattern:** Store actions inside the store.
- **Performance:** MUST use `useShallow` or atomic selectors.
- **Rule:** Only use for global UI state (theme, sidebar status) or complex flows (multi-step form).
- **Example:**

  ```typescript
  // ❌ Bad (Re-renders on any store change)
  const { toggleUnit } = useWeatherStore();

  // ✅ Good (Atomic Selector)
  const toggleUnit = useWeatherStore((s) => s.actions.toggleUnit);

  // ✅ Good (Shallow Comparison)
  const userData = useUserStore(useShallow((s) => ({ name: s.name, role: s.role })));
  ```

- **Persistence:** Use MMKV adapter.
