# `perf-fiscal/no-inline-context-value`

Disallow passing an inline object or array literal to a React `Context.Provider` `value` prop. The rule inspects every `<Something.Provider>` element and flags a `value` that is a freshly-created object/array, since a new reference on each render forces every consumer to re-render.

## Why it matters

React compares the Context `value` by identity. When the provider receives an inline literal such as `value={{ user, role }}` or `value={[a, b]}`, a brand-new reference is allocated on every parent render. Each render therefore invalidates the Context for **all** consumers, regardless of whether the underlying data actually changed. In large trees this cascades into widespread, avoidable re-renders.

Hoisting the value into a `useMemo` (or out of the component when it is static) keeps the reference stable so consumers only re-render when the data genuinely changes.

## Default behavior

The rule reports when the `value` prop of a `*.Provider` element is, after unwrapping parentheses and `as` assertions, an inline `ObjectExpression` or `ArrayExpression`. References to memoized or hoisted variables are not flagged.

## Options

This rule has no options.

## Examples

### ❌ Problem

```tsx
function App({ name, role, refetch }) {
  return (
    <UserContext.Provider value={{ name, role, refresh: () => refetch() }}>
      <Dashboard />
    </UserContext.Provider>
  );
}
```

### ✅ Fix

```tsx
function App({ name, role, refetch }) {
  const providerValue = useMemo(
    () => ({ name, role, refresh: () => refetch() }),
    [name, role, refetch]
  );

  return (
    <UserContext.Provider value={providerValue}>
      <Dashboard />
    </UserContext.Provider>
  );
}
```

## When not to use it

Disable the rule for providers whose consumers are trivial or never re-render in practice, or when the surrounding component itself only renders once (e.g. an app-root provider mounted a single time). In those cases the extra `useMemo` adds noise without a measurable win.
