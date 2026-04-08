# Pattern: Clickable Cards

Cards and teasers often need to be entirely clickable while remaining accessible. Naive approaches create problems for screen readers, keyboard users, or break native browser behavior.

## Requirements for Accessible Clickable Cards

- Entire card must be clickable for mouse/touch users
- Screen readers should announce only ONE link per card (not multiple redundant links)
- Link text must be meaningful (not empty or generic "read more")
- Right-click / middle-click / URL preview must work
- Text within the card must remain selectable
- Focus indicator must be visible

## Solution 1: Wrapping Everything in `<a>` (NOT Recommended)

```html
<a href="/detail" class="ce-teaser">
    <img src="image.jpg" alt="" class="ce-teaser__image">
    <h3 class="ce-teaser__title">Card Title</h3>
    <p class="ce-teaser__text">Description text that explains the card content.</p>
</a>
```

**Problems:** Screen readers read ALL content as one long link text -- extremely verbose and confusing. Cannot nest interactive elements (buttons, other links) inside an `<a>` element.

## Solution 2: Separate Links (NOT Recommended)

```html
<div class="ce-teaser">
    <a href="/detail"><img src="image.jpg" alt="Card Title" class="ce-teaser__image"></a>
    <h3 class="ce-teaser__title"><a href="/detail">Card Title</a></h3>
    <p class="ce-teaser__text">Description text.</p>
    <a href="/detail" class="btn btn-primary">Read more</a>
</div>
```

**Problems:** Creates 3 tab stops all pointing to the same URL. Keyboard users must tab through redundant links. Screen readers announce the same destination multiple times.

## Solution 3: Empty Link Overlay (Acceptable)

```html
<div class="ce-teaser">
    <a href="/detail" class="ce-teaser__overlay" aria-label="Card Title"></a>
    <img src="image.jpg" alt="" class="ce-teaser__image">
    <h3 class="ce-teaser__title">Card Title</h3>
    <p class="ce-teaser__text">Description text.</p>
</div>
```

```scss
.ce-teaser {
    position: relative;

    &__overlay {
        position: absolute;
        inset: 0;
        z-index: 1;
    }
}
```

**Problems:** Text selection does not work anywhere on the card. The `aria-label` must be kept in sync with the heading manually. Empty links are not ideal semantically.

## Solution 4: Pseudo-Element Stretch (RECOMMENDED)

The heading link gets a `::after` pseudo-element stretched over the entire card. This is the recommended default for all card/teaser components.

**Why this pattern wins:**
- Only one link and one tab stop per card
- Meaningful link text comes from the heading itself
- Right-click and middle-click work on the heading link
- Text outside the heading is still selectable (z-index layering)
- No extra markup or JavaScript needed

### Fluid Component Template

```html
<!-- ContentElements/Partials/Molecule/Teaser.html -->
<html xmlns:f="http://typo3.org/ns/TYPO3/CMS/Fluid/ViewHelpers"
      data-namespace-uri-known-prefixed-attribute="true">

<article class="ce-teaser">
    <f:if condition="{image}">
        <div class="ce-teaser__media">
            <f:image image="{image}" alt="" class="ce-teaser__image" loading="lazy" />
        </div>
    </f:if>
    <div class="ce-teaser__body">
        <h3 class="ce-teaser__title">
            <f:link.typolink parameter="{link}" class="ce-teaser__link">
                {title}
            </f:link.typolink>
        </h3>
        <f:if condition="{text}">
            <p class="ce-teaser__text">{text}</p>
        </f:if>
        <f:if condition="{tags}">
            <ul class="ce-teaser__tags" role="list">
                <f:for each="{tags}" as="tag">
                    <li class="ce-teaser__tag">{tag}</li>
                </f:for>
            </ul>
        </f:if>
    </div>
</article>

</html>
```

### SCSS

