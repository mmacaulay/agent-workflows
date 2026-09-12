---
name: widget-scaffold
description: Scaffold a new dashboard widget - card component, settings screen, type additions, and test stubs - following the conventions this app family shares. Use when the user wants to add a new widget type.
disable-model-invocation: true
---

Both dashboard apps share a file layout for widgets, so this scaffolds the same shape in either.

Ask the user for the widget name in PascalCase, for example `ShoppingList`.

Before creating anything, read an existing widget in **this** repo and follow it rather than the
names below: the apps ship different widgets, so the nearest example differs. Look in
`src/components/` for a `*Card.tsx` and take the closest one.

Create:

**Component** - `src/components/<Name>Card.tsx`
A functional component stub taking the standard card props. Named export.

**Settings screen** - `app/settings/widget/<name-lowercase>.tsx`
A settings screen stub using the existing layout. This is a route file, so default export per
Expo Router conventions.

**Tests**
- `__tests__/<Name>Card.test.tsx` - component test using jest-expo and
  @testing-library/react-native
- `__tests__/<Name>Settings.test.tsx` - settings screen test

**Types** - add the widget type in `src/types/`, following the existing widget config pattern in
`DashboardConfig`. Per-widget visible configuration belongs in `DashboardConfig`, not
`AppSettings`. Check the repo's own `AGENTS.md`, which states this and may carry app-specific
exceptions.

Do **not** wire the widget into any template, store, or settings navigation. Leave that to the
user once the scaffold is in place, and say plainly what still needs doing.

A widget that exists but no template places is unreachable, so if the user stops after
scaffolding, make sure they know the widget is not yet live.
