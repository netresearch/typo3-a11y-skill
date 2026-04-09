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
| `patterns-clickable-cards.md` | Five clickable card patterns with trade-off analysis |
| `patterns-responsive-tables.md` | Horizontal scroll and card reflow patterns for mobile tables |

## License

[MIT](LICENSE)
