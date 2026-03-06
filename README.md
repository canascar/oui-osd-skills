# OUI Prototype Workspace

A prototyping workspace built with the [OpenSearch UI (OUI) Framework](https://github.com/opensearch-project/oui) and React + TypeScript via Vite.

## Quick Start

```bash
npm create vite@latest oui-prototype -- --template react-ts
cd oui-prototype
```

Install OUI and its peer dependencies:

```bash
npm install @opensearch-project/oui --legacy-peer-deps
npm install react react-dom moment prop-types --legacy-peer-deps
npm install -D @types/react @types/react-dom sass --legacy-peer-deps
```

> Note: `--legacy-peer-deps` is required because OUI declares peer deps for React 16 types, but works correctly with React 18.

Install the datemath alias (OUI imports `@opensearch/datemath` which is a security placeholder):

```bash
npm install @elastic/datemath --legacy-peer-deps
```

Configure `vite.config.ts` with the datemath alias:

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@opensearch/datemath': '@elastic/datemath',
    },
  },
});
```

Add the `global` polyfill in `index.html` before the module script tag (OUI dependencies expect Node's `global` variable):

```html
<script>var global = globalThis;</script>
<script type="module" src="/src/main.tsx"></script>
```

Run the dev server:

```bash
npm run dev
```

## Theme System

OUI ships with 6 theme CSS files. Every prototype should include a ThemeProvider and ThemePicker.

### Available Themes

| Theme Key   | CSS File                    | Description                  |
|-------------|-----------------------------|------------------------------|
| OUI Light   | `oui_theme_light.css`       | Standard light theme         |
| OUI Dark    | `oui_theme_dark.css`        | Standard dark theme          |
| Next Light  | `oui_theme_next_light.css`  | Refreshed/modern light theme |
| Next Dark   | `oui_theme_next_dark.css`   | Refreshed/modern dark theme  |
| V9 Light    | `oui_theme_v9_light.css`    | V9 variant light theme       |
| V9 Dark     | `oui_theme_v9_dark.css`     | V9 variant dark theme        |

### ThemeContext Pattern

Create `src/ThemeContext.tsx` that:
- Uses dynamic `import('...css?url')` for each theme so Vite bundles all CSS files
- Provides `theme` (current ThemeKey), `setTheme`, and `isDark` via React context
- Dynamically swaps a `<link>` element's `href` in `document.head` on theme change
- Do NOT use static CSS imports in `main.tsx` — the ThemeProvider handles loading

### ThemePicker Pattern

Create `src/components/ThemePicker.tsx` that:
- Renders an `OuiButtonIcon` with `display="empty"` and `size="s"`
- Uses `iconType="moon"` for light themes, `iconType="invert"` for dark themes (OUI does NOT have a `sun` icon)
- On click, shows a dropdown panel (use `OuiPanel` with absolute positioning, NOT `OuiPopover` which has portal rendering issues in React 18)
- Lists all 6 themes with a `check` icon next to the active one
- Place the ThemePicker in the top-right of every page, or in the `OuiHeader` right section

### Wrap the App

In `main.tsx`:
```tsx
import { ThemeProvider } from './ThemeContext';
// Do NOT import any OUI CSS here — ThemeProvider handles it

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <ThemeProvider>
      <App />
    </ThemeProvider>
  </React.StrictMode>
);
```

## OpenSearch Branding

- Use `OuiHeaderLogo` with `iconType="logoOpenSearch"` in the header for branding
- The `logoOpenSearch` icon works correctly in both light and dark themes
- For standalone logo display, use `<OuiIcon type="logoOpenSearch" size="xxl" />`
- Do NOT use custom SVGs or images for the OpenSearch logo

### Header Branding Pattern

```tsx
<OuiHeader position="fixed">
  <OuiHeaderSection grow={false}>
    <OuiHeaderSectionItem border="right">
      {/* Collapsible nav button goes here */}
    </OuiHeaderSectionItem>
    <OuiHeaderSectionItem>
      <OuiHeaderLogo iconType="logoOpenSearch">OpenSearch</OuiHeaderLogo>
    </OuiHeaderSectionItem>
  </OuiHeaderSection>
  <OuiHeaderSection side="right">
    <OuiHeaderSectionItem>
      <OuiHeaderLinks>
        <OuiHeaderLink href="#">Documentation</OuiHeaderLink>
      </OuiHeaderLinks>
    </OuiHeaderSectionItem>
    <OuiHeaderSectionItem>
      <ThemePicker />
    </OuiHeaderSectionItem>
  </OuiHeaderSection>
