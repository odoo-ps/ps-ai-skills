# CSS / SCSS Guidelines

## Formatting
- 4-space indents, no tabs
- Max 80 columns wide
- Opening `{`: space after last selector
- Closing `}`: on its own line
- One declaration per line

## Properties Order
Order from "outside in": position → box model → typography → decorative (`font`, `filter`, etc.).
Scoped SCSS variables and CSS variables go at the **very top**, followed by an empty line.

## Class Naming
- Prefix all classes: `o_<module_name>_...` (e.g., `o_sale_order_form`, `o_forum_post`)
- Exception: webclient uses just `o_` prefix
- Avoid id selectors
- Use "Grandchild" approach — avoid hyper-specific nested class names

## SCSS Variables
Standard convention: `$o-[root]-[element]-[property]-[modifier]`
```scss
$o-sale-total-color: #000;
$o-sale-header-bg-color-active: $primary;
```

## Scoped SCSS Variables (within blocks)
Convention: `$-[variable-name]`
```scss
.o_my_component {
    $-spacing: 8px;
    padding: $-spacing;
}
```

## SCSS Mixins and Functions
Convention: `o-[name]`, use imperative verbs for functions (`o-get-color`, `o-make-shadow`).
Optional arguments use scoped variable form: `$-argument`.

## CSS Variables
Convention: BEM — `--[root]__[element]-[property]--[modifier]`
```css
--sale__header-color--active: var(--primary);
```
- CSS variables are DOM-context-scoped (not global design system)
- Defined inside component's main block with default fallbacks
- Avoid defining on `:root` pseudo-class (use SCSS variables for global design system instead)

## Bootstrap Utilities
Prefer Bootstrap utility classes over inline `style=""` where an equivalent exists (e.g. `text-end`, `fw-bold`, `mb-3`, `ms-3`). Unavoidable layout values (e.g. `min-width`, `width: auto`) are acceptable inline.
