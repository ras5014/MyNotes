# 📚 Single-SPA Implementation Guide

A comprehensive learning guide for implementing single-spa in a React micro-frontend architecture.

---

## 📋 Table of Contents

1. [What is Single-SPA?](#what-is-single-spa)
2. [Problem & Solution](#problem--solution)
3. [Core Concepts](#core-concepts)
4. [Architecture Overview](#architecture-overview)
5. [Step-by-Step Implementation](#step-by-step-implementation)
6. [Project Structure](#project-structure)
7. [Running the Project](#running-the-project)
8. [Key Files & Examples](#key-files--examples)
9. [Common Questions](#common-questions)
10. [Resources](#resources)

---

## What is Single-SPA?

**single-spa** (single-spa applications) is a JavaScript framework that brings **router-driven architecture** to micro-frontends.

### In Simple Terms:
- It's like a **central hub** that manages when to show/hide your micro-frontend apps
- It **watches URL changes** and decides which apps to mount
- It **automates cleanup** when switching between apps
- It's **framework-agnostic** (works with React, Vue, Angular, Svelte, etc.)

### Official Definition:
> A top-level router that ensures only the necessary micro-frontends are loaded, bootstrapped, mounted, and unmounted at any given time.

---

## Problem & Solution

### The Problem (Current Setup with Just Module Federation)

Your current architecture uses **Webpack Module Federation** which has limitations:

```
┌─────────────────────────────────────┐
│          HOST APP (port 3000)        │
│                                      │
│  ├─ Header (always loaded)           │
│  ├─ Counter (always loaded)          │
│  ├─ Products (always loaded)         │
│  └─ Redux Store (shared)             │
│                                      │
│  Problems:                           │
│  ❌ All apps in memory at startup    │
│  ❌ No automatic lifecycle management│
│  ❌ Hardcoded URLs in config         │
│  ❌ One broken app crashes all       │
│  ❌ Hard to deploy independently     │
└─────────────────────────────────────┘
```

### The Solution (Adding Single-SPA)

single-spa provides **route-based orchestration**:

```
┌──────────────────────────────────────┐
│          HOST APP (port 3000)         │
│  ┌──────────────────────────────────┐│
│  │  single-spa Router                ││
│  │  - Watches URL changes            ││
│  │  - Mounts/Unmounts apps           ││
│  │  - Manages lifecycle              ││
│  └──────────────────────────────────┘│
└──────────────────────────────────────┘
           ↓
    URL Changes: /counter
           ↓
  ┌────────────────────────┐
  │ Counter App            │
  │ ✅ bootstrap() [once]  │
  │ ✅ mount()             │
  │ ✅ unmount() [cleanup] │
  └────────────────────────┘

Benefits:
✅ Apps load on demand
✅ Only active app in memory
✅ Automatic cleanup
✅ Independent deployment
✅ Error isolation
```

---

## Core Concepts

### 1. **Application Registration**

Tell single-spa about your micro-frontends:

```javascript
singleSpa.registerApplication({
  name: '@app/counter',                    // Unique identifier
  app: () => System.import('@app/counter/singleSpaEntry'),  // How to load
  activeWhen: '/counter',                  // When to show
  props: { /* shared data */ }             // Data to pass
});
```

### 2. **The Three Lifecycle Functions**

Every micro-frontend exports these three functions:

```javascript
// 1️⃣ BOOTSTRAP - Called once on first load
export async function bootstrap(props) {
  console.log('Setting up app...');
  // Initialize services, load config, etc.
}

// 2️⃣ MOUNT - Called when route matches
export async function mount(props) {
  console.log('Rendering app...');
  // Render to DOM
  ReactDOM.createRoot(el).render(<App />);
}

// 3️⃣ UNMOUNT - Called when leaving route
export async function unmount(props) {
  console.log('Cleaning up...');
  // Remove from DOM, cleanup listeners
  root.unmount();
}
```

### 3. **Lifecycle Flow**

```
First Visit to /counter:
┌─────────────┐
│  bootstrap()│  ← One time only
└──────┬──────┘
       ↓
┌─────────────┐
│  mount()    │  ← Render to DOM
└──────┬──────┘
       ↓
   User sees app
       ↓
(Navigation away)
       ↓
┌──────────────┐
│  unmount()   │  ← Clean up
└──────┬───────┘
       ↓
(Navigation back to /counter)
       ↓
┌─────────────┐
│  mount()    │  ← Render again (bootstrap skipped!)
└─────────────┘
```

### 4. **Props (Shared Data)**

Host passes data to micro-frontends:

```javascript
// Host registers app with props
singleSpa.registerApplication({
  name: '@app/counter',
  app: () => System.import('@app/counter/singleSpaEntry'),
  activeWhen: '/counter',
  props: {
    store: reduxStore,
    userId: 'user123',
    apiUrl: 'https://api.example.com'
  }
});

// Micro-frontend receives props
export async function mount(props) {
  const { store, userId, apiUrl } = props;
  // Use shared data
}
```

### 5. **activeWhen Patterns**

Control when apps are mounted based on routes:

```javascript
activeWhen: '/'              // Always visible
activeWhen: '/counter'       // Only on /counter
activeWhen: '/products'      // Only on /products
activeWhen: ['/settings', '/admin']  // Multiple routes
activeWhen: (location) => {   // Custom function
  return location.pathname.startsWith('/user');
}
```

---

## Architecture Overview

### File Structure

```
react/
├── host/                          # Main shell app
│   ├── src/
│   │   ├── main.jsx              # Entry point
│   │   ├── App.jsx               # Host UI with routing
│   │   ├── registerMicroFrontends.js  # NEW: single-spa registration
│   │   ├── app/
│   │   │   └── store.jsx         # Redux store
│   │   ├── features/
│   │   │   └── counter/
│   │   │       └── counterSlice.js
│   │   └── providers/
│   │       └── AppStoreProvider.jsx
│   ├── webpack.config.js
│   └── package.json
│
├── counter/                       # Micro-frontend 1
│   ├── src/
│   │   ├── main.jsx              # CHANGED: exports bootstrap, mount, unmount
│   │   ├── Counter.jsx           # Component
│   │   ├── features/
│   │   │   └── counter/
│   │   │       └── counterSlice.js
│   │   └── providers/
│   │       └── FallbackStoreProvider.jsx
│   ├── webpack.config.js         # UPDATED: expose singleSpaEntry
│   └── package.json
│
├── header/                        # Micro-frontend 2
│   ├── src/
│   │   ├── main.jsx              # CHANGED: exports bootstrap, mount, unmount
│   │   ├── Header.jsx
│   ├── webpack.config.js
│   └── package.json
│
├── products/                      # Micro-frontend 3
│   ├── src/
│   │   ├── main.jsx              # CHANGED: exports bootstrap, mount, unmount
│   │   ├── Products.jsx
│   │   ├── ProductList.jsx
│   ├── webpack.config.js
│   └── package.json
│
└── SINGLE_SPA_GUIDE.md           # This file
```

### Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Routing** | single-spa | App registration & lifecycle |
| **Module Loading** | Webpack Module Federation | Code splitting & sharing |
| **State Management** | Redux + React-Redux | Shared state across micro-frontends |
| **Bundling** | Webpack 5 | Build and serve apps |
| **Framework** | React 18 | UI library |

---

## Step-by-Step Implementation

### Phase 1: Installation

#### Step 1.1: Install single-spa in all apps

```bash
# In host/
npm install single-spa

# In counter/
npm install single-spa

# In header/
npm install single-spa

# In products/
npm install single-spa
```

---

### Phase 2: Update Micro-Frontends

#### Step 2.1: Update Counter's main.jsx

**File:** `counter/src/main.jsx`

**Current (synchronous rendering):**
```javascript
import React from "react";
import ReactDOM from "react-dom/client";
import { Counter } from "./Counter.jsx";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<Counter />);
```

**New (single-spa lifecycle):**
```javascript
import React from "react";
import ReactDOM from "react-dom/client";
import { Counter } from "./Counter.jsx";

let root = null;

/**
 * BOOTSTRAP - One-time setup
 * Called once when app is first registered
 */
export async function bootstrap(props) {
  console.log("🔧 Counter: Bootstrap called", props);
  // Initialize services, load config, etc.
}

/**
 * MOUNT - Render when active
 * Called when route matches
 */
export async function mount(props) {
  console.log("📌 Counter: Mount called with props:", props);
  const rootElement = document.getElementById("counter-app");
  root = ReactDOM.createRoot(rootElement);
  root.render(<Counter />);
}

/**
 * UNMOUNT - Clean up
 * Called when route changes away
 */
export async function unmount(props) {
  console.log("🗑️ Counter: Unmount called");
  if (root) {
    root.unmount();
    root = null;
  }
}
```

**Key changes:**
- Use `let root = null` to track the instance
- Export three async functions
- Use specific DOM element ID (`counter-app`)
- Proper cleanup in unmount

#### Step 2.2: Update Header's main.jsx

**File:** `header/src/main.jsx`

Same pattern as Counter:

```javascript
import React from "react";
import ReactDOM from "react-dom/client";
import { Header } from "./Header.jsx";

let root = null;

export async function bootstrap(props) {
  console.log("🔧 Header: Bootstrap called");
}

export async function mount(props) {
  console.log("📌 Header: Mount called");
  const rootElement = document.getElementById("header-app");
  root = ReactDOM.createRoot(rootElement);
  root.render(<Header />);
}

export async function unmount(props) {
  console.log("🗑️ Header: Unmount called");
  if (root) {
    root.unmount();
    root = null;
  }
}
```

#### Step 2.3: Update Products's main.jsx

**File:** `products/src/main.jsx`

Same pattern:

```javascript
import React from "react";
import ReactDOM from "react-dom/client";
import { Products } from "./Products.jsx";

let root = null;

export async function bootstrap(props) {
  console.log("🔧 Products: Bootstrap called");
}

export async function mount(props) {
  console.log("📌 Products: Mount called");
  const rootElement = document.getElementById("products-app");
  root = ReactDOM.createRoot(rootElement);
  root.render(<Products />);
}

export async function unmount(props) {
  console.log("🗑️ Products: Unmount called");
  if (root) {
    root.unmount();
    root = null;
  }
}
```

---

### Phase 3: Update Webpack Configurations

#### Step 3.1: Update Counter's webpack.config.js

**File:** `counter/webpack.config.js`

In the `ModuleFederationPlugin` section, add the `singleSpaEntry` expose:

```javascript
exposes: {
  "./Counter": "./src/Counter.jsx",
  "./singleSpaEntry": "./src/main.jsx",  // ← ADD THIS LINE
},
```

#### Step 3.2: Update Header's webpack.config.js

**File:** `header/webpack.config.js`

```javascript
exposes: {
  "./Header": "./src/Header.jsx",
  "./singleSpaEntry": "./src/main.jsx",  // ← ADD THIS LINE
},
```

#### Step 3.3: Update Products's webpack.config.js

**File:** `products/webpack.config.js`

```javascript
exposes: {
  "./Products": "./src/Products.jsx",
  "./singleSpaEntry": "./src/main.jsx",  // ← ADD THIS LINE
},
```

#### Step 3.4: Update Host's webpack.config.js

**File:** `host/webpack.config.js`

Add `single-spa` to shared dependencies:

```javascript
shared: {
  react: { singleton: true, eager: true },
  "react-dom": { singleton: true, eager: true },
  "react-redux": { singleton: true, eager: true },
  "@reduxjs/toolkit": { singleton: true, eager: true },
  "single-spa": { singleton: true },  // ← ADD THIS LINE
},
```

---

### Phase 4: Create single-spa Registration File

#### Step 4.1: Create registerMicroFrontends.js

**File:** `host/src/registerMicroFrontends.js`

```javascript
import * as singleSpa from 'single-spa';

/**
 * Register all micro-frontend applications
 * This tells single-spa about each app and when to activate it
 */

// Register Header - Always visible
singleSpa.registerApplication({
  name: '@app/header',
  app: () => System.import('@app/header/singleSpaEntry'),
  activeWhen: '/',  // Always active on root path
  props: {},
});

// Register Counter - Show only on /counter
singleSpa.registerApplication({
  name: '@app/counter',
  app: () => System.import('@app/counter/singleSpaEntry'),
  activeWhen: '/counter',
  props: {},
});

// Register Products - Show only on /products
singleSpa.registerApplication({
  name: '@app/products',
  app: () => System.import('@app/products/singleSpaEntry'),
  activeWhen: '/products',
  props: {},
});

console.log('✅ Micro-frontends registered with single-spa');
```

---

### Phase 5: Update Host App.jsx

#### Step 5.1: Update App.jsx with Routing

**File:** `host/src/App.jsx`

Replace the current implementation with route-based rendering:

```javascript
import React from "react";
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';

export function App() {
  return (
    <Router>
      <div>
        {/* Navigation */}
        <nav style={{ padding: '20px', borderBottom: '1px solid #ccc' }}>
          <Link to="/" style={{ marginRight: '20px' }}>Header</Link>
          <Link to="/counter" style={{ marginRight: '20px' }}>Counter</Link>
          <Link to="/products" style={{ marginRight: '20px' }}>Products</Link>
        </nav>

        {/* Container for Header (always visible) */}
        <div id="header-app" style={{ padding: '10px' }}></div>

        {/* Container for Counter (mounted when route is /counter) */}
        <div id="counter-app" style={{ padding: '10px' }}></div>

        {/* Container for Products (mounted when route is /products) */}
        <div id="products-app" style={{ padding: '10px' }}></div>
      </div>
    </Router>
  );
}
```

---

### Phase 6: Update Host main.jsx

#### Step 6.1: Initialize single-spa

**File:** `host/src/main.jsx`

```javascript
import React from "react";
import ReactDOM from "react-dom/client";
import { App } from "./App.jsx";
import "./index.css";
import { AppStoreProvider } from "./providers/AppStoreProvider.jsx";

// IMPORTANT: Register micro-frontends BEFORE starting single-spa
import './registerMicroFrontends';
import * as singleSpa from 'single-spa';

const root = ReactDOM.createRoot(document.getElementById("root"));

root.render(
  <AppStoreProvider>
    <App />
  </AppStoreProvider>,
);

// Start single-spa routing
// This must happen AFTER React app is mounted
singleSpa.start();

console.log('✅ single-spa started');
```

---

## Key Files & Examples

### Counter Lifecycle Example

When user navigates to `/counter`:

```
1. Browser URL changes: / → /counter

2. single-spa detects URL change
   └─> Runs all apps' activeWhen checks

3. Header's activeWhen: '/' 
   └─> Does NOT match (route is /counter)
   └─> Calls: unmount()
   └─> Header component removed from DOM

4. Counter's activeWhen: '/counter'
   └─> MATCHES! (route is /counter)
   └─> Has Counter been bootstrapped before?
       ├─> NO: Call bootstrap() first
       └─> YES: Skip bootstrap
   └─> Call: mount()
   └─> Counter rendered to DOM

5. Products's activeWhen: '/products'
   └─> Does NOT match (route is /counter)
   └─> Calls: unmount()
   └─> Products component removed from DOM

Result: Only Counter visible on screen ✅
```

### Shared State with Redux

Counter still uses Redux to share state with Host:

```javascript
// Host creates Redux store
const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

// Host passes store to Counter via props
singleSpa.registerApplication({
  name: '@app/counter',
  app: () => System.import('@app/counter/singleSpaEntry'),
  activeWhen: '/counter',
  props: { store }  // ← Pass store here
});

// Counter receives store in mount()
export async function mount(props) {
  const { store } = props;
  root.render(
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}
```

---

## Running the Project

### Prerequisites

- Node.js 14+ installed
- npm or yarn

### Start All Applications

**Terminal 1: Host**
```bash
cd host
npm start
# Runs on http://localhost:3000
```

**Terminal 2: Counter**
```bash
cd counter
npm start
# Runs on http://localhost:3003
```

**Terminal 3: Header**
```bash
cd header
npm start
# Runs on http://localhost:3001
```

**Terminal 4: Products**
```bash
cd products
npm start
# Runs on http://localhost:3002
```

### Test the Implementation

1. Open http://localhost:3000 in browser
2. Open browser DevTools Console (F12)
3. Click navigation links:
   - Click "Header" → See console logs for Header mount/unmount
   - Click "Counter" → See console logs for Counter bootstrap/mount, Header unmount
   - Click "Products" → See console logs for Products bootstrap/mount, Counter unmount
4. Watch the DOM in DevTools to see elements appear/disappear

### Expected Console Output

```
✅ Micro-frontends registered with single-spa
✅ single-spa started

📌 Header: Mount called

(Click "Counter" link)
🗑️ Header: Unmount called
🔧 Counter: Bootstrap called
📌 Counter: Mount called

(Click "Products" link)
🗑️ Counter: Unmount called
🔧 Products: Bootstrap called
📌 Products: Mount called
```

---

## Common Questions

### Q: Why does bootstrap() only run once?

**A:** Initialization is expensive (loading config, setting up services). single-spa caches the bootstrapped state so if you return to a route, mount() runs without re-initializing.

```
First visit to /counter:
  bootstrap() ← One time
  mount()     ← Render

Leave /counter, then return:
  mount()     ← Render again (bootstrap skipped!)
```

### Q: What happens if unmount() fails?

**A:** single-spa stops the unmounting process and logs an error. The old app remains mounted. This prevents breaking other apps.

### Q: Can multiple micro-frontends be visible at once?

**A:** Yes! Use overlapping activeWhen patterns:

```javascript
// Header always visible
registerApplication({
  name: '@app/header',
  activeWhen: '/',
});

// Counter also visible
registerApplication({
  name: '@app/counter',
  activeWhen: '/counter',
});

// Now on /counter, BOTH Header and Counter are visible
```

### Q: How do I share state between micro-frontends?

**A:** Three approaches:

1. **Redux** (your current setup)
   - Host creates store
   - Passes via props to micro-frontends
   - All use same Redux store

2. **Props from Host**
   - Host passes data via `props` in registration
   - Simple but limited

3. **Custom Event System**
   - Micro-frontends emit/listen to custom events
   - Decoupled communication

### Q: Does single-spa replace Module Federation?

**A:** No! They work together:
- **Module Federation**: *How* to load code (handles bundling)
- **single-spa**: *When* to load apps (handles routing/lifecycle)

### Q: Can micro-frontends fail silently?

**A:** Add error handling:

```javascript
registerApplication({
  name: '@app/counter',
  app: async () => {
    try {
      return await System.import('@app/counter/singleSpaEntry');
    } catch (error) {
      console.error('Failed to load counter:', error);
      // Return dummy lifecycle functions
      return {
        bootstrap: async () => {},
        mount: async () => {
          document.getElementById('counter-app').innerHTML = 
            '<div style="color:red;">Failed to load Counter</div>';
        },
        unmount: async () => {},
      };
    }
  },
  activeWhen: '/counter',
});
```

### Q: How do I debug single-spa?

**A:** Enable devtools:

```javascript
// In host/src/main.jsx before singleSpa.start()
window.__SINGLE_SPA_DEVTOOLS__ = true;

// Then in browser console:
window.singleSpa.getMountedApps();           // See active apps
window.singleSpa.getAppStatus('@app/counter');  // Check app status
```

---

## Resources

### Official Documentation
- [single-spa Official Docs](https://single-spa.js.org)
- [single-spa API Reference](https://single-spa.js.org/docs/api)
- [Webpack Module Federation](https://webpack.js.org/concepts/module-federation/)

### Learning Resources
- [single-spa Video Tutorial](https://www.youtube.com/playlist?list=PLM_Na2U-_FO3b3EbmFt7aI_CHx9E5UhNX)
- [Micro Frontends with single-spa](https://www.pluralsight.com/courses/micro-frontends-single-spa)

### Related Tools
- [@react-micro-frontend/devtools](https://www.npmjs.com/package/@react-micro-frontend/devtools) - Browser DevTools for single-spa
- [single-spa-layout](https://www.npmjs.com/package/single-spa-layout) - Declarative router for single-spa

---

## Summary

| Concept | What It Does |
|---------|------------|
| **single-spa** | Routes between micro-frontend apps |
| **registerApplication** | Tell single-spa about your apps |
| **activeWhen** | Define when apps should be visible |
| **bootstrap()** | One-time initialization |
| **mount()** | Render app to DOM |
| **unmount()** | Clean up from DOM |
| **Module Federation** | Load code from remote apps |
| **Props** | Pass shared data to micro-frontends |

---

**Last Updated:** January 29, 2026  
**Status:** Learning Guide Complete
