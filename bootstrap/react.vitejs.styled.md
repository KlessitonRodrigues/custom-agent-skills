# Bootstrap: Vite + React + Styled Components

Step-by-step guide to create a new SPA from zero using this project as template.

---

## Prerequisites

- Node.js 22.x
- Yarn

---

## 1. Project Initialization

```bash
mkdir my-app && cd my-app
yarn init -y
```

---

## 2. package.json

Replace the generated `package.json`:

```json
{
  "name": "@apps/my-app",
  "version": "1.0.0",
  "license": "MIT",
  "type": "module",
  "scripts": {
    "dev": "vite --host",
    "build": "vite build",
    "preview": "vite preview --host",
    "format": "prettier -c -w ./src"
  },
  "dependencies": {
    "react": "^19.2.7",
    "react-dom": "^19.2.7"
  },
  "devDependencies": {
    "@emotion/is-prop-valid": "^1.2.2",
    "@trivago/prettier-plugin-sort-imports": "^6.0.2",
    "@types/react": "^19.2.17",
    "@types/react-dom": "^19.2.3",
    "@types/styled-components": "^5.1.36",
    "@vitejs/plugin-react": "^6.0.3",
    "react-icons": "^5.7.0",
    "react-router-dom": "^7.18.1",
    "styled-components": "^6.4.3",
    "typescript": "^6.0.3",
    "vite": "^8.1.3",
    "prettier": "^3.9.4"
  }
}
```

```bash
yarn install
```

---

## 3. TypeScript Config

**`tsconfig.json`**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "allowSyntheticDefaultImports": true,
    "allowJs": true,
    "sourceMap": true,
    "paths": {
      "src/*": ["./src/*"]
    },
    "strict": true,
    "noImplicitAny": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"]
}
```

---

## 4. Vite Config

**`vite.config.ts`**

```typescript
import react from "@vitejs/plugin-react";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [react()],
  appType: "spa",
  base: "",
  resolve: {
    tsconfigPaths: true,
  },
});
```

---

## 5. Prettier Config

**`.prettierrc`**

```json
{
  "tabWidth": 2,
  "useTabs": false,
  "printWidth": 100,
  "singleQuote": true,
  "arrowParens": "avoid",
  "semi": true,
  "importOrder": ["^src/(.*)$", "^[./]"],
  "importOrderSeparation": true,
  "importOrderSortSpecifiers": true,
  "plugins": ["@trivago/prettier-plugin-sort-imports"]
}
```

---

## 6. Directory Structure

```
src/
├── app.tsx                          # App entry — wires ThemeProvider + RouterProvider
├── global.d.ts                      # Global type declarations (e.g., asset imports)
├── theme.d.ts                       # styled-components DefaultTheme augmentation
├── constants/
│   └── urls.ts                      # External URLs
├── hooks/
│   └── useTheme/
│       └── index.tsx                # ThemeProvider + useTheme hook
├── lib/
│   ├── components/
│   │   └── AppProviders/
│   │       └── index.tsx            # StyleSheetManager + ThemeProvider + GlobalCSS
│   └── common/
│       └── StyledComponents/
│           ├── Containers.ts        # PageContainer, PageContent, Section, Card, …
│           ├── Flex.ts              # Row, Column, …
│           └── …
├── pages/
│   ├── routes.tsx                   # createBrowserRouter route definitions
│   └── <page>/
│       └── index.tsx                # Page component
├── styles/
│   ├── colors.ts                    # Raw color palettes (material, ant, …)
│   ├── global.ts                    # createGlobalStyle — resets, fonts, body defaults
│   ├── theme.ts                     # IThemeProps, getTheme, getLocalTheme, saveLocalTheme
│   └── utils.ts                     # cssSize, screenSize, animations, gradients, IStyledProps
└── index.html                       # Vite HTML entry
```

---

## 7. Source Files

### `src/styles/utils.ts`

```typescript
import { keyframes } from "styled-components";

export type IStyledProps = {
  active?: boolean;
  size?: number;
  gap?: number;
  w?: string | number;
  h?: string | number;
  maxw?: string;
  maxh?: string;
  p?: string;
  m?: string;
  mb?: number;
  left?: boolean;
  right?: boolean;
  top?: boolean;
  bottom?: boolean;
  wrap?: boolean;
  responsive?: boolean | "reverse-left";
  reverse?: boolean;
  half?: boolean;
  fit?: boolean;
  between?: boolean;
  hidden?: boolean;
  full?: boolean;
};

export const cssSize = (size: number | string) => {
  if (typeof size === "string") return size;
  return `${(size * 0.2).toFixed(1)}rem`;
};

export const screenSize = {
  mobileS: 320,
  mobileM: 425,
  mobileL: 480,
  tablet: 768,
  laptopS: 900,
  laptopM: 1024,
  laptopL: 1180,
  desktopS: 1440,
  desktopM: 1980,
  desktopL: 2560,
};

