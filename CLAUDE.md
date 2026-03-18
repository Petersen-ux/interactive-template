# Webflow itemplate.design — Projektkontext

## Rolle & Sprache
Claude agiert als **Senior Webflow Engineer / Design System Architect**.
Kommunikation auf Deutsch. Stephan kommuniziert knapp und direktiv ("Go", "Done").
Bei Unklarheiten: `TODO:` markieren — niemals Werte erfinden.

## Projekt-IDs
- Site: `699f435633df7bdd84c75b23` (itemplate.design.webflow.com)
- Designer: `https://itemplate.design.webflow.com?app=dc8209c65e3ec02254d15275ca056539c89f6d15741893a0adf29ad6f381eb99`
- Home-Page: `69a078e576a988d6b843b653`
- Staging "Klassen und Elemente": `699f435733df7bdd84c75b6c`
- Styleguide: `69a97d34aa666caae2172cb0`
- Navbar-Component: `1154efb0-99e9-e2e1-cc01-27e760a1e28a`
- Footer-Component: `6a66de7c-4f63-b000-0122-4edf6210a909`

---

## LEITPRINZIP: Webflow-native Lösungen zuerst

**Prioritäts-Reihenfolge (verbindlich):**
1. **Webflow-Designer-Bordmittel** — Styles, Variables, Interactions, Flexbox/Grid, native Elemente (Navbar, Tabs, Slider, Form)
2. **Webflow Variables** — alle Farben, Fonts, Sizes über Variable-Collections, nie als Hardcode
3. **Combo Classes** — Varianten über zusätzliche Klassen, nicht über Duplikate
4. **Custom CSS nur als letzter Ausweg** — wenn Webflow-Bordmittel nachweislich nicht ausreichen (z.B. `height:auto` ist nicht animierbar → Custom CSS erlaubt)

> Jeder Custom-Code-Einsatz muss begründet werden. Ziel: möglichst wenig eingebettetes CSS/JS.

---

## Struktur-Hierarchie (harte Pflicht)

```
SECTION → BASE-CONTAINER → INHALTSELEMENT(E)
```

Keine Ausnahmen. Jede `section` enthält genau 1× `base-container` als direktes Kind.

**Beobachtetes Pipely-Pattern (Template-Bestand):**
```
section_[name]
  └─ padding-global          (px: 2.5rem, @small: 1.25rem)
       └─ container-large    (max-width: 80rem, margin: auto)
            └─ padding-section-[size]  (vertikales Spacing)
                 └─ [component]_wrapper / _grid / _content
```

**Wrapper-Ebenen:**
- `page-wrapper` → `main-wrapper` → Sections
- Sections tragen: `section` + semantischer Modifier (`section_hero`, `section_team`, `section footer`)

---

## CSS-Klassenbenennung

**Kanonisch: kebab-case.** Webflow ist case-insensitive bei Klassennamen.
Keine Leerzeichen, keine PascalCase-Klassen als Neuzugang.

**Legacy-Migration:** PascalCase- und Space-Klassen (aus Finexo-Import) werden systematisch auf kebab-case migriert. Gruppen 1–5 abgeschlossen, Gruppen 6–8 offen.

**Klassen-Systematik:**
- Struktur: `section`, `base-container`, `padding-global`, `container-large`
- Typografie-Base: `standard-h1` … `standard-h4`, `standard-paragraph`, `standard-teaser`, `standard-ui`, `standard-small`
- Typografie-Combo: `.white`, `.muted`, `.white-soft`, `.white-muted`, `.bold`, `.semibold`
- Farben: `text-color-[name]`, `bg-[name]`
- Buttons: `button` + Combo (`is-secondary`, `is-tertiary`, `is-outline`, `is-small`, `is-icon`)
- Visibility: `hide`, `hide-tablet`, `hide-mobile-landscape`, `hide-mobile`
- Components: `navbar_[part]`, `hero_[part]`, `footer_[part]`, `team_[part]`

**Beispiel-Kombination in Webflow:**
- `standard-h1` + `white` → weiße H1 auf dunklem BG
- `standard-ui` + `bold` + `white` → Button-Label auf Primary-BG
- `standard-small` + `white-muted` → Footer-Copyright

---

## Design Tokens (SSOT: @knowledge/styles.css)

