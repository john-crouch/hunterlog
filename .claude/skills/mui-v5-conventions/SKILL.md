---
name: mui-v5-conventions
description: React 18 + TypeScript + Material UI v5 implementation guide. Use when implementing or refactoring UI components with MUI v5, ensuring consistent styling, accessibility, type safety, and adherence to MUI best practices. Covers component patterns, styling conventions, accessibility requirements, and quality standards for production-ready React applications.
---

# MUI v5 Conventions

Guide for implementing clean, consistent, and accessible React 18 + TypeScript UI using Material UI v5.

## Primary Goals

* Produce clean, consistent UI that matches MUI best practices
* Minimize visual regressions during refactors
* Keep code readable and type-safe

## Hard Rules

1. **React 18 functional components** - Use functional components with TypeScript types for props and API shapes
2. **Prefer MUI components** - Use MUI components over custom HTML unless there is a clear reason
3. **Theme-based values** - Use theme spacing, palette, and typography instead of magic numbers
4. **MUI v5 styling only** - Do not introduce new styling libraries (styled-components, emotion directly, etc.)
5. **No inline styles** - Except trivial one-offs. Prefer `sx` prop for component-local styling
6. **Composition over duplication** - Extract shared UI patterns into reusable components

## Styling Conventions

### When to Use Each Styling Method

* **`sx` prop** - For local styles and layout tweaks on individual components
* **`styled()`** - Only when the style is reused, complex, or benefits from component encapsulation
* **Theme values** - Always use `theme.spacing(n)`, palette colors, and typography variants

### Spacing and Layout

* Use `sx` shorthand for spacing: `p: 2`, `mt: 1`, `mx: 'auto'`
* Use theme spacing scale consistently: `theme.spacing(1)` = 8px (default)
* Avoid hard-coded pixel values

### Colors and Typography

* Use theme palette: `color: 'primary.main'`, `bgcolor: 'background.paper'`
* Use typography variants: `variant="h1"`, `variant="body2"`
* Avoid hard-coded colors like `#FF0000` or `rgb(255, 0, 0)`

### Consistency

* Ensure consistent spacing scale, border radii, and shadow usage across components
* Match existing patterns in the codebase before introducing new patterns

## Component Patterns

### Forms

* Use `TextField`, `Select`, `Autocomplete`, `FormControl`, `FormHelperText` consistently
* Always provide labels for form fields
* Use controlled components with proper state management

**Example:**
```typescript
<TextField
  label="Email"
  value={email}
  onChange={(e) => setEmail(e.target.value)}
  error={!!emailError}
  helperText={emailError}
  fullWidth
/>
```

### Validation

* Show errors using MUI `error` prop and `helperText`
* Keep error messages short and specific
* Validate on blur or submit, not on every keystroke (unless appropriate)

### Dialogs

* Use `Dialog`, `DialogTitle`, `DialogContent`, `DialogActions`
* Keep actions right-aligned in `DialogActions`
* Provide a clear close mechanism (X button or Cancel action)

**Example:**
```typescript
<Dialog open={open} onClose={handleClose}>
  <DialogTitle>Confirm Action</DialogTitle>
  <DialogContent>
    Are you sure you want to proceed?
  </DialogContent>
  <DialogActions>
    <Button onClick={handleClose}>Cancel</Button>
    <Button onClick={handleConfirm} variant="contained">Confirm</Button>
  </DialogActions>
</Dialog>
```

### Tables

* Use MUI `Table`, `TableHead`, `TableBody`, `TableRow`, `TableCell` components
* Avoid heavy custom table CSS
* Use `stickyHeader` for long tables if needed

### Layout

* **Stack** - For simple vertical or horizontal layouts with consistent spacing
* **Box** - For generic containers with `sx` styling
* **Grid** - For responsive grid layouts
* **Container** - For page-level max-width containers

Use responsive props for breakpoints:
```typescript
<Box sx={{ display: { xs: 'block', md: 'flex' } }}>
```

### Icons

* Use `@mui/icons-material` for all icons
* Keep icon usage consistent (same size, color scheme)
* Provide accessible labels for icon-only buttons

**Example:**
```typescript
<IconButton aria-label="delete" onClick={handleDelete}>
  <DeleteIcon />
</IconButton>
```

## Accessibility

Ensure all components are accessible:

1. **Labels** - Provide labels for all inputs, buttons, and icon buttons
2. **ARIA attributes** - Set `aria-label` when visual label is missing
3. **Interactive elements** - Avoid clickable divs. Use `ButtonBase` or `Button`
4. **Keyboard navigation** - Ensure focus states are visible and keyboard navigation works
5. **Contrast** - Ensure reasonable color contrast (WCAG AA minimum)
6. **Focus indicators** - Do not remove focus outlines without providing alternatives

**Bad:**
```typescript
<div onClick={handleClick}>Click me</div>
```

**Good:**
```typescript
<Button onClick={handleClick}>Click me</Button>
```

For detailed accessibility patterns, see [references/accessibility.md](references/accessibility.md).

## Component Organization

* **Shared components** - Place in `components/common/` or similar
* **Feature components** - Organize by feature domain
* **Reusable patterns** - Extract when used 2+ times

## Deliverables for Each Change

1. **Updated component code** - With proper TypeScript types for props
2. **Shared components** - New reusable components in sensible folders (e.g., `components/common`)
3. **Storybook stories** - Add or update stories if Storybook is present
4. **Tests** - Update or add minimal tests for critical behavior if tests exist

## Pre-Finalization Checklist

Before finalizing any component changes:

1. **Run linters** - `npm run lint` or equivalent if available
2. **Type check** - `npm run typecheck` or `tsc --noEmit` if available
3. **Visual verification** - Confirm visual changes are intentional, minimal, and consistent with the rest of the UI
4. **Accessibility check** - Verify keyboard navigation, focus states, and labels
5. **Responsive check** - Test on different screen sizes if layout changes were made

## Additional Resources

For detailed component patterns and examples:
* See [references/component-patterns.md](references/component-patterns.md) for comprehensive patterns
* See [references/accessibility.md](references/accessibility.md) for WCAG guidelines and MUI-specific accessibility patterns
