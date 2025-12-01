# Coding Standards (Zero Tolerance)

## 1. i18n (Internationalization)
- **Rule:** Hardcoded strings are **FORBIDDEN**.
- **Rule:** Keys must be typed.
- **Example:**
  ```tsx
  // ✅ Allowed
  <Text>{t('home.welcome_message', { name })}</Text>
  
  // ❌ Forbidden (Compilation Error)
  <Text>Welcome back</Text>
  ```

## 2. Data & Validation (Zod)
- **Rule:** Never use raw API data in the UI.
- **Rule:** All external data must be parsed and transformed via Zod.
- **Rule:** Use `z.infer<>` for TypeScript types, never manual interfaces for API responses.
- **Example:**
  ```typescript
  const UserSchema = z.object({
    first_name: z.string(),
  }).transform(d => ({ firstName: d.first_name }));

  // API Call
  const fetchUser = async () => 
    api.get('/user').then(res => UserSchema.parse(res.data));
  ```

## 3. Async & Error Handling
- **Rule:** Promises must be handled. No floating promises.
- **Rule:** Use `try/catch` in async functions or return the promise to the caller.
- **Rule:** API errors must be mapped to user-friendly messages, not raw `500` texts.

## 4. Accessibility (a11y) - MANDATORY
- **Rule:** Every interactive element must have `accessibilityLabel` and `accessibilityRole`.
- **Rule:** Tests must target a11y labels, not implementation details.
- **Rule:** Minimum touch target size is 44x44px.

## 5. Type Safety
- **Rule:** No `any`. No `as unknown as`.
- **Rule:** Use `satisfies` operator for stricter type checking on constants.
- **Rule:** Generic naming (T, K, V) is allowed only in utility types. In business logic, use descriptive names (`TUser`, `TResponse`).

## 6. Naming Conventions
- **Files:** `PascalCase.tsx` for components, `camelCase.ts` for utils/hooks.
- **Folders:** `kebab-case` for features (`features/user-profile`).
- **Booleans:** Must start with `is`, `has`, `should`, `can`.
- **Event Handlers:** Props start with `on` (`onPress`), Handlers start with `handle` (`handlePress`).

## 7. Documentation
- **Rule:** Exported functions and components must have JSDoc.
- **Rule:** Code should be self-documenting; use comments for "WHY", not "WHAT".
