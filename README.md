# Code Architecture Guide

A collection of frontend development and UI/UX patterns generalized from a real-world React/TypeScript project. These guidelines are intended to help maintain consistency, improve developer productivity, and foster accessible user experiences in modern web applications for the colomo.io organization.

## Table of Contents

1. [Project Architecture](#project-architecture)
2. [File Naming Conventions](#file-naming-conventions)
3. [React Component Patterns](#react-component-patterns)
4. [Custom Hooks](#custom-hooks)
5. [Routing](#routing)
6. [Styling and Tailwind CSS](#styling-and-tailwind-css)
7. [Design System Overview](#design-system-overview)
8. [Internationalization (i18n)](#internationalization-i18n)
9. [Animations](#animations)
10. [Testing](#testing)
11. [Type Safety Patterns](#type-safety-patterns)
12. [UI/UX Design Principles](#uiux-design-principles)
13. [Quick Reference](#quick-reference)

---

## Project Architecture

Organize the codebase by domain instead of by technology. This keeps related logic, types, and assets together and makes the structure easier to navigate as the application grows.

```
src/
  [domain]/           # e.g. workout/
    domain/           # Types, business logic, utilities
    repositories/     # Data access layer
    db/               # Static data (JSON)
    ui/               # Components and hooks
      hooks/          # Domain-specific hooks
    assets/           # Domain-specific assets
  core/               # Shared infrastructure
    i18n/             # Internationalization
    ui/               # Design system components
  components/         # Feature components (page-level)
  pages/              # File-based routes (e.g. TanStack Router)
  utility/            # General utilities
```

---

## File Naming Conventions

| Type         | Convention          | Example                       |
| ------------ | ------------------- | ----------------------------- |
| Components   | `kebab-case.tsx`    | `workout-plan-overview.tsx`   |
| Hooks        | `use-kebab-case.ts` | `use-gloria-engine.ts`        |
| Domain types | `PascalCase.ts`     | `Workout.ts`                  |
| Tests        | `*.test.tsx`        | `celebration-screen.test.tsx` |
| Stories      | `*.stories.tsx`     | `exercise-row.stories.tsx`    |
| Utilities    | `kebab-case.ts`     | `formatter.ts`                |

Components use **PascalCase** for the exported function name. Always use named exports; avoid default exports to keep imports explicit.

---

## React Component Patterns

### Standard Component Template

```typescript
import type { ReactNode } from "react";

interface MyComponentProps {
  title: string;
  children: ReactNode;
  onAction?: () => void;
}

export function MyComponent({ title, children, onAction }: MyComponentProps) {
  return (
    <div className="flex flex-col gap-4" data-testid="my-component">
      <Heading size="2xl" weight="bold" t="my.translation.key" />
      {children}
    </div>
  );
}
```

**Rules**

- Named exports only — no default exports
- Use an `interface` for props (not `type`)
- Destructure props in the function signature
- Add `data-testid` on interactive/testable elements
- Separate type imports (`import type { X } from "path"`)
- Do not use inline styles; rely on utility classes (e.g., Tailwind)

---

## Custom Hooks

Keep business logic out of components by encapsulating it in reusable hooks. Follow the `use-kebab-case` file convention.

```typescript
// src/workout/ui/hooks/use-workout-timer.ts
export function useWorkoutTimer(duration: number) {
  const [remaining, setRemaining] = useState(duration);
  const [isRunning, setIsRunning] = useState(false);
  // ...logic
  return { remaining, isRunning, pause, play, reset };
}
```

---

## Routing

For projects using TanStack Router or similar file-based routers:

- Place route files under `src/pages/`
- Use `loader` for server-side data fetching and `Route.useLoaderData()` for access
- Dynamic routes follow a `$paramName.tsx` convention
- Avoid manually editing generated route trees

```typescript
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/my-route")({
  loader: async () => {
    return { data: await fetchData() };
  },
  head: () => ({
    meta: [{ title: "Page Title" }],
  }),
  component: MyRoutePage,
});

function MyRoutePage() {
  const { data } = Route.useLoaderData();
  return <div>{/* page content */}</div>;
}
```

---

## Styling and Tailwind CSS

When using Tailwind CSS (v4 or later) with a custom theme:

- Define colors, fonts, and utilities in a central styles file (e.g., `src/styles.css`)
- Use the `cn()` utility (classnames) for conditional class names
- Create custom utilities for shadows, text effects, etc.

```tsx
<div className="bg-primary border border-black shadow-solid-sm rounded-xl p-4">
  <Heading size="2xl" weight="bold" className="text-shadow">
    Let's Move!
  </Heading>
</div>
```

Avoid inline styles. Keep design tokens in documentation (e.g., `docs/tokens.md`) for reference.

---

## Design System Overview

Shared UI components should live in a `core/ui/` directory and expose consistent props.

### Text Components

Use a `TextProps` design for typography:

```tsx
<Heading size="3xl" weight="bold" t="workout.title" />
<P t={{ id: "celebration.exercises", values: { count: 5 } }} />
<Text size="sm" color="secondary">Supporting text</Text>
```

**TextProps options**

- `size`: xs, sm, base, lg, xl, 2xl, 3xl, 4xl, 5xl, 6xl
- `weight`: thin, extralight, light, normal, medium, semibold, bold, extrabold, black
- `color`: primary, secondary, destructive, warning, text
- `t`: translation key or object `{ id, values }`
- `trim`: boolean for overflow control

### Layout & Interactive Components

- `Box` — generic container
- `Stack` — flex column layout
- `ContentBox` — padded section wrapper
- `Button` / `LinkButton` — actionable elements (router-aware when needed)

Design system components should encapsulate common styling and accessibility patterns.

---

## Internationalization (i18n)

Workflow for adding translations:

1. Add keys to locale files (e.g., `src/core/i18n/en.ts`, `es.ts`)
2. Use flat dot notation for keys (`"workout.timer.pause"`)
3. Consume translations via the `t` prop on text components or `useIntl()` hook

Locale detection can be cookie-based or header-based; implement a centralized loader to provide the current locale data.

---

## Animations

Use Framer Motion or a similar library for declarative animations:

```tsx
import { motion } from "framer-motion";

<motion.div
  initial={{ opacity: 0, scale: 0.8 }}
  animate={{ opacity: 1, scale: 1 }}
  transition={{ duration: 0.5, ease: "easeOut" }}
>
  {/* Content */}
</motion.div>;
```

Keep animations subtle, performant, and aligned with the app's tone.

---

## Testing

### E2E (e.g., Playwright)

```typescript
import { test, expect } from "@playwright/test";

test("user can start workout", async ({ page }) => {
  await page.goto("/workout/abc-123");
  await page.getByTestId("start-button").click();
  await expect(page.getByTestId("timer")).toBeVisible();
});
```

### Unit (e.g., Vitest)

Co-locate tests with implementation files:

```typescript
import { describe, it, expect } from "vitest";

describe("myFunction", () => {
  it("returns expected result", () => {
    expect(myFunction(input)).toBe(expected);
  });
});
```

### Storybook (CSF3)

```typescript
import type { Meta, StoryObj } from "@storybook/react";
import { MyComponent } from "./my-component";

const meta = {
  component: MyComponent,
} satisfies Meta<typeof MyComponent>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: { title: "Example" },
};
```

Add `data-testid` attributes to facilitate both unit and E2E tests.

---

## Type Safety Patterns

```typescript
// Type guard
export const isWorkoutPlan = (
  plan: WorkoutPlan | undefined,
): plan is WorkoutPlan => plan !== undefined;

// Union types for state
type WorkoutStatus = "overview" | "working-out" | "finished";

// Type-only imports
import type { Workout, Exercise } from "../domain/Workout";
```

Favor explicit types and leverage TypeScript for compile-time safety.

---

## UI/UX Design Principles

These guidelines support inclusive, consistent, and scalable interface design:

- Prioritize accessibility from the start (WCAG 2.1/2.2 AA compliance)
- Use design tokens and a component library to enforce visual consistency
- Conduct user research and usability testing to validate decisions
- Maintain responsive layouts and progressive enhancement
- Document components with usage examples and accessibility notes
- Apply a systematic approach to color, typography, and spacing
- Build for performance and consider animation impact on CPU/battery
- Collaborate closely with developers during design handoff

This section is intended for frontend engineers and designers working together; adapt as needed for your context.

---

## Quick Reference

| Pattern    | Convention                                         |
| ---------- | -------------------------------------------------- |
| Exports    | Named only, no defaults                            |
| Props      | `interface`, not `type`                            |
| Files      | kebab-case                                         |
| Components | PascalCase names                                   |
| Hooks      | `use-` prefix, kebab-case files                    |
| Styling    | Utility classes (Tailwind), `cn()` for conditional |
| i18n       | `t` prop on Text components                        |
| Testing    | `data-testid` on interactive elements              |
| State      | Local `useState`, hooks for complex logic          |
| Data       | Router loaders, repository pattern                 |
| Animations | Declarative (Framer Motion)                        |

---