**Fonts:** Manrope (Headings), Inter (Body), Montserrat (Funktional/Buttons)
**Farben — Webflow Variable-Collections:**
- Markenfarben (`collection-b2fab82a-eea3-3a30-0ceb-4eebf2dc5d08`): Primär `#1E0EAB`, Sekundär `#F54927`, Kontrast `#F7920F`, Akzent `#584efe`
- Base (`collection-ceeee9fa-f5eb-63b5-9d80-d5ab63773639`): Sizing/Spacing

**Type Scale (aus styles.css :root):**
- H1: 3.5rem / lh 4.2rem / weight 700
- H2: 2.25rem / lh 2.7rem / weight 600
- H3: 1.5rem / lh 1.8rem / weight 600
- H4: 1.25rem / lh 1.5rem / weight 600
- Body: 1rem / lh 1.2rem / weight 400
- UI: 0.9375rem / lh 1 / weight 500
- Small: 0.875rem / lh 1.05rem / weight 400

**Spacing:** Container max 1920px, Padding `clamp(16px, 4vw, 48px)`
**Radii:** 8px / 16px / 40px (Marke), Template: 1.25rem (team_item)
**Breakpoints:** Desktop → 991px (Tablet) → 767px (Mobile Landscape) → 479px (Mobile Portrait)

---

## Build-Regeln

1. `section` → `base-container` → Content (Grid/Flex)
2. Pipely-kompatible Utilities nutzen: `padding-global` + `padding-section-*` + `container-*`
3. Nur Tokens/Utilities/Components — keine freien Inline-Styles
4. Farben ausschließlich via `text-color-*` und `bg-*` Klassen
5. Typografie über `standard-h1..h4`, `standard-paragraph`, `standard-ui` etc.
6. Responsive: 991/767/479 beachten, Min-Width 360px ohne Overflow
7. Pro Seite genau ein H1, korrekte Heading-Hierarchie
8. Alt-Texte für alle bedeutungstragenden Bilder

---

## Webflow MCP API — Kritische Constraints

- `set_style` **ersetzt alle Klassen** — Combo-Classes werden gelöscht wenn nicht explizit im `style_names`-Array mitgegeben
- Webflow ist **case-insensitive** bei Style-Uniqueness — `create_style("teaser")` scheitert wenn `Teaser` existiert
- `set_text` funktioniert **nicht auf DivBlock** — Child-Span mit `set_dom_config: {dom_tag: 'span'}` nötig
- `skip_properties: false` bei `get_styles` explizit setzen — Default lässt CSS-Werte weg
- `set_image_asset` ist korrekt für Image-Elemente — `add_or_update_attribute` mit `src` scheitert
- Style-Namen mit Whitespace-Anomalien können **nicht per API** umbenannt werden → manuell im Designer
- `element_builder`-Batches **klein halten** — große verschachtelte Strukturen → Timeout
- `update_style` mit `remove_properties` entfernt Breakpoint-Overrides korrekt
- Line-Height-Variablen sind **absolute rem-Werte**, keine unitless Ratios
- Variable-Mode-Erstellung nur per Name — Breakpoint-Zuweisung **manuell im Designer**
- **Designer-Tab muss sichtbar bleiben** — Backgrounding → MCP-Connection-Drop (Chrome Memory Saver deaktivieren)

---

## Offene Aufgaben

- CSS-Migration Gruppen 6–8 (PascalCase → kebab-case)
- 3 Klassen manuell umbenennen: `Teaser`, `Button-Wrapper`, `Second` (API-Limit)
- Mobile Portrait Variable Mode: Breakpoint-Zuweisung im Designer
- App-Showcase Slider: Landscape-Layout, max-height 500px, two-col (Topline/H2/Body)
- Accordion Arrow-Rotation (via `w--open` + CSS transform)
- forumgruppe.de: Projektanalyse und CSS-Konventionen (Designer-Zugang nötig)

---

## Referenz-Dateien (via @-Import in Claude Code)

- Vollständige Projektregeln: @knowledge/projekt-basis.md
- CSS SSOT (1.815 Zeilen): @knowledge/styles.css
- Design-System-Dokumentation: @knowledge/design.md
- Team-Komponentenprofil: @knowledge/section_team_profile.json
- HTML-Exports: @knowledge/blog.txt, @knowledge/services.txt, @knowledge/case-studies.txt, @knowledge/about.txt, @knowledge/design-home.txt, @knowledge/design-contact.txt
