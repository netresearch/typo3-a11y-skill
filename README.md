# typo3-a11y-skill

WCAG 2.1 AA accessibility patterns for TYPO3 v13+ sitepackage frontend development. A Claude Code skill that provides comprehensive accessibility guidelines, HTML/ARIA patterns, SCSS examples, and TypeScript implementations.

## Installation

### Claude Code Marketplace

```bash
claude install netresearch/typo3-a11y-skill
```

### Composer

```bash
composer require netresearch/typo3-a11y-skill
```

## References

| File | Description |
|---|---|
| `accessibility.md` | WCAG 2.1 AA comprehensive guide -- language, landmarks, headings, links, buttons, color, focus, ARIA, testing |
| `patterns-skiplinks.md` | Mandatory skip link navigation with Fluid, SCSS, and Playwright tests |
| `patterns-accessible-navigation.md` | Main navigation, submenus, mobile toggle with b13/menus TreeMenu |
| `patterns-accessible-forms.md` | Form labels, error handling, fieldsets, multi-step forms |
| `patterns-accessible-filter.md` | Filtering, pagination, sorting, semantic table structure |
| `patterns-disclosure-widget.md` | Accordions, collapsible sections, content hiding techniques |
| `patterns-clickable-cards.md` | Accessible clickable-card pattern, with rejected alternatives noted |
| `patterns-responsive-tables.md` | Horizontal scroll and card reflow patterns for mobile tables |
| `patterns-sticky-header.md` | Scroll-triggered fixed header with IntersectionObserver |
| `patterns-lazy-loading.md` | Deferred component initialization with placeholder content |
| `patterns-breadcrumb.md` | Breadcrumb navigation with JSON-LD structured data |
| `patterns-language-switcher.md` | Multi-language navigation with b13/menus LanguageMenu |
| `patterns-animations.md` | Scroll animations with prefers-reduced-motion support |
| `patterns-scroll-to-anchor.md` | Smooth scroll with sticky header offset compensation |
| `patterns-skeleton-loading.md` | CSS placeholder animations for content loading |
| `patterns-toast-notification.md` | Auto-dismiss notifications with ARIA live region |
| `patterns-back-to-top.md` | Scroll-to-top button with visibility threshold |

## License

This project uses split licensing:

- Code (`scripts/**`, `.github/workflows/**`, config files) is licensed under the [MIT License](LICENSE-MIT).
- Documentation and skill content (`skills/**`, `references/**`, `README.md`) is licensed under [CC-BY-SA-4.0](LICENSE-CC-BY-SA-4.0).

SPDX expression: `(MIT AND CC-BY-SA-4.0)`.

## Maintainer

Maintained by [Netresearch DTT GmbH](https://www.netresearch.de).