export const animations = {
  fadeIn: keyframes`from { opacity: 0; } to { opacity: 1; }`,
  fadeUp: keyframes`from { transform: translateY(3rem); opacity: 0; } to { opacity: 1 }`,
  slideUp: keyframes`from { transform: translateY(12rem); opacity: 0.4; } to { opacity: 1; }`,
};

export const isMobileScreen = () => window.innerWidth <= screenSize.tablet;
```

### `src/styles/theme.ts`

```typescript
export type IThemeColors = "indigo" | "purple" | "blue" | "green" | "red";

export type IThemeProps = {
  colors: {
    main: string;
    mainBg: string;
    mainText: string;
    bg1: string;
    bg2: string;
    bg3: string;
    bg4: string;
    text1: string;
    text2: string;
    text3: string;
    text4: string;
    gray: string;
    yellow: string;
    red: string;
    blue: string;
    green: string;
    white: string;
    black: string;
  };
  fontSize: {
    md: string;
    sm: string;
    xs: string;
    label: string;
    h1: string;
    h2: string;
    h3: string;
    h4: string;
    h5: string;
    h6: string;
  };
  border: { sm: string; md: string; lg: string };
  radius: { sm: string; md: string; lg: string; xl: string; full: string };
  shadow: { sm: string; md: string; lg: string };
};

export const getTheme = (dark: boolean, color: IThemeColors): IThemeProps => {
  // Return full theme object; swap bg/text tokens when dark === true
  // See existing implementation for reference values
  throw new Error("implement getTheme");
};

export const getLocalTheme = () => {
  const mode = localStorage.getItem("theme_mode") || "light";
  const color = (localStorage.getItem("theme_color") ||
    "indigo") as IThemeColors;
  return { mode, color } as { mode: "light" | "dark"; color: IThemeColors };
};

export const saveLocalTheme = (mode: "light" | "dark", color: IThemeColors) => {
  if (mode) localStorage.setItem("theme_mode", mode);
  if (color) localStorage.setItem("theme_color", color);
};
```

### `src/theme.d.ts`

```typescript
import "styled-components";
import { IThemeProps } from "src/styles/theme";

declare module "styled-components" {
  export interface DefaultTheme extends IThemeProps {}
}
```

### `src/styles/global.ts`

```typescript
import { createGlobalStyle, css } from "styled-components";

export default createGlobalStyle(
  ({ theme }) => css`
    * {
      margin: 0;
      padding: 0;
      outline: 0;
      box-sizing: border-box;
    }
    html {
      font-size: 16px;
    }
    body {
      font-family: "Roboto", sans-serif;
      font-size: ${theme.fontSize.md};
      min-height: 100vh;
      background-color: ${theme.colors.bg4};
      color: ${theme.colors.text4};
    }
  `,
);
```

### `src/hooks/useTheme/index.tsx`

```typescript
import { PropsWithChildren, createContext, useContext, useEffect, useMemo, useState } from 'react';

import { IThemeColors, getLocalTheme, saveLocalTheme } from 'src/styles/theme';

type IUseThemeCtx = {
  isDark: boolean;
  color: IThemeColors;
  setDark: (value: boolean) => void;
  setColor: (value: IThemeColors) => void;
};

// @ts-expect-error typeError
const ThemeCtx = createContext<IUseThemeCtx>(null);

const localTheme = getLocalTheme();

export const ThemeProvider = (props: PropsWithChildren) => {
  const [isDark, setDark] = useState(localTheme.mode === 'dark');
  const [color, setColor] = useState<IThemeColors>(localTheme.color);

  useEffect(() => {
    saveLocalTheme(isDark ? 'dark' : 'light', color);
  }, [isDark, color]);

  const value = useMemo(
    () => ({ isDark, color, setDark, setColor }),
    [isDark, color],
  );

  return <ThemeCtx.Provider value={value}>{props.children}</ThemeCtx.Provider>;
};

export default () => useContext(ThemeCtx);
```

### `src/lib/components/AppProviders/index.tsx`

```typescript
import isValidProp from '@emotion/is-prop-valid';
import { PropsWithChildren } from 'react';
import { StyleSheetManager, ThemeProvider } from 'styled-components';

import useTheme from 'src/hooks/useTheme';
import GlobalCSS from 'src/styles/global';
import { getTheme } from 'src/styles/theme';

const AppProviders = (props: PropsWithChildren) => {
  const { isDark, color } = useTheme();
  const theme = getTheme(isDark, color);

  return (
    <StyleSheetManager disableCSSOMInjection shouldForwardProp={isValidProp}>
      <ThemeProvider theme={theme}>
        <GlobalCSS />
        {props.children}
      </ThemeProvider>
    </StyleSheetManager>
  );
};

export default AppProviders;
```

### `src/lib/common/StyledComponents/Containers.ts`

```typescript
import styled, { css } from "styled-components";

