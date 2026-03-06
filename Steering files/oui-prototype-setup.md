---
inclusion: auto
---

# OUI Prototype Development — Steering Rules

This steering document defines the setup process, build configuration, and UX design standards
for prototyping with the OpenSearch UI (OUI) Framework (`@opensearch-project/oui`).

All prototype work in this workspace MUST follow these rules.

---

## 1. Project Initialization

When scaffolding a new OUI prototype React app:

1. Use Vite with the `react-ts` template:
   ```bash
   npm create vite@latest <project-name> -- --template react-ts
   cd <project-name>
   ```

2. Install OUI and its peer dependencies:
   ```bash
   npm install @opensearch-project/oui --legacy-peer-deps
   npm install react react-dom moment prop-types --legacy-peer-deps
   npm install -D @types/react @types/react-dom sass --legacy-peer-deps
   ```
   Note: `--legacy-peer-deps` is required because OUI declares peer deps for React 16 types,
   but works correctly with React 18.
   ```

3. Install the datemath alias (OUI imports `@opensearch/datemath` which is a security placeholder):
   ```bash
   npm install @elastic/datemath --registry https://registry.npmjs.org --legacy-peer-deps
   ```
   Then alias it in `vite.config.ts`:
   ```ts
   resolve: {
     alias: {
       '@opensearch/datemath': '@elastic/datemath',
     },
   },
   ```

4. Import the OUI theme CSS in your app entry point (`src/main.tsx`):
   ```tsx
   import '@opensearch-project/oui/dist/oui_theme_light.css';
   ```
   For dark mode prototypes, use `oui_theme_dark.css` instead.

5. Wrap your app root with `<OuiProvider>` if available, or simply ensure the CSS import is loaded globally.

---

## 2. Project Structure

Organize prototype files like this:

```
src/
  components/       # Reusable composed components built from OUI primitives
  pages/            # Full page prototypes / views
  tokens/           # Any local design token overrides or extensions
  App.tsx           # Root app component
  main.tsx          # Entry point with OUI CSS import
```

- Keep each prototype page in its own file under `pages/`.
- Composed components that combine multiple OUI components go in `components/`.
- Never duplicate OUI component logic — always import from `@opensearch-project/oui`.

---

## 3. Component Usage Rules

- ALWAYS prefer OUI components over custom HTML elements or third-party UI libraries.
  Use `OuiButton`, `OuiPage`, `OuiFlexGroup`, `OuiText`, `OuiPanel`, etc.
- Import components individually:
  ```tsx
  import { OuiButton, OuiFlexGroup, OuiFlexItem } from '@opensearch-project/oui';
  ```
- Do NOT install or use other UI component libraries (MUI, Chakra, Ant Design, etc.)
  alongside OUI in the same prototype. OUI is the single source of truth for UI.
- When OUI does not have a component for a specific need, build a custom component
  that follows OUI's visual language and spacing conventions.

---

## 4. Design Token & Theming Standards

OUI uses Sass-based theming with pre-built CSS theme files. Follow these rules:

- Use OUI's built-in color variables and spacing. Do NOT hardcode hex colors or pixel values.
- Reference OUI's Sass variables when writing custom styles:
  ```scss
  @import '@opensearch-project/oui/src/themes/oui/oui_colors_light';
  ```
- Stick to OUI's 4px/8px spacing grid. Common spacing values: 4, 8, 12, 16, 24, 32, 48.
- For typography, use `OuiText`, `OuiTitle`, and `OuiCode` components rather than raw
  `<h1>`, `<p>`, or `<code>` tags.
- When extending the theme, place overrides in `src/tokens/` and document what you changed and why.

---

## 5. Layout Standards

- Use `OuiPage`, `OuiPageBody`, `OuiPageContent`, and `OuiPageSideBar` for page-level layout.
- Use `OuiFlexGroup` and `OuiFlexItem` for all flex-based layouts. Do NOT use raw CSS flexbox.
- Use `OuiSpacer` for vertical spacing between sections. Do NOT use margin hacks.
- Use `OuiPanel` for content cards and grouped sections.
- Responsive behavior should rely on OUI's built-in responsive props where available.

---

## 6. Accessibility Standards

OUI components are built with accessibility in mind. Maintain that standard:

- Never remove or override `aria-*` attributes that OUI components provide.
- All interactive elements must be keyboard navigable.
- Use `OuiScreenReaderOnly` for visually hidden but screen-reader-accessible content.
- Color contrast must meet WCAG 2.1 AA minimum (4.5:1 for normal text, 3:1 for large text).
- Every form input must have an associated `<label>` or use OUI's `OuiFormRow` which handles this.
- Images and icons must have meaningful `alt` text or `aria-label`.
- Do NOT rely on color alone to convey information — use icons, text, or patterns as well.

---

## 7. Code Quality Rules

- All prototype components must be written in TypeScript (`.tsx`).
- Props must be typed — no `any` types for component props.
- Use functional components with hooks. No class components.
- Keep components small and focused. If a component exceeds ~150 lines, split it.
- Name files in PascalCase matching the component name: `DataTable.tsx`, `SearchPanel.tsx`.

---

## 8. Build & Dev Server

- OUI dependencies expect Node's `global` variable. Add this polyfill in `index.html` before
  the module script tag:
  ```html
  <script>var global = globalThis;</script>
  ```
- Use `npm run dev` to start the Vite dev server for local prototyping.
- Use `npm run build` to produce a production build for sharing prototypes.
- If Sass compilation issues arise, ensure `sass` is installed as a dev dependency.

---

## 9. Reference Links

- OUI GitHub: https://github.com/opensearch-project/oui
- OUI Living Style Guide: https://oui.opensearch.org/ (component examples and docs)
- OUI FAQ: https://github.com/opensearch-project/oui/blob/master/FAQ.md
- OUI npm: https://www.npmjs.com/package/@opensearch-project/oui