</OuiHeader>
```

### Available Logo Icons

`logoOpenSearch`, `logoAWS`, `logoAWSMono`, `logoElastic`, `logoKibana`, `logoGithub`,
`logoSlack`, `logoDocker`, `logoKubernetes`, `logoGCP`, `logoGCPMono`, `logoAzure`,
`logoAzureMono`, `logoIBM`, `logoIBMMono`

## Navigation Components

### OuiHeader (Global Header Bar)
- Use `OuiHeader` with `position="fixed"` for a sticky top bar
- `OuiHeaderSection` groups left/right content
- `OuiHeaderSectionItem` wraps individual items; use `border="right"` for dividers
- `OuiHeaderSectionItemButton` for icon buttons in the header
- `OuiHeaderLinks` / `OuiHeaderLink` for nav links
- When using a fixed header, add `paddingTop: 48` to the page content below it

### OuiCollapsibleNav (Flyout Side Navigation)
- Renders as a flyout panel from the left side
- Requires `isOpen`, `onClose`, and a `button` prop (the hamburger toggle)
- Use `OuiCollapsibleNavGroup` inside for grouped sections with titles and icons
- Place `OuiSideNav` inside groups for the actual menu tree

Hamburger button pattern:
```tsx
const navButton = (
  <OuiHeaderSectionItemButton
    aria-label="Toggle navigation"
    onClick={() => setNavIsOpen(!navIsOpen)}
  >
    <OuiIcon type="menu" size="m" />
  </OuiHeaderSectionItemButton>
);
```

### OuiSideNav (Static Side Navigation)
- Takes an `items` array with nested `items` for tree structure
- Each item has `name`, `id`, `isSelected`, and `onClick`
- Use inside `OuiCollapsibleNavGroup` or standalone in a sidebar

### Workspace Icons for Navigation
OUI includes workspace-specific icons for nav groups:
`wsAnalytics`, `wsEssentials`, `wsObservability`, `wsSearch`, `wsSecurityAnalytics`, `wsSelector`

## Icon Cache (Required for React 18)

OUI's `OuiIcon` is a class component that uses dynamic `import()` to load icon SVGs at runtime. This triggers a `setState` before mount in React 18 strict mode, causing icons to render as empty gray boxes.

Fix: Pre-load all icons your app uses via `appendIconComponentCache` before the first render.

Create `src/iconCache.ts`:
```ts
import { appendIconComponentCache } from '@opensearch-project/oui/es/components/icon/icon';
import { icon as logoOpenSearch } from '@opensearch-project/oui/es/components/icon/assets/logo_opensearch';
import { icon as user } from '@opensearch-project/oui/es/components/icon/assets/user';
// ... import each icon your app uses

