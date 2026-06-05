# Roadmap

Prioritized backlog for Perf Fiscal. Tags: **P0–P2** priority · effort (S/M/L) · impact (low/med/high).
Pick from the top of each section — items are roughly ordered by ROI.

## 🚀 Quick wins — infra & DX

- [ ] **CI on GitHub Actions** — P0 · S · high
  - No `.github/workflows/` exists today (only an unmerged `ci/tests-workflow` branch).
  - Run `npm ci && npm run build && npm run lint && npm test` on every PR (Node 18/20/22 matrix).
  - This would have caught the broken Vue rules before merge.
- [ ] **Add `meta.docs.url` to every rule** — P1 · S · med
  - None of the 17 rules expose `url`, so IDEs / `eslint --help` can't deep-link to `docs/rules/*.md`.
  - Set it once in the `createRule` factory (`src/utils/create-rule.ts`) from the rule name.
- [ ] **`all` / `strict` preset** — P1 · S · med
  - Only `recommended` and `vue` exist in `src/index.ts`. Add an `all` preset (every rule as `error`)
    and consider `flat/all` for parity.
- [ ] **Decide fate of unwired scaffolding** — P1 · S · med
  - `src/integrations/cicd.ts`, `src/integrations/lsp.ts`, `src/reporting/json-report.ts` are not
    exported from `index.ts` and have no tests, yet ship in `dist`.
  - Either wire them up (a real `--format json` reporter / GitHub PR comment step) **or** drop them from the build.

## 🧩 New rules — fill performance gaps

Each item = rule + tests + `docs/rules/<name>.md` + registration in `src/index.ts` (and a preset).

- [ ] **`no-await-in-loop`** — P0 · M · high
  - Sequential `await` inside a loop that could be `Promise.all`. Complements `prefer-promise-all-settled`.
- [ ] **`no-accumulator-spread`** — P1 · M · high
  - `acc = { ...acc, k }` / `[...acc, x]` inside `reduce`/loops → silent O(n²). Fits the `no-quadratic-complexity` theme.
- [ ] **`no-array-index-key`** (React) — P1 · S · med
  - `key={index}` in lists; classic re-render / reconciliation pitfall.
- [ ] **`require-effect-cleanup`** (React) — P1 · M · high
  - `useEffect` adding `addEventListener` / `setInterval` / subscriptions without a cleanup return → leaks.
- [ ] **`prefer-set-has`** — P2 · M · med
  - `array.includes(x)` inside a loop → suggest a `Set`. O(n²) → O(n).
- [ ] **`no-regex-in-hot-path`** — P2 · M · med
  - Regex literal recompiled inside a loop / render. Reuses `no-redos-regex` infra.
- [ ] **`prefer-lazy-component`** (React) — P2 · M · med
  - Static import of a heavy component that should be `React.lazy` / dynamic `import()`.
- [ ] **`vue-prefer-shallow-ref`** (Vue) — P2 · M · med
  - Large/immutable structures in `ref()`/`reactive()` where `shallowRef`/`shallowReactive` suffices.
    Extends `vue-optimize-reactivity` + `src/utils/vue-ast-utils.ts`.

## 🏗️ Bigger bets

- [ ] **Autofix / suggestions where missing** — P1 · M · high
  - 5 rules set `fixable`; several report-only rules could add safe `suggest` fixers
    (e.g. `no-inline-context-value` → wrap value in `useMemo`).
- [ ] **Unblock the Rust core** — P2 · L · med
  - The binary doesn't compile (SWC/serde conflict), so the JS fallback is always used.
  - Pragmatic path from `docs/rust-core-improvements.md`: pin a known-good `Cargo.lock`, or migrate to `tree-sitter`.
- [ ] **Additional frameworks** — P2 · L · med
  - Svelte 5 (runes) and Solid have reactivity pitfalls analogous to Vue; reuse the `vue-ast-utils` pattern.

## ✅ Recently completed

- [x] Repaired 3 broken Vue rules (invalid grouped `:exit` selectors, infinite `parent` recursion,
      duplicate diagnostics); lint 7→0; added missing `no-inline-context-value` docs. *(PR #16)*
- [x] Wired the shared strictness / file-skipping helper (`src/utils/rule-options.ts`) into configurable rules,
      updated schemas + docs, and extended regression tests.
