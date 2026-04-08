# Accessibility Standards

WCAG 2.1 Level AA compliance for TYPO3 sitepackage frontend code.

## Table of Contents

1. [Language and Page Metadata](#language-and-page-metadata) -- WCAG 3.1.1, 2.4.2
2. [Page Structure and Landmarks](#page-structure-and-landmarks) -- WCAG 1.3.1, 2.4.1
3. [Headings and Content Order](#headings-and-content-order) -- WCAG 2.4.6, 1.3.2
4. [Links](#links) -- WCAG 2.4.4, 1.4.1
5. [Buttons](#buttons) -- WCAG 4.1.2, 1.3.1
6. [Color and Contrast](#color-and-contrast) -- WCAG 1.4.3
7. [CSS Units and User Preferences](#css-units-and-user-preferences) -- WCAG 1.4.4
8. [Focus Management](#focus-management) -- WCAG 2.4.7, 2.4.3, 2.1.2
9. [ARIA Reference](#aria-reference) -- WCAG 4.1.2
10. [Automated Accessibility Testing](#automated-accessibility-testing)
11. [Content Element Checklist](#content-element-checklist)

Cross-references to dedicated pattern files:
- `patterns-skiplinks.md` -- Skip link navigation
- `patterns-clickable-cards.md` -- Clickable card patterns
- `patterns-accessible-navigation.md` -- Navigation patterns
- `patterns-disclosure-widget.md` -- Disclosure and accordion
- `patterns-accessible-forms.md` -- Form patterns
- `patterns-accessible-filter.md` -- Filtering and tables
- `patterns-responsive-tables.md` -- Mobile table patterns

---

## Language and Page Metadata

### Natural Language

Set `lang` on `<html>` -- affects screen reader pronunciation, hyphenation, quotation marks, and translation tools. In TYPO3, this is handled via site configuration.

```html
<html lang="{siteLanguage.locale.languageCode}">
```

For inline foreign-language text, set `lang` on the containing element:

```html
<p>The term <span lang="ja-Latn">Kaizen</span> means continuous improvement.</p>
```

Use sparingly -- frequent voice profile switches interrupt reading flow. Well-established loanwords (Download, Workshop, Link) don't need it.

### Page Title

Every page must have a unique, descriptive `<title>`. In TYPO3, this comes from `config.pageTitleFirst = 1` and the page title field.

Rules:
- **Unique per page** -- never the same title on different pages
- **Concise** -- under 60 characters
- **Page name first, then site name**: `Products - Shop Name` (not `Shop Name - Products`)
- **Context-dependent information** when relevant:

```html
<!-- Checkout step -->
<title>Checkout (step 3 of 4) - Shop Name</title>
<!-- Form errors -->
<title>2 errors - Contact - Site Name</title>
<!-- Search results -->
<title>21 results for "term" - Site Name</title>
<!-- Paginated results -->
<title>Page 2 - Products - Site Name</title>
```

### Viewport

Only this viewport meta tag is allowed:

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

**Never use:**
- `user-scalable=no` -- disables zoom for low-vision users
- `maximum-scale=1` -- disables zoom in some browsers
- Fixed width values like `width=500`

---

## Page Structure and Landmarks

### Landmarks

Every page layout must contain these semantic regions:

```html
<header class="main-header" id="main-header">        <!-- banner -->
<nav aria-label="Main navigation">                     <!-- navigation -->
<main id="main-content">                               <!-- main -->
<aside>                                                 <!-- complementary -->
<footer class="main-footer" id="main-footer">          <!-- contentinfo -->
```

### Navigation Landmarks

Significant groups of links must be wrapped in `<nav>` with a label:

```html
<nav aria-label="{f:translate(key: 'mainNavigation', extensionName: 'my_sitepackage')}">
<nav aria-label="{f:translate(key: 'breadcrumb', extensionName: 'my_sitepackage')}">
<nav aria-label="{f:translate(key: 'footerNavigation', extensionName: 'my_sitepackage')}">
```

### Form Landmarks

Search forms are landmarks -- use `role="search"` or the `<search>` element:

```html
<search>
    <form action="/search" method="get">
        <label for="search-input" class="visually-hidden">
            <f:translate key="searchLabel" extensionName="my_sitepackage" />
        </label>
        <input type="search" id="search-input" name="q"
               placeholder="{f:translate(key: 'searchPlaceholder', extensionName: 'my_sitepackage')}">
        <button type="submit">
            <f:translate key="searchSubmit" extensionName="my_sitepackage" />
        </button>
    </form>
</search>
```

### Label Landmarks

When multiple landmarks of the same type exist, label them to differentiate:

```html
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Footer navigation">...</nav>
```

Without labels, screen reader users cannot distinguish between multiple `<nav>` elements.

### Structure Main Content

Use landmarks, headings, and lists to provide structure within `<main>`. Screen reader users rely on these to navigate complex pages. Group related content with `<section>` and label each with a heading or `aria-label`.

---

## Headings and Content Order

### Heading Hierarchy

- Exactly one `<h1>` per page
- Never skip heading levels (`<h1>` then `<h3>` without `<h2>`)
- Headings create the document outline -- screen reader users navigate by headings
- Every content section should start with a heading

```html
<h1>Page Title</h1>
  <h2>Section</h2>
    <h3>Subsection</h3>
    <h3>Subsection</h3>
  <h2>Another Section</h2>
```

### Content Order

DOM order must match visual order. Content must make sense without CSS.

**Never use these to reorder content semantically:**
- `order` in Flexbox/Grid (visual reorder for responsive layout is OK if DOM stays logical)
- `flex-direction: row-reverse` or `column-reverse` to reverse reading order
- `tabindex` values > 0 to override tab order
- CSS `float` tricks that put content before its heading in the DOM

---

## Links

### Link vs. Button Decision

| Use | Element | Behavior |
|---|---|---|
| Navigate to URL/anchor | `<a href="...">` | Changes page/location |
| Trigger action on current page | `<button>` | Toggle, submit, open dialog |
| Download file | `<a href="..." download>` | Initiates download |

**Never:**
- Use `<div onclick>` or `<span onclick>` as links or buttons
- Use `<a>` without `href` (removes it from tab order)
- Use `<a href="javascript:void(0)">` -- use `<button>` instead
- Use `<button>` for navigation -- use `<a>` instead

### Link Styling

Links in body text **must be underlined**. Color alone is not sufficient -- 8% of men have color vision deficiencies and cannot distinguish link color from text color.

```scss
// Basic/_links.scss

// Links in running text must be underlined
.ce-textmedia,
.news-detail__content,
.accordion-body {
    a:not([class]) {
        text-decoration: underline;
        text-underline-offset: 0.1875rem;

        &:hover {
            text-decoration-thickness: 0.125rem;
        }
    }
}

// Navigation links are exempted (context makes purpose obvious)
.nav-link,
.btn {
    text-decoration: none;
}
```

### Download Links

Download links must communicate: file type, file size, and that it's a download.

```html
<a href="/files/report.pdf" download>
    Annual Report 2024 (PDF, 2.4 MB)
</a>
```

### Email Links

Always show the email address as visible text:

```html
<!-- Good -->
<a href="mailto:info@example.com">info@example.com</a>

<!-- Bad: hides the address -->
<a href="mailto:info@example.com">Contact us</a>
```

### Linked Images

The image `alt` text becomes the link's accessible name:

```html
<!-- Logo link to homepage -->
<a href="/">
    <img src="/logo.svg" alt="Company Name - Back to homepage">
</a>

<!-- Image + text link: empty alt to avoid redundancy -->
<a href="/products/widget">
    <img src="/widget.jpg" alt="">
    Widget Pro 3000
</a>
```

### Links Opening in New Window

When using `target="_blank"`, inform users:

```html
<a href="https://external.com" target="_blank"
   rel="noopener noreferrer">
    External Resource
    <span class="visually-hidden">(opens in new tab)</span>
</a>
```

Or use a visual icon with screen reader text. Never open links in new tabs without indication.

### Client-Side Rendering

Not applicable for standard TYPO3 sitepackages (server-side rendering). If using JS-heavy frontend components that manipulate history: ensure focus management on route changes and update `<title>` dynamically.

### Clickable Card Patterns

See `references/patterns-clickable-cards.md` for the 5 patterns with trade-offs. **Recommended: pseudo-element stretch pattern.**

---

## Buttons

### Button Labeling

Every button must have an accessible name. Three patterns:

```html
<!-- 1. Text content (best) -->
<button type="button">Save changes</button>

<!-- 2. Icon button with visually hidden text (preferred) -->
<button type="button">
    <svg aria-hidden="true">...</svg>
    <span class="visually-hidden">Close dialog</span>
</button>

<!-- 3. Icon button with aria-label (acceptable) -->
<button type="button" aria-label="Close dialog">
    <svg aria-hidden="true">...</svg>
</button>
```

**Never** use `title` as the only accessible name for buttons.

### Resetting Button Styles

When buttons need custom styling, reset properly but keep focus styles:

```scss
.btn-reset {
    appearance: none;
    background: none;
    border: none;
    padding: 0;
    font: inherit;
    color: inherit;
    cursor: pointer;

    &:focus-visible {
        outline: 0.1875rem solid $primary;
        outline-offset: 0.125rem;
    }
}
```

CSS `all: unset` also works but removes ALL styles including focus -- always re-add focus styles.

### Button States and Properties

```html
<!-- Toggle button (show/hide) -->
<button type="button" aria-expanded="false" aria-controls="panel-1">
    Show details
</button>

<!-- Pressed toggle (bold/italic toolbar) -->
<button type="button" aria-pressed="false">Bold</button>

<!-- Button with popup -->
<button type="button" aria-haspopup="true" aria-expanded="false">
    Options
</button>
```

Update `aria-expanded` and `aria-pressed` via JavaScript when toggling.

### Don't Disable Buttons

**Never use `disabled` on submit buttons.** Disabled buttons:
- Are not focusable -- keyboard users cannot find them
- Have no hover state -- users get no feedback why they can't submit
- Have low contrast by default -- hard to read
- Provide no explanation of WHY they are disabled

Instead: keep the button enabled, validate on click, and show error messages:

```html
<!-- Bad -->
<button type="submit" disabled>Submit</button>

<!-- Good: always enabled, show errors on click -->
<button type="submit">Submit</button>
```

---

## Color and Contrast

### Minimum Contrast Ratios (WCAG AA)

| Element | Ratio |
|---|---|
| Normal text (<24px / <19px bold) | 4.5:1 |
| Large text (>=24px / >=19px bold) | 3:1 |
| UI components (borders, icons, focus indicators) | 3:1 |

Rules:
- Never convey information through color alone -- always add icons, patterns, underlines, or text
- Test all Bootstrap theme color combinations against their backgrounds
- Test with Chrome DevTools color contrast tools and emulated color deficiencies
- The current WCAG contrast formula has known limitations -- use judgment alongside the numbers

```scss
// Verify these combinations in your theme:
$primary: #0069b4;    // Must have 4.5:1 against $white for text
$secondary: #d1530f;  // Must have 4.5:1 against $white for text
$danger: #dc3545;     // Must have 4.5:1 against $white for text
```

---

## CSS Units and User Preferences

### Relative Units Only

Key points:

- `rem` for font sizes, spacing, padding, margins, media queries
- `em` for component-relative sizing (e.g., icon size relative to text)
- Never `px` except for 1px borders and box-shadows
- Users who set larger browser font sizes must get proportionally larger layouts
- At 200% zoom, no content loss or horizontal scrolling may occur

### Media Queries for User Settings

Respond to these user preferences via CSS media queries:

```scss
// Dark mode
@media (prefers-color-scheme: dark) {
    // Swap colors, adjust image brightness/contrast
}

// Increased contrast
@media (prefers-contrast: more) {
    // Increase contrast ratios, thicken borders, remove subtle backgrounds
}

// Forced colors / Windows High Contrast Mode
@media (forced-colors: active) {
    // Use system colors, don't override backgrounds
    // Borders become the primary visual structure
    // Custom backgrounds and box-shadows disappear
}

// Reduced transparency
@media (prefers-reduced-transparency: reduce) {
    // Replace semi-transparent overlays with solid colors
}

// Reduced motion
@media (prefers-reduced-motion: reduce) { ... }
```

`forced-colors` is critical for Windows High Contrast Mode users. When active, the browser overrides all colors -- custom backgrounds and shadows disappear. Ensure layouts work with borders as the primary visual structure.

### `display: contents` Danger

`display: contents` removes the element's box from layout but **also removes its semantics** from the accessibility tree in some browsers.

**Never use on:**
- `<button>` -- loses button role
- `<a>` -- loses link role
- `<table>`, `<tr>`, `<td>` -- loses table structure
- Any interactive element

**Safe to use on:**
- Wrapper `<div>` or `<span>` that exist only for layout purposes

### `list-style: none` Removes List Semantics

Safari + VoiceOver removes list semantics when `list-style: none` is applied. Fix by adding `role="list"` explicitly:

```scss
// When using list-style: none, ALWAYS add role="list" in the template
.nav-list,
.skiplinks__list,
.breadcrumb,
.pagination {
    list-style: none;
}
```

```html
<!-- Required in Fluid when list-style: none is used -->
<ul class="nav-list" role="list">
    <li>...</li>
</ul>
```

### Reduced Motion

```scss
// Basic/_accessibility.scss
@media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
        scroll-behavior: auto !important;
    }
}
```

When adding animations, provide a subtle fallback (e.g., opacity fade) for reduced-motion users instead of removing all animation. For animation patterns with `prefers-reduced-motion` support, see the typo3-frontend-patterns skill.

---

## Focus Management

### Focus Styles

All interactive elements must have visible focus indicators:

```scss
// Basic/_accessibility.scss
*:focus-visible {
    outline: 0.1875rem solid $primary;
    outline-offset: 0.125rem;
}

*:focus:not(:focus-visible) {
    outline: none;
}
```

`:focus-visible` shows focus only for keyboard navigation, not mouse clicks. The outline must have 3:1 contrast against adjacent colors.

### Making Elements Focusable

| Attribute | Behavior | Use case |
|---|---|---|
| No tabindex | Native focusable elements (`<a href>`, `<button>`, `<input>`) | Default |
| `tabindex="0"` | Adds to natural tab order | Custom interactive elements |
| `tabindex="-1"` | Focusable via JS `.focus()` only | Programmatic focus targets |
| `tabindex="1+"` | **Never use** | Overrides natural order, creates chaos |

### Moving Focus

When opening modals, drawers, or overlays:
1. Save the previously focused element
2. Move focus to the new content (first focusable element or the container itself)
3. When closing, return focus to the saved element

```typescript
function openDialog(dialog: HTMLElement, trigger: HTMLElement): void {
    const previousFocus = document.activeElement as HTMLElement;

    dialog.removeAttribute('hidden');
    dialog.querySelector<HTMLElement>('[autofocus], button, a, input')?.focus();

    dialog.addEventListener('close', () => {
        previousFocus?.focus();
    }, { once: true });
}
```

### Focus Containment with `inert`

The modern alternative to manual focus trapping is the `inert` attribute:

```typescript
function openModal(modal: HTMLElement): void {
    document.querySelectorAll('body > *:not(.modal-overlay)').forEach((el) => {
        el.setAttribute('inert', '');
    });
    modal.removeAttribute('inert');
    modal.querySelector<HTMLElement>('[autofocus], button')?.focus();
}

function closeModal(modal: HTMLElement, trigger: HTMLElement): void {
    document.querySelectorAll('[inert]').forEach((el) => {
        el.removeAttribute('inert');
    });
    trigger.focus();
}
```

`inert` makes elements non-focusable AND invisible to screen readers. For legacy browser support, keep the manual `trapFocus()` function as fallback:

```typescript
function trapFocus(element: HTMLElement): void {
    const focusable = element.querySelectorAll<HTMLElement>(
        'a[href], button:not([disabled]), input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])',
    );
    const first = focusable[0];
    const last = focusable[focusable.length - 1];

    element.addEventListener('keydown', (e: KeyboardEvent) => {
        if (e.key !== 'Tab') return;
        if (e.shiftKey && document.activeElement === first) {
            e.preventDefault();
            last.focus();
        } else if (!e.shiftKey && document.activeElement === last) {
            e.preventDefault();
            first.focus();
        }
    });
    first?.focus();
}
```

### Preserve Order

DOM order must match visual order. CSS properties that break this:
- `order` in Flex/Grid
- `flex-direction: row-reverse` / `column-reverse`
- `position: absolute` moving elements visually out of sequence

### Skip Links

See `references/patterns-skiplinks.md`. Mandatory for every page.

### Keyboard Navigation Summary

| Key | Action |
|---|---|
| `Tab` / `Shift+Tab` | Sequential focus navigation |
| `Enter` / `Space` | Activate button or link |
| `Escape` | Close dropdown, modal, popover |
| Arrow keys | Navigate within tabs, accordions (optional enhancement) |

---

## ARIA Reference

### Core ARIA Patterns

ARIA creates relationships between elements using ID references:

```html
<!-- aria-labelledby: element labeled BY another element -->
<div role="region" aria-labelledby="section-title">
    <h2 id="section-title">Latest News</h2>
</div>

<!-- aria-describedby: element described BY another element -->
<input type="email" aria-describedby="email-help">
<p id="email-help">We'll never share your email.</p>

<!-- aria-controls: element controls another element -->
<button aria-expanded="false" aria-controls="panel-1">Toggle</button>
<div id="panel-1" hidden>Panel content</div>
```

### ARIA Rules

1. **Don't use ARIA if native HTML works** -- `<button>` over `<div role="button">`
2. **Don't change native semantics unnecessarily** -- don't add `role="button"` to `<a>`
3. **All interactive ARIA elements must be keyboard accessible**
4. **Don't use `role="presentation"` or `aria-hidden="true"` on focusable elements**
5. **All interactive elements must have an accessible name**

### Images

```html
<!-- Informative image -->
<img src="..." alt="Description of what the image shows">

<!-- Decorative image (empty alt is sufficient, role="presentation" is optional) -->
<img src="..." alt="">

<!-- Complex image (chart, infographic) -->
<figure>
    <img src="..." alt="Brief description" aria-describedby="desc-{uid}">
    <figcaption id="desc-{uid}">Detailed description...</figcaption>
</figure>
```

### Live Regions

For dynamic content updates that screen readers should announce:

```html
<!-- Polite: announced at next pause (filter results, status updates) -->
<div aria-live="polite" aria-atomic="true">
    3 results found
</div>

<!-- Assertive: interrupts immediately (errors, urgent alerts) -->
<div role="alert">
    Session expires in 2 minutes
</div>

<!-- Status: polite announcement for form/process status -->
<div role="status">
    Form saved successfully
</div>
```

---

## Automated Accessibility Testing

### axe-core Integration

Add automated accessibility testing to Playwright:

```typescript
// tests/e2e/accessibility.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

const pages = ['/', '/contact', '/news'];

for (const page of pages) {
    test(`has no critical a11y violations on ${page}`, async ({ page: browserPage }) => {
        await browserPage.goto(page);
        const results = await new AxeBuilder({ page: browserPage })
            .withTags(['wcag2a', 'wcag2aa', 'best-practice'])
            .analyze();
        expect(results.violations).toEqual([]);
    });
}
```

Install: `npm install --save-dev @axe-core/playwright` in the root project.

### Browser DevTools Workflows

**Accessibility Tree:**
Chrome DevTools > Elements > Accessibility pane shows computed accessible name, role, and properties. Enable "Full accessibility tree" in DevTools settings for the tree view.

**Debug Roles and Names:**
Inspect element > Accessibility pane > "Computed Properties" shows resolved `role`, `name`, `description`. Common issue: missing accessible name on icon buttons or empty links.

**Visualize Tab Order:**
Chrome DevTools > Elements > Accessibility > check "Show tab order". Numbers overlay shows actual tab sequence -- verify it matches visual reading order.

**Emulate Vision Deficiencies:**
Chrome DevTools > Rendering > "Emulate vision deficiencies". Options: Protanopia, Deuteranopia, Tritanopia, Achromatopsia, Blurred vision. Also emulate: `prefers-reduced-motion`, `prefers-color-scheme`, `prefers-contrast`, `forced-colors`.

**Custom Debugging Selectors:**

```css
/* Find images without alt */
img:not([alt]) { outline: 0.25rem solid red !important; }

/* Find links without accessible text */
a:empty:not([aria-label]):not([aria-labelledby]) { outline: 0.25rem solid red !important; }

/* Find buttons without accessible text */
button:empty:not([aria-label]):not([aria-labelledby]) { outline: 0.25rem solid red !important; }

/* Find missing lang attribute */
html:not([lang]) { outline: 0.25rem solid red !important; }
```

### Linter Rules

Add accessibility Stylelint rules to `Build/.stylelintrc.json`:

```json
{
    "plugins": ["stylelint-a11y"],
    "rules": {
        "a11y/media-prefers-reduced-motion": true,
        "a11y/no-outline-none": true,
        "a11y/no-text-size-adjust": true
    }
}
```

### CI Integration

Add axe-core to the Playwright CI pipeline. Configure the accessibility test job in your CI pipeline.

---

## Responsive Accessibility

### Touch Targets

Minimum 44x44px for all interactive elements on mobile (WCAG 2.5.8):

```scss
@media (pointer: coarse) {
    .btn,
    .nav-link,
    .dropdown-item,
    .form-check-label {
        min-height: 2.75rem;
        min-width: 2.75rem;
    }
}
```

---

## Content Element Checklist

For every new content element, verify:

- [ ] Page has unique, descriptive `<title>`
- [ ] `lang` attribute set on `<html>` (and inline foreign text where needed)
- [ ] Viewport allows zoom (no `user-scalable=no` or `maximum-scale=1`)
- [ ] Heading hierarchy is correct (no skipped levels, one `<h1>`)
- [ ] DOM order matches visual order
- [ ] All images have meaningful `alt` text (or `alt=""` if decorative)
- [ ] Links in body text are underlined
- [ ] Download links show file type and size
- [ ] Links opening in new tab inform users
- [ ] Buttons have accessible names (no empty icon buttons)
- [ ] Buttons are never `disabled` -- validate on click instead
- [ ] Interactive elements use correct element (`<a>` vs `<button>`)
- [ ] Interactive elements are keyboard-accessible
- [ ] ARIA attributes are correct and complete
- [ ] `aria-expanded` toggles on disclosure triggers
- [ ] Color contrast meets WCAG AA (4.5:1 text, 3:1 UI)
- [ ] Information is not conveyed by color alone
- [ ] Focus order follows visual/DOM order
- [ ] Focus indicator is visible (3:1 contrast)
- [ ] `prefers-reduced-motion` disables animations
- [ ] `forced-colors` does not break layout
- [ ] Touch targets are at least 44x44px
- [ ] `list-style: none` lists have `role="list"`
- [ ] Form fields have associated `<label>`
- [ ] Error messages are announced to screen readers (`aria-live` or `role="alert"`)
- [ ] Live regions announce dynamic content updates
- [ ] Labels exist in both DE and EN
- [ ] axe-core Playwright test passes

---

## Recommended Reading

*Web Accessibility Cookbook* by Manuel Matuzovic (O'Reilly, 2024) -- comprehensive guide to web accessibility with practical recipes.
