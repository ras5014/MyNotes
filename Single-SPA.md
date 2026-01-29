# Building a Single-spa Microfrontend Demo from Scratch

## Introduction

In this guide, we'll build a complete working demo with:
- **Root App** (the host/orchestrator)
- **Navbar MFE** (shared navigation that stays visible)
- **PageA MFE** (a page-level microfrontend)
- **PageB MFE** (another page-level microfrontend)

By the end, you'll have a fully functional microfrontend system using single-spa and Module Federation.

---

## Prerequisites

Before starting, make sure you have:
- **Node.js** v14+ and **npm** or **yarn**
- **TypeScript** knowledge (we'll use TS throughout)
- **React** familiarity
- **Webpack** basics
- A code editor (VS Code recommended)

---

# Phase 1: Project Structure Setup

## Step 1: Create the Monorepo Structure

Let's organize our demo as a monorepo with separate directories for each MFE.

```
demo-microfrontends/
├── packages/
│   ├── root-app/              # The orchestrator
│   ├── navbar-mfe/            # Navigation microfrontend
│   ├── page-a-mfe/            # Page A microfrontend
│   └── page-b-mfe/            # Page B microfrontend
└── package.json               # Root package.json (optional, for workspace)
```

### Create the root directory:

```bash
mkdir demo-microfrontends
cd demo-microfrontends
npm init -y  # Creates root package.json
```

### Create subdirectories:

```bash
mkdir -p packages/{root-app,navbar-mfe,page-a-mfe,page-b-mfe}
```

---

# Phase 2: Root App (The Orchestrator)

## Step 2: Initialize Root App Project

### Navigate to root-app:

```bash
cd packages/root-app
npm init -y
```

### Install dependencies:

```bash
npm install react react-dom react-router-dom
npm install --save-dev webpack webpack-cli webpack-dev-server \
  @webpack-cli/serve html-webpack-plugin ts-loader typescript \
  webpack-dev-middleware webpack-hot-middleware \
  @babel/core @babel/preset-react @babel/preset-typescript \
  babel-loader css-loader style-loader

# Single-spa and Module Federation
npm install single-spa
```

### Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "baseUrl": ".",
    "paths": {
      "@root/*": ["src/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

### Create directory structure:

```bash
mkdir -p src/components
mkdir -p public
```

---

## Step 3: Create Root App Source Files

### `src/main.tsx` (Entry point)

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { registerApplication, start } from 'single-spa';
import { App } from './components/App';

// Import this file to render the root app
const container = document.getElementById('root');
const root = ReactDOM.createRoot(container!);

// The root app itself is registered as an MFE
export const bootstrap = async () => {
  console.log('[Root] Bootstrapping...');
  // One-time setup
};

export const mount = async () => {
  console.log('[Root] Mounting...');
  root.render(<App />);
};

export const unmount = async () => {
  console.log('[Root] Unmounting...');
  root.unmount();
};

// Register all MFEs
const registerMFEs = () => {
  // Navbar MFE - always visible (activeWhen: '/')
  registerApplication({
    name: '@mfe/navbar',
    app: () => System.import('@mfe/navbar'),
    activeWhen: '/',
    customProps: {
      sharedData: {
        basePath: '/',
      },
    },
  });

  // Page A MFE - active when URL contains /page-a
  registerApplication({
    name: '@mfe/page-a',
    app: () => System.import('@mfe/page-a'),
    activeWhen: '/page-a',
    customProps: {
      basePath: '/page-a',
    },
  });

  // Page B MFE - active when URL contains /page-b
  registerApplication({
    name: '@mfe/page-b',
    app: () => System.import('@mfe/page-b'),
    activeWhen: '/page-b',
    customProps: {
      basePath: '/page-b',
    },
  });
};

// Start single-spa
registerMFEs();
start();
```

### `src/components/App.tsx` (Root layout)

```typescript
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import './App.css';

export function App() {
  const navigate = useNavigate();

  return (
    <div className="app-container">
      <header className="app-header">
        <h1>Microfrontend Demo</h1>
        <p>Using single-spa + Module Federation</p>
      </header>

      <div className="app-content">
        {/* Navbar MFE will be mounted here */}
        <div id="navbar-container" className="navbar-container">
          <nav className="loading">Loading navbar...</nav>
        </div>

        {/* Page MFEs will be mounted here */}
        <main className="page-container">
          <div id="page-container">
            <p>Select a page from the navbar to get started!</p>
          </div>
        </main>
      </div>

      <footer className="app-footer">
        <p>&copy; 2024 Microfrontend Demo</p>
      </footer>
    </div>
  );
}
```

### `src/components/App.css`:

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #f5f5f5;
}

.app-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 2rem;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.app-header h1 {
  font-size: 2.5rem;
  margin-bottom: 0.5rem;
}

.app-header p {
  font-size: 1rem;
  opacity: 0.9;
}

.app-content {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.navbar-container {
  background: white;
  border-bottom: 1px solid #e0e0e0;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
}

.navbar-container nav {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  gap: 2rem;
}

.navbar-container nav.loading {
  color: #999;
  font-style: italic;
}

.page-container {
  flex: 1;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

.app-footer {
  background: #333;
  color: white;
  padding: 1.5rem;
  text-align: center;
  border-top: 1px solid #ddd;
}

.app-footer p {
  margin: 0;
}
```

### `public/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Microfrontend Demo - single-spa</title>
  </head>
  <body>
    <div id="root"></div>
    <!-- Webpack Module Federation will handle all script loading -->
  </body>
</html>
```

---

## Step 4: Create Webpack Configuration for Root App

### `webpack.config.js` (in root-app directory):

```javascript
const path = require('path');
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { ModuleFederationPlugin } = require('webpack').container;

const deps = require('./package.json').dependencies;

module.exports = {
  mode: 'development',
  entry: './src/main.tsx',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js',
    publicPath: 'http://localhost:3000/',
    clean: true,
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
  },
  module: {
    rules: [
      {
        test: /\.(ts|tsx|js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              '@babel/preset-react',
              '@babel/preset-typescript',
            ],
          },
        },
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'root_app',
      filename: 'remoteEntry.js',
      remotes: {
        '@mfe/navbar': 'navbar_mfe@http://localhost:3001/remoteEntry.js',
        '@mfe/page-a': 'page_a_mfe@http://localhost:3002/remoteEntry.js',
        '@mfe/page-b': 'page_b_mfe@http://localhost:3003/remoteEntry.js',
      },
      shared: {
        react: {
          singleton: true,
          requiredVersion: deps.react,
          eager: true,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: deps['react-dom'],
          eager: true,
        },
        'single-spa': {
          singleton: true,
          requiredVersion: deps['single-spa'],
          eager: true,
        },
      },
    }),
    new HtmlWebpackPlugin({
      template: './public/index.html',
    }),
    new webpack.HotModuleReplacementPlugin(),
  ],
  devServer: {
    port: 3000,
    historyApiFallback: true,
    hot: true,
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  },
};
```

### Update `package.json` scripts:

```json
{
  "scripts": {
    "start": "webpack serve --mode development",
    "build": "webpack --mode production",
    "dev": "webpack serve --mode development --watch"
  }
}
```

---

# Phase 3: Navbar MFE

## Step 5: Create Navbar Microfrontend

### Setup:

```bash
cd packages/navbar-mfe
npm init -y
npm install react react-dom react-router-dom single-spa
npm install --save-dev webpack webpack-cli webpack-dev-server \
  @webpack-cli/serve html-webpack-plugin ts-loader typescript \
  @babel/core @babel/preset-react @babel/preset-typescript babel-loader \
  css-loader style-loader
