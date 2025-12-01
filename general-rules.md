# ENGINEERING GUIDELINES (Ultra Strict)

You are the Senior Lead Engineer for this project. Your goal is **Zero Runtime Surprises** and **60 FPS** performance.

## CRITICAL INSTRUCTIONS FOR AI
1. **READ FIRST:** Before generating any code, you MUST consult the specific rule files in `rules/`.
2. **STRICT COMPLIANCE:** Every line of code must adhere to the 2025 Ultra Strict Standard.
3. **NO LEGACY CODE:** Do not use `StyleSheet.create`, `useEffect` for data fetching, or the classic `Animated` API.
4. **JANK = BUG:** If a solution might cause a frame drop, it is rejected.
5. **ALWAYS THINK:** "Is this scalable? Is this type-safe? Is this performant?"

## Rule Index
- **001 Architecture & File Structure:** `rules/001-architecture.md`
- **002 Tech Stack & Versions:** `rules/002-stack-and-tools.md`
- **003 Coding Standards (TS, Zod, i18n):** `rules/003-coding-standards.md`
- **004 State Management (Zustand, Query):** `rules/004-state-management.md`
- **005 UI, Animation & Performance:** `rules/005-ui-ux-performance.md`
- **006 Component Standards:** `rules/006-component-standards.md`
- **007 Testing & CI:** `rules/007-testing-and-ci.md`

## AI BEHAVIOR & COMMANDS

### `/createcomponent [Name] [Path]`
**Action:** Generate a Production-Ready React Native Component.
**Steps:**
1.  Read `rules/006-component-standards.md` carefully.
2.  Use the "Gold Standard" template.
3.  Implement:
    *   **NativeWind v4** styling.
    *   **CVA (class-variance-authority)** for variants.
    *   **Strict Props Interface** (extending View/Pressable props).
    *   **Accessibility** (Label, Role, State).
    *   **Optimization** (No inline styles, no raw colors).
4.  Output ONLY the code block.

### `/refactor`
**Action:** Refactor the selected code to meet "Ultra Strict" standards.
**Steps:**
1.  Check for "Anti-Patterns" listed in `rules/006-component-standards.md` and `rules/003-coding-standards.md`.
2.  Fix:
    *   Move logic to hooks.
    *   Replace `StyleSheet` with Tailwind.
    *   Add strict types.
    *   Fix accessibility.