```scss
// ContentElements/_ce-teaser.scss

.ce-teaser {
    position: relative;
    display: flex;
    flex-direction: column;
    height: 100%;
    background-color: $white;
    border: 1px solid $border-color;
    border-radius: $border-radius;
    overflow: hidden;
    transition: box-shadow 0.2s ease;

    // Focus-visible indicator on the card when the link is focused
    &:has(.ce-teaser__link:focus-visible) {
        outline: 0.1875rem solid $primary;
        outline-offset: 0.125rem;
    }

    &:hover {
        box-shadow: 0 0.25rem 1rem rgba(0, 0, 0, 0.1);
    }

    &__media {
        aspect-ratio: 16 / 9;
        overflow: hidden;
    }

    &__image {
        width: 100%;
        height: 100%;
        object-fit: cover;
        transition: transform 0.3s ease;

        .ce-teaser:hover & {
            transform: scale(1.03);
        }
    }

    &__body {
        display: flex;
        flex-direction: column;
        flex-grow: 1;
        padding: 1.25rem;
    }

    &__title {
        font-size: 1.125rem;
        margin-bottom: 0.5rem;
    }

    // The pseudo-element stretches the link over the entire card
    &__link {
        color: inherit;
        text-decoration: none;

        &::after {
            content: '';
            position: absolute;
            inset: 0;
            z-index: 1;
        }

        &:hover {
            text-decoration: underline;
        }

        // Hide default outline -- card itself shows focus indicator via :has()
        &:focus-visible {
            outline: none;
        }
    }

    &__text {
        font-size: 0.875rem;
        color: $text-muted;
        margin-bottom: 0.75rem;
        // Raise above the pseudo-element so text is selectable
        position: relative;
        z-index: 2;
    }

    &__tags {
        list-style: none;
        padding: 0;
        margin: 0;
        margin-top: auto;
        display: flex;
        flex-wrap: wrap;
        gap: 0.5rem;
        // Raise above the pseudo-element so tags are selectable
        position: relative;
        z-index: 2;
    }

    &__tag {
        font-size: 0.75rem;
        padding: 0.125rem 0.5rem;
        background-color: $gray-100;
        border-radius: $border-radius-sm;
        color: $text-muted;
    }
}
```

**Z-index layering explained:** The `::after` pseudo-element sits at `z-index: 1`, making the entire card clickable. Elements that should remain selectable (text, tags) get `position: relative; z-index: 2`, raising them above the overlay. The heading link itself stays at the default stacking level, so clicks on non-raised areas pass through to `::after`.

## Solution 5: JavaScript Click Delegation (Acceptable)

```typescript
// Assets/JavaScript/Components/ClickableCard.ts

export function initClickableCards(): void {
    const cards = document.querySelectorAll<HTMLElement>('[data-clickable-card]');

    cards.forEach((card) => {
        const primaryLink = card.querySelector<HTMLAnchorElement>('a[data-card-link]');
        if (!primaryLink) return;

        card.style.cursor = 'pointer';

        card.addEventListener('click', (event: MouseEvent) => {
            // Do not hijack clicks on interactive elements
            const target = event.target as HTMLElement;
            if (target.closest('a, button, input, select, textarea')) return;

            if (event.ctrlKey || event.metaKey) {
                window.open(primaryLink.href, '_blank');
            } else {
                primaryLink.click();
            }
        });
    });
}
```

**Problems:** No URL preview on hover over the card body. No native right-click context menu with link options. Requires JavaScript -- card is not clickable without it.

## Labels

```xml
<!-- locallang.xlf (EN) -->
<trans-unit id="teaser.readMore"><source>Read more about %s</source></trans-unit>

<!-- de.locallang.xlf (DE) -->
<trans-unit id="teaser.readMore">
    <source>Read more about %s</source>
    <target>Mehr erfahren über %s</target>
</trans-unit>
```

## Playwright E2E Test

```typescript
// tests/e2e/clickable-card.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Clickable Card (Pseudo-Element Pattern)', () => {
    test.beforeEach(async ({ page }) => {
        await page.goto('/teaser-page');
    });

    test('card has exactly one link', async ({ page }) => {
        const links = page.locator('.ce-teaser').first().locator('a');
        await expect(links).toHaveCount(1);
    });

    test('link has meaningful text', async ({ page }) => {
        const linkText = await page.locator('.ce-teaser__link').first().textContent();
        expect(linkText?.trim()).not.toBe('');
    });

    test('card is focusable via keyboard', async ({ page }) => {
        await page.keyboard.press('Tab');
        const focused = page.locator(':focus');
        await expect(focused).toHaveClass(/ce-teaser__link/);
    });

    test('card text is selectable', async ({ page }) => {
        const zIndex = await page.locator('.ce-teaser__text').first().evaluate(
            (el) => window.getComputedStyle(el).zIndex,
        );
        expect(Number(zIndex)).toBeGreaterThan(1);
    });
});
```

## Key Rules

1. **Use Solution 4 (pseudo-element) as default** for all card/teaser components
2. **One link per card** -- screen readers should encounter exactly one link
3. **Meaningful link text** -- heading text serves as the link label
4. **Text selectability** -- raise non-link content above the overlay with `z-index: 2`
5. **Focus indicator on the card** -- use `:has(:focus-visible)` to show outline on the card boundary
6. **No px units** -- use rem throughout, except for 1px borders
7. **ce- prefix** -- content element cards use `ce-teaser`, `ce-card`, etc.