```

### Create directory structure:

```bash
mkdir -p src
mkdir -p public
```

### `src/Navbar.tsx`:

```typescript
import React from 'react';
import { useNavigate, useLocation } from 'react-router-dom';
import './Navbar.css';

export function Navbar() {
  const navigate = useNavigate();
  const location = useLocation();

  return (
    <nav className="navbar">
      <div className="navbar-brand">
        <span>MyApp</span>
      </div>
      
      <ul className="navbar-menu">
        <li>
          <a
            href="/page-a"
            className={location.pathname === '/page-a' ? 'active' : ''}
            onClick={(e) => {
              e.preventDefault();
              navigate('/page-a');
            }}
          >
            Page A
          </a>
        </li>
        <li>
          <a
            href="/page-b"
            className={location.pathname === '/page-b' ? 'active' : ''}
            onClick={(e) => {
              e.preventDefault();
              navigate('/page-b');
            }}
          >
            Page B
          </a>
        </li>
      </ul>

      <div className="navbar-info">
        <small>Navbar MFE (Persistent)</small>
      </div>
    </nav>
  );
}
```

### `src/Navbar.css`:

```css
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background: white;
  border-bottom: 2px solid #667eea;
}

.navbar-brand {
  font-size: 1.5rem;
  font-weight: bold;
  color: #667eea;
}

