# Bubs' Shelf: React Build Plan

Oct 18, 2026 · @Sidney

## Overview

Bubs' Shelf is one React and TypeScript app with two tabs: a plant care tracker with a calm, timed watering session, and a Labubu collection tracker. You build it in six stages, and each one ends with something that works.

1. **Setup.** An empty Vite project running in your browser.
2. **Plant list.** Add, edit, and delete plants that survive a refresh.
3. **Due today.** The app works out which plants need water and logs each watering.
4. **Watering session.** A Pomodoro-style mode that walks you through one plant at a time.
5. **Labubu tab.** A second section that reuses your components.
6. **Next.js.** Move the app to Next.js with a database, so it works on every device.

How to use this plan: finish each stage's checklist before moving on, and commit to git at the end of every stage. Don't worry about styling until Stage 2 is done. Rough time is 2 to 4 weekends for Stages 0 to 4, plus 2 to 3 more for Stage 5 at a relaxed pace.

## Accessibility in every stage

Accessibility is built into each stage below, not saved for the end. Fixing it later means rewriting components you've already finished. Aim for [WCAG 2.2](https://www.w3.org/WAI/standards-guidelines/wcag/) level AA, the standard most teams and job postings mean.

Your toolkit:

- **Keyboard:** unplug your mouse mentally. Tab, Shift+Tab, Enter, Space, and Esc should do everything.
- **Screen reader:** VoiceOver on Mac (Cmd+F5) or iPhone, NVDA on Windows (free), TalkBack on Android.
- **axe DevTools:** a free browser extension that flags common problems on the current page.
- **Lighthouse:** built into Chrome DevTools, gives an accessibility score and a list of fixes.
- **eslint-plugin-jsx-a11y:** catches problems in your JSX as you type.

End-of-stage routine: run axe, fix what it finds, then use the stage's main feature with only the keyboard and once with a screen reader. Automated tools catch only some issues, so the manual check matters most.

## TypeScript from day one

You'll write the whole app in TypeScript, starting in Stage 0. The compiler catches mistakes like a misspelled prop, or a string where a number belongs, before you even open the browser.

Keep `"strict": true` in `tsconfig.json` (Vite and Next.js both turn it on). Avoid `any`: if you're stuck, use `unknown` and narrow it.

| Stage | TypeScript you'll practice |
| --- | --- |
| 0. Setup | `tsconfig.json`, `.ts` vs `.tsx` files |
| 1. Plant list | Interfaces, typed props, `useState<T>`, event types, a generic custom hook |
| 2. Due today | Function signatures, union types, letting the compiler guide a refactor |
| 3. Watering session | Discriminated unions for reducer actions, `useRef<T>`, `as const` |
| 4. Labubu tab | Nested types, typing `useReducer`, generic components |
| 5. Next.js | Types generated from a database schema, typed server actions |

Keep these open: the [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) and the [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/).

## Stage 0: Setup

Goal: a blank React app running locally, with a folder layout you'll grow into.

- [x] Install Node.js (the LTS version) and VS Code
- [x] Run `npm create vite@latest bubs-shelf`, then pick React and TypeScript
- [x] `cd bubs-shelf`, `npm install`, `npm run dev`, and open the local URL
- [x] Delete the demo content in `App.tsx` and render a heading that says "Bubs' Shelf"
- [x] Create the folders below
- [x] `git init` and make your first commit

```
src/
  components/   PlantCard.tsx, PlantForm.tsx, Timer.tsx...
  hooks/        useLocalStorage.ts, useTimer.ts
  types/        plant.ts, session.ts, labubu.ts
  utils/        dates.ts
  pages/        PlantsPage.tsx, SessionPage.tsx, LabubuPage.tsx
  App.tsx
```

Tip: install the React Developer Tools browser extension now. Watching state change live makes everything after this easier.

**Accessibility**

- [x] Install the axe DevTools extension
- [x] Add `eslint-plugin-jsx-a11y` to your ESLint config
- [x] Confirm `index.html` has `<html lang="en">` so screen readers use the right voice
- [ ] Spend 15 minutes learning basic VoiceOver or NVDA commands on any website

## Stage 1: Plant list

Goal: add, edit, and delete plants, and keep them saved after you refresh the page.

Start by deciding what one plant looks like as data. Everything else is built around this shape.

```
// src/types/plant.ts
export interface Plant {
  id: string;
  name: string;
  nickname?: string;
  waterEveryDays: number;
  lastWatered: string;   // "YYYY-MM-DD"
  notes: string;
  photoUrl?: string;
}

const monstera: Plant = {
  id: crypto.randomUUID(),
  name: "Monstera",
  nickname: "Monty",
  waterEveryDays: 7,
  lastWatered: "2026-10-14",
  notes: "Water when top 2 inches are dry"
};
```

