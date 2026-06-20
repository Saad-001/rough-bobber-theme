# Rough Bobber — Theme Development Conventions

This file is the single source of truth for building custom sections, snippets, and blocks
on top of the **Dawn** theme for the Rough Bobber Shopify store.

Claude Code MUST read and follow this file before writing any Liquid, CSS, or JS.
If a request conflicts with these conventions, stop and flag it rather than guessing.

Project: Rough Bobber — print-on-demand apparel brand (Printful), brand-first aesthetic.
Store is **bilingual: English + Spanish**. All customer-facing strings must be translatable
(use locale files / `{{ 'key' | t }}`, never hardcoded copy in section markup).

---

## 1. Tech rules (non-negotiable)

- **Vanilla CSS only.** No Tailwind, no Bootstrap, no SCSS, no CSS framework of any kind.
- No JS frameworks. Plain vanilla JS only, matching Dawn's existing patterns.
- Follow Dawn's existing conventions (BEM-ish class naming, `assets/` CSS per section
  loaded via `{{ 'section-name.css' | asset_url | stylesheet_tag }}`).
- Each custom section gets its own CSS file in `assets/`, named to match the section file.
- Mobile-first, responsive. Test at 750px and 990px Dawn breakpoints.
- No inline styles except where a Liquid setting must drive a value
  (e.g. `style="--section-padding-top: {{ section.settings.padding_top }}px"`).
- Accessibility: semantic HTML, alt text, focus states, ARIA where Dawn uses it.

---

## 2. Brand tokens

> Source of truth = **Webkit** (per client instruction "include the webkit font").
> Brandkit alternates are noted as comments in case the client's spec doc overrides.
> Do NOT silently mix the two sets — change them in one place if updated.

### Colors

```css
:root {
  --rb-black: #121315; /* Webkit near-black.  Brandkit alt: #131416 */
  --rb-gold: #b3916b; /* Webkit bronze.      Brandkit alt: #b17b21 (richer) */
  --rb-sand: #c5b8a8; /* warm sand / taupe (from Brandkit, used for light bg) */
  --rb-cream: #f5f4f2; /* body/light background seen in mockups */
}
```

- Dark sections: `--rb-black` background, cream/sand text, gold accents.
- Light sections: cream/sand background, `--rb-black` text, gold accents.
- Gold (`--rb-gold`) is for accents, buttons, hovers, dividers — not large fills.

### Fonts

- **Display / headings:** `Amnestia` (distressed display face). Headlines, hero,
  section titles, product titles. Often uppercase in the design.
- **Body / UI:** `Open Sans` (Webkit body font). Paragraphs, descriptions, labels,
  buttons, nav, captions.
- Brandkit alternates (do NOT use unless client overrides): Amnestia Distressed +
  Bebas Neue Pro Middle. Bebas is condensed all-caps — avoid for body copy.
- Load custom fonts via `@font-face` in a single `assets/rb-fonts.css`, referencing
  font files uploaded to `assets/`. Provide `font-display: swap`.
- Never rely on Dawn's `settings.type_*` font pickers for Amnestia — it's a custom face.

### Brand copy / strings

- Primary tagline: **"Built by Ride. Worn for Life."**
- Secondary: **"Dirt. Speed. Glory."**
- Brand vibe words: **Raw · Authentic · Timeless · Biker Lifestyle**
- Value props (icon strip): Built by Ride / Ride Free / Stay Rough / Built to Last
- These are defaults only — expose as section settings AND make them translatable.

---

## 3. Common conventions — EVERY custom section MUST include these

Every custom section, without exception, includes the following three things.

### 3.1 Top & bottom padding controlled by theme settings

Each section exposes editable top and bottom padding in the theme customizer, following
**Dawn's exact native pattern** — a per-section padding class scoped by `section.id`, with
the values scaled to 75% on mobile and restored to 100% at the 750px breakpoint.

In `{% schema %}` settings (use Dawn's standard keys/labels):

```json
{
  "type": "range",
  "id": "padding_top",
  "min": 0, "max": 100, "step": 4, "unit": "px",
  "label": "t:sections.all.padding.padding_top",
  "default": 36
},
{
  "type": "range",
  "id": "padding_bottom",
  "min": 0, "max": 100, "step": 4, "unit": "px",
  "label": "t:sections.all.padding.padding_bottom",
  "default": 36
}
```

Apply Dawn's padding class on the section wrapper:

```liquid
<div class="rb-section section-{{ section.id }}-padding ...">
```

And emit Dawn's scoped padding CSS in the section (this is the exact Dawn pattern —
75% on mobile, full value from 750px up):

```liquid
{%- style -%}
  .section-{{ section.id }}-padding {
    padding-top: {{ section.settings.padding_top | times: 0.75 | round: 0 }}px;
    padding-bottom: {{ section.settings.padding_bottom | times: 0.75 | round: 0 }}px;
  }

  @media screen and (min-width: 750px) {
    .section-{{ section.id }}-padding {
      padding-top: {{ section.settings.padding_top }}px;
      padding-bottom: {{ section.settings.padding_bottom }}px;
    }
  }
{%- endstyle -%}
```

- Do NOT use custom CSS variables for padding — match Dawn's `.section-{{ section.id }}-padding`
  class so the behavior is identical to every native Dawn section.
- Do NOT hardcode padding in the section's `assets/rb-<name>.css` file; padding always comes
  from these settings via the scoped `{%- style -%}` block.

### 3.2 Global width & horizontal padding via Dawn's `page-width`

