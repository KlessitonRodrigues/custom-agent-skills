# Bootstrap: Next.js + DaisyUI + Storybook Component Library

Use this guide to create a React component library documented with Storybook, using the
Next.js/Vite Storybook framework and Tailwind CSS 4 with DaisyUI.

This setup uses Next.js and React as dependencies and Storybook as the development and build
interface. It does not define a Next.js application `dev` or `start` command.

## Prerequisites

- Node.js 22.x
- Yarn 1.22.x

## 1. Project Initialization

Create the package in the appropriate workspace, then install its dependencies from the monorepo
root:

```bash
mkdir -p apps/component-library
cd apps/component-library
yarn init -y
```

## 2. `package.json`

Use a generic workspace package name and preserve the versions and public entry points:

```json
{
  "name": "@apps/component-library",
  "version": "1.0.0",
  "main": "./src/index.tsx",
  "types": "./src/index.tsx",
  "license": "MIT",
  "private": true,
  "scripts": {
    "dev": "storybook dev -p 6006",
    "build": "storybook build"
  },
  "dependencies": {
    "next": "^16.2.10",
    "react": "^19.2.7",
    "react-dom": "^19.2.8"
  },
  "devDependencies": {
    "@chromatic-com/storybook": "latest",
    "@storybook/addon-a11y": "^10.5.10",
    "@storybook/addon-docs": "^10.5.10",
    "@storybook/addon-vitest": "^10.5.10",
    "@storybook/nextjs-vite": "^10.5.10",
    "@tailwindcss/postcss": "^4.3.3",
    "@trivago/prettier-plugin-sort-imports": "^6.0.2",
    "@types/node": "26.2.0",
    "@types/react": "19.2.18",
    "@vitest/browser-playwright": "latest",
    "@vitest/coverage-v8": "latest",
    "daisyui": "^5.6.16",
    "playwright": "latest",
    "prettier": "^3.9.6",
    "react-icons": "^5.7.0",
    "react-is": "^19.2.8",
    "react-toastify": "^11.0.5",
    "recharts": "^3.9.2",
    "storybook": "^10.5.10",
    "tailwind-merge": "^3.4.1",
    "tailwindcss": "^4.3.2",
    "typescript": "7.0.2",
    "vite": "latest",
    "vitest": "latest"
  },
  "exports": {
    ".": "./src/index.tsx",
    "./*": "./src/*"
  }
}
```

Install the workspace dependencies from the repository root:

```bash
yarn install
```

Keep internal workspace package references on the monorepo's established version convention.
Do not add Next.js application scripts unless the package is also intended to be a standalone
Next.js app.

## 3. Directory Structure

```text
apps/component-library/
├── .storybook/
│   ├── main.ts
│   └── preview.tsx
├── src/
│   ├── components/
│   │   ├── authentication/
│   │   ├── cards/
│   │   ├── charts/
│   │   ├── common/
│   │   ├── navbars/
│   │   ├── settings/
│   │   ├── showcase/
│   │   └── users/
│   ├── stories/
│   │   └── <component>.stories.tsx
│   ├── utils/
│   ├── global.css
│   └── index.tsx
├── .gitignore
├── .prettierrc
├── package.json
├── postcss.config.mjs
└── tsconfig.json
```

## 4. TypeScript Configuration

Configure strict, no-emit TypeScript with the React JSX transform and Next.js plugin:

```json
{
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "react-jsx",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "target": "ES2017"
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    ".next/dev/types/**/*.ts"
  ],
  "exclude": ["node_modules"]
}
```

## 5. Storybook Configuration

Use the Next.js Vite framework, discover TypeScript stories, and enable accessibility, docs, and
Vitest addons.

**`.storybook/main.ts`**

```typescript
import type { StorybookConfig } from '@storybook/nextjs-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(js|jsx|mjs|ts|tsx)'],
  addons: ['@storybook/addon-a11y', '@storybook/addon-docs', '@storybook/addon-vitest'],
  framework: '@storybook/nextjs-vite',
};

export default config;
```

**`.storybook/preview.tsx`**

