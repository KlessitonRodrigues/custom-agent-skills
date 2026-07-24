# Vite + React + Styled Components Coding Rules

## Folder Structure

```
src/
├── app.tsx                          # App entry — wires ThemeProvider + RouterProvider
├── theme.d.ts                       # DefaultTheme augmentation for styled-components
├── constants/
│   └── urls.ts                      # External link constants
├── hooks/
│   └── useTheme/
│       └── index.tsx                # ThemeProvider + useTheme hook
├── lib/
│   ├── components/
│   │   └── AppProviders/
│   │       └── index.tsx            # StyleSheetManager + ThemeProvider + GlobalCSS
│   └── common/
│       └── StyledComponents/
│           ├── Containers.ts        # PageContainer, PageContent, Section, Card
│           ├── Flex.ts              # Row, Column
│           └── …
├── pages/
│   ├── routes.tsx                   # Route definitions array
│   └── <page>/
│       └── index.tsx                # Page component
├── styles/
│   ├── colors.ts                    # Raw color palettes only — no logic
│   ├── global.ts                    # createGlobalStyle — resets and body defaults
│   ├── theme.ts                     # IThemeProps, getTheme, getLocalTheme, saveLocalTheme
│   └── utils.ts                     # cssSize, screenSize, animations, IStyledProps
└── index.html
```

// Never place page-specific components outside their page folder
// Never import from `../../` beyond one level — always use `src/` alias

## Theme (`styles/theme.ts`)

// `IThemeProps` defines the full shape of the theme — extend it when adding new tokens
// `getTheme(dark, color)` is the single source of truth — never construct partial themes
// Dark mode swaps only bg/text/border tokens — main/accent colors stay the same
// `getLocalTheme()` / `saveLocalTheme()` use `localStorage` keys `theme_mode` and `theme_color`

```typescript
export type IThemeColors = 'indigo' | 'purple' | 'blue' | 'green' | 'red';

export type IThemeProps = {
  colors: { main: string; mainBg: string; bg1: string; bg2: string; bg3: string; bg4: string; text1: string; text2: string; text3: string; text4: string; /* ... */ };
  fontSize: { md: string; sm: string; xs: string; label: string; h1: string; /* ... */ };
  border:   { sm: string; md: string; lg: string };
  radius:   { sm: string; md: string; lg: string; xl: string; full: string };
  shadow:   { sm: string; md: string; lg: string };
};
```

## styled-components DefaultTheme (`theme.d.ts`)

// Always augment `DefaultTheme` so `theme` is fully typed in every styled component

```typescript
import 'styled-components';
import { IThemeProps } from 'src/styles/theme';

declare module 'styled-components' {
  export interface DefaultTheme extends IThemeProps {}
}
```

## AppProviders (`lib/components/AppProviders/index.tsx`)

// Always wrap with `StyleSheetManager disableCSSOMInjection shouldForwardProp={isValidProp}`
// `isValidProp` comes from `@emotion/is-prop-valid` — prevents custom props leaking to DOM
// `GlobalCSS` must be rendered inside `ThemeProvider` so it can access theme tokens

```typescript
import isValidProp from '@emotion/is-prop-valid';
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
```

## useTheme Hook (`hooks/useTheme/index.tsx`)

// `ThemeProvider` persists mode and color to localStorage on every change
// Export the hook as a default anonymous arrow: `export default () => useContext(ThemeCtx)`
// Wrap context value in `useMemo` keyed on `[isDark, color]` to avoid unnecessary re-renders

```typescript
export const ThemeProvider = (props: PropsWithChildren) => {
  const [isDark, setDark] = useState(localTheme.mode === 'dark');
  const [color, setColor] = useState<IThemeColors>(localTheme.color);

  useEffect(() => { saveLocalTheme(isDark ? 'dark' : 'light', color); }, [isDark, color]);

  const value = useMemo(() => ({ isDark, color, setDark, setColor }), [isDark, color]);

  return <ThemeCtx.Provider value={value}>{props.children}</ThemeCtx.Provider>;
};

export default () => useContext(ThemeCtx);
```

## Styled Components

// Use `css\`\`` helper inside every `styled.*()` call so `theme` is typed
// Use `IStyledProps` for all generic layout/utility props — never widen with `any`
// Use `cssSize(n)` for all spacing — 1 unit = 0.2rem (e.g. `cssSize(4)` = 0.8rem)
// Use `screenSize.*` constants for all media query breakpoints — never hardcode px values
// Define shared layout primitives (Row, Column, Section, Card) in `lib/common/StyledComponents/`
// Never hardcode color hex values in components — always reference `theme.colors.*`

```typescript
import styled, { css } from 'styled-components';
import { IStyledProps, cssSize, screenSize } from 'src/styles/utils';

export const Row = styled.div<IStyledProps>(
  ({ gap, left, responsive }) => css`
    display: flex;
    gap: ${cssSize(gap ?? 2)};
    ${left && 'justify-content: flex-start;'}

    @media (max-width: ${screenSize.laptopS}px) {
      ${responsive && 'flex-direction: column;'}
    }
  `,
);

export const Card = styled.div(
  ({ theme }) => css`
    border-radius: ${theme.radius.md};
    background-color: ${theme.colors.bg2};
    padding: ${cssSize(4)};
  `,
);
```

## Pages (`pages/<page>/index.tsx`)

// Each page is a single default-exported functional component
// Pages compose section components from `src/lib/sections/<page>/`
// Pages must not contain direct styled-component definitions — delegate to sections or common

```typescript
import PageSection from 'src/lib/sections/home/PageSection';

const HomePage = () => (
  <>
    <PageSection />
  </>
);

export default HomePage;
```

## Routes (`pages/routes.tsx`)

// Export a single `RouteObject[]` array — never call `createBrowserRouter` here
// `createBrowserRouter` is called once in `app.tsx`

```typescript
import { RouteObject } from 'react-router-dom';
import HomePage from 'src/pages/home';

const Router: RouteObject[] = [
  { path: '/', element: <HomePage /> },
];

export default Router;
```

## App Entry (`app.tsx`)

// Provider nesting order is fixed: `ThemeProvider` → `AppProviders` → `RouterProvider`
// `ThemeProvider` must be outermost so `useTheme` is available inside `AppProviders`
// Always guard `getElementById('root')` with a null check before calling `createRoot`

```typescript
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

## Global Styles (`styles/global.ts`)

// Use `createGlobalStyle` with the `css` helper for theme access
// Includes CSS reset, font-family, font-size, background and text color from theme
// Import font files directly for Vite to bundle them

```typescript
export default createGlobalStyle(
  ({ theme }) => css`
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: 'Roboto', sans-serif;
      background-color: ${theme.colors.bg4};
      color: ${theme.colors.text4};
    }
  `,
);
```

## Sizing & Utilities (`styles/utils.ts`)

// `cssSize(n)` converts a unitless number to rem: `n * 0.2rem` (e.g. 5 → 1.0rem)
// `screenSize` is the single source for breakpoints — never duplicate these values
// `IStyledProps` is the shared prop interface for all layout styled components
// `animations` exports named `keyframes` objects — reference them with `animation: 0.5s ${animations.fadeIn}`

## Imports

// Use `src/` path alias for all cross-directory imports
// Import order enforced by prettier-plugin-sort-imports: external packages → `src/` → relative
// Never use deep relative paths (`../../..`) — always use `src/` alias

```typescript
// Correct
import useTheme from 'src/hooks/useTheme';
import { Section } from 'src/lib/common/StyledComponents/Containers';

// Wrong
import useTheme from '../../../hooks/useTheme';
```
