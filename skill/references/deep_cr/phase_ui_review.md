# UI Design Review

## What to Look For

### Visual Hierarchy and Layout
- Content is organized with clear visual hierarchy — headings, spacing, grouping communicate structure
- Layout is consistent with the rest of the application (spacing units, grid usage, alignment)
- Interactive elements are visually distinguishable from static content
- Z-index management is intentional, not hacked with arbitrarily high values

### Accessibility
- ARIA labels and roles on interactive elements (buttons, form controls, custom widgets)
- Keyboard navigation works — focusable elements have visible focus indicators, tab order is logical
- Color contrast meets WCAG AA minimums (4.5:1 for body text, 3:1 for large text)
- Screen reader compatibility — semantic HTML elements used appropriately (nav, main, article, aside, not div soup)
- Images have alt text (decorative images use empty alt, not missing alt)
- Form inputs have associated labels (not just placeholder text)

### Responsive Design
- Layout adapts at standard breakpoints — content doesn't overflow or become unusable
- Touch targets are at least 44x44px on mobile
- Text remains readable without horizontal scrolling on small screens
- Images and media scale appropriately (no fixed pixel widths that break on mobile)

### String Quality
- No typos, misspellings, or grammatical errors in user-facing text
- Tone is consistent with the rest of the application
- Strings are externalized for i18n if the project supports localization
- Error messages are helpful — they tell the user what happened and what to do about it

### State Handling
- **Loading states**: Users see feedback while data is being fetched (spinners, skeletons, progress bars)
- **Empty states**: Blank screens are never shown — empty states explain what's missing and what to do
- **Error states**: Errors are shown inline where possible, with retry options where applicable
- **Disabled states**: Disabled controls explain why they're disabled (tooltip, help text)
- **Optimistic updates**: If used, rollback behavior on failure is correct

### Design System Consistency
- Uses existing component library components rather than custom implementations
- Colors, typography, spacing match the design system tokens
- Icon usage is consistent (same icon set, same sizes)
- No one-off styles that duplicate or conflict with existing patterns

### Form UX
- Inline validation shows errors near the relevant field, not just at the top of the form
- Error messages appear on blur or submit, not on every keystroke
- Submit button shows loading state during submission
- Form preserves input on validation failure (doesn't clear fields)
- Required vs optional fields are clearly indicated

### Navigation and Information Architecture
- Navigation patterns are consistent with the rest of the application
- Breadcrumbs or back navigation is available where needed
- Deep links work — pages are directly accessible via URL
- Page titles and browser tab text are descriptive

## Common Issues

- **Div soup**: Using `<div>` and `<span>` for everything instead of semantic HTML. Makes the page inaccessible and hard to maintain.
- **Click handlers on non-interactive elements**: Adding onClick to divs/spans instead of using buttons or links. Breaks keyboard navigation and screen readers.
- **Placeholder-only labels**: Using placeholder text as the only label for form inputs. Placeholder disappears on focus, leaving the user confused.
- **Fixed dimensions**: Hardcoded pixel widths/heights that break on different screen sizes or content lengths.
- **Missing loading states**: Showing stale data or nothing while fetching. Users think the app is broken.
- **Alert/confirm for everything**: Using browser alert() or confirm() instead of inline UI for user feedback.
- **Inconsistent spacing**: Mixing different spacing values or using magic numbers instead of design system tokens.
- **Text truncation without affordance**: Truncating text with ellipsis but no way to see the full content (tooltip, expand, detail view).

## Severity Guidance

**Critical:**
- Interactive elements that are completely inaccessible (no keyboard access, no screen reader announcement)
- Broken layout that makes content unreadable or unusable
- Missing error states that leave users stuck with no feedback
- User-facing text with offensive or seriously misleading content

**Moderate:**
- Accessibility issues that degrade but don't block usage (missing ARIA labels, poor contrast)
- Inconsistent patterns that confuse users (different interaction models for similar actions)
- Missing loading or empty states (functional but poor UX)
- Responsive breakage on common screen sizes

**Mild:**
- Minor spacing or alignment inconsistencies
- Slightly inconsistent tone in copy
- Missing hover states or micro-interactions
- Semantic HTML improvements that don't affect current functionality