Do NOT invent custom max-widths or left/right padding. Wrap inner content in Dawn's
global container so width and side padding stay consistent with the rest of the theme:

```liquid
<div class="rb-section ...">
  <div class="page-width">
    <!-- section content here -->
  </div>
</div>
```

- `page-width` controls max content width + responsive left/right padding globally.
- For full-bleed backgrounds (e.g. hero image edge-to-edge), let the OUTER wrapper be
  full width and keep the `page-width` div INSIDE for the text/content only.
- Never hardcode `margin: 0 auto; max-width: 1200px` — rely on `page-width`.

### 3.3 Color scheme picker per section

Every section exposes Dawn's color scheme setting so the merchant can switch the section
between brand schemes (dark / cream / sand) from the customizer.

In `{% schema %}`:

```json
{
  "type": "color_scheme",
  "id": "color_scheme",
  "label": "t:sections.all.colors.label",
  "default": "scheme-1"
}
```

On the wrapper, apply Dawn's color-scheme class:

```liquid
<div class="rb-section color-{{ section.settings.color_scheme }} gradient" ...>
```

- This inherits Dawn's `--color-background`, `--color-foreground`, etc.
- Define Rough Bobber's schemes (dark/cream/sand + gold accent) in theme settings
  (`config/settings_data.json` color schemes) so the picker has brand-correct options.
- Brand-token CSS vars (`--rb-gold` etc.) may still be used for fixed accent details,
  but background/text should respect the chosen scheme.

### 3.4 Optional per-element colour overrides (scheme is the base)

The `color_scheme` picker (3.3) is always the **base** — it must work correctly with
no overrides set. On top of it, a section MAY expose optional `color` settings (e.g.
heading, text, button background, button text) that let the merchant fine-tune
contrast, which matters most for text/buttons sitting over imagery or video.

The rule: **an override applies only when it has a value; otherwise the section falls
back to the chosen scheme.** Implement this with a per-element CSS variable that
defaults to the scheme colour in the section's `assets/rb-<name>.css`, and is
overridden in the scoped `{%- style -%}` block **only if** the setting is non-blank.

In `assets/rb-<name>.css` — default every override var to the scheme colour:

```css
.rb-section {
  --rb-heading: rgb(var(--color-foreground));
  --rb-btn-bg: rgb(var(--color-button));
  --rb-btn-text: rgb(var(--color-button-text));
}
.rb-section__heading { color: var(--rb-heading); }
```

In the section's scoped `{%- style -%}` block — emit a var **only when set** (the
id selector wins over the class default above):

```liquid
#rb-<name>-{{ section.id }} {
  {%- if s.heading_color != blank %}--rb-heading: {{ s.heading_color }};{% endif -%}
  {%- if s.button_bg != blank %}--rb-btn-bg: {{ s.button_bg }};{% endif -%}
  {%- if s.button_text_color != blank %}--rb-btn-text: {{ s.button_text_color }};{% endif -%}
}
```

- Leave override `color` settings with **no `default`** so they start blank and the
  scheme shows through until the merchant deliberately overrides.
- Label them clearly as optional (e.g. "Heading override (optional)") and group them
  under the colour header, after the `color_scheme` picker.
- Never hardcode an override colour in the CSS file — the CSS only holds the
  scheme-based fallback; actual override values come from the scoped `{%- style -%}`.
- Image/media overlay colours (overlay tint, gradient) are legibility controls, not
  scheme colours — those stay as their own `color`/`range` settings.

---

## 4. Section schema baseline

Every custom section's `{% schema %}` should at minimum contain:

- A `color_scheme` setting (3.3)
- `padding_top` and `padding_bottom` range settings (3.1)
- A clear `name`, sensible `settings`, and `presets` so it's addable in the customizer
- Block-level settings where the section is repeatable (cards, slides, value props)
- All labels referencing locale keys (`t:...`) — no hardcoded English in schema labels
  where a translation is expected.

---

## 5. File / naming conventions

- Section: `sections/rb-<name>.liquid` (e.g. `sections/rb-hero.liquid`)
- Section CSS: `assets/rb-<name>.css`, loaded inside the section via `stylesheet_tag`
- Snippets: `snippets/rb-<name>.liquid`
- Prefix all custom classes with `rb-` to avoid collisions with Dawn.
- Keep one component's styles in one file. No global style bleed.

---

## 6. Translation (EN + ES)

- Wrap every customer-facing string in `{{ '...' | t }}` with keys in `locales/en.default.json`
  and `locales/es.json`.
- Section setting defaults can hold English, but real copy lives in locale files / the
  Translate & Adapt app for store content.
- Do not hardcode Spanish or English directly in section markup.

---

## 7. Before finishing any section — checklist

- [ ] Vanilla CSS only, scoped to `assets/rb-<name>.css`
- [ ] `page-width` used for content width + side padding
- [ ] `padding_top` / `padding_bottom` settings wired via Dawn's `.section-{{ section.id }}-padding` pattern (75% mobile, full at 750px)
- [ ] `color_scheme` picker present and applied; works correctly with no overrides
- [ ] Any per-element colour overrides default to blank and fall back to the scheme (3.4)
- [ ] All strings translatable (EN/ES)
- [ ] Amnestia for display, Open Sans for body
- [ ] Brand colors via CSS vars / color scheme, gold used as accent only
- [ ] Responsive at 750 / 990 breakpoints
- [ ] `presets` block so the section is addable in the customizer