- [ ] Define the `Plant` type in `types/plant.ts`
- [ ] Hardcode 3 sample plants as `const samplePlants: Plant[]` and render them with `.map()`
- [ ] Build a `PlantCard` component and type its props: `function PlantCard({ plant }: { plant: Plant })`
- [ ] Move the array into state with `useState<Plant[]>(samplePlants)`
- [ ] Build `PlantForm` with controlled inputs, typing change handlers as `React.ChangeEvent<HTMLInputElement>`
- [ ] Notice that number inputs give you strings: convert with `Number()`, or TypeScript will complain (this is it doing its job)
- [ ] Add a plant: append to the array with `setPlants([...plants, newPlant])`
- [ ] Delete a plant: `filter` it out
- [ ] Edit a plant: reuse `PlantForm`, prefilled, and `map` to replace the changed one
- [ ] Save to localStorage with `useEffect`, and load it back as the initial state (`JSON.parse` returns `any`, so give the result a type)
- [ ] Bonus: pull the saving logic into a generic hook, `useLocalStorage<T>(key: string, initial: T)`

**Accessibility**

- [ ] Give every input a visible `<label>` connected with `htmlFor`, not just placeholder text
- [ ] Use real `<button>` elements for every action, never a clickable `<div>`
- [ ] Give buttons on each card a specific name, like "Delete Monstera" instead of just "Delete"
- [ ] Write `alt` text for plant photos ("Monstera with three new leaves"), or `alt=""` if a photo is purely decorative
- [ ] Render the plants as a `<ul>` of `<li>` cards, with one `<h1>` on the page and `<h2>` per card
- [ ] Show form errors in text, linked to the input with `aria-describedby`
- [ ] After adding or deleting a plant, move focus somewhere sensible so keyboard users aren't stranded

**What you'll learn:** components, props, `useState`, lists and keys, controlled forms, immutable updates, `useEffect`.

**Done when:** you can add your real plants, refresh, and they're all still there.

## Stage 2: Due today

Goal: the app tells you which plants need water today, which are overdue, and which can wait.

The key idea is **derived state**: don't store "days until thirsty." Calculate it from `lastWatered` and `waterEveryDays` every time the app renders, so it can never go out of date.

- [ ] In `utils/dates.ts`, write `daysUntilDue(plant: Plant): number` (negative means overdue)
- [ ] Define `type DueStatus = "overdue" | "today" | "later"` and show a badge on each `PlantCard`
- [ ] Sort the list so the thirstiest plants come first (copy first: `[...plants].sort(...)`)
- [ ] Add filter buttons: All, Due today, Overdue, stored as `useState<"all" | "today" | "overdue">("all")`
- [ ] Add a "Watered" button that sets `lastWatered` to today
- [ ] Add `history: string[]` to the `Plant` type, then fix every error the compiler shows you. It lists every place that needs updating.
- [ ] Show a "3 plants need water today" summary at the top
- [ ] Now style it: cards in a grid, colors for overdue and due

Watch out: dates are tricky in JavaScript. Store dates as `"YYYY-MM-DD"` strings and compare whole days, not timestamps, or a plant watered at 11pm will show up wrong the next morning.

**Accessibility**

- [ ] Never rely on color alone: Overdue badges need text (and optionally an icon), not just red
- [ ] Check text contrast is at least 4.5:1 (axe and DevTools both show this)
- [ ] Mark the active filter button with `aria-pressed="true"`
- [ ] Add a `role="status"` region that announces "Monstera watered" when you tap Watered
- [ ] Make sure the focus outline is clearly visible on every button and input

**What you'll learn:** derived state, pure helper functions, sorting and filtering without changing the original array, conditional rendering.

**Done when:** you open the app in the morning and it tells you exactly what to water.

## Stage 3: Watering session

Goal: tap Start, and the app walks you through today's plants one at a time, so watering feels like a series of small steps instead of one big chore.

```
idle ──START──▶ watering ──DONE_PLANT / SKIP_PLANT──▶ ◇ queue empty?
                   ▲                                   │ yes ──▶ done
                   │                                   │ no, 3rd plant ──▶ break
                   │                                   │ no ──▶ watering (next plant)
                   └────────────── END_BREAK ◀──────── break
```

Each box is one value of a `status` field: `idle`, `watering`, `break`, or `done`. The whole session is one state object, updated by a reducer.

```
// src/types/session.ts
export type SessionStatus = "idle" | "watering" | "break" | "done";

export interface Session {
  status: SessionStatus;
  queue: string[];         // ids of plants due today, thirstiest first
  index: number;           // which plant you're on
  secondsLeft: number;
  wateredCount: number;
  startedAt: number | null;
}

export type SessionAction =
  | { type: "START"; plantIds: string[] }
  | { type: "DONE_PLANT" }
  | { type: "SKIP_PLANT" }
  | { type: "END_BREAK" }
  | { type: "FINISH" };
```