```typescript
import type { Preview } from '@storybook/nextjs-vite';

import '../src/global.css';

const preview: Preview = {
  tags: ['autodocs'],
  parameters: {
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/i,
      },
    },
  },
};

export default preview;
```

## 6. Tailwind CSS and DaisyUI

Tailwind CSS 4 is processed through PostCSS. DaisyUI is loaded from the global stylesheet and its
design tokens can be customized with CSS variables.

**`postcss.config.mjs`**

```javascript
const config = {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};

export default config;
```

**`src/global.css`**

```css
@import 'tailwindcss';

@plugin "daisyui" {
  themes: light --default;
}

@source "../";

:root {
  --color-primary: #3b82f6;
  --color-primary-content: #ffffff;
  --color-base-100: #ffffff;
  --color-base-content: #1f2937;
}
```

Import the global stylesheet once in the Storybook preview so every story uses the same theme.
Extend the root CSS variables for the component library's semantic colors, typography, radii, and
shadows; use those tokens instead of hard-coded values in components.

## 7. Component and Story Examples

Export the public component API from `src/index.tsx`, grouping exports by component area and
utility. Keep implementation modules under `src/components/` and `src/utils/`.

**`src/components/common/common.button.tsx`**

```tsx
import type { ButtonHTMLAttributes } from 'react';
import { twMerge } from 'tailwind-merge';

export interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  color?: 'primary' | 'secondary' | 'accent' | 'neutral' | 'info' | 'success' | 'warning' | 'error';
  loading?: boolean;
}

export const Button = (props: ButtonProps) => {
  const { children, className, color = 'primary', loading = false, disabled, ...buttonProps } = props;

  return (
    <button
      className={twMerge('btn', `btn-${color}`, className)}
      disabled={disabled || loading}
      aria-busy={loading || undefined}
      {...buttonProps}
    >
      {loading && <span aria-hidden="true" className="loading loading-spinner loading-xs" />}
      {children}
    </button>
  );
};
```

Extend the native button attributes so consumers can use standard button props. Use DaisyUI classes
for styling and `tailwind-merge` to let callers safely override class names.

**`src/stories/button.stories.tsx`**

```tsx
import type { Meta, StoryObj } from '@storybook/nextjs-vite';

import { Button } from '../components/common/common.button';

const meta = {
  component: Button,
  title: 'Common/Button',
  args: {
    children: 'Continue',
  },
} satisfies Meta<typeof Button>;

export default meta;

type Story = StoryObj<typeof meta>;

export const Default = {
  args: {
    color: 'primary',
  },
} satisfies Story;

export const Loading = {
  args: {
    loading: true,
    children: 'Saving',
  },
} satisfies Story;
```

Add a story for each component with representative default, variant, and state examples. Export
public components from `src/index.tsx`:

```typescript
export { Button } from './components/common/common.button';
export type { ButtonProps } from './components/common/common.button';
```

Use Storybook stories as the component examples and documentation. The Vitest addon and browser
dependencies are included, but add an explicit test script and Vitest configuration only when
tests are configured for the package.

## 8. Formatting and Ignore Rules

**`.prettierrc`**

```json
{
  "tabWidth": 2,
  "useTabs": false,
  "printWidth": 100,
  "singleQuote": true,
  "arrowParens": "avoid",
  "semi": true,
  "importOrder": ["^src/(.*)$", "^server/(.*)$", "^[./]"],
  "importOrderSeparation": true,
  "importOrderSortSpecifiers": true,
  "plugins": ["@trivago/prettier-plugin-sort-imports"]
}
```

**`.gitignore`**

```gitignore
node_modules/
.next/
dist/
cdk.out/
next-env.d.ts
*storybook.log
storybook-static
```

## 9. Development Workflow

Run workspace scripts from the monorepo root:

```bash
yarn workspace @apps/component-library dev
yarn workspace @apps/component-library build
```

`dev` starts Storybook on port 6006. `build` creates the static Storybook site; deploy it only
through the hosting setup required by the project. Verify the package build after adding or
changing components and stories.