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
```
import type { Preview } from '@storybook/react-vite';

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


## Login Box Story And Component

```
import type { Meta, StoryObj } from '@storybook/react-vite';
import { within, userEvent, waitFor, expect } from 'storybook/test';
import { LoginBox } from './LoginBox.js';

const meta: Meta<typeof LoginBox> = {
  title: 'Components/LoginBox',
  component: LoginBox,
  tags: ['autodocs'],
  parameters: {
    a11y: { disable: false },
    chromatic: { disableSnapshot: false },
  },
};
export default meta;

type Story = StoryObj<typeof LoginBox>;

export const Default: Story = {
  args: {
    onLogin: () => {},
    loading: false,
    error: '',
  },
};

export const WithError: Story = {
  args: {
    onLogin: () => {},
    loading: false,
    error: 'Invalid username or password',
  },
};

export const Loading: Story = {
  args: {
    onLogin: () => {},
    loading: true,
    error: '',
  },
};

export const DisabledButton: Story = {
  args: {
    onLogin: () => {},
    loading: true,
    error: '',
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button', { name: /logging in/i });
    expect(button).toBeDisabled();
  },
};

export const ValidationTest: Story = {
  args: {
    onLogin: () => {},
    loading: false,
    error: '',
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    // Try submitting empty form
    await userEvent.click(canvas.getByRole('button', { name: /login/i }));
    await expect(canvas.getByText('Username is required')).toBeInTheDocument();
    await expect(canvas.getByText('Password is required')).toBeInTheDocument();
  },
};

export const SuccessfulLoginTest: Story = {
  args: {
    onLogin: () => {},
    loading: false,
    error: '',
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    await userEvent.type(canvas.getByLabelText('Username'), 'storybook');
    await userEvent.type(canvas.getByLabelText('Password'), 'testpass');
    await userEvent.click(canvas.getByRole('button', { name: /login/i }));
    // No error messages should be present
    await waitFor(() => {
      expect(canvas.queryByText('Username is required')).toBeNull();
      expect(canvas.queryByText('Password is required')).toBeNull();
    });
  },
};

export const AccessibilityTest: Story = {
  args: {
    onLogin: () => {},
    loading: false,
    error: '',
  },
  parameters: {
    a11y: {
      // Custom accessibility rules can be added here
    },
  },
};

export const SuccessfulClickTest: Story = {
  args: {
    onLogin: () => {},
    loading: false,
    error: '',
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button', { name: /login/i });
    await userEvent.click(button);
    expect(button).not.toBeDisabled();
  },
};
```

```
import React, { useState } from 'react';

export interface LoginBoxProps {
  onLogin: (username: string, password: string) => void;
  error?: string;
  loading?: boolean;
}

export const LoginBox: React.FC<LoginBoxProps> = ({ onLogin, error, loading }) => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [touched, setTouched] = useState({ username: false, password: false });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    setTouched({ username: true, password: true });
    if (username && password) {
      onLogin(username, password);
    }
  };

  const usernameError = touched.username && !username ? 'Username is required' : '';
  const passwordError = touched.password && !password ? 'Password is required' : '';

  return (
    <form
      onSubmit={handleSubmit}
      aria-label="login form"
      style={{
        maxWidth: 320,
        margin: '2rem auto',
        padding: '2rem',
        border: '1px solid #ccc',
        borderRadius: 8,
        background: '#fff',
        boxShadow: '0 2px 8px rgba(0,0,0,0.05)',
        fontFamily: 'inherit',
      }}
    >
      <h2 style={{ marginBottom: '1rem' }}>Login</h2>
      <div style={{ marginBottom: '1rem' }}>
        <label htmlFor="username" style={{ display: 'block', marginBottom: 4 }}>
          Username
        </label>
        <input
          id="username"
          type="text"
          value={username}
          autoComplete="username"
          onChange={e => setUsername(e.target.value)}
          onBlur={() => setTouched(t => ({ ...t, username: true }))}
          aria-invalid={!!usernameError}
          aria-describedby="username-error"
          style={{
            width: '100%',
            padding: 8,
            border: usernameError ? '1px solid #d32f2f' : '1px solid #ccc',
            borderRadius: 4,
          }}
        />
        {usernameError && (
          <div id="username-error" style={{ color: '#d32f2f', fontSize: 12 }}>
            {usernameError}
          </div>
        )}
      </div>
      <div style={{ marginBottom: '1rem' }}>
        <label htmlFor="password" style={{ display: 'block', marginBottom: 4 }}>
          Password
        </label>
        <input
          id="password"
          type="password"
          value={password}
          autoComplete="current-password"
          onChange={e => setPassword(e.target.value)}
          onBlur={() => setTouched(t => ({ ...t, password: true }))}
          aria-invalid={!!passwordError}
          aria-describedby="password-error"
          style={{
            width: '100%',
            padding: 8,
            border: passwordError ? '1px solid #d32f2f' : '1px solid #ccc',
            borderRadius: 4,
          }}
        />
        {passwordError && (
          <div id="password-error" style={{ color: '#d32f2f', fontSize: 12 }}>
            {passwordError}
          </div>
        )}
      </div>
      {error && (
        <div role="alert" style={{ color: '#d32f2f', marginBottom: '1rem' }}>
          {error}
        </div>
      )}
      <button
        type="submit"
        disabled={loading}
        style={{
          width: '100%',
          padding: 10,
          background: '#1ea7fd',
          color: '#fff',
          border: 'none',
          borderRadius: 4,
          fontWeight: 600,
          cursor: loading ? 'not-allowed' : 'pointer',
          opacity: loading ? 0.7 : 1,
        }}
      >
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};

export default LoginBox;
```