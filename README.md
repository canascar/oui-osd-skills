# OUI Prototype Workspace

A prototyping workspace built with the [OpenSearch UI (OUI) Framework](https://github.com/opensearch-project/oui) and React + TypeScript via Vite.

## Quick Start

```bash
npm create vite@latest my-prototype -- --template react-ts
cd my-prototype
```

Install OUI and its peer dependencies:

```bash
npm install @opensearch-project/oui --legacy-peer-deps
npm install react react-dom moment prop-types --legacy-peer-deps
npm install -D @types/react @types/react-dom sass --legacy-peer-deps
```

Install the datemath alias (OUI imports `@opensearch/datemath` which is a security placeholder):

```bash
npm install @elastic/datemath --registry https://registry.npmjs.org --legacy-peer-deps
```

Add the alias in `vite.config.ts`:

```ts
resolve: {
  alias: {
    '@opensearch/datemath': '@elastic/datemath',
  },
},
```

Import the OUI theme in `src/main.tsx`:

```tsx
import '@opensearch-project/oui/dist/oui_theme_light.css';
```

Add the `global` polyfill in `index.html` before the module script tag:

```html
<script>var global = globalThis;</script>
```

Run the dev server:

```bash
npm run dev
```

## Project Structure

```
src/
  components/       # Reusable composed components built from OUI primitives
  pages/            # Full page prototypes / views
  tokens/           # Local design token overrides or extensions
  App.tsx           # Root app component
  main.tsx          # Entry point with OUI CSS import
```

## Component Usage

- Always prefer OUI components (`OuiButton`, `OuiPage`, `OuiFlexGroup`, `OuiText`, `OuiPanel`, etc.) over custom HTML or third-party UI libraries.
- Import components individually:
  ```tsx
  import { OuiButton, OuiFlexGroup, OuiFlexItem } from '@opensearch-project/oui';
  ```
- Do not install other UI libraries (MUI, Chakra, Ant Design, etc.) alongside OUI.

## Layout

- Use `OuiPage`, `OuiPageBody`, `OuiPageContent`, and `OuiPageSideBar` for page layout.
- Use `OuiFlexGroup` / `OuiFlexItem` for flex layouts instead of raw CSS flexbox.
- Use `OuiSpacer` for vertical spacing and `OuiPanel` for content cards.

## Theming

- Use OUI's built-in color variables and spacing. Don't hardcode hex colors or pixel values.
- Stick to OUI's 4px/8px spacing grid (4, 8, 12, 16, 24, 32, 48).
- For typography, use `OuiText`, `OuiTitle`, and `OuiCode` instead of raw HTML tags.
- Place any theme overrides in `src/tokens/` with documentation on what changed and why.
- For dark mode, swap to `oui_theme_dark.css`.

## Accessibility

- Don't remove or override `aria-*` attributes provided by OUI components.
- All interactive elements must be keyboard navigable.
- Use `OuiScreenReaderOnly` for visually hidden but screen-reader-accessible content.
- Color contrast must meet WCAG 2.1 AA minimum (4.5:1 normal text, 3:1 large text).
- Every form input needs an associated label — use `OuiFormRow` which handles this.
- Don't rely on color alone to convey information.

## Code Standards

- All components in TypeScript (`.tsx`) with typed props (no `any`).
- Functional components with hooks only — no class components.
- Keep components under ~150 lines. Split if they grow beyond that.
- File names in PascalCase matching the component: `DataTable.tsx`, `SearchPanel.tsx`.

## Build

```bash
npm run build
```

## References

- [OUI GitHub](https://github.com/opensearch-project/oui)
- [OUI Living Style Guide](https://oui.opensearch.org/) — component examples and docs
- [OUI FAQ](https://github.com/opensearch-project/oui/blob/master/FAQ.md)
- [OUI on npm](https://www.npmjs.com/package/@opensearch-project/oui)
