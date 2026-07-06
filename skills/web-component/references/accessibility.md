# Accessibility

Every component meets these before it's done.

## Semantic HTML

Use the correct element for the job — never a `div` for something interactive.

```tsx
// ✅
<button onClick={onPress}>Submit</button>
<a href="/path">Link</a>
<input type="text" />
<nav aria-label="Main navigation">…</nav>

// ❌
<div onClick={onPress}>Submit</div>
<div className="link">Link</div>
```

## Keyboard navigation

- All interactive elements reachable via Tab.
- Buttons activate on Enter and Space; links on Enter.
- Custom widgets (dropdown, modal, tabs) follow the matching WAI-ARIA pattern.

## Focus visibility

Every focusable element needs a **visible focus ring**. Never leave bare `outline-none`.

```tsx
// ✅ replace outline-none with a ring
'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-border-focus focus-visible:ring-offset-2'

// ❌ removes focus with no replacement
'focus:outline-none'
```

## Labels & ARIA

```tsx
// Form input — associate the label
<label htmlFor={inputId}>Email</label>
<input id={inputId} />

// Icon-only button — always aria-label
<button aria-label="Close dialog"><X className="h-4 w-4" aria-hidden="true" /></button>

// Decorative icon — aria-hidden
<CheckCircle className="h-4 w-4 text-success" aria-hidden="true" />

// Loading — aria-busy
<button aria-busy={loading} disabled={loading}>…</button>

// Error message — role + wiring
<span id={errorId} role="alert">Invalid email</span>
<input aria-describedby={errorId} aria-invalid="true" />
```

## Color contrast (WCAG AA)

- Normal text on background: ≥ 4.5:1.
- Large text (18px+, or 14px+ bold): ≥ 3:1.
- UI components and focus indicators: ≥ 3:1.
- **Never use color alone** to convey state — pair it with an icon or text.

## Motion

Respect reduced-motion preferences.

```tsx
'transition-colors motion-reduce:transition-none'
'motion-safe:hover:scale-105'
```
