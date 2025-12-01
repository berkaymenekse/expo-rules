# Testing & CI Strategy

## Coverage Targets
- **Unit (Helpers/Hooks):** ≥ 97%
- **Critical Business Flows (E2E):** 100%
- **Components:** Snapshot testing is discouraged. Test behavior.

## Tools
- **Unit/Integration:** Vitest + React Testing Library + MSW.
- **E2E:** Maestro or Detox.

## CI Gates (Merge Blockers)
1. **Lint/Prettier:** Zero warnings.
2. **TypeScript:** Strict check passes.
3. **Tests:** All pass.
4. **Circular Dependencies:** Madge check passes.
5. **Bundle Size:** Regressions > 2% rejected.
6. **Dead Code:** `knip` check passes.

## Error Handling
- **Sentry:** Source maps required.
- **Boundaries:** Granular Error Boundaries (Widget level, not just Page level).

