# Polymarket UI Design & Frontend Execution Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task.
> **Design Context:** This plan MUST be executed using the `@frontend-design`, `@adapt`, and `@polish` skills. The aesthetic direction is "Stripe-like Refined Premium Tech" — deeply meticulous, high contrast, subtle micro-interactions, and zero generic "AI slop" aesthetics.

**Goal:** Implement a production-grade, visually striking, and highly polished UI for the Polymarket Watcher dashboard using Tailwind CSS and React/TanStack Start.

**Architecture:** A responsive, fluid layout utilizing modern CSS (OKLCH colors, `clamp()` fluid typography, CSS Grid) and optimistic/real-time UI patterns for the incoming signals.

**Tech Stack:** React, Tailwind CSS, Lucide React (for icons), Framer Motion (for layout animations).

---

## Aesthetic Direction: "Refined Intelligence"

- **Typography:** Inter or a high-quality geometric sans-serif (e.g., Geist or custom local font). Tight tracking on headings, generous line-height on body text.
- **Color Palette (OKLCH):** 
  - Background: Deep slate/charcoal dark mode (`oklch(0.15 0.01 250)` tint) or a stark, crisp white mode with deep navy/black text. Let's build a **Premium Dark Theme** by default to match the terminal/trading vibe, but heavily tinted (not pure `#000000`).
  - Accents: Vibrant, high-contrast indicators for trading signals (e.g., sharp emerald for buys, bright coral for anomalies).
- **Spatial:** Fluid padding. No "cards inside cards". Asymmetric grid for the dashboard.
- **Motion:** Exponential easing (`ease-out-expo`) for incoming real-time signals. No bouncy spring animations.

---

### Task 1: Design System, Tailwind, and shadcn/ui Configuration

**Files:**
- Modify: `dashboard/tailwind.config.js` / `dashboard/components.json`
- Modify: `dashboard/app/app.css` (or `globals.css`)

- [ ] **Step 1: Configure OKLCH colors and shadcn variables**
shadcn uses CSS variables. Update `app.css` to define custom colors using OKLCH while mapping them to shadcn's required CSS variable structure (`--background`, `--foreground`, `--card`, etc.).
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 250 0.01 18%; /* Dark mode by default */
    --foreground: 250 0.01 95%;
    --card: 250 0.015 22%;
    --card-foreground: 250 0.01 95%;
    --muted: 250 0.02 70%;
    /* Accent variables for signals */
    --accent-bull: 160 0.15 70%; 
    --accent-bear: 25 0.20 65%;
  }
}
```

- [ ] **Step 2: Update Tailwind config**
Update `tailwind.config.js` to ensure shadcn picks up the OKLCH values using `oklch(var(--value))` if supported, or adapt to HSL if the shadcn CLI enforced HSL during init. Also, add the fluid typography scale and custom easing curve `out-expo: cubic-bezier(0.16, 1, 0.3, 1)`.

- [ ] **Step 3: Commit**
```bash
git add dashboard/tailwind.config.js dashboard/app/app.css
git commit -m "design: integrate shadcn CSS variables with OKLCH color palette and fluid typography"
```

---

### Task 2: Core Layout & Navigation (Apply `@adapt` skill)

**Files:**
- Modify: `dashboard/app/routes/__root.tsx`

- [ ] **Step 1: Build the responsive shell**
Implement a layout that uses a side navigation drawer on desktop, but adapts to a bottom navigation bar or top header hamburger on mobile. Use CSS Grid for the main content area.
Ensure touch targets on mobile are at least `44x44px`.

- [ ] **Step 2: Add visual polish to the header**
Instead of a generic solid header, use a subtle border (`border-b border-surfaceHover/50`) and a backdrop blur if appropriate. Add the "Harke" or "Watcher" branding with the specific typography requested earlier (clean, lowercase, sans-serif).

- [ ] **Step 3: Commit**
```bash
git add dashboard/app/routes/__root.tsx
git commit -m "design: implement adaptive shell layout for mobile and desktop"
```

---

### Task 3: Market Cards & Data Visualization

**Files:**
- Create: `dashboard/app/components/MarketCard.tsx`
- Modify: `dashboard/app/routes/index.tsx`

- [ ] **Step 1: Design the Market Card**
Build `MarketCard.tsx` wrapping the shadcn `Card` components (`Card`, `CardHeader`, `CardTitle`, `CardContent`).
**Anti-pattern to avoid:** Do not use identical grid templates for everything. 
**Design:** Make the card feel like a precise instrument. Show the market title (large), volume/liquidity (muted, small), and current leading probability (large, accented using a shadcn `Badge`).
Include a hover state that slightly elevates the surface color (`hover:bg-accent/50`) and uses `transition-colors duration-300 ease-out-expo`.

- [ ] **Step 2: Empty States & Loading Skeletons (Apply `@polish` skill)**
Create a beautifully designed empty state for when no markets are tracked using shadcn utility components. Use a subtle Lucide icon and clear instructions.
Create a loading skeleton using the shadcn `Skeleton` component (`<Skeleton className="h-[200px] w-full rounded-xl" />`).

- [ ] **Step 3: Integrate into Dashboard**
Map the `MarketCard` components in `index.tsx` using a responsive CSS grid (`grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-clamp(1rem, 2vw, 2rem)`).

- [ ] **Step 4: Commit**
```bash
git add dashboard/app/components/MarketCard.tsx dashboard/app/routes/index.tsx
git commit -m "design: build polished MarketCard with interaction states and empty states"
```

---

### Task 4: Real-time Signal Feed (Motion & Polish)

**Files:**
- Create: `dashboard/app/components/SignalFeed.tsx`
- Modify: `dashboard/app/routes/index.tsx`

- [ ] **Step 1: Install Framer Motion**
```bash
cd dashboard
bun add framer-motion
```

- [ ] **Step 2: Build the Signal Feed Component**
Create `SignalFeed.tsx`. This list will update frequently via SSE.
Use `<AnimatePresence>` from Framer Motion so new signals slide down and fade in gracefully (`ease-out-expo`), rather than jarringly snapping the layout.
Each signal item should have a clear visual hierarchy:
- Timestamp (muted, monospaced)
- Market Name (medium weight)
- Action/Signal type (using shadcn `Badge` color-coded: Bull/Bear/Anomaly)

- [ ] **Step 3: Polish the Feed Layout**
Constrain the feed height and wrap the list in a shadcn `ScrollArea` component to keep the UI clean with a custom styled scrollbar. Ensure the newest signal always appears at the top and visually flashes or highlights briefly when it arrives.

- [ ] **Step 4: Commit**
```bash
git add package.json dashboard/app/components/SignalFeed.tsx dashboard/app/routes/index.tsx
git commit -m "design: implement animated real-time signal feed using framer-motion"
```

---

### Task 5: Final Polish & Audit (Apply `@polish` skill)

**Files:**
- Audit across all components.

- [ ] **Step 1: Contrast & Accessibility Check**
Ensure all text over `--card` background meets WCAG AA contrast. Ensure focus states (`focus-visible:ring-ring`) are present on all interactive elements.

- [ ] **Step 2: Spacing Consistency**
Check that gaps between sections feel intentional and rhythmic. Avoid tight, cramped layouts.

- [ ] **Step 3: Review against "AI Slop" anti-patterns**
Ensure there are no arbitrary gradient texts, no overly rounded corners mixed with sharp ones, no glowing drop shadows, and no generic "cards inside cards". The UI should look like it belongs in a Stripe or Vercel product showcase.

- [ ] **Step 4: Commit**
```bash
git commit -a -m "design: final visual polish, spacing rhythm, and accessibility audit"
```
