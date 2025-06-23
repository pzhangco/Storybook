## 1 Create a workspace & initialise pnpm
```
pnpm init          # writes an package.json
```
## 2 Add runtime dependencies
```
pnpm add react react-dom
```
Why both packages?
react supplies the component API; react-dom mounts those components into the browser’s DOM tree.

## 3 Add the development toolchain
``` 
pnpm add -D typescript         # compiler
pnpm add -D vite               # lightning-fast dev server & bundler
pnpm add -D @vitejs/plugin-react   # React + Fast Refresh for Vite
pnpm add -D @types/react @types/react-dom  # TS type stubs
```

## 4 Install Storybook for React + Vite
```
pnpm add -D storybook @storybook/react-vite @storybook/addon-docs
```
`@storybook/react-vite` pulls in React renderer, and the Vite builder automatically.

## 5 Create your TypeScript config
We’ll hand-write the minimal config instead of running tsc --init.

`tsconfig.json`
```
{
    "compilerOptions": {
      "target": "ES2020",
      "module": "NodeNext",
      "moduleResolution": "NodeNext",
      "jsx": "react-jsx",
      "strict": true,
      "esModuleInterop": true,
      "forceConsistentCasingInFileNames": true,
      "skipLibCheck": true,
      "outDir": "dist",
      "rootDir": "src"
    },
    "include": ["src", ".storybook"]
  }
```
`"jsx": "react-jsx"` enables the automatic JSX runtime introduced in React 17—no more import React from 'react' at the top of every file.

## 6 Add a Vite config
`vite.config.ts`

```
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()]
});
```

## 7 Wire up npm scripts
Edit package.json and add:
```
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview",
  "storybook": "storybook dev -p 6006",
  "build-storybook": "storybook build"
}
```
## 8 Configure Storybook
Create a .storybook folder.

.storybook/main.ts
```
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.mdx', '../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: ['@storybook/addon-docs'], 
  framework: { name: '@storybook/react-vite', options: {} }
};

export default config;
```
.storybook/preview.ts
```import type { Preview } from '@storybook/react-vite';

export const parameters: Preview['parameters'] = {
  actions: { argTypesRegex: '^on[A-Z].*' },
  controls: {
    matchers: { color: /(background|color)$/i, date: /Date$/ }
  },
};

export const tags = ['autodocs'];

```
## 9 Scaffold your first component & story
```
mkdir -p src/components
```

src/components/Button.tsx

```
import React from 'react';

export interface ButtonProps {
  label: string;
  color?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  onClick?: () => void;
}

const colorMap = {
  primary: '#1ea7fd',
  secondary: '#e2e2e2',
  danger: '#d32f2f',
};

const sizeMap = {
  small: '0.5rem 1rem',
  medium: '0.75rem 1.5rem',
  large: '1rem 2rem',
};

export const Button: React.FC<ButtonProps> = ({
  label,
  color = 'primary',
  size = 'medium',
  disabled = false,
  onClick,
}) => {
  return (
    <button
      style={{
        backgroundColor: colorMap[color],
        color: color === 'secondary' ? '#333' : '#fff',
        padding: sizeMap[size],
        border: 'none',
        borderRadius: '4px',
        fontSize: '1rem',
        opacity: disabled ? 0.5 : 1,
        cursor: disabled ? 'not-allowed' : 'pointer',
        transition: 'background 0.2s',
      }}
      disabled={disabled}
      onClick={onClick}
    >
      {label}
    </button>
  );
};

export default Button;


```

src/components/Button.stories.tsx
```
// src/components/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react-vite';
import { Button, ButtonProps } from './Button.js';
import { userEvent, within, expect } from 'storybook/test';

export default  { 
    title: 'Components/Button',
    component: Button,
    tags: ['autodocs'],
    argTypes: {
    onClick: { action: 'clicked' },
    },
};

type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    label: 'Primary Button',
    color: 'primary',
    size: "medium",
    disabled: false,
  },
};

export const Secondary: Story = {
  args: {
    label: 'Secondary Button',
    color: 'secondary',
    size: 'medium',
    disabled: false,
  },
};

export const Danger: Story = {
  args: {
    label: 'Danger Button',
    color: 'danger',
    size: 'medium',
    disabled: false,
  },
};

export const Disabled: Story = {
  args: {
    label: 'Disabled Button',
    color: 'primary',
    size: 'medium',
    disabled: true,
  },
};

export const SuccessfulClickTest: Story = {
  args: {
    label: 'Click Me',
    color: 'primary',
    size: 'medium',
    disabled: false,
  },
  play: async ({ canvasElement, args }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button', { name: /click me/i });
    await userEvent.click(button);
    // The onClick action will be logged in the actions panel
    expect(button).not.toBeDisabled();
  },
};
```
## 10 Run it!
```
pnpm dev         # Vite app → http://localhost:5173
pnpm storybook   # Storybook → http://localhost:6006
```