.navbar-menu {
  display: flex;
  list-style: none;
  gap: 2rem;
  margin: 0;
  padding: 0;
}

.navbar-menu a {
  text-decoration: none;
  color: #333;
  font-weight: 500;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  transition: all 0.3s ease;
}

.navbar-menu a:hover {
  background: #f0f0f0;
  color: #667eea;
}

.navbar-menu a.active {
  background: #667eea;
  color: white;
}

.navbar-info {
  color: #999;
  font-size: 0.8rem;
}
```

### `src/index.tsx` (Entry point for Navbar MFE):

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import { Navbar } from './Navbar';

let root: any = null;

export const bootstrap = async () => {
  console.log('[Navbar MFE] Bootstrapping...');
  // One-time initialization
};

export const mount = async (props: any) => {
  console.log('[Navbar MFE] Mounting with props:', props);
  const container = document.getElementById('navbar-container');
  
  if (!container) {
    console.error('Navbar container not found!');
    return;
  }

  root = ReactDOM.createRoot(container);
  root.render(
    <BrowserRouter>
      <Navbar />
    </BrowserRouter>
  );
};

export const unmount = async () => {
  console.log('[Navbar MFE] Unmounting...');
  if (root) {
    root.unmount();
  }
};
```

### `webpack.config.js`:

```javascript
const path = require('path');
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { ModuleFederationPlugin } = require('webpack').container;

const deps = require('./package.json').dependencies;

module.exports = {
  mode: 'development',
  entry: './src/index.tsx',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js',
    publicPath: 'http://localhost:3001/',
    clean: true,
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
  },
  module: {
    rules: [
      {
        test: /\.(ts|tsx|js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              '@babel/preset-react',
              '@babel/preset-typescript',
            ],
          },
        },
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'navbar_mfe',
      filename: 'remoteEntry.js',
      exposes: {
        './sspa': './src/index.tsx', // Export the bootstrap/mount/unmount functions
      },
      shared: {
        react: {
          singleton: true,
          requiredVersion: deps.react,
          eager: true,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: deps['react-dom'],
          eager: true,
        },
        'react-router-dom': {
          singleton: true,
          requiredVersion: deps['react-router-dom'],
        },
        'single-spa': {
          singleton: true,
          requiredVersion: deps['single-spa'],
          eager: true,
        },
      },
    }),
    new webpack.HotModuleReplacementPlugin(),
  ],
  devServer: {
    port: 3001,
    historyApiFallback: true,
    hot: true,
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  },
};
```

### Update `package.json` scripts:

```json
{
  "scripts": {
    "start": "webpack serve --mode development",
    "build": "webpack --mode production",
    "dev": "webpack serve --mode development --watch"
  }
}
```

---

# Phase 4: Page A MFE

## Step 6: Create Page A Microfrontend

### Setup:

```bash
cd packages/page-a-mfe
npm init -y
npm install react react-dom react-router-dom single-spa
npm install --save-dev webpack webpack-cli webpack-dev-server \
  html-webpack-plugin ts-loader typescript \
  @babel/core @babel/preset-react @babel/preset-typescript babel-loader \
  css-loader style-loader
```

### Create directory structure:

```bash
mkdir -p src
```

### `src/PageA.tsx`:

```typescript
import React, { useState } from 'react';
import './PageA.css';

export function PageA() {
  const [count, setCount] = useState(0);

  return (
    <div className="page-a">
      <h2>Page A Microfrontend</h2>
      
      <div className="page-content">
        <p>
          This page is rendered by the Page A MFE. It's mounted/unmounted based on
          the URL. Try navigating to Page B and back to see the remounting behavior.
        </p>

        <div className="counter-section">
          <h3>Counter (local state)</h3>
          <p>Current count: <strong>{count}</strong></p>
          <button onClick={() => setCount(count + 1)}>Increment</button>
          <button onClick={() => setCount(0)}>Reset</button>
          <p className="info">
            ⓘ Try navigating away and back. The counter resets because the MFE
            unmounts and remounts.
          </p>
        </div>

        <div className="info-box">
          <h3>What's happening:</h3>
          <ul>
            <li>You're viewing a React component from a separate bundle</li>
            <li>This bundle is loaded dynamically via Module Federation</li>
            <li>The Navbar above never unmounted</li>
            <li>When you navigate to Page B, this component will unmount</li>
          </ul>
        </div>
      </div>
    </div>
  );
}
```

### `src/PageA.css`:

```css
.page-a {
  max-width: 800px;
  margin: 0 auto;
}

.page-a h2 {
  color: #667eea;
  margin-bottom: 1.5rem;
  font-size: 2rem;
}

.page-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.page-content p {
  line-height: 1.6;
  color: #333;
  margin-bottom: 1rem;
}

.counter-section {
  background: #f9f9f9;
  padding: 1.5rem;
  border-radius: 8px;
  margin: 1.5rem 0;
  border-left: 4px solid #667eea;
}

.counter-section h3 {
  color: #667eea;
  margin-top: 0;
}

.counter-section p {
  margin: 0.5rem 0;
}

.counter-section button {
  background: #667eea;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  margin-right: 0.5rem;
  transition: background 0.3s ease;
}

.counter-section button:hover {
  background: #764ba2;
}

.info {
  font-size: 0.9rem;
  color: #666;
  font-style: italic;
}

.info-box {
  background: #e8f4f8;
  padding: 1.5rem;
  border-radius: 8px;
  margin-top: 1.5rem;
  border: 1px solid #b3d9e8;
}

.info-box h3 {
  color: #0277bd;
  margin-top: 0;
}

.info-box ul {
  list-style: disc;
  margin-left: 1.5rem;
}

.info-box li {
  margin-bottom: 0.5rem;
  color: #333;
}
```

### `src/index.tsx`:

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { PageA } from './PageA';

let root: any = null;

export const bootstrap = async () => {
  console.log('[Page A MFE] Bootstrapping...');
};

export const mount = async (props: any) => {
  console.log('[Page A MFE] Mounting with props:', props);
  const container = document.getElementById('page-container');
  
  if (!container) {
    console.error('Page container not found!');
    return;
  }

  root = ReactDOM.createRoot(container);
  root.render(<PageA />);
};

export const unmount = async () => {
  console.log('[Page A MFE] Unmounting...');
  if (root) {
    root.unmount();
  }
};
```

### `webpack.config.js`:

```javascript
const path = require('path');
const webpack = require('webpack');
const { ModuleFederationPlugin } = require('webpack').container;

const deps = require('./package.json').dependencies;

module.exports = {
  mode: 'development',
  entry: './src/index.tsx',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js',
    publicPath: 'http://localhost:3002/',
    clean: true,
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
  },
  module: {
    rules: [
      {
        test: /\.(ts|tsx|js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              '@babel/preset-react',
              '@babel/preset-typescript',
            ],
          },
        },
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'page_a_mfe',
      filename: 'remoteEntry.js',
      exposes: {
        './sspa': './src/index.tsx',
      },
      shared: {
        react: {
          singleton: true,
          requiredVersion: deps.react,
          eager: true,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: deps['react-dom'],
          eager: true,
        },
        'single-spa': {
          singleton: true,
          requiredVersion: deps['single-spa'],
          eager: true,
        },
      },
    }),
    new webpack.HotModuleReplacementPlugin(),
  ],
  devServer: {
    port: 3002,
    hot: true,
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  },
};
```

### Update `package.json` scripts:

```json
{
  "scripts": {
    "start": "webpack serve --mode development",
    "build": "webpack --mode production",
    "dev": "webpack serve --mode development --watch"
  }
}
```

---

# Phase 5: Page B MFE

## Step 7: Create Page B Microfrontend

Follow the same pattern as Page A, but with different content:

### `src/PageB.tsx`:

```typescript
import React, { useEffect, useState } from 'react';
import './PageB.css';

export function PageB() {
  const [items, setItems] = useState<string[]>([]);
  const [input, setInput] = useState('');

  useEffect(() => {
    console.log('[PageB] Component mounted');
    return () => console.log('[PageB] Component unmounting');
  }, []);

  const addItem = () => {
    if (input.trim()) {
      setItems([...items, input]);
      setInput('');
    }
  };

  return (
    <div className="page-b">
      <h2>Page B Microfrontend</h2>
      
      <div className="page-content">
        <p>
          This is a different MFE with its own state and lifecycle. Notice how
          the Navbar remained visible during navigation.
        </p>

        <div className="todo-section">
          <h3>Simple Todo List</h3>
          
          <div className="input-group">
            <input
              type="text"
              value={input}
              onChange={(e) => setInput(e.target.value)}
              placeholder="Enter a task..."
              onKeyPress={(e) => e.key === 'Enter' && addItem()}
            />
            <button onClick={addItem}>Add</button>
          </div>

          <ul className="todo-list">
            {items.map((item, idx) => (
              <li key={idx}>{item}</li>
            ))}
          </ul>

          {items.length === 0 && <p className="empty">No items yet</p>}
        </div>

        <div className="info-box">
          <h3>MFE Independence:</h3>
          <ul>
            <li>Page B has its own React state and lifecycle</li>
            <li>It's loaded from a separate bundle (port 3003)</li>
            <li>Navigation between pages unmounts/remounts the current MFE</li>
            <li>Module Federation shares React to avoid duplication</li>
            <li>Check console logs to see lifecycle events</li>
          </ul>
        </div>
      </div>
    </div>
  );
}
```

### `src/PageB.css`:

```css
.page-b {
  max-width: 800px;
  margin: 0 auto;
}

.page-b h2 {
  color: #764ba2;
  margin-bottom: 1.5rem;
  font-size: 2rem;
}

.page-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.page-content p {
  line-height: 1.6;
  color: #333;
  margin-bottom: 1rem;
}

.todo-section {
  background: #fafafa;
  padding: 1.5rem;
  border-radius: 8px;
  margin: 1.5rem 0;
  border-left: 4px solid #764ba2;
}

.todo-section h3 {
  color: #764ba2;
  margin-top: 0;
}

.input-group {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.input-group input {
  flex: 1;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

.input-group button {
  background: #764ba2;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background 0.3s ease;
}

.input-group button:hover {
  background: #667eea;
}

.todo-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.todo-list li {
  background: white;
  padding: 0.75rem 1rem;
  margin: 0.5rem 0;
  border-radius: 4px;
  border-left: 3px solid #764ba2;
}

.empty {
  color: #999;
  font-style: italic;
  text-align: center;
  padding: 1rem;
}

.info-box {
  background: #f3e5f5;
  padding: 1.5rem;
  border-radius: 8px;
  margin-top: 1.5rem;
  border: 1px solid #e1bee7;
}

.info-box h3 {
  color: #6a1b9a;
  margin-top: 0;
}

.info-box ul {
  list-style: disc;
  margin-left: 1.5rem;
}

.info-box li {
  margin-bottom: 0.5rem;
  color: #333;
}
```

### `src/index.tsx`:

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { PageB } from './PageB';

let root: any = null;

export const bootstrap = async () => {
  console.log('[Page B MFE] Bootstrapping...');
};

export const mount = async (props: any) => {
  console.log('[Page B MFE] Mounting with props:', props);
  const container = document.getElementById('page-container');
  
  if (!container) {
    console.error('Page container not found!');
    return;
  }

  root = ReactDOM.createRoot(container);
  root.render(<PageB />);
};

export const unmount = async () => {
  console.log('[Page B MFE] Unmounting...');
  if (root) {
    root.unmount();
  }
};
```

### `webpack.config.js`:

```javascript
const path = require('path');
const webpack = require('webpack');
const { ModuleFederationPlugin } = require('webpack').container;

const deps = require('./package.json').dependencies;

module.exports = {
  mode: 'development',
  entry: './src/index.tsx',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js',
    publicPath: 'http://localhost:3003/',
    clean: true,
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
  },
  module: {
    rules: [
      {
        test: /\.(ts|tsx|js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              '@babel/preset-react',
              '@babel/preset-typescript',
            ],
          },
        },
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'page_b_mfe',
      filename: 'remoteEntry.js',
      exposes: {
        './sspa': './src/index.tsx',
      },
      shared: {
        react: {
          singleton: true,
          requiredVersion: deps.react,
          eager: true,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: deps['react-dom'],
          eager: true,
        },
        'single-spa': {
          singleton: true,
          requiredVersion: deps['single-spa'],
          eager: true,
        },
      },
    }),
    new webpack.HotModuleReplacementPlugin(),
  ],
  devServer: {
    port: 3003,
    hot: true,
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  },
};
```

### Update `package.json` scripts:

```json
{
  "scripts": {
    "start": "webpack serve --mode development",
    "build": "webpack --mode production",
    "dev": "webpack serve --mode development --watch"
  }
}
```

---

# Phase 6: Running the Demo

## Step 8: Start All Services

### Terminal 1 - Start Root App:

```bash
cd packages/root-app
npm start
# Runs on http://localhost:3000
```

### Terminal 2 - Start Navbar MFE:

```bash
cd packages/navbar-mfe
npm start
# Runs on http://localhost:3001
```

### Terminal 3 - Start Page A MFE:

```bash
cd packages/page-a-mfe
npm start
# Runs on http://localhost:3002
```

### Terminal 4 - Start Page B MFE:

```bash
cd packages/page-b-mfe
npm start
# Runs on http://localhost:3003
```

---

## Step 9: Test the Demo

1. **Navigate to http://localhost:3000**
   - You should see the header and footer
   - The navbar should load (you may see "Loading navbar..." briefly)

2. **Click "Page A" in the navbar**
   - Page A MFE loads
   - Try the counter to verify state management

3. **Click "Page B"**
   - Page A unmounts
   - Page B mounts
   - Notice navbar stayed visible (never unmounted)
   - Try the todo list

4. **Click "Page A" again**
   - Page B unmounts
   - Page A mounts with the counter reset

5. **Check the Browser Console**
   - You'll see lifecycle logs:
     ```
     [Root] Mounting...
     [Navbar MFE] Bootstrapping...
     [Navbar MFE] Mounting...
     [Page A MFE] Bootstrapping...
     [Page A MFE] Mounting...
     [Page B MFE] Bootstrapping...
     [Page B MFE] Mounting...
     [Page A MFE] Unmounting...
     ```

---

# Phase 7: Understanding What Just Happened

## Why Did This Work?

### 1. **Module Federation**
   - Each MFE runs on its own port with its own webpack dev server
   - webpack's `ModuleFederationPlugin` allows MFEs to expose modules
   - Root app's `remotes` config tells it where to find each MFE
   - `shared` config ensures React is loaded only once

### 2. **single-spa Registration**
   - Root app registered all MFEs with `registerApplication()`
   - `activeWhen` functions/paths determined when each MFE should mount
   - `start()` began monitoring URL changes

### 3. **Lifecycle Methods**
   - Each MFE exported `bootstrap`, `mount`, `unmount`
   - single-spa called these at the right time
   - Bootstrap happens once; mount/unmount happens on every activation

### 4. **DOM Containers**
   - Root app provided `<div id="page-container" />` for pages
   - Root app provided `<div id="navbar-container" />` for navbar
   - Each MFE rendered into the appropriate container

---

# Phase 8: Next Steps & Improvements

### Things You Can Try:

1. **Add a shared state layer** (Redux or Context)
   ```typescript
   // Create a shared store in root-app, pass via customProps
   shared: {
     reduxStore: sharedStore,
   }
   ```

2. **Add error handling** in activity functions
   ```typescript
   activeWhen: (location) => {
     try {
       return checkConditions(location);
     } catch (err) {
       console.error('Error evaluating activity:', err);
       return false;
     }
   }
   ```

3. **Implement feature flags**
   ```typescript
   const isEnabled = featureFlags.get('page-b-enabled');
   activeWhen: isEnabled ? '/page-b' : '/(^(?!/page-b).)*$',
   ```

4. **Add a root-level navigation router**
   ```typescript
   // Use react-router-dom in root-app for client-side routing
   // MFEs subscribe to route changes via customProps
   ```

5. **Share API clients** via customProps
   ```typescript
   customProps: {
     apiClient: new ApiClient(),
     eventBus: new EventBus(),
   }
   ```

---

## Common Issues & Troubleshooting

### Issue: "Cannot find module '@mfe/...'"
**Solution**: Make sure all MFE dev servers are running on their assigned ports.

### Issue: "CORS errors"
**Solution**: Check webpack devServer config includes:
```javascript
headers: {
  'Access-Control-Allow-Origin': '*',
}
```

### Issue: "React not found"
**Solution**: Ensure `eager: true` and `singleton: true` in Module Federation shared config.

### Issue: "MFE not mounting"
**Solution**: 
- Check browser console for bootstrap/mount errors
- Verify the container element exists: `document.getElementById('page-container')`
- Ensure MFE export lifecycle functions with correct names

### Issue: "Page reloads instead of navigation"
**Solution**: Check root-app is properly preventing default behavior and using single-spa's routing.

---

## Summary

You've built a complete microfrontend system with:

✓ **Root app** that orchestrates everything
✓ **Navbar** that persists across page changes
✓ **Independent page MFEs** that can be mounted/unmounted
✓ **Module Federation** for efficient code sharing
✓ **single-spa** managing the lifecycle

The key concepts:
- **activeWhen** determines when MFEs load
- **Lifecycle methods** control bootstrap/mount/unmount
- **Module Federation** shares dependencies
- **customProps** passes data from root to MFEs

Now you understand how the production system works! 🎉
