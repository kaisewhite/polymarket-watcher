# Polymarket UI Interface Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a real-time dashboard UI for the Polymarket Watcher project using TanStack Start to visualize watched markets and incoming AI trading signals.

**Architecture:** A standalone TanStack Start application living in a new `dashboard/` directory. It will connect to the existing Redis instance (populated by the `application/` service) to load active markets and historical signals, while using an API route with Server-Sent Events (SSE) to push real-time anomaly detections to the frontend.

**Tech Stack:** Bun, TypeScript, TanStack Start (React Router 7), Tailwind CSS, `ioredis` (for data access).

---

### Task 1: Initialize TanStack Start Project

**Files:**
- Create: `dashboard/` (via CLI)

- [ ] **Step 1: Run the initialization command**
```bash
cd /Volumes/Samsung/repositories/kaisewhite/polymarket-watcher
bunx @tanstack/create-start@latest dashboard
```
*(Note: Since this is interactive, if running via agent, we may need to scaffold the minimum files manually or accept defaults if supported. For this plan, we assume standard scaffolding with Tailwind and TypeScript).*

- [ ] **Step 2: Install dependencies & shadcn/ui**
```bash
cd dashboard
bun install
bun add ioredis
bun add -D tailwindcss postcss autoprefixer
bunx tailwindcss init -p
bunx shadcn@latest init -d
```

- [ ] **Step 3: Add core shadcn components**
```bash
bunx shadcn@latest add card badge button skeleton scroll-area
```

- [ ] **Step 4: Configure Tailwind**
Modify `dashboard/tailwind.config.js`:
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./app/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```
Modify `dashboard/app/app.css` to include Tailwind directives:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

- [ ] **Step 5: Verify it runs**
Run `bun run dev` inside `dashboard/` and ensure it starts on port 3000 without errors.

- [ ] **Step 6: Commit**
```bash
git add dashboard/
git commit -m "chore: initialize TanStack Start dashboard with Tailwind, shadcn/ui, and ioredis"
```

---

### Task 2: Configure Redis Data Access Layer

**Files:**
- Create: `dashboard/app/utils/redis.ts`
- Create: `dashboard/.env`

- [ ] **Step 1: Add environment variables**
Create `dashboard/.env`:
```env
REDIS_URL=redis://localhost:6379
```

- [ ] **Step 2: Create Redis client utility**
Create `dashboard/app/utils/redis.ts`:
```typescript
import Redis from 'ioredis';

const redisUrl = process.env.REDIS_URL || 'redis://localhost:6379';
export const redis = new Redis(redisUrl);

export async function getActiveMarkets() {
  // Assuming keys are stored like 'market:*' or tracked in a set
  // This will need to map to the exact keys used by the watcher application
  const keys = await redis.keys('market:*');
  if (keys.length === 0) return [];
  
  const pipeline = redis.pipeline();
  keys.forEach(key => pipeline.get(key));
  const results = await pipeline.exec();
  
  return results?.map(([err, val]) => val ? JSON.parse(val as string) : null).filter(Boolean) || [];
}

export async function getRecentSignals() {
  // Fetch recent AI signals (assuming a list or zset like 'signals:recent')
  const signals = await redis.lrange('signals:recent', 0, 50);
  return signals.map(s => JSON.parse(s));
}
```

- [ ] **Step 3: Test Redis connection**
Create a quick test script `dashboard/test-redis.ts`:
```typescript
import { redis } from './app/utils/redis.ts';
async function test() {
  console.log(await redis.ping());
  process.exit(0);
}
test();
```
Run: `bun run test-redis.ts`
Expected: Outputs "PONG"

- [ ] **Step 4: Commit**
```bash
git add dashboard/app/utils/redis.ts dashboard/.env
git commit -m "feat: add Redis data access layer for reading markets and signals"
```

---

### Task 3: Build the Dashboard UI Layout

**Files:**
- Modify: `dashboard/app/routes/__root.tsx`
- Modify: `dashboard/app/routes/index.tsx`

- [ ] **Step 1: Update Root Layout**
Update `dashboard/app/routes/__root.tsx` to include a persistent sidebar/header for navigation.
```tsx
import { Outlet, createRootRoute } from '@tanstack/react-router'
import { Meta, Scripts } from '@tanstack/start'

export const Route = createRootRoute({
  component: RootComponent,
})