appendIconComponentCache({
  logoOpenSearch,
  user,
  // ... register each icon
});
```

Add a type declaration file `src/oui-icons.d.ts`:
```ts
declare module '@opensearch-project/oui/es/components/icon/icon' {
  export const appendIconComponentCache: (map: Record<string, React.ComponentType>) => void;
}
declare module '@opensearch-project/oui/es/components/icon/assets/*' {
  import { ComponentType } from 'react';
  export const icon: ComponentType;
}
```

Import the cache as the FIRST line in `main.tsx`:
```ts
import './iconCache'; // Must be first
```

Icon asset file names use snake_case: `logo_opensearch`, `eye_closed`, `arrow_down`, etc.
Find the exact file name in `node_modules/@opensearch-project/oui/es/components/icon/assets/`.

## Icon Reference

OUI has 400+ built-in icons. Always use OUI icons — do NOT import external icon libraries.

### How to Use Icons
```tsx
import { OuiIcon } from '@opensearch-project/oui';
<OuiIcon type="iconName" size="m" />
```

Sizes: `s`, `m`, `l`, `xl`, `xxl`

### Common Icon Categories

**Navigation & UI:**
`menu`, `menuDown`, `menuLeft`, `menuRight`, `menuUp`, `arrowDown`, `arrowLeft`,
`arrowRight`, `arrowUp`, `sortDown`, `sortUp`, `sortLeft`, `sortRight`, `sortable`,
`popout`, `expand`, `expandMini`, `minimize`, `fullScreen`, `fullScreenExit`,
`cross`, `check`, `plus`, `minus`, `pencil`, `trash`, `copy`, `cut`, `save`,
`refresh`, `search`, `filter`, `gear`, `wrench`, `link`, `unlink`, `share`,
`download`, `exportAction`, `importAction`

**Status & Feedback:**
`check`, `checkInCircleFilled`, `cross`, `crossInCircleFilled`, `alert`, `warning`,
`danger`, `help`, `iInCircle`, `questionInCircle`, `bell`, `bellSlash`,
`faceHappy`, `faceNeutral`, `faceSad`, `heart`, `starFilled`, `starEmpty`,
`thumbsUp`, `thumbsDown`

**Data & Visualization:**
`visArea`, `visAreaStacked`, `visBarVertical`, `visBarVerticalStacked`,
`visBarHorizontal`, `visBarHorizontalStacked`, `visLine`, `visPie`, `visMetric`,
`visGauge`, `visGoal`, `visTable`, `visTagCloud`, `visText`, `visTimelion`,
`visVega`, `visMapCoordinate`, `visMapRegion`, `heatmap`, `lineChart`, `stats`

**Theme & Display:**
`moon` (for light-to-dark toggle), `invert` (for dark-to-light toggle),
`color`, `eye`, `eyeClosed`, `glasses`, `image`, `document`, `documents`

**App & Feature Icons:**
`dashboardApp`, `discoverApp`, `consoleApp`, `devToolsApp`, `managementApp`,
`visualizeApp`, `securityApp`, `securityAnalyticsApp`, `indexManagementApp`,
`notebookApp`, `reportingApp`, `spacesApp`

**Navigation-Specific (nav prefix):**
`navOverview`, `navDashboards`, `navDiscover`, `navDevtools`, `navData`,
`navManage`, `navNotifications`, `navReports`, `navIntegrations`, `navModels`,
`navServices`, `navServiceMap`, `navInfo`, `navGetStarted`, `navAlerting`,
`navAnomalyDetection`, `navDetectionRules`, `navSecurityCases`, `navSecurityFindings`

**Workspace Icons (ws prefix):**
`wsAnalytics`, `wsEssentials`, `wsObservability`, `wsSearch`,
`wsSecurityAnalytics`, `wsSelector`

**Important:** OUI does NOT have a `sun` icon — use `invert` for light mode toggle. Use `moon` for dark mode toggle.

## Project Structure

```
src/
  components/       # Reusable composed components (ThemePicker, etc.)
  pages/            # Full page prototypes / views
  tokens/           # Local design token overrides or extensions
  ThemeContext.tsx   # Theme provider with all 6 themes
  App.tsx           # Root app component with routing
  main.tsx          # Entry point — wraps App in ThemeProvider (no CSS import)
```

- Keep each prototype page in its own file under `pages/`.
- Composed components that combine multiple OUI components go in `components/`.
- Never duplicate OUI component logic — always import from `@opensearch-project/oui`.

## Component Usage

- Always prefer OUI components over custom HTML or third-party UI libraries.
- Import components individually from `@opensearch-project/oui`.
- Do NOT install other UI libraries (MUI, Chakra, Ant Design, etc.) alongside OUI.
- When OUI does not have a component for a specific need, build a custom component that follows OUI's visual language and spacing conventions.
- NOTE: `OuiPopover` has portal rendering issues in React 18 with Vite. For dropdown menus, use `OuiPanel` with absolute positioning and a click-outside handler instead.

## Layout

- Use `OuiPage`, `OuiPageBody`, `OuiPageContent`, and `OuiPageSideBar` for page layout.
- Use `OuiFlexGroup` / `OuiFlexItem` for flex layouts instead of raw CSS flexbox.
- Use `OuiSpacer` for vertical spacing and `OuiPanel` for content cards.
- Responsive behavior should rely on OUI's built-in responsive props where available.

## Design Tokens & Theming

- Use OUI's built-in color variables and spacing. Don't hardcode hex colors or pixel values.
- Use CSS custom properties from the active theme: `var(--ouiColorPrimary)`, `var(--ouiColorLightestShade)`, etc.
- Stick to OUI's 4px/8px spacing grid (4, 8, 12, 16, 24, 32, 48).
- For typography, use `OuiText`, `OuiTitle`, and `OuiCode` instead of raw HTML tags.
- Place any theme overrides in `src/tokens/` with documentation on what changed and why.

## Accessibility

- Don't remove or override `aria-*` attributes provided by OUI components.
- All interactive elements must be keyboard navigable.
- Use `OuiScreenReaderOnly` for visually hidden but screen-reader-accessible content.
- Color contrast must meet WCAG 2.1 AA minimum (4.5:1 normal text, 3:1 large text).
- Every form input needs an associated label — use `OuiFormRow` which handles this.
- Images and icons must have meaningful `alt` text or `aria-label`.
- Don't rely on color alone to convey information — use icons, text, or patterns as well.

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