`SessionAction` is a discriminated union: inside `case "START":`, TypeScript knows `action.plantIds` exists. Add a `never` check in the `default` case and the compiler will flag any action you forget to handle.

**Part A: the timer on its own**

- [ ] Build a plain countdown on a test page: Start, Pause, Reset
- [ ] Use `setInterval` inside `useEffect`, and clear it in the cleanup function
- [ ] Count down with a functional update, `setSecondsLeft(s => s - 1)`, not `secondsLeft - 1`
- [ ] Keep the interval id in `useRef` so Pause can stop it without a re-render
- [ ] Move it all into a `useTimer` hook

Try it yourself first, then compare with this sketch:

```
function useTimer(isRunning: boolean) {
  const [secondsLeft, setSecondsLeft] = useState(0);
  const intervalRef = useRef<number | null>(null);

  useEffect(() => {
    if (!isRunning) return;
    intervalRef.current = window.setInterval(() => {
      setSecondsLeft(s => Math.max(s - 1, 0));
    }, 1000);
    return () => {
      if (intervalRef.current !== null) clearInterval(intervalRef.current);
    };
  }, [isRunning]);

  return [secondsLeft, setSecondsLeft] as const;
}
```

Why `window.setInterval`? It returns a plain number. Bare `setInterval` can get typed as Node's `Timeout` instead, a classic TypeScript head-scratcher. And `as const` makes the hook return a typed pair, like `useState` does, instead of a loose array.

**Part B: the session**

- [ ] Write a `sessionReducer` with actions `START`, `DONE_PLANT`, `SKIP_PLANT`, `END_BREAK`, `FINISH`
- [ ] On `START`, build the queue from Stage 2's due-today list
- [ ] Show one plant big on screen: name, photo, care notes, and the timer (2 to 3 minutes)
- [ ] On `DONE_PLANT`, mark it watered (reuse your Stage 2 function) and move to the next one
- [ ] After every 3 plants, switch to `break` for 1 minute
- [ ] When the queue is empty, show the summary: "8 plants watered in 19 minutes"
- [ ] Add a session cap: "I have 10 minutes" only queues the most overdue plants that fit
- [ ] Make the timer gentle: when it runs out, nothing is failed, it just asks "Done?"

Watch out: if the timer speeds up or doesn't stop, an old interval is still running. That almost always means a missing cleanup function.

**Accessibility**

Timers are one of the trickiest things to make accessible, which makes this the best a11y practice in the plan.

- [ ] Let people pause and extend the timer (WCAG requires adjustable time limits)
- [ ] Don't announce every second: a screen reader reading "2:59, 2:58..." is unusable. Announce changes like "1 minute left" and "Time's up" in a polite live region
- [ ] When the session moves to the next plant, move focus to that plant's heading
- [ ] Never signal the end of a timer with sound or animation alone, always with text too
- [ ] Respect `prefers-reduced-motion` for any countdown rings or transitions
- [ ] Do a full session with only the keyboard, then with your eyes closed using a screen reader

**What you'll learn:** `useEffect` cleanup, `useRef`, stale state, `useReducer`, state machines, custom hooks.

**Done when:** you can water every due plant using only the session screen, and it feels calmer than before.

## Stage 4: Labubu collection tab

Goal: a second section for your collection, built mostly from pieces you already have.

This data is nested: a series holds figures. That's the new challenge.

```
// src/types/labubu.ts
export type FigureStatus = "wanted" | "owned" | "duplicate";

export interface Figure {
  id: string;
  name: string;
  status: FigureStatus;
  count: number;
  isSecret: boolean;
}

export interface Series {
  id: string;
  seriesName: string;
  figures: Figure[];
}

const macaron: Series = {
  id: "s1",
  seriesName: "Exciting Macaron",
  figures: [
    { id: "f1", name: "Lychee Berry", status: "owned", count: 1, isSecret: false },
    { id: "f2", name: "Soy Milk", status: "wanted", count: 0, isSecret: false }
  ]
};
```

- [ ] Install React Router and add routes: `/plants`, `/session`, `/labubu`
- [ ] Build a shared `Layout` with a nav bar and an `<Outlet />`
- [ ] Render series as sections, figures as cards inside them
- [ ] Cycle a figure's status with a tap: wanted, owned, duplicate
- [ ] Show "7 of 12" and a progress bar per series (derived, like Stage 2)
- [ ] Add a Trade list view that shows only duplicates
- [ ] Move collection updates into `useReducer` with actions like `ADD_FIGURE` and `SET_STATUS`
- [ ] Look at `PlantCard` and your new figure card side by side and pull out a shared `Card` component
- [ ] Save to localStorage with the `useLocalStorage` hook you made in Stage 1