function RootComponent() {
  return (
    <html>
      <head>
        <Meta />
      </head>
      <body className="bg-gray-900 text-white min-h-screen font-sans">
        <header className="p-4 bg-gray-800 border-b border-gray-700">
          <h1 className="text-xl font-bold text-blue-400">Polymarket Watcher</h1>
        </header>
        <main className="p-6">
          <Outlet />
        </main>
        <Scripts />
      </body>
    </html>
  )
}
```

- [ ] **Step 2: Create Server Function to Load Data**
Update `dashboard/app/routes/index.tsx` to fetch data on the server:
```tsx
import { createFileRoute } from '@tanstack/react-router'
import { getActiveMarkets, getRecentSignals } from '../utils/redis'

export const Route = createFileRoute('/')({
  component: Dashboard,
  loader: async () => {
    // In TanStack Start, loaders run on the server
    const markets = await getActiveMarkets();
    const signals = await getRecentSignals();
    return { markets, signals };
  }
})

function Dashboard() {
  const { markets, signals } = Route.useLoaderData();
  
  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <div className="col-span-2 bg-gray-800 p-4 rounded-lg shadow">
        <h2 className="text-lg font-semibold mb-4">Active Markets</h2>
        <div className="grid grid-cols-2 gap-4">
          {markets.length === 0 ? <p className="text-gray-400">No active markets found.</p> : null}
          {/* Map through markets here */}
        </div>
      </div>
      <div className="col-span-1 bg-gray-800 p-4 rounded-lg shadow">
        <h2 className="text-lg font-semibold mb-4">Live Signals</h2>
        <div className="flex flex-col gap-3">
          {signals.length === 0 ? <p className="text-gray-400">No recent signals.</p> : null}
          {/* Map through signals here */}
        </div>
      </div>
    </div>
  )
}
```

- [ ] **Step 3: Run to verify layout renders**
Run `cd dashboard && bun run dev`. Open `http://localhost:3000` and verify the dark mode UI loads without errors.

- [ ] **Step 4: Commit**
```bash
git add dashboard/app/routes/
git commit -m "feat: implement main dashboard layout and server-side data loader"
```

---

### Task 4: Add Real-time Streaming (Server-Sent Events)

**Files:**
- Create: `dashboard/app/routes/api/signals.ts`
- Modify: `dashboard/app/routes/index.tsx`

- [ ] **Step 1: Create SSE API Route**
Create `dashboard/app/routes/api/signals.ts`:
```typescript
import { createAPIFileRoute } from '@tanstack/start/api'
import Redis from 'ioredis'

export const Route = createAPIFileRoute('/api/signals')({
  GET: ({ request }) => {
    const stream = new ReadableStream({
      start(controller) {
        // Create a dedicated pub/sub redis client
        const subUrl = process.env.REDIS_URL || 'redis://localhost:6379';
        const sub = new Redis(subUrl);
        
        sub.subscribe('signals:live', (err) => {
          if (err) console.error('Failed to subscribe', err);
        });

        sub.on('message', (channel, message) => {
          controller.enqueue(new TextEncoder().encode(`data: ${message}\n\n`));
        });

        // Cleanup on disconnect
        request.signal.addEventListener('abort', () => {
          sub.quit();
          controller.close();
        });
      }
    });

    return new Response(stream, {
      headers: {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive',
      },
    });
  }
})
```

- [ ] **Step 2: Connect Frontend to SSE stream**
Modify `dashboard/app/routes/index.tsx` to add React state and an EventSource:
```tsx
import { createFileRoute } from '@tanstack/react-router'
import { useEffect, useState } from 'react'
import { getActiveMarkets, getRecentSignals } from '../utils/redis'

// ... Keep existing loader ...

function Dashboard() {
  const { markets, signals: initialSignals } = Route.useLoaderData();
  const [liveSignals, setLiveSignals] = useState(initialSignals);

  useEffect(() => {
    const eventSource = new EventSource('/api/signals');
    
    eventSource.onmessage = (event) => {
      const newSignal = JSON.parse(event.data);
      setLiveSignals(prev => [newSignal, ...prev].slice(0, 50)); // Keep last 50
    };

    return () => eventSource.close();
  }, []);
  
  // ... render liveSignals instead of initialSignals ...
}
```

- [ ] **Step 3: Test Real-time connection**
In one terminal: `cd dashboard && bun run dev`
In another terminal: `redis-cli PUBLISH signals:live '{"market": "Trump vs Biden", "action": "WHALE_BUY", "confidence": 0.9}'`
Expected: The new signal instantly appears on the UI without reloading.

- [ ] **Step 4: Commit**
```bash
git add dashboard/app/routes/
git commit -m "feat: implement SSE endpoint and frontend hook for real-time signal streaming"
```
