# COMPONENT & STYLING ARCHITECTURE (2025/Q4)

**Role:** Senior Lead Architect (IQ: 200)  
**Context:** Ultra-Strict FSD Implementation  
**Status:** **Non-Negotiable**

In this architecture, a component is not just a UI fragment; it is a **contract** between the Design System, the Data Layer, and the User. We do not paint pixels; we engineer interfaces.

Here are the **Immutable Laws** for creating UI in our ecosystem.

---

## Part 1: Component Architecture (The Logic)

### 1. Location Dictates Responsibility (The FSD Law)
Where you place the file determines what it is allowed to know.
*   **`shared/ui` (Atoms/Molecules):**
    *   **IQ Check:** Is it purely visual? (Button, Card, Input).
    *   **Rule:** **Zero business logic.** Cannot import from `features` or `entities`. Receives data via primitives or slots.
    *   **Naming:** `Button.tsx`, `Card.tsx` (PascalCase).
*   **`features/*/ui` (Organisms):**
    *   **IQ Check:** Does it solve a specific domain problem? (WeatherCard, LoginForm).
    *   **Rule:** Can use domain models. Connects to `model` (Zustand) via hooks.
*   **`widgets` (Templates):**
    *   **IQ Check:** Does it orchestrate multiple features? (DashboardLayout).
    *   **Rule:** Layout only. No direct API calls.

### 2. The "Props Explosion" Anti-Pattern
Passing 10+ props is a failure of abstraction.
*   **Bad:** `<Header name={...} age={...} role={...} onX={...} onY={...} />`
*   **Mandatory:** Use **Compound Components** or **Slots** for complex UI.
*   **Mandatory:** Pass Domain Objects (`user: User`) for Feature components, but generic props for Shared components.

### 3. Logic Decoupling (The "Humble Component")
The UI component should be "dumb."
*   **Forbidden:** Calling `axios`, `fetch`, or complex logic inside the component.
*   **Mandatory:** Extract logic to Custom Hooks (`useWeatherLogic`).
    *   *Component:* `(State) => UI`
    *   *Hook:* `(Events) => State`

### 4. Zero Layout Shift & Performance
*   **Skeleton:** Returning `null` or a spinner that collapses the layout is forbidden. You must render a Skeleton that matches the exact dimensions.
*   **Lists:** `FlashList` is mandatory. `ScrollView` is banned for dynamic lists.
*   **Images:** `expo-image` with `cachePolicy="memory-disk"` and `transition={200}` is mandatory.

---

## Part 2: Styling Architecture (The Visual Contract)

We use **NativeWind v4**. We do not write "styles"; we apply **tokens**.

### 1. The "No Magic Numbers" Rule
*   **Forbidden:** `width: 37`, `margin: 13`, `fontSize: 15`.
*   **Mandatory:** Use Tailwind tokens. `w-10` (40px), `m-4` (16px), `text-base` (16px).
*   **Why:** If the design system changes, we change the config, not 500 files.

### 2. Semantic Coloring (The Dark Mode Law)
Never use raw colors like `bg-white` or `text-black` for layout structures.
*   **Forbidden:** `bg-white text-black` (This breaks in Dark Mode).
*   **Mandatory:** Use Semantic Tokens defined in `tailwind.config.js`.
    *   `bg-card` (White in light, Slate-900 in dark).
    *   `text-foreground` (Black in light, White in dark).
    *   `text-muted-foreground` (Gray-500 in light, Gray-400 in dark).

### 3. The Safe Area Strategy
Never use padding for safe areas manually.
*   **Forbidden:** `pt-[40px]` (Assumes specific device notch).
*   **Mandatory:** Use NativeWind safe-area utilities: `pt-safe`, `mb-safe`.

### 4. Variant Management (CVA)
Do not use ternary operators for complex style switching. Use `class-variance-authority`.
*   **Bad:** `` className={`p-4 ${isActive ? 'bg-blue-500' : 'bg-gray-200'} ${isLarge ? 'p-8' : 'p-4'}`} ``
*   **Good:** `className={buttonVariants({ variant: 'primary', size: 'lg' })}`

---

## Part 3: The "Gold Standard" Implementation

**When the user runs `/createcomponent`, YOU MUST COPY THIS STRUCTURE EXACTLY.**

```tsx
/**
 * COMPONENT: [ComponentName]
 * LAYER: [shared/ui | features/x/ui]
 */

import React from 'react';
import { View, Text, Pressable } from 'react-native';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/shared/lib/utils'; // Ensure this path exists

// 1. VARIANT DEFINITIONS
const containerVariants = cva(
  "flex-row items-center justify-between rounded-xl p-4 transition-all",
  {
    variants: {
      variant: {
        default: "bg-card border border-border",
        ghost: "bg-transparent border-transparent",
        destructive: "bg-destructive border-destructive",
      },
      size: {
        sm: "h-12",
        md: "h-16",
        lg: "h-20",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "md",
    },
  }
);

// 2. TYPES
interface Props extends 
  React.ComponentPropsWithoutRef<typeof Pressable>,
  VariantProps<typeof containerVariants> {
  label: string;
  description?: string;
  leftElement?: React.ReactNode;
  rightElement?: React.ReactNode;
}

// 3. COMPONENT
export const [ComponentName] = ({ 
  label, 
  description,
  variant, 
  size, 
  leftElement,
  rightElement,
  className,
  ...props 
}: Props) => {
  return (
    <Pressable 
      className={cn(containerVariants({ variant, size }), className)}
      accessibilityRole="button"
      accessibilityLabel={label}
      accessibilityHint={description}
      {...props}
    >
      <View className="flex-row items-center gap-3">
        {leftElement && <View>{leftElement}</View>}
        
        <View className="flex-1">
          <Text className="text-base font-semibold text-foreground">
            {label}
          </Text>
          {description && (
            <Text className="text-sm text-muted-foreground mt-0.5">
              {description}
            </Text>
          )}
        </View>
      </View>

      {rightElement && <View>{rightElement}</View>}
    </Pressable>
  );
};
```

---

## Part 4: The Reviewer's Kill-List (Automatic Rejection)

If a Code Reviewer spots any of these, the PR is closed immediately.

1.  ❌ **Inline Objects:** `style={{ marginTop: 20 }}` -> **REJECT.**
2.  ❌ **Hardcoded Colors:** `text-[#333]` -> **REJECT.** (Use `text-foreground`).
3.  ❌ **Missing Dark Mode:** "I'll add it later" -> **REJECT.** (Add `dark:` classes now).
4.  ❌ **Logic in UI:** `useEffect(() => { fetch... })` -> **REJECT.** (Use a hook).
5.  ❌ **Missing Accessibility:** No `accessibilityLabel` or `Role` -> **REJECT.**
6.  ❌ **Pixel Values:** `w-[37px]` -> **REJECT.** (Use `w-9` or `w-10`).
7.  ❌ **Z-Index Hack:** `z-50` without a documented stacking context -> **REJECT.**