Watch out: updating one figure inside one series means copying both levels. If an update "doesn't show," you probably changed the old object instead of making a new one.

**Accessibility**

- [ ] Wrap the nav in `<nav>` and the page content in `<main>`, so screen reader users can jump between them
- [ ] Use React Router's `NavLink`, which marks the current page with `aria-current="page"`
- [ ] On every route change, update `document.title` and move focus to the new page's `<h1>`
- [ ] Add a "Skip to main content" link as the first thing on the page
- [ ] Use a `<progress>` element (or `role="progressbar"` with value attributes) for series completion
- [ ] Make the status-cycling button say its current state, like "Soy Milk: owned. Change status"

**What you'll learn:** routing, layouts, nested immutable updates, `useReducer`, reusable components.

**Done when:** you can check off figures as you unbox them and see your trade list at a glance.

## Stage 5: Move to Next.js

Goal: the same app, rebuilt on Next.js with a real database, so Bubs' Shelf works on your phone and your laptop with the same data.

You already understand every feature, so each change shows you exactly what Next.js adds. Make a fresh Next.js project and move your code over piece by piece rather than converting the Vite project in place.

| Vite version | Next.js version |
| --- | --- |
| React Router routes | Folders in `app/`: `app/plants/page.tsx`, `app/session/page.tsx`, `app/labubu/page.tsx` |
| Shared `Layout` with `<Outlet />` | `app/layout.tsx` |
| localStorage | A database (SQLite to start, hosted Postgres when you deploy) |
| Updating state, then saving | Server actions that write to the database |
| Hand-written types in `types/` | Types generated from your database schema |
| Runs only in your browser | Deployed online, open on any device |

**Part A: the move**

- [ ] Run `npx create-next-app@latest bubs-shelf-next` and choose TypeScript and the App Router
- [ ] Copy `components/`, `hooks/`, `utils/`, and `types/` across
- [ ] Recreate each page as a folder in `app/`, and move your nav into `app/layout.tsx`
- [ ] Swap React Router's `<Link>` for Next's `<Link>` from `next/link`
- [ ] Add `"use client"` to the top of any component that uses state, effects, refs, or click handlers, and notice which ones need it
- [ ] Get everything working again, still on localStorage, with zero errors from `npx tsc --noEmit`

**Part B: the database**

- [ ] Add an ORM (Prisma or Drizzle) and define tables for plants, waterings, series, and figures. Both generate TypeScript types from your schema, so most of your hand-written types can go
- [ ] Seed it with your current data (export your localStorage as JSON first)
- [ ] Load the plant list in a server component, straight from the database
- [ ] Turn "Watered," "Add plant," and "Set status" into server actions
- [ ] Keep the watering session a client component: the timer has to run in the browser, but `DONE_PLANT` calls a server action to save

**Part C: online**

- [ ] Add login (Auth.js is a common choice) so your shelf is private
- [ ] Move to a hosted Postgres database
- [ ] Deploy on Vercel, then add the site to your phone's home screen

Watch out: an error about hooks or `window` in Next.js almost always means a component needs `"use client"`, or is running on the server when you expected the browser.

**Accessibility**

- [ ] Set `lang` on `<html>` in `app/layout.tsx`, and a `title` for each page with Next's metadata
- [ ] Use `next/image` for photos, and keep writing real `alt` text
- [ ] Check that Next's built-in route announcer reads each new page aloud
- [ ] Show and announce pending states while server actions save ("Saving...")
- [ ] Make sure the login form has labels, clear error messages, and works with password managers
- [ ] Run Lighthouse on the deployed site and test it on your phone with VoiceOver or TalkBack

**What you'll learn:** server vs. client components, file-based routing, layouts, server actions, databases and ORMs, authentication, deployment.

**Done when:** you water a plant from your phone and it shows as watered on your laptop.

## Stretch goals and getting unstuck

Pick these up once the core app works, in any order.

- **Photos:** upload a photo per plant or figure and store it as a data URL
- **Notifications:** a browser notification when a plant becomes due
- **Watering streaks:** a calendar heatmap built from each plant's `history`
- **Session stats:** average minutes per session, plants watered per week
- **TypeScript:** turn on stricter settings like noUncheckedIndexedAccess, or validate form input with Zod
- **Deploy it:** if you'd like it online before Stage 5, put the Vite version on Netlify

When you get stuck:

1. Read the error message all the way through. React's errors usually name the exact component and line.
2. Add `console.log` right before the line that misbehaves, and check what the value really is.
3. Open React DevTools and look at the component's state and props.
4. Shrink the problem: comment things out until the bug disappears, then add them back one at a time.
5. Check the official docs at [react.dev](https://react.dev/learn). They're the best free React tutorial there is.
6. Ask me, and paste the component plus the error. That's the fastest way to get a useful answer.
