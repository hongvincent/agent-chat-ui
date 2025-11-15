# CLAUDE.md - Agent Chat UI Codebase Guide for AI Assistants

*English | [한국어](./CLAUDE.ko.md)*

This document provides comprehensive guidance for AI assistants working with the Agent Chat UI codebase. It covers architecture, conventions, patterns, and best practices.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Codebase Structure](#codebase-structure)
3. [Development Environment](#development-environment)
4. [Architecture & Patterns](#architecture--patterns)
5. [Coding Conventions](#coding-conventions)
6. [Component Patterns](#component-patterns)
7. [State Management](#state-management)
8. [API Integration](#api-integration)
9. [Styling Approach](#styling-approach)
10. [Common Development Tasks](#common-development-tasks)
11. [Testing & Quality](#testing--quality)
12. [Important Files Reference](#important-files-reference)

---

## Project Overview

**Agent Chat UI** is a production-grade Next.js 15 application that provides a highly customizable chat interface for LangGraph agents. It's a fork of the original LangChain project, enhanced with additional UX/UI features.

### Tech Stack

- **Framework**: Next.js 15.2.3 (App Router)
- **Runtime**: React 19.0.0
- **Language**: TypeScript 5.7.2 (strict mode)
- **Styling**: Tailwind CSS 4.0.13
- **UI Components**: Radix UI primitives (shadcn/ui pattern)
- **Agent Integration**: LangGraph SDK 0.2.63
- **State Management**: React Context + nuqs (URL state)
- **Package Manager**: pnpm 10.5.1
- **Animations**: Framer Motion 12.4.9

### Key Features

- Real-time streaming chat with LangGraph agents
- File upload support (images, PDFs)
- Thread/conversation history management
- Artifact rendering in side panel
- Interrupt handling for agent interactions
- YAML-based runtime configuration
- Dark mode support
- Responsive design (mobile/desktop)
- Markdown rendering with math (KaTeX) and code highlighting

---

## Codebase Structure

### Directory Layout

```
/home/user/agent-chat-ui/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── api/[..._path]/    # Catch-all API proxy route
│   │   ├── layout.tsx         # Root layout with NuqsAdapter
│   │   ├── page.tsx           # Main chat page
│   │   └── globals.css        # Global styles (Tailwind + CSS vars)
│   │
│   ├── components/            # React components
│   │   ├── ui/               # shadcn/ui primitives (22 components)
│   │   │   ├── button.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── input.tsx
│   │   │   └── ...
│   │   │
│   │   ├── thread/           # Chat thread components
│   │   │   ├── index.tsx     # Main Thread orchestrator (655 lines)
│   │   │   ├── artifact.tsx  # Portal-based artifact system
│   │   │   │
│   │   │   ├── messages/     # Message rendering
│   │   │   │   ├── ai.tsx           # AI message display
│   │   │   │   ├── human.tsx        # User message display
│   │   │   │   ├── tool-calls.tsx   # Tool execution display
│   │   │   │   └── shared.tsx       # Shared message components
│   │   │   │
│   │   │   ├── history/      # Thread history sidebar
│   │   │   │   ├── index.tsx
│   │   │   │   ├── ThreadList.tsx
│   │   │   │   ├── DesktopSidebar.tsx
│   │   │   │   ├── MobileSidebar.tsx
│   │   │   │   └── components/thread-item/
│   │   │   │
│   │   │   └── agent-inbox/  # Interrupt handling
│   │   │       └── components/
│   │   │
│   │   ├── settings/         # Settings dialogs
│   │   └── icons/            # Custom SVG icons
│   │
│   ├── hooks/                # Custom React hooks
│   │   ├── useMediaQuery.tsx        # Responsive breakpoints
│   │   └── use-file-upload.tsx      # File upload management (168 lines)
│   │
│   ├── lib/                  # Utilities & API clients
│   │   ├── utils.ts                 # cn() utility for Tailwind
│   │   ├── config.ts                # YAML config loader
│   │   ├── assistant-api.ts         # Assistant API functions
│   │   ├── multimodal-utils.ts      # File/image handling
│   │   ├── constants.ts             # App constants
│   │   └── ...
│   │
│   └── providers/            # React Context providers
│       ├── Stream.tsx        # LangGraph streaming (308 lines)
│       ├── Thread.tsx        # Thread list management
│       ├── Settings.tsx      # App configuration + user settings
│       ├── AssistantConfig.tsx      # Assistant-specific config
│       └── client.ts         # LangGraph client creation
│
├── public/                   # Static assets
│   ├── settings.yaml        # Runtime app configuration ⚠️ IMPORTANT
│   ├── full-description.md  # User guide (shown in landing page)
│   ├── logo.png            # App logo
│   └── ...
│
├── .env.example             # Environment variable template
├── next.config.mjs         # Next.js configuration (10mb body limit)
├── tsconfig.json           # TypeScript config (strict mode, @ alias)
├── tailwind.config.js      # Tailwind configuration
├── eslint.config.js        # ESLint flat config
├── prettier.config.js      # Prettier configuration
├── components.json         # shadcn/ui configuration
└── package.json            # Dependencies and scripts
```

### Path Aliases

The codebase uses a path alias configured in `tsconfig.json`:

```typescript
// Import pattern:
import { Button } from "@/components/ui/button";
import { useSettings } from "@/providers/Settings";
import { cn } from "@/lib/utils";
```

**Always use `@/` imports** instead of relative paths for cleaner, more maintainable code.

---

## Development Environment

### Prerequisites

- **Node.js**: 18.x or higher
- **pnpm**: 10.x (required - do not use npm or yarn)
- **Backend**: LangGraph server running on http://localhost:2024

### Initial Setup

1. **Install dependencies**:
   ```bash
   pnpm install
   ```

2. **Configure environment variables**:
   ```bash
   cp .env.example .env
   ```

   Edit `.env`:
   ```env
   # Local development
   NEXT_PUBLIC_API_URL=http://localhost:2024
   NEXT_PUBLIC_ASSISTANT_ID=agent

   # Server-side only (DO NOT prefix with NEXT_PUBLIC_)
   LANGSMITH_API_KEY=lsv2_...
   ```

3. **Start development server**:
   ```bash
   pnpm dev
   ```

### NPM Scripts

```bash
pnpm dev           # Start Next.js dev server (http://localhost:3000)
pnpm build         # Production build (.next directory)
pnpm start         # Start production server
pnpm lint          # Run ESLint checks
pnpm lint:fix      # Auto-fix ESLint issues
pnpm format        # Format code with Prettier
pnpm format:check  # Check code formatting
```

### Environment Variables

#### Public Variables (Exposed to Browser)
- `NEXT_PUBLIC_API_URL`: API endpoint (local: http://localhost:2024, prod: https://yoursite.com/api)
- `NEXT_PUBLIC_ASSISTANT_ID`: Assistant/graph ID (default: "agent")

#### Private Variables (Server-Side Only)
- `LANGGRAPH_API_URL`: LangGraph deployment URL
- `LANGSMITH_API_KEY`: LangSmith API key (starts with lsv2_)

**CRITICAL**: Never prefix server-only variables with `NEXT_PUBLIC_` - they will be exposed to the client!

---

## Architecture & Patterns

### 1. Next.js App Router Pattern

This project uses the **App Router** (not Pages Router). Key differences:

```typescript
// File-system routing:
src/app/page.tsx          → /
src/app/layout.tsx        → Root layout
src/app/api/[..._path]/route.ts → /api/* (catch-all)
```

- **Server Components by default**: Use `"use client"` directive only when needed
- **No getServerSideProps/getStaticProps**: Use async Server Components instead
- **Layouts**: Shared UI that persists across routes

### 2. API Proxy Pattern

All LangGraph API calls go through a Next.js edge function proxy:

```typescript
// src/app/api/[..._path]/route.ts
export const { GET, POST, PUT, PATCH, DELETE, OPTIONS } =
  initApiPassthrough({
    apiUrl: process.env.LANGGRAPH_API_URL,
    apiKey: process.env.LANGSMITH_API_KEY,
    runtime: "edge",
  });
```

**Purpose**:
- Hide API keys from client
- Enable CORS
- Edge runtime for global performance

**Client-side usage**:
```typescript
const client = new Client({
  apiUrl: NEXT_PUBLIC_API_URL,  // Points to /api, not LangGraph directly
});
```

### 3. Portal-Based Artifact System

The artifact panel uses a **headless portal pattern** to avoid prop drilling:

```typescript
// src/components/thread/artifact.tsx
export function useArtifact() {
  // Returns tuple: [Component, { open, setOpen, title, setTitle }]
}

// Usage in Thread component:
const [ArtifactContent, artifact] = useArtifact();

// Deep in component tree:
<ArtifactContent>
  <CodeEditor code={artifactCode} />
</ArtifactContent>
```

**Benefits**:
- No prop drilling through multiple layers
- Centralized artifact state
- Clean component hierarchy

### 4. Streaming + Optimistic Updates

The Stream provider handles real-time LangGraph streaming:

```typescript
// src/providers/Stream.tsx
const stream = useTypedStream({
  apiUrl, apiKey, assistantId, threadId,
  fetchStateHistory: true,
  onCustomEvent: handleCustomEvent,
});

// Optimistic updates for instant UI feedback:
stream.submit(input, {
  optimisticValues: (prev) => ({
    ...prev,
    messages: [...prev.messages, newMessage],
  }),
});
```

### 5. DO_NOT_RENDER Pattern

Special prefix to hide messages from UI while satisfying LangGraph requirements:

```typescript
// src/lib/constants.ts
export const DO_NOT_RENDER_ID_PREFIX = "do-not-render-";

// Usage:
const toolResponseId = `${DO_NOT_RENDER_ID_PREFIX}${uuid()}`;
```

Components check for this prefix and skip rendering.

### 6. YAML-Driven Configuration

Runtime configuration from `public/settings.yaml` allows customization without rebuilding:

```typescript
// src/lib/config.ts
export async function loadConfig(): Promise<ChatConfig> {
  const response = await fetch("/settings.yaml");
  const yaml = await response.text();
  return parseYaml(yaml);
}
```

**Important**: Changes to `settings.yaml` take effect immediately (hard refresh may be needed to clear cache).

### 7. Drag Counter Pattern

File upload uses a drag counter to prevent flickering when hovering over child elements:

```typescript
// src/hooks/use-file-upload.tsx
const dragCounter = useRef(0);

const handleDragEnter = (e: DragEvent) => {
  e.preventDefault();
  dragCounter.current++;
  if (dragCounter.current === 1) setIsDragging(true);
};

const handleDragLeave = (e: DragEvent) => {
  e.preventDefault();
  dragCounter.current--;
  if (dragCounter.current === 0) setIsDragging(false);
};
```

### 8. Responsive Design with Media Queries

Single source of truth for breakpoints:

```typescript
// src/hooks/useMediaQuery.tsx
export function useMediaQuery(query: string): boolean

// Usage:
const isLargeScreen = useMediaQuery("(min-width: 1024px)");
```

---

## Coding Conventions

### TypeScript Standards

1. **Strict Mode**: Always enabled (`"strict": true` in tsconfig.json)
   - No implicit any
   - Strict null checks
   - Strict function types

2. **Explicit Types**: Prefer explicit return types for exported functions
   ```typescript
   // Good
   export function getAssistant(id: string): Promise<Assistant> {
     // ...
   }

   // Avoid
   export function getAssistant(id: string) {
     // ...
   }
   ```

3. **Type Guards**: Use for runtime validation
   ```typescript
   export function isBase64ContentBlock(block: unknown): block is Base64ContentBlock {
     return (
       typeof block === "object" &&
       block !== null &&
       "type" in block &&
       block.type === "image"
     );
   }
   ```

4. **Interface vs Type**:
   - Use `interface` for object shapes (extendable)
   - Use `type` for unions, intersections, primitives

5. **No `any`**: Allowed by ESLint config but discouraged - use `unknown` instead

### React Conventions

1. **Client Components**: Use `"use client"` directive at top of file when needed
   ```typescript
   "use client";

   import { useState } from "react";
   ```

2. **Component Structure**:
   ```typescript
   // 1. Imports (React, external, internal, types)
   // 2. Type definitions
   // 3. Component definition
   // 4. Exports

   "use client";

   import React from "react";
   import { Button } from "@/components/ui/button";
   import type { Message } from "@langchain/langgraph-sdk";

   interface Props {
     message: Message;
   }

   export function MessageComponent({ message }: Props) {
     // Hooks first
     const [expanded, setExpanded] = useState(false);

     // Event handlers
     const handleClick = () => setExpanded(!expanded);

     // Render
     return <div onClick={handleClick}>{/* ... */}</div>;
   }
   ```

3. **Hooks Order**:
   - State hooks (`useState`, `useReducer`)
   - Context hooks (`useContext`, custom context hooks)
   - Ref hooks (`useRef`)
   - Effect hooks (`useEffect`, `useLayoutEffect`)
   - Custom hooks
   - Derived values (useMemo, useCallback)

4. **Props Naming**:
   - Event handlers: `on[Event]` (e.g., `onClick`, `onSubmit`)
   - Boolean props: `is[State]`, `has[State]`, `enable[Feature]`
   - Render props: `render[Element]` or just element name

### ESLint Configuration

```javascript
// eslint.config.js (flat config format)
rules: {
  ...reactHooks.configs.recommended.rules,
  "@typescript-eslint/no-explicit-any": 0,  // Warning (not error)
  "@typescript-eslint/no-unused-vars": ["warn", {
    args: "none",
    argsIgnorePattern: "^_",
    varsIgnorePattern: "^_"
  }],
  "react-refresh/only-export-components": ["warn", {
    allowConstantExport: true
  }],
}
```

**Key points**:
- Unused variables starting with `_` are ignored
- React Hooks rules are enforced
- Components must be exported for Fast Refresh

### Prettier Configuration

```javascript
// prettier.config.js
{
  endOfLine: "auto",
  singleAttributePerLine: true,  // Each JSX prop on own line
  plugins: ["prettier-plugin-tailwindcss"],  // Auto-sort Tailwind classes
}
```

**Always run** `pnpm format` before committing code.

### File Naming

- **Components**: PascalCase (`ThreadList.tsx`, `ArtifactPanel.tsx`)
- **Hooks**: camelCase with `use` prefix (`useMediaQuery.tsx`, `use-file-upload.tsx`)
- **Utils/Libs**: kebab-case (`multimodal-utils.ts`, `file-validation.ts`)
- **Types**: PascalCase (`Message.ts`, `AssistantConfig.ts`)
- **Constants**: kebab-case (`constants.ts`)

### Imports Organization

```typescript
// 1. React imports
import React, { useState, useEffect } from "react";

// 2. External libraries (alphabetical)
import { Button } from "@/components/ui/button";
import { motion } from "framer-motion";
import { Loader2 } from "lucide-react";

// 3. Internal imports (@ alias, alphabetical)
import { useSettings } from "@/providers/Settings";
import { cn } from "@/lib/utils";

// 4. Types (separate from values)
import type { Message } from "@langchain/langgraph-sdk";
import type { ChatConfig } from "@/lib/config";
```

---

## Component Patterns

### shadcn/ui Pattern

All UI components follow the shadcn/ui pattern:

```typescript
// src/components/ui/button.tsx
import * as React from "react";
import { Slot } from "@radix-ui/react-slot";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const buttonVariants = cva(
  "inline-flex items-center justify-center...",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground...",
        destructive: "bg-destructive text-destructive-foreground...",
        outline: "border border-input...",
        // ...
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button";
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);
Button.displayName = "Button";

export { Button, buttonVariants };
```

**Key characteristics**:
- Uses `class-variance-authority` (cva) for variant management
- Radix UI primitives for accessibility
- Forward refs for parent component access
- `asChild` prop for composition
- `cn()` utility for class merging

### Adding New UI Components

Use the shadcn/ui CLI (if available) or manually create following the pattern:

```bash
# If shadcn CLI is available:
npx shadcn-ui@latest add [component-name]

# Components are configured to use:
# - Style: New York
# - Color: Neutral
# - Tailwind config: tailwind.config.js
# - CSS variables: src/app/globals.css
```

### Custom Component Example

```typescript
// src/components/thread/MessageBubble.tsx
"use client";

import React from "react";
import { motion } from "framer-motion";
import { cn } from "@/lib/utils";
import type { Message } from "@langchain/langgraph-sdk";

interface MessageBubbleProps {
  message: Message;
  isUser: boolean;
  className?: string;
}

export function MessageBubble({
  message,
  isUser,
  className
}: MessageBubbleProps) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.3 }}
      className={cn(
        "rounded-lg p-4",
        isUser ? "bg-primary text-primary-foreground" : "bg-muted",
        className
      )}
    >
      {message.content}
    </motion.div>
  );
}
```

**Pattern notes**:
- Props interface defined before component
- `className` prop for customization
- `cn()` for conditional classes
- Motion for animations
- Semantic HTML

---

## State Management

### Provider Architecture

The app uses React Context providers for different concerns:

#### 1. Stream Provider (LangGraph Integration)

```typescript
// src/providers/Stream.tsx
export function StreamProvider({ children }: { children: ReactNode }) {
  const stream = useTypedStream({
    apiUrl, apiKey, assistantId, threadId,
    fetchStateHistory: true,
    onCustomEvent: (event) => {
      // Handle custom events (title generation, etc.)
    },
  });

  return (
    <StreamContext.Provider value={stream}>
      {children}
    </StreamContext.Provider>
  );
}

export function useStream() {
  return useContext(StreamContext);
}
```

**Usage**:
```typescript
const stream = useStream();
stream.submit(input);
stream.regenerate();
stream.setBranch(branch);
```

#### 2. Settings Provider (Configuration + User Prefs)

```typescript
// src/providers/Settings.tsx
export function SettingsProvider({ children }: { children: ReactNode }) {
  const [config, setConfig] = useState<ChatConfig | null>(null);
  const [userSettings, setUserSettings] = useLocalStorage("userSettings");

  return (
    <SettingsContext.Provider value={{
      config,
      userSettings,
      updateUserSettings,
      resetUserSettings
    }}>
      {children}
    </SettingsContext.Provider>
  );
}

export function useSettings() {
  return useContext(SettingsContext);
}
```

**Two-layer settings**:
- `config`: YAML-based app configuration (read-only from `settings.yaml`)
- `userSettings`: User preferences (stored in localStorage)

#### 3. Thread Provider (Conversation Management)

```typescript
// src/providers/Thread.tsx
export function ThreadProvider({ children }: { children: ReactNode }) {
  const [threads, setThreads] = useState<Thread[]>([]);

  const getThreads = useCallback(async () => {
    const result = await client.threads.list();
    setThreads(result);
  }, [client]);

  return (
    <ThreadContext.Provider value={{ threads, getThreads, setThreads }}>
      {children}
    </ThreadContext.Provider>
  );
}
```

#### 4. Assistant Config Provider

```typescript
// src/providers/AssistantConfig.tsx
export function AssistantConfigProvider({ children }: { children: ReactNode }) {
  const [config, setConfig] = useState<Assistant | null>(null);
  const [schemas, setSchemas] = useState<AssistantSchemas | null>(null);

  return (
    <AssistantConfigContext.Provider value={{
      config,
      schemas,
      updateConfig,
      refetchConfig
    }}>
      {children}
    </AssistantConfigContext.Provider>
  );
}
```

### URL State Management (nuqs)

For shareable/bookmarkable state, use nuqs:

```typescript
import { useQueryState, parseAsBoolean } from "nuqs";

// Simple string state
const [threadId, setThreadId] = useQueryState("threadId");

// Parsed state with default
const [hideToolCalls, setHideToolCalls] = useQueryState(
  "hideToolCalls",
  parseAsBoolean.withDefault(false)
);

// Update URL state
setThreadId("thread-123");  // URL: ?threadId=thread-123
setHideToolCalls(true);     // URL: ?threadId=thread-123&hideToolCalls=true
```

**NuqsAdapter required** in root layout:

```typescript
// src/app/layout.tsx
import { NuqsAdapter } from "nuqs/adapters/next/app";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <NuqsAdapter>{children}</NuqsAdapter>
      </body>
    </html>
  );
}
```

### Local Storage

For persistent user preferences:

```typescript
// Pattern: Check localStorage, fallback to default
const savedSettings = localStorage.getItem("userSettings");
const userSettings = savedSettings ? JSON.parse(savedSettings) : defaultSettings;

// Update
localStorage.setItem("userSettings", JSON.stringify(newSettings));
```

**Used for**:
- User theme preferences
- Font size/family overrides
- UI preferences (sidebar state, etc.)

---

## API Integration

### LangGraph Client Setup

```typescript
// src/providers/client.ts
import { Client } from "@langchain/langgraph-sdk";

export function createClient(apiUrl: string, apiKey?: string) {
  return new Client({ apiKey, apiUrl });
}

// Usage
const client = createClient(
  process.env.NEXT_PUBLIC_API_URL!,
  process.env.LANGSMITH_API_KEY
);
```

### Assistant API Functions

```typescript
// src/lib/assistant-api.ts

// Get assistant by ID or graph_id
export async function getAssistant(
  client: Client,
  assistantId: string
): Promise<Assistant> {
  // Try as assistant_id first
  const assistant = await client.assistants.get(assistantId);
  if (assistant) return assistant;

  // Fall back to graph_id search
  return await searchAssistants(client, assistantId);
}

// Update assistant configuration
export async function updateAssistantConfig(
  client: Client,
  assistantId: string,
  config: Partial<AssistantConfig>
): Promise<Assistant> {
  return await client.assistants.update(assistantId, { config });
}

// Get input/output schemas
export async function getAssistantSchemas(
  client: Client,
  graphId: string
): Promise<AssistantSchemas> {
  return await client.assistants.getSchemas(graphId);
}
```

### Thread Operations

```typescript
// Create new thread
const thread = await client.threads.create();

// Get thread state
const state = await client.threads.getState(threadId);

// List threads
const threads = await client.threads.list({ limit: 50 });

// Update thread metadata
await client.threads.update(threadId, {
  metadata: { title: "New Title" }
});

// Delete thread
await client.threads.delete(threadId);
```

### Streaming Messages

```typescript
// src/providers/Stream.tsx
const stream = useTypedStream({
  apiUrl,
  apiKey,
  assistantId,
  threadId,
  fetchStateHistory: true,

  // Handle custom events (e.g., title generation)
  onCustomEvent: (event) => {
    if (event.event === "title_generated") {
      setTitle(event.data.title);
    }
  },
});

// Submit message
stream.submit({
  messages: [{ role: "user", content: "Hello" }]
}, {
  // Optimistic update for instant UI feedback
  optimisticValues: (prev) => ({
    ...prev,
    messages: [...prev.messages, newMessage],
  }),
});
```

### Error Handling

```typescript
import { toast } from "sonner";

try {
  const result = await client.threads.create();
} catch (error) {
  console.error("Failed to create thread:", error);
  toast.error("Failed to create conversation", {
    description: error instanceof Error ? error.message : "Unknown error",
  });
}
```

---

## Styling Approach

### Tailwind CSS 4.x

This project uses **Tailwind CSS 4.0**, which has a different architecture than v3:

```css
/* src/app/globals.css */
@import "tailwindcss";

/* CSS variables for theming */
:root {
  --background: 0 0% 100%;
  --foreground: 0 0% 3.9%;
  --primary: 0 0% 9%;
  --primary-foreground: 0 0% 98%;
  /* ... */
}

.dark {
  --background: 0 0% 3.9%;
  --foreground: 0 0% 98%;
  /* ... */
}
```

### Design System

**Color Palette**:
- Uses `oklch()` color space for perceptually uniform colors
- Monochrome neutral palette (inspired by Claude.ai/ChatGPT)
- Semantic naming: `background`, `foreground`, `muted`, `accent`, `primary`, `destructive`

**Theme Variables**:
```css
/* Light mode */
--background: oklch(99% 0.01 297);
--foreground: oklch(31% 0.01 297);

/* Dark mode */
.dark {
  --background: oklch(22% 0.01 297);
  --foreground: oklch(96% 0.01 297);
}
```

### Utility Classes

```typescript
// src/lib/utils.ts
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

**Usage**:
```typescript
<div className={cn(
  "base-classes",
  condition && "conditional-classes",
  anotherCondition ? "true-classes" : "false-classes",
  className  // Allow parent override
)} />
```

### Custom Utilities

```css
/* src/app/globals.css */
.scrollbar-pretty {
  scrollbar-width: thin;
  scrollbar-color: hsl(var(--border)) transparent;
}

.shadow-inner-right {
  box-shadow: inset -10px 0 10px -10px rgba(0, 0, 0, 0.1);
}
```

### Responsive Design

Tailwind breakpoints:
- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px
- `2xl`: 1536px

**Usage**:
```typescript
<div className="px-4 md:px-6 lg:px-8">
  <h1 className="text-2xl md:text-3xl lg:text-4xl">Title</h1>
</div>
```

### Dark Mode

```typescript
// Uses next-themes for dark mode
import { ThemeProvider } from "next-themes";

<ThemeProvider attribute="class" defaultTheme="system">
  {children}
</ThemeProvider>

// Toggle theme
import { useTheme } from "next-themes";

const { theme, setTheme } = useTheme();
setTheme("dark");  // "light" | "dark" | "system"
```

### Animation

**Framer Motion** for complex animations:

```typescript
import { motion } from "framer-motion";

<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0, y: -20 }}
  transition={{ duration: 0.3 }}
>
  Content
</motion.div>
```

**Tailwind Animate** for simple transitions:

```typescript
<div className="transition-all duration-300 hover:scale-105">
  Hover me
</div>
```

---

## Common Development Tasks

### Adding a New Component

1. **Create component file**:
   ```bash
   # UI primitive (shadcn/ui pattern)
   touch src/components/ui/new-component.tsx

   # Feature component
   touch src/components/thread/NewFeature.tsx
   ```

2. **Follow component pattern**:
   ```typescript
   "use client";  // If using hooks/state

   import React from "react";
   import { cn } from "@/lib/utils";

   interface NewComponentProps {
     // Props definition
   }

   export function NewComponent({ }: NewComponentProps) {
     return <div>Component</div>;
   }
   ```

3. **Export from index** (if creating a module):
   ```typescript
   // src/components/ui/index.ts
   export { Button } from "./button";
   export { NewComponent } from "./new-component";
   ```

### Adding a New Hook

1. **Create hook file**:
   ```bash
   touch src/hooks/use-new-feature.tsx
   ```

2. **Follow hook pattern**:
   ```typescript
   import { useState, useEffect } from "react";

   export function useNewFeature(param: string) {
     const [state, setState] = useState<Type>(initialValue);

     useEffect(() => {
       // Effect logic
     }, [param]);

     return { state, setState };
   }
   ```

### Updating Configuration

**App Configuration** (`public/settings.yaml`):

```yaml
branding:
  appName: "My Chat App"
  logoPath: "/logo.png"
  description: "Ask me anything"
  chatOpeners:
    - "What's the weather?"
    - "Tell me a joke"

buttons:
  enableFileUpload: true
  chatInputPlaceholder: "Type a message..."

threads:
  showHistory: true
  enableDeletion: true

theme:
  fontFamily: "sans"  # sans | serif | mono
  fontSize: "medium"   # small | medium | large
  colorScheme: "light" # light | dark | auto

ui:
  autoCollapseToolCalls: true
  chatWidth: "wide"    # default | wide
```

**Changes take effect immediately** (hard refresh browser cache).

### Adding File Upload Support

File upload is handled by `use-file-upload.tsx`:

```typescript
const {
  files,
  isDragging,
  addFiles,
  removeFile,
  clearFiles,
  handleDrop,
  handleDragEnter,
  handleDragLeave,
  handleDragOver,
  handlePaste,
} = useFileUpload();

// Supported formats: JPEG, PNG, GIF, WebP, PDF
// Validation in src/lib/file-validation.ts
```

### Adding Custom Events

```typescript
// Backend sends custom event:
{
  "event": "custom_event_name",
  "data": { "key": "value" }
}

// Frontend handles in Stream provider:
const stream = useTypedStream({
  // ...
  onCustomEvent: (event) => {
    if (event.event === "custom_event_name") {
      // Handle event
      console.log(event.data);
    }
  },
});
```

### Modifying Message Rendering

```typescript
// src/components/thread/messages/ai.tsx
export function AIMessage({ message }: { message: Message }) {
  // Add custom rendering logic

  if (message.metadata?.customType) {
    return <CustomRenderer data={message.metadata} />;
  }

  // Default rendering
  return <DefaultRenderer content={message.content} />;
}
```

### Adding Artifact Types

```typescript
// Artifacts are rendered in side panel
const [ArtifactContent, artifact] = useArtifact();

// Set artifact content
artifact.setTitle("Code Preview");
artifact.setOpen(true);

<ArtifactContent>
  <CodeEditor code={code} language="typescript" />
</ArtifactContent>
```

---

## Testing & Quality

### Linting

```bash
# Check for issues
pnpm lint

# Auto-fix issues
pnpm lint:fix
```

**ESLint rules**:
- React Hooks rules (enforced)
- TypeScript strict mode
- Unused variables warning (ignored if prefixed with `_`)
- React Refresh component exports

### Formatting

```bash
# Format all files
pnpm format

# Check formatting without writing
pnpm format:check
```

**Prettier rules**:
- End of line: auto
- Single attribute per line (JSX)
- Tailwind classes auto-sorted

### Type Checking

```bash
# Check types (no build)
npx tsc --noEmit

# Production build (includes type checking)
pnpm build
```

### Pre-Commit Checklist

Before committing code:

1. ✅ Run `pnpm lint:fix`
2. ✅ Run `pnpm format`
3. ✅ Test in browser (dev mode)
4. ✅ Check TypeScript errors (`npx tsc --noEmit`)
5. ✅ Ensure no console errors
6. ✅ Test responsive design (mobile/desktop)
7. ✅ Test dark mode if UI changes

### Common Issues

**Issue**: TypeScript errors after adding dependency
**Solution**: Restart TypeScript server in IDE

**Issue**: Tailwind classes not working
**Solution**: Check `tailwind.config.js` content paths, restart dev server

**Issue**: Environment variable not available
**Solution**: Prefix with `NEXT_PUBLIC_` for client access, restart dev server

**Issue**: File upload not working
**Solution**: Check `next.config.mjs` has `bodySizeLimit: "10mb"`

**Issue**: Settings changes not reflected
**Solution**: Hard refresh browser (Cmd+Shift+R / Ctrl+Shift+R)

---

## Important Files Reference

### Configuration Files

| File | Purpose |
|------|---------|
| `next.config.mjs` | Next.js configuration (body size limit, experimental features) |
| `tsconfig.json` | TypeScript configuration (strict mode, path aliases) |
| `tailwind.config.js` | Tailwind CSS configuration (theme, plugins) |
| `eslint.config.js` | ESLint rules (flat config format) |
| `prettier.config.js` | Prettier formatting rules |
| `components.json` | shadcn/ui configuration |
| `.env.example` | Environment variable template |
| `public/settings.yaml` | **Runtime app configuration** ⚠️ |

### Core Application Files

| File | Purpose | Lines |
|------|---------|-------|
| `src/app/layout.tsx` | Root layout with NuqsAdapter | 24 |
| `src/app/page.tsx` | Main chat page | - |
| `src/app/globals.css` | Global styles + CSS variables | 171 |
| `src/app/api/[..._path]/route.ts` | API proxy to LangGraph | - |

### Provider Files

| File | Purpose | Lines |
|------|---------|-------|
| `src/providers/Stream.tsx` | LangGraph streaming + optimistic updates | 308 |
| `src/providers/Settings.tsx` | App config + user preferences | 150 |
| `src/providers/Thread.tsx` | Thread list management | 80 |
| `src/providers/AssistantConfig.tsx` | Assistant-specific configuration | 167 |
| `src/providers/client.ts` | LangGraph client creation | - |

### Component Files

| File | Purpose | Lines |
|------|---------|-------|
| `src/components/thread/index.tsx` | Main Thread orchestrator | 655 |
| `src/components/thread/artifact.tsx` | Portal-based artifact system | 190 |
| `src/components/thread/messages/ai.tsx` | AI message rendering | - |
| `src/components/thread/messages/human.tsx` | User message rendering | - |
| `src/components/thread/messages/tool-calls.tsx` | Tool execution display | - |
| `src/components/thread/history/` | Thread history sidebar | - |
| `src/components/thread/agent-inbox/` | Interrupt handling | - |

### Utility Files

| File | Purpose |
|------|---------|
| `src/lib/utils.ts` | `cn()` utility for Tailwind class merging |
| `src/lib/config.ts` | YAML configuration loader |
| `src/lib/assistant-api.ts` | Assistant API helper functions |
| `src/lib/multimodal-utils.ts` | File/image handling utilities |
| `src/lib/constants.ts` | App constants (DO_NOT_RENDER prefix, etc.) |
| `src/lib/file-validation.ts` | File upload validation |

### Hook Files

| File | Purpose | Lines |
|------|---------|-------|
| `src/hooks/useMediaQuery.tsx` | Responsive breakpoint detection | - |
| `src/hooks/use-file-upload.tsx` | File upload management (drag/drop/paste) | 168 |

---

## Best Practices for AI Assistants

### When Making Changes

1. **Read before writing**: Always read the full file before editing
2. **Preserve patterns**: Follow existing patterns in the file
3. **Use path aliases**: Always use `@/` imports, never relative paths
4. **Type safety**: Add proper TypeScript types for new code
5. **Format code**: Run `pnpm format` after making changes
6. **Test thoroughly**: Check in browser, test responsive design, verify dark mode

### When Adding Features

1. **Check existing patterns**: Look for similar features first
2. **Use existing components**: Leverage shadcn/ui components
3. **Follow provider pattern**: Add state to appropriate provider
4. **Update settings.yaml**: If feature needs configuration
5. **Handle errors gracefully**: Use toast notifications for user feedback
6. **Consider responsive design**: Test on mobile and desktop

### When Debugging

1. **Check console**: Look for TypeScript/ESLint errors
2. **Verify environment variables**: Ensure proper prefixes
3. **Check provider hierarchy**: Ensure component is wrapped in needed providers
4. **Inspect network**: Check API proxy is working
5. **Test with hard refresh**: Clear browser cache for config changes

### Common Gotchas

- **Environment variables**: Must restart dev server after changes
- **Public variables**: Must prefix with `NEXT_PUBLIC_` for client access
- **Settings.yaml**: Changes need hard refresh to clear cache
- **Tailwind classes**: Must be complete strings (no string concatenation)
- **Next.js caching**: Sometimes need to delete `.next` directory
- **File uploads**: Limited to 10MB (configured in `next.config.mjs`)

---

## Additional Resources

- **README.md**: User-facing documentation (Korean)
- **public/full-description.md**: End-user guide
- **LangGraph SDK Docs**: https://langchain-ai.github.io/langgraph/
- **Next.js Docs**: https://nextjs.org/docs
- **Tailwind CSS Docs**: https://tailwindcss.com/docs
- **shadcn/ui Docs**: https://ui.shadcn.com/
- **Radix UI Docs**: https://www.radix-ui.com/

---

**Last Updated**: 2025-11-15
**Project Version**: 0.0.0
**Next.js Version**: 15.2.3
**React Version**: 19.0.0