import { cssSize, screenSize } from "src/styles/utils";

export const PageContainer = styled.div(
  () => css`
    width: 100%;
    height: 100%;
    z-index: 3;
  `,
);

export const PageContent = styled.div(
  () => css`
    width: 100%;
    margin: auto;
    min-height: 97vh;
    max-width: ${screenSize.laptopL}px;
  `,
);

export const Section = styled.section(
  ({ theme }) => css`
    margin: ${cssSize(4)} ${cssSize(2)};
    font-size: ${theme.fontSize.h4};
    color: ${theme.colors.text2};
  `,
);

export const Card = styled.div(
  ({ theme }) => css`
    position: relative;
    width: 100%;
    border-radius: ${theme.radius.md};
    background-color: ${theme.colors.bg2};
    padding: ${cssSize(4)};
  `,
);
```

### `src/lib/common/StyledComponents/Flex.ts`

```typescript
import styled, { css } from "styled-components";

import { IStyledProps, cssSize, screenSize } from "src/styles/utils";

const FlexBase = styled.div<IStyledProps>(
  ({ w, h, fit }) => css`
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    ${w && `width: ${w};`}
    ${h && `height: ${h};`}
    ${fit && `width: fit-content;`}
  `,
);

export const Row = styled(FlexBase)(
  ({ gap, left, right, top, bottom, wrap, between, responsive }) => css`
    flex-direction: row;
    gap: ${cssSize(gap ?? 2)};
    ${left && "justify-content: flex-start;"}
    ${right && "justify-content: flex-end;"}
    ${between && "justify-content: space-between;"}
    ${top && "align-items: flex-start;"}
    ${bottom && "align-items: flex-end;"}
    ${wrap && "flex-wrap: wrap;"}

    @media (max-width: ${screenSize.laptopS}px) {
      ${responsive && "flex-direction: column;"}
      ${responsive && left && "align-items: flex-start;"}
    }
  `,
);

export const Column = styled(FlexBase)(
  ({ gap, left, right, top, bottom }) => css`
    flex-direction: column;
    gap: ${cssSize(gap ?? 2)};
    ${left && "align-items: flex-start;"}
    ${right && "align-items: flex-end;"}
    ${top && "justify-content: flex-start;"}
    ${bottom && "justify-content: flex-end;"}
  `,
);
```

### `src/pages/routes.tsx`

```typescript
import { RouteObject } from 'react-router-dom';

import HomePage from 'src/pages/home';

const Router: RouteObject[] = [
  { path: '/', element: <HomePage /> },
];

export default Router;
```

### `src/app.tsx`

```typescript
import { createRoot } from 'react-dom/client';
import { RouterProvider, createBrowserRouter } from 'react-router-dom';

import Router from 'src/pages/routes';

import { ThemeProvider } from './hooks/useTheme';
import AppProviders from './lib/components/AppProviders';

const App = () => (
  <ThemeProvider>
    <AppProviders>
      <RouterProvider router={createBrowserRouter(Router)} />
    </AppProviders>
  </ThemeProvider>
);

const rootEl = document.getElementById('root');
if (rootEl) createRoot(rootEl).render(<App />);
```

### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/app.tsx"></script>
  </body>
</html>
```

---

## 8. Development Workflow

```bash
# Start local dev server (port 5173 by default)
yarn dev

# Type-check
yarn tsc --noEmit

# Build for production
yarn build

# Preview production build
yarn preview

# Format source files
yarn format
```

---

## 9. Adding a New Page

1. Create `src/pages/<page>/index.tsx` with the page component
2. Add the route to `src/pages/routes.tsx`
3. Add any page-specific sections under `src/lib/sections/<page>/`

---

## 10. Conventions

| Rule              | Detail                                                                                             |
| ----------------- | -------------------------------------------------------------------------------------------------- |
| Theme tokens      | Always use `theme.colors.*`, `theme.fontSize.*`, `theme.radius.*` — never hardcode colors or sizes |
| Sizing            | Use `cssSize(n)` for spacing/padding (1 unit = 0.2rem)                                             |
| Breakpoints       | Use `screenSize.*` constants from `src/styles/utils.ts`                                            |
| Styled components | Define with `css\`\``helper for theme access; use`IStyledProps` for shared prop types              |
| CSS injection     | Always wrap with `StyleSheetManager disableCSSOMInjection shouldForwardProp={isValidProp}`         |
| Theme forwarding  | Use `@emotion/is-prop-valid` to prevent custom props from leaking to DOM                           |
| Imports           | Use `src/` path alias — never use relative `../../` beyond one level                               |
| Import order      | Enforced by `@trivago/prettier-plugin-sort-imports`: external → `src/` → relative                  |
| App entry         | `ThemeProvider` wraps `AppProviders` wraps `RouterProvider` — order is required                    |
| Dark mode         | Persisted to `localStorage` under `theme_mode` and `theme_color` keys                              |
