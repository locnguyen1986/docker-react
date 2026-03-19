# React with Bun

This report explores using [Bun](https://bun.sh) as a runtime, package manager, bundler, and dev server for React applications. It covers what Bun is, how to start new React projects with it, how to migrate the existing Create React App project in this repository (which uses `react-scripts 3.4.3`), and the key differences and gotchas compared to the standard npm/CRA workflow.

---

## Table of Contents

1. [What is Bun?](#1-what-is-bun)
2. [Setting Up a New React Project with Bun](#2-setting-up-a-new-react-project-with-bun)
3. [Migrating the Existing CRA Project to Bun](#3-migrating-the-existing-cra-project-to-bun)
4. [Bun as Dev Server and Bundler (Native Approach)](#4-bun-as-dev-server-and-bundler-native-approach)
5. [Key Differences and Gotchas vs npm/CRA](#5-key-differences-and-gotchas-vs-npmcra)
6. [Practical Code Snippets](#6-practical-code-snippets)
7. [Summary Comparison Table](#7-summary-comparison-table)
8. [References](#8-references)

---

## 1. What is Bun?

Bun is an all-in-one JavaScript runtime and toolkit designed as a drop-in replacement for Node.js with dramatically faster performance. It is written in Zig and uses JavaScriptCore (the engine behind Safari) rather than V8, which contributes to its startup speed and throughput advantages.

### Core capabilities

| Capability | Description |
|---|---|
| **Runtime** | Execute `.js`, `.ts`, `.jsx`, `.tsx` files directly — no compilation step needed |
| **Package manager** | `bun install` reads `package.json` and resolves dependencies up to 30× faster than npm |
| **Bundler** | `bun build` produces optimised browser-ready bundles with tree-shaking |
| **Test runner** | `bun test` runs Jest-compatible tests with a built-in test harness |
| **Dev server** | `Bun.serve()` provides an HTTP server with native JSX/TSX hot-module support |

### Why Bun matters for React development

- **Faster installs.** `bun install` uses a binary lockfile (`bun.lockb`) and a global cache, making repeated installs almost instant. A fresh install of a typical React app takes seconds rather than the 30–60 seconds common with npm.
- **Native JSX/TSX transpilation.** Bun can parse and execute `.jsx` and `.tsx` files out of the box without a Babel configuration. There is no need for `@babel/preset-react` or any additional transpiler plugin.
- **Fast Hot Module Replacement (HMR).** When paired with Vite, Bun's speed at resolving and serving modules means HMR round-trips are perceptibly faster than with Node.js.
- **Zero-config TypeScript.** Bun strips TypeScript types at runtime without invoking `tsc`, so TypeScript "just works" without a separate compile step during development.
- **Single binary.** Unlike the Node.js ecosystem where you need `node`, `npm`/`yarn`/`pnpm`, and separate bundler tools, Bun ships as one binary that handles all of these roles.

### Bun version note

This report targets **Bun v1.x** (the stable series as of early 2026). Install it with:

```bash
curl -fsSL https://bun.sh/install | bash
# or on macOS via Homebrew
brew install oven-sh/bun/bun

bun --version
```

---

## 2. Setting Up a New React Project with Bun

There are two main approaches to creating a new React project that uses Bun: pairing Bun with Vite (the recommended approach for most teams) or using Bun entirely natively without an additional dev-server framework.

### 2.1 Approach A: Vite + Bun (Recommended)

Vite is a framework-agnostic dev server and bundler that integrates seamlessly with Bun. Using Bun as the package manager and runtime alongside Vite gives you the best of both worlds: Vite's battle-tested React plugin ecosystem and Bun's fast installs and execution.

#### Create the project

```bash
bun create vite my-app --template react-ts
cd my-app
bun install
```

For a JavaScript-only project (no TypeScript):

```bash
bun create vite my-app --template react
cd my-app
bun install
```

#### Start the dev server

```bash
bun run dev
```

#### Build for production

```bash
bun run build
```

#### Preview the production build locally

```bash
bun run preview
```

#### Resulting `package.json` scripts section

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite --preview"
  }
}
```

> **Tip:** You can also run Vite directly through Bun's `bunx` tool (analogous to `npx`):
> ```bash
> bunx vite
> ```

### 2.2 Approach B: Bun Native (No Vite)

For maximum control — or when you want to avoid adding Vite as a dependency — you can bootstrap a React project using only Bun's built-in capabilities.

#### Initialise a new project

```bash
mkdir my-react-app && cd my-react-app
bun init
```

`bun init` asks a few questions (package name, entry point) and creates a minimal `package.json`, `index.ts`, `tsconfig.json`, and `.gitignore`.

#### Add React dependencies

```bash
bun add react react-dom
bun add -d @types/react @types/react-dom
```

#### Create the entry point

Create `index.tsx`:

```tsx
import { createRoot } from "react-dom/client";

function App() {
  return <h1>Hello from Bun + React!</h1>;
}

const container = document.getElementById("root");
if (!container) throw new Error("Root element not found");

createRoot(container).render(<App />);
```

#### Create a minimal HTML shell

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My Bun React App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="./index.tsx"></script>
  </body>
</html>
```

#### Configure JSX in `bunfig.toml`

```toml
[jsx]
# Use the classic React JSX transform
jsx = "react"
jsxImportSource = "react"
```

#### Add scripts to `package.json`

```json
{
  "scripts": {
    "dev": "bun --hot index.tsx",
    "build": "bun build ./index.tsx --outdir ./dist --target browser",
    "test": "bun test"
  }
}
```

---

## 3. Migrating the Existing CRA Project to Bun

The repository at `/workspace/repo` is a standard Create React App project bootstrapped with `react-scripts 3.4.3`, using React 16 and plain JavaScript (no TypeScript). The `package.json` looks like:

```json
{
  "name": "frontend",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^4.2.4",
    "@testing-library/react": "^9.5.0",
    "@testing-library/user-event": "^7.2.1",
    "react": "^16.13.1",
    "react-dom": "^16.13.1",
    "react-scripts": "3.4.3"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  }
}
```

There are two levels of migration:

- **Level 1 (Drop-in):** Replace only npm/Node with Bun, keeping `react-scripts` intact.
- **Level 2 (Full migration):** Replace `react-scripts` with Vite + Bun for a modern, maintained stack.

### 3.1 Level 1 — Drop-in: Replace npm with Bun (keep react-scripts)

This is the lowest-risk option. Everything stays the same; Bun just handles package installation and script running instead of npm.

#### Step 1: Install Bun

```bash
curl -fsSL https://bun.sh/install | bash
# Reload your shell so `bun` is in PATH
source ~/.bashrc   # or ~/.zshrc
```

#### Step 2: Install dependencies with Bun

```bash
cd /workspace/repo
bun install
```

Bun reads the existing `package.json` and `package-lock.json`, resolves all dependencies, and produces a `bun.lockb` binary lockfile alongside the existing lock file. Your `node_modules` folder is populated just as with `npm install`.

#### Step 3: Run the dev server

```bash
bun run start
# equivalent to: react-scripts start
```

#### Step 4: Build for production

```bash
bun run build
# equivalent to: react-scripts build
```

#### Step 5: Run tests

```bash
bun run test
# equivalent to: react-scripts test (which uses Jest under the hood)
```

> **Note:** At Level 1, `react-scripts` is still executing under Node.js internally — Bun only acts as the package manager and script runner. You get faster installs but the same webpack-based build pipeline.

#### Step 6 (Optional): Add `bun.lockb` to `.gitignore` or commit it

If your team standardises on Bun, commit `bun.lockb` and remove `package-lock.json`:

```bash
# In .gitignore — remove the line for bun.lockb if it's there
# and optionally add package-lock.json
echo "package-lock.json" >> .gitignore
git add bun.lockb
```

### 3.2 Level 2 — Full Migration: Replace react-scripts with Vite + Bun

`react-scripts 3.4.3` is based on webpack 4 and Create React App, which reached end-of-life in early 2023. Migrating to Vite gives you a modern, actively maintained build toolchain with faster HMR and build times.

#### Step 1: Install Vite and the React plugin

```bash
bun add -d vite @vitejs/plugin-react
```

#### Step 2: Remove react-scripts

```bash
bun remove react-scripts
```

#### Step 3: Create `vite.config.js` in the project root

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000, // keep the same port as CRA for muscle memory
    open: true,
  },
  build: {
    outDir: "build", // CRA outputs to `build/`, Vite defaults to `dist/`
  },
});
```

#### Step 4: Move `public/index.html` to the project root

Vite expects `index.html` at the root of the project, not inside `public/`. CRA stores it in `public/index.html`.

```bash
mv public/index.html ./index.html
```

Then update `index.html` to reference your entry point directly (Vite does not inject scripts automatically the same way CRA does):

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <title>React App</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
    <!-- Vite entry point — replaces CRA's injected script -->
    <script type="module" src="/src/index.js"></script>
  </body>
</html>
```

#### Step 5: Update `package.json` scripts

```json
{
  "scripts": {
    "start": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "bun test"
  }
}
```

#### Step 6: Handle `process.env` references

CRA exposes environment variables as `process.env.REACT_APP_*`. Vite uses `import.meta.env.VITE_*`. You will need to:

1. Rename `.env` variables from `REACT_APP_FOO` to `VITE_FOO`.
2. Replace all `process.env.REACT_APP_FOO` references in source files with `import.meta.env.VITE_FOO`.

```bash
# Find all process.env.REACT_APP_ usages
grep -r "process\.env\.REACT_APP_" src/
```

#### Step 7: Handle static assets in `public/`

Vite serves the `public/` folder as static assets at the root path, just like CRA. Files in `public/` (other than `index.html` which you moved) can stay in place.

#### Step 8: Upgrade React 16 → 18 (Optional but recommended)

`react-scripts 3.4.3` ships with React 16. If you are moving to Vite, upgrading to React 18 is straightforward:

```bash
bun add react@^18 react-dom@^18
```

Update `src/index.js` to use the new root API:

```jsx
// Before (React 16)
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

ReactDOM.render(<App />, document.getElementById("root"));
```

```jsx
// After (React 18)
import React from "react";
import { createRoot } from "react-dom/client";
import App from "./App";

const container = document.getElementById("root");
const root = createRoot(container);
root.render(<App />);
```

> **React 18 upgrade notes:**
> - `ReactDOM.render` is deprecated but still works (with a console warning).
> - Concurrent mode features (Suspense, `useTransition`, `useDeferredValue`) become available.
> - `@testing-library/react` should be upgraded to v14+ for React 18 compatibility.
> - `@testing-library/user-event` should be upgraded to v14+ as well.

#### Step 9: Verify the migration

```bash
bun run start    # should open the app at http://localhost:3000
bun run build    # should produce a production build in ./build
```

---

## 4. Bun as Dev Server and Bundler (Native Approach)

Beyond acting as a package manager, Bun has first-class built-in capabilities for serving and building React applications without any external framework.

### 4.1 The `Bun.serve()` API

Bun's HTTP server (`Bun.serve`) can serve a React application with file watching and JSX transpilation baked in. Here is a minimal dev server:

```ts
// server.ts
import index from "./index.html";

const server = Bun.serve({
  port: 3000,
  routes: {
    // Serve the HTML shell for all routes (SPA pattern)
    "/*": index,
  },
  development: true, // enables source maps and better error messages
});

console.log(`Listening on http://localhost:${server.port}`);
```

Run with:

```bash
bun --hot server.ts
```

The `--hot` flag enables Bun's hot-reloading: when source files change, modules are reloaded without restarting the process.

### 4.2 `bun build` for production bundles

`bun build` is Bun's native bundler, similar to webpack or Rollup.

```bash
# Bundle index.tsx for the browser
bun build ./src/index.tsx \
  --outdir ./dist \
  --target browser \
  --minify \
  --sourcemap external
```

Key flags:

| Flag | Description |
|---|---|
| `--outdir` | Output directory (default: none — prints to stdout) |
| `--target` | `browser`, `bun`, or `node` |
| `--minify` | Enable minification (identifiers, whitespace, syntax) |
| `--sourcemap` | `inline`, `external`, or `none` |
| `--splitting` | Enable code splitting for lazy imports |
| `--public-path` | CDN/asset prefix for output files |

Example with code splitting:

```bash
bun build ./src/index.tsx \
  --outdir ./dist \
  --target browser \
  --splitting \
  --minify
```

### 4.3 Configuring JSX with `bunfig.toml`

`bunfig.toml` is Bun's project configuration file (analogous to `.npmrc` or `jest.config.js` for their respective tools).

```toml
# bunfig.toml

[jsx]
# "react" = classic transform (React.createElement)
# "react-jsx" = automatic transform (no React import needed)
jsx = "react-jsx"
jsxImportSource = "react"

[install]
# Use exact versions instead of ^ ranges when adding packages
exact = true

[test]
# Test file pattern
include = ["**/*.test.{ts,tsx,js,jsx}", "**/*.spec.{ts,tsx,js,jsx}"]
```

With `jsx = "react-jsx"` (the automatic transform), you do not need to import React in every file that uses JSX:

```tsx
// No "import React from 'react'" needed with react-jsx transform
export function Greeting({ name }: { name: string }) {
  return <p>Hello, {name}!</p>;
}
```

### 4.4 TypeScript configuration for Bun

When using Bun natively, the recommended `tsconfig.json` includes Bun's type declarations:

```json
{
  "compilerOptions": {
    "lib": ["ESNext", "DOM"],
    "module": "ESNext",
    "target": "ESNext",
    "moduleResolution": "bundler",
    "moduleDetection": "force",
    "allowImportingTsExtensions": true,
    "noEmit": true,
    "composite": false,
    "strict": true,
    "downlevelIteration": true,
    "skipLibCheck": true,
    "jsx": "react-jsx",
    "jsxImportSource": "react",
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "types": ["bun-types"]
  }
}
```

Install `bun-types` for full IDE support:

```bash
bun add -d bun-types
```

---

## 5. Key Differences and Gotchas vs npm/CRA

### 5.1 react-scripts compatibility

`react-scripts` runs webpack under the hood using Node.js APIs. When you run `bun run start` with `react-scripts` in the scripts, Bun spawns a child process that still runs through Node.js. This means:

- ✅ **It works** — you get faster installs and script-runner overhead is eliminated.
- ⚠️ **The build pipeline is still webpack-based** — you do not get Bun's native bundler speed for the actual compilation step.
- ⚠️ **`react-scripts 3.4.3` is very old** — it uses webpack 4, which has known performance limits on large codebases.

### 5.2 `bun test` vs Jest

`bun test` is a Jest-compatible test runner but with meaningful differences:

| Feature | Jest (via react-scripts) | bun test |
|---|---|---|
| Globals | Auto-injected (`describe`, `it`, `expect`) | Auto-injected in `bun test` context |
| `jest.mock()` | Full support | Partial — `mock.module()` is the Bun equivalent |
| Fake timers | `jest.useFakeTimers()` | `mock.timers` API (slightly different) |
| Coverage | `--coverage` | `--coverage` (uses V8 coverage) |
| Config file | `jest.config.js` | `bunfig.toml [test]` section |
| Speed | Slower (Node.js startup) | Faster (Bun runtime) |

> When running tests through `react-scripts test`, Jest runs inside Node.js regardless of whether you use `bun run test`. If you want the speed of `bun test`, you need to eject from react-scripts or migrate to Vite.

### 5.3 Environment variables

| Tool | Prefix | Access |
|---|---|---|
| CRA (react-scripts) | `REACT_APP_` | `process.env.REACT_APP_FOO` |
| Vite | `VITE_` | `import.meta.env.VITE_FOO` |
| Bun native | None required | `process.env.FOO` or `Bun.env.FOO` |

Bun reads `.env`, `.env.local`, `.env.development`, and `.env.production` files automatically — no `dotenv` package needed.

### 5.4 Missing or different Node.js built-in modules

Bun implements the most commonly used Node.js built-ins (`fs`, `path`, `crypto`, `http`, `stream`, etc.) but there are edge cases:

- Some native Node.js add-ons (`.node` files compiled with `node-gyp`) may not work with Bun.
- `node:vm` has limited support in Bun v1.x.
- Some packages that use internal Node.js C++ APIs directly will not work.

For React apps (which run in the browser, not in Bun's runtime), this is rarely an issue — it only matters for build scripts or server-side code.

### 5.5 `bun:sqlite` vs better-sqlite3

If your project uses `better-sqlite3` for any build-time scripting or SSR, Bun provides a built-in `bun:sqlite` module that is significantly faster:

```ts
// Instead of: const Database = require("better-sqlite3");
import { Database } from "bun:sqlite";

const db = new Database("mydb.sqlite");
const query = db.query("SELECT * FROM users WHERE id = $id");
const user = query.get({ $id: 1 });
```

### 5.6 Windows support

As of Bun v1.x, Windows support is production-ready but has some caveats:

- Bun on Windows requires Windows 10 version 1809 or later.
- Some shell scripts in `package.json` that use Unix-specific syntax (pipes, `&&`, environment variable exports) may behave differently. Use `cross-env` for environment variable setting if your team works across platforms.
- File paths with backslashes can cause issues in some configurations — use forward slashes in config files.

### 5.7 Lockfile format

Bun uses a binary lockfile (`bun.lockb`) rather than a text-based one (`package-lock.json` or `yarn.lock`). To see its contents:

```bash
bun bun.lockb  # prints a human-readable representation
```

Teams with strict audit requirements should be aware that `bun.lockb` is not human-readable in the standard diff sense. You can configure Bun to also output a `yarn.lock` for compatibility:

```toml
# bunfig.toml
[install]
saveTextLockfile = true  # generates yarn.lock alongside bun.lockb
```

### 5.8 `node:` prefix for built-in modules

Older Bun versions (before v1.0.4) had incomplete support for the `node:` prefix on built-in modules (e.g., `import { readFile } from "node:fs/promises"`). In Bun v1.x this is fully supported, but if you are pinned to an older Bun version and see `Cannot find module "node:fs"` errors, update Bun:

```bash
bun upgrade
```

### 5.9 Peer dependency warnings

Bun is more lenient about peer dependency version mismatches than npm. If your team relies on npm's strict peer dependency enforcement to catch version conflicts, be aware that `bun install` may not surface those warnings.

---

## 6. Practical Code Snippets

### 6.1 Minimal `index.tsx` entry point (React 18 + Bun)

```tsx
// src/index.tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App";
import "./index.css";

const container = document.getElementById("root");

if (!container) {
  throw new Error(
    "Root element with id='root' not found in the DOM. " +
    "Ensure your index.html has <div id=\"root\"></div>."
  );
}

const root = createRoot(container);

root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

### 6.2 Minimal `App.tsx`

```tsx
// src/App.tsx
import { useState } from "react";

interface CounterProps {
  initialCount?: number;
}

function Counter({ initialCount = 0 }: CounterProps) {
  const [count, setCount] = useState(initialCount);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
      <button onClick={() => setCount((c) => c - 1)}>Decrement</button>
    </div>
  );
}

export default function App() {
  return (
    <main>
      <h1>React + Bun</h1>
      <Counter initialCount={0} />
    </main>
  );
}
```

### 6.3 `bunfig.toml` — full example

```toml
# bunfig.toml
# Bun project configuration file

[jsx]
# Use the automatic JSX transform (no "import React" needed in every file)
jsx = "react-jsx"
jsxImportSource = "react"

[install]
# Equivalent of `npm install --save-exact` — avoids ^ version ranges
exact = false
# Cache directory (default: ~/.bun/install/cache)
# cache = "~/.bun/install/cache"

[install.scopes]
# Configure private registry for a scope
# "@mycompany" = { url = "https://npm.mycompany.com/", token = "$NPM_TOKEN" }

[test]
# Files to include when running `bun test`
include = [
  "**/*.test.{ts,tsx,js,jsx}",
  "**/*.spec.{ts,tsx,js,jsx}"
]
# Timeout per test in milliseconds
timeout = 5000
# Bail after N failures
# bail = 1

[run]
# Run scripts with bun instead of node when bun is not specified
# bun = true
```

### 6.4 `vite.config.ts` — full example for the Vite + Bun approach

```ts
// vite.config.ts
import { defineConfig, loadEnv } from "vite";
import react from "@vitejs/plugin-react";
import { resolve } from "path";

export default defineConfig(({ mode }) => {
  // Load env file based on mode (development, production, etc.)
  const env = loadEnv(mode, process.cwd(), "");

  return {
    plugins: [
      react({
        // Use SWC-based fast refresh (alternative: babel)
        // Remove this option to use babel instead
      }),
    ],

    resolve: {
      alias: {
        // Allows: import Button from "@/components/Button"
        "@": resolve(__dirname, "./src"),
      },
    },

    server: {
      port: 3000,
      open: true,
      // Proxy API calls to avoid CORS during development
      proxy: {
        "/api": {
          target: env.VITE_API_URL || "http://localhost:8080",
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, ""),
        },
      },
    },

    build: {
      // Match CRA's default output directory
      outDir: "build",
      // Generate source maps for production debugging
      sourcemap: true,
      rollupOptions: {
        output: {
          // Split vendor chunks for better caching
          manualChunks: {
            vendor: ["react", "react-dom"],
          },
        },
      },
    },

    // Expose env variables to the client (must be prefixed with VITE_)
    // Access as: import.meta.env.VITE_MY_VAR
    define: {
      // Polyfill process.env for libraries that still use it
      "process.env.NODE_ENV": JSON.stringify(mode),
    },
  };
});
```

### 6.5 Minimal `Bun.serve()` dev server with React

```ts
// dev-server.ts
// Run with: bun --hot dev-server.ts

const server = Bun.serve({
  port: Number(process.env.PORT) || 3000,

  async fetch(req: Request): Promise<Response> {
    const url = new URL(req.url);

    // Serve static files from the public directory
    const publicFile = Bun.file(`./public${url.pathname}`);
    if (await publicFile.exists()) {
      return new Response(publicFile);
    }

    // Serve the HTML shell for all other routes (SPA fallback)
    const html = await Bun.file("./index.html").text();
    return new Response(html, {
      headers: { "Content-Type": "text/html" },
    });
  },

  error(err: Error): Response {
    return new Response(`Server error: ${err.message}`, { status: 500 });
  },
});

console.log(`Dev server running at http://localhost:${server.port}`);
```

### 6.6 Example `bun test` test file

```tsx
// src/App.test.tsx
import { describe, it, expect, beforeEach } from "bun:test";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import App from "./App";

describe("App", () => {
  it("renders the heading", () => {
    render(<App />);
    expect(screen.getByText("React + Bun")).toBeDefined();
  });

  it("counter increments on button click", async () => {
    const user = userEvent.setup();
    render(<App />);

    const incrementBtn = screen.getByText("Increment");
    await user.click(incrementBtn);

    expect(screen.getByText("Count: 1")).toBeDefined();
  });
});
```

---

## 7. Summary Comparison Table

| Approach | Setup Command | Dev Server | Build Tool | HMR | TypeScript | Notes |
|---|---|---|---|---|---|---|
| **CRA (react-scripts 3.4.3)** | `npm install` | `react-scripts start` (webpack-dev-server) | webpack 4 | ✅ (slow) | ✅ (via Babel) | End-of-life; this repo's current setup; slow rebuilds on large codebases |
| **Vite + Bun** | `bun create vite my-app --template react-ts` | `vite` (esbuild-based) | Rollup (via Vite) | ✅ (fast) | ✅ (native) | Recommended migration path; fast HMR; active maintenance; env vars use `import.meta.env` |
| **Bun Native** | `bun init && bun add react react-dom` | `bun --hot server.ts` | `bun build` | ✅ (via `--hot`) | ✅ (native) | Maximum control; fewest dependencies; less ecosystem tooling; best fit for greenfield projects |

### Expanded notes

**CRA (react-scripts 3.4.3):**
- CRA reached end-of-life. Security patches are no longer being released.
- webpack 4 does not support native ESM and has limited tree-shaking.
- This is the current state of `/workspace/repo`. Level 1 Bun migration (package manager swap) is risk-free.

**Vite + Bun:**
- Vite uses esbuild (written in Go) for dev-time transpilation and Rollup for production bundles, giving it fast cold starts and HMR.
- Running Vite via Bun (`bun run dev`) gives you Bun's fast package resolution for installs while Vite handles the dev server.
- This is the most practical migration target for the existing CRA repo.

**Bun Native:**
- Using only `bun build` and `Bun.serve` eliminates all external tooling.
- `bun build` is competitive with esbuild and significantly faster than webpack.
- Best for new projects where you want a minimal dependency surface.
- Some ecosystem tools (e.g., Storybook, CRA-specific plugins) may require adaptation.

---

## 8. References

- [Bun Documentation](https://bun.sh/docs) — Official Bun runtime docs, getting started, API reference
- [Bun Bundler](https://bun.sh/docs/bundler) — `bun build` CLI, configuration, code splitting, plugins
- [Bun JSX/TSX Support](https://bun.sh/docs/runtime/jsx) — Native JSX transpilation, `bunfig.toml` JSX configuration
- [Vite Guide](https://vitejs.dev/guide/) — Getting started with Vite, framework guides, migration from CRA
- [React Documentation](https://react.dev/) — Official React docs, including the React 18 upgrade guide
