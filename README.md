<div align="center">

# ⚡ Synax

**The semantic bridge between AI agents and accessible React experiences.**

An open-source Server-Driven UI engine that turns agent-generated JSON into accessible, design-system-agnostic interfaces.

[![CI](https://img.shields.io/github/actions/workflow/status/your-username/synax/ci.yml?style=flat-square&label=tests)](https://github.com/your-username/synax/actions)
[![npm](https://img.shields.io/npm/v/@synax/core?style=flat-square&color=cb3837)](https://www.npmjs.com/package/@synax/core)
[![TypeScript](https://img.shields.io/badge/TypeScript-strict-007ACC?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![React](https://img.shields.io/badge/React-18%2B-20232A?style=flat-square&logo=react&logoColor=61DAFB)](https://react.dev/)
[![a11y](https://img.shields.io/badge/a11y-axe--core%20tested-brightgreen?style=flat-square)](#-accessibility)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](./LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-ff69b4?style=flat-square)](#-contributing)

<br />

<!-- TODO: Replace with a short GIF/video (~800x400) showing a split screen:
     the agent emitting JSON on one side, the UI rendering on the other,
     then switching from Material UI to Radix/Tailwind instantly. -->
![Synax demo](https://via.placeholder.com/800x400.png?text=Synax+Interactive+Demo+Animation)

[Playground (coming soon)]() · [Documentation]() · [MCP Guides]() · [Report a Bug](https://github.com/your-username/synax/issues)

</div>

---

## Table of Contents

- [Why Synax?](#-why-synax)
- [Features](#-features)
- [How It Works](#-how-it-works)
- [Quick Start](#-quick-start)
- [The Intent Contract](#-the-intent-contract)
- [Packages](#-packages)
- [Accessibility](#-accessibility)
- [Repository Structure](#-repository-structure)
- [Contributing](#-contributing)
- [Roadmap](#-roadmap)
- [License](#-license)

---

## 🧠 Why Synax?

**Intent over implementation.**

In AI-first apps, LLMs are often asked to generate raw frontend code or heavy styling markup. The result: inconsistent interfaces, broken design-system rules, and little to no accessibility.

Synax fixes this with a **strict intent contract**.

The agent never says a button should be *blue* with *16px padding*. It only states a semantic intent:

```json
{ "type": "Button", "intent": "primary" }
```

Synax is the rendering middleware that reads that contract and resolves each node to the right component from **your own design system**. The agent decides *what* the user needs; your design system decides *how* it looks.

It fits naturally into agent ecosystems built on the **Model Context Protocol (MCP)**, LangChain, or any dynamic dashboard where the UI is decided at runtime.

## ✨ Features

- 🛡️ **Strict AI contracts (Zod)** — `@synax/core` ships strict schemas. If an agent hallucinates visual props that don't exist, the validator intercepts them and falls back gracefully instead of crashing your UI.
- 🎨 **Agnostic rendering (Inversion of Control)** — The engine has zero built-in styles. Bring your own components (Material UI, Radix, Chakra, Tailwind…) through *adapter* packages.
- ♿ **Accessibility by default** — End-to-end tests (Playwright + axe-core) check that generated UI stays within WCAG guidelines.
- ⚡ **SSR & lazy loading ready** — Works with `next/dynamic` or `React.lazy`. If the payload doesn't ask for a `DataGrid`, its JavaScript chunk is never sent to the client.
- 🔌 **MCP-friendly** — A small, well-typed JSON surface that is easy for agents and MCP servers to produce.

## 🏗️ How It Works

<!-- TODO: Replace with a clean block diagram of the flow below. -->
![Synax architecture](https://via.placeholder.com/800x250.png?text=LLM/Agent+%E2%86%92+JSON+Schema+%E2%86%92+Synax+Core+(Validator)+%E2%86%92+Adapter+(Registry)+%E2%86%92+React+UI)

```
Agent / MCP server ──▶ JSON payload ──▶ @synax/core (validate) ──▶ Adapter registry ──▶ React UI
```

1. **Agent / MCP server** emits a structured JSON payload.
2. **`@synax/core`** validates the tree against the intent dictionary.
3. **`SduiProvider`** uses dependency injection to look up the concrete components in your adapter registry.
4. **`SduiRenderer`** recursively builds the React tree, lazy-loading components as needed.

## 🚀 Quick Start

### 1. Install

Synax is modular: install the core plus the adapter for your UI ecosystem.

```bash
# Core engine (React + Zod only, no UI dependencies)
pnpm add @synax/core

# An adapter, e.g. Radix UI
pnpm add @synax/adapter-radix
```

<details>
<summary>npm / yarn</summary>

```bash
npm install @synax/core @synax/adapter-radix
# or
yarn add @synax/core @synax/adapter-radix
```

</details>

### 2. Get a payload from your agent

This is the contract the AI produces:

```json
{
  "type": "Card",
  "props": { "title": "Delete User" },
  "children": [
    { "type": "Text", "content": "This action cannot be undone." },
    { "type": "Button", "intent": "destructive", "actionId": "delete_user_123" }
  ]
}
```

### 3. Render it

```tsx
import { SduiProvider, SduiRenderer } from '@synax/core';
import { radixAdapterRegistry } from '@synax/adapter-radix'; // or your own custom registry

// `schema` is the JSON payload coming from your server / MCP agent
export default function AgentView({ schema }) {
  return (
    // Inversion of Control in action:
    // swap in `muiAdapterRegistry` and the whole UI changes instantly.
    <SduiProvider registry={radixAdapterRegistry}>
      <SduiRenderer payload={schema} />
    </SduiProvider>
  );
}
```

That's it. The same payload now renders with Radix. Swap the registry and it renders with Material UI, with no changes on the agent side.

## 📜 The Intent Contract

Payloads are trees of nodes. Each node describes **what** it is, never **how** it looks:

| Field      | Purpose                                                                      |
| ---------- | ---------------------------------------------------------------------------- |
| `type`     | The semantic component (`Card`, `Text`, `Button`, …).                        |
| `intent`   | The role of the element (`primary`, `destructive`, …), not its appearance.   |
| `props`    | Validated, component-specific properties.                                    |
| `content`  | Text content of the node.                                                    |
| `children` | Nested nodes.                                                                |
| `actionId` | An opaque identifier that your app maps to a real handler.                   |

Anything outside the schema (raw CSS, arbitrary class names, unknown component types) is rejected by the Zod validator and replaced with a safe fallback.

> 💡 The agent never ships executable code or styles, only data your app already knows how to interpret.

## 📦 Packages

| Package                                        | Description                                                            |
| ---------------------------------------------- | ---------------------------------------------------------------------- |
| [`@synax/core`](./packages/core)               | Zod schemas, TypeScript types, provider and recursive renderer.        |
| [`@synax/adapter-radix`](./packages/adapter-radix) | Maps the core contract to Radix Primitives + Tailwind.             |
| [`@synax/adapter-mui`](./packages/adapter-mui) | Maps the core contract to Material UI components.                      |

Want a different design system? See [Contributing](#-contributing): adapters are the easiest way to get involved.

## ♿ Accessibility

Accessibility isn't an afterthought; it's part of the test suite. The `e2e/` workspace runs **Playwright + axe-core** against the demo app for every adapter, so a change that makes AI-generated UI inaccessible fails CI.

## 📂 Repository Structure

The repo uses **Turborepo** and **pnpm workspaces** so adapters can scale independently:

```
synax/
├── apps/
│   ├── demo/              # Next.js split-screen playground showcasing the engine
│   └── docs/              # Official docs and MCP guides (Nextra)
├── packages/
│   ├── core/              # Zod validator, TypeScript types, recursive renderer
│   ├── adapter-mui/       # Core → Material UI mapping
│   └── adapter-radix/     # Core → Radix Primitives + Tailwind mapping
└── e2e/                   # Automated tests (Playwright + axe-core)
```

## 🤝 Contributing

Contributions are very welcome. We'd love to grow the adapter ecosystem (Chakra UI, Ant Design, and more), and we're equally happy with bug reports, docs improvements, and new ideas.

```bash
# Clone and install
git clone https://github.com/your-username/synax.git
cd synax
pnpm install

# Run the demo playground
pnpm --filter demo dev

# Run tests (unit + accessibility)
pnpm test
```

Please read the [Contributing Guide](./CONTRIBUTING.md) before opening a pull request, and check the [open issues](https://github.com/your-username/synax/issues) for a good place to start.

## 🗺️ Roadmap

- [x] Core validator and recursive renderer
- [x] Radix + Tailwind adapter
- [x] Material UI adapter
- [ ] Chakra UI adapter
- [ ] Ant Design adapter
- [ ] Interactive online playground
- [ ] Complete MCP integration guides

## 📄 License

Distributed under the [MIT License](./LICENSE).

<div align="center">

Built with ❤️ and ☕ by the Synax contributors.

</div>
