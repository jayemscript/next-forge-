# Next.js 16 — Micro-SaaS & Mini Full Stack Project Structure

> Based on Next.js 16.2.2 (App Router, April 2026).
> Stack: TypeScript, Tailwind CSS, Drizzle ORM, Auth.js, Stripe, Resend.
> Intended for solo builders and small teams shipping micro-SaaS products.

---

## Repo Name

```
nextforge
```

Alternatives if taken:

```
nextforge-starter
forge-saas
nextplate
stackforge
```

`nextforge` works because it implies building something solid and production-ready from a template, without being tied to a specific niche. Short, memorable, and available on most platforms.

---

## Who This Structure Is For

- Solo developers or teams of 1-3 building a focused SaaS product
- Projects that need auth, payments, a dashboard, and a landing page
- Apps where one codebase handles both the frontend and the backend
- People who want to ship fast without over-engineering the architecture

This structure avoids monorepo complexity. Everything lives in one Next.js app. When your product outgrows it, the separation points are already obvious.

---

## Root Structure

```
nextforge/
├── src/                        # All application source code
├── public/                     # Static assets (images, fonts, icons)
├── drizzle/                    # DB migration files (Drizzle ORM)
├── .env.local                  # Secret env variables — never commit this
├── .env.example                # Template showing required env keys
├── .eslintrc.json              # ESLint config
├── .prettierrc                 # Prettier config
├── next.config.ts              # Next.js configuration
├── tailwind.config.ts          # Tailwind CSS config
├── tsconfig.json               # TypeScript config
├── middleware.ts               # Global middleware (runs before every request)
└── package.json
```

Notes:
- `next build` in v16 no longer runs the linter automatically. Add `"lint": "eslint"` to your scripts.
- `.next/dev` (new in v16) separates dev and build artifacts. You can run `next dev` and `next build` at the same time without conflicts.
- Keep `.env.example` committed so teammates and your future self know what variables are needed.

---

## `src/` — Application Source

```
src/
├── app/                        # App Router — all routes and API handlers
├── components/                 # Shared UI components
├── db/                         # Database client and schema
├── services/                   # Business logic (pure functions, no Next.js internals)
├── lib/                        # Utilities, helpers, shared logic
├── hooks/                      # Custom React hooks
├── types/                      # TypeScript types and interfaces
└── config/                     # App-wide configuration and validated env vars
```

---

## `src/app/` — Routes and API

```
src/app/
├── layout.tsx                  # Root layout — html + body tags required here
├── page.tsx                    # Landing page → /
├── loading.tsx                 # Root-level loading skeleton
├── error.tsx                   # Root-level error boundary
├── not-found.tsx               # 404 page
├── globals.css                 # Global styles, imported in layout.tsx
│
├── (marketing)/                # Route group — public pages, no URL prefix added
│   ├── layout.tsx              # Shared marketing layout (navbar, footer)
│   ├── pricing/
│   │   └── page.tsx            # → /pricing
│   └── blog/
│       ├── page.tsx            # → /blog
│       └── [slug]/
│           └── page.tsx        # → /blog/:slug
│
├── (auth)/                     # Route group — authentication pages
│   ├── layout.tsx              # Centered card layout, no sidebar
│   ├── sign-in/
│   │   └── page.tsx            # → /sign-in
│   ├── sign-up/
│   │   └── page.tsx            # → /sign-up
│   └── forgot-password/
│       └── page.tsx            # → /forgot-password
│
├── (dashboard)/                # Route group — protected app pages
│   ├── layout.tsx              # Sidebar + topbar layout, auth-guarded in middleware
│   ├── dashboard/
│   │   ├── page.tsx            # → /dashboard
│   │   ├── loading.tsx         # Skeleton shown while page data loads
│   │   ├── _components/        # Private: only used by this route
│   │   │   ├── stats-card.tsx
│   │   │   └── activity-feed.tsx
│   │   └── _lib/               # Private: data loaders and server actions for this route
│   │       ├── dashboard.loader.ts
│   │       └── dashboard.actions.ts
│   ├── settings/
│   │   ├── page.tsx            # → /settings
│   │   ├── profile/
│   │   │   └── page.tsx        # → /settings/profile
│   │   └── billing/
│   │       └── page.tsx        # → /settings/billing
│   └── [feature]/              # Placeholder for your core product feature
│       ├── page.tsx
│       ├── _components/
│       └── _lib/
│           └── [feature].actions.ts
│
└── api/                        # API routes — for external consumers and webhooks only
    ├── auth/
    │   └── [...nextauth]/
    │       └── route.ts        # Auth.js session handler
    ├── users/
    │   ├── route.ts            # GET /api/users, POST /api/users
    │   └── [id]/
    │       └── route.ts        # GET, PUT, DELETE /api/users/:id
    ├── webhooks/
    │   └── stripe/
    │       └── route.ts        # Stripe payment events → /api/webhooks/stripe
    └── healthcheck/
        └── route.ts            # → /api/healthcheck (for uptime monitoring)
```

### App Router File Conventions

| File | Purpose |
|------|---------|
| `layout.tsx` | Persistent shell wrapping child routes |
| `page.tsx` | The visible page content for a route |
| `loading.tsx` | Suspense fallback shown while page loads |
| `error.tsx` | Error boundary scoped to this segment |
| `not-found.tsx` | Custom 404 for this segment |
| `route.ts` | API endpoint — no UI, returns data |
| `middleware.ts` | Runs on every request before rendering |

### Route Group and Folder Conventions

| Pattern | Example | Effect |
|---------|---------|--------|
| `(group)` | `(dashboard)` | Groups routes without adding a URL segment |
| `[param]` | `[id]` | Dynamic URL segment |
| `[...slug]` | `[...slug]` | Catch-all — matches multiple path segments |
| `[[...slug]]` | `[[...slug]]` | Optional catch-all |
| `_folder` | `_components` | Private folder — excluded from routing entirely |

---

## `src/components/` — Shared UI Components

Components here are used across multiple routes. If a component is only used inside one route, colocate it in that route's `_components/` folder instead.

```
src/components/
├── ui/                         # Base design system components
│   ├── button.tsx
│   ├── input.tsx
│   ├── card.tsx
│   ├── modal.tsx
│   ├── badge.tsx
│   └── index.ts                # Barrel export
├── layout/                     # Structural layout components
│   ├── header.tsx
│   ├── footer.tsx
│   ├── sidebar.tsx
│   └── nav.tsx
├── forms/                      # Reusable form components
│   ├── login-form.tsx
│   └── signup-form.tsx
└── providers/                  # React context providers
    ├── query-provider.tsx      # TanStack Query setup
    ├── theme-provider.tsx
    └── auth-provider.tsx
```

---

## `src/db/` — Database Layer

```
src/db/
├── index.ts                    # Drizzle client singleton — import db from here
├── schema/                     # One file per table
│   ├── users.ts
│   ├── subscriptions.ts
│   ├── sessions.ts
│   └── index.ts                # Re-exports all schemas
└── migrations/                 # Auto-generated by drizzle-kit
    └── 0001_initial.sql
```

### `src/db/index.ts`

```ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema';

const client = postgres(process.env.DATABASE_URL!);
export const db = drizzle(client, { schema });
```

### `src/db/schema/users.ts`

```ts
import { pgTable, text, timestamp, uuid } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: text('email').notNull().unique(),
  name: text('name'),
  plan: text('plan').notNull().default('free'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

export type User = typeof users.$inferSelect;
export type NewUser = typeof users.$inferInsert;
```

---

## `src/services/` — Business Logic Layer

Services contain pure functions — no `cookies()`, `headers()`, or `revalidatePath()`. That makes them testable in isolation and reusable from API routes, Server Actions, webhooks, and cron jobs.

```
src/services/
├── user.service.ts             # User CRUD and account logic
├── subscription.service.ts     # Plan checks, upgrade/downgrade logic
├── auth.service.ts             # Password hashing, token validation
├── email.service.ts            # Send emails via Resend
├── stripe.service.ts           # Create checkout sessions, manage subscriptions
└── upload.service.ts           # File upload handling
```

### `src/services/user.service.ts`

```ts
import { db } from '@/db';
import { users } from '@/db/schema';
import { eq } from 'drizzle-orm';
import type { NewUser } from '@/db/schema/users';

export async function getUserById(id: string) {
  return db.query.users.findFirst({ where: eq(users.id, id) });
}

export async function getUserByEmail(email: string) {
  return db.query.users.findFirst({ where: eq(users.email, email) });
}

export async function createUser(data: NewUser) {
  const [user] = await db.insert(users).values(data).returning();
  return user;
}

export async function updateUserPlan(id: string, plan: string) {
  const [user] = await db
    .update(users)
    .set({ plan })
    .where(eq(users.id, id))
    .returning();
  return user;
}
```

---

## `src/lib/` — Utilities and Helpers

```
src/lib/
├── utils.ts                    # cn(), formatDate(), slugify(), truncate()
├── validations.ts              # Zod schemas for all forms and API inputs
├── auth.ts                     # Auth.js config and session helpers
├── constants.ts                # Plans, limits, feature flags, routes
└── errors.ts                   # Custom error classes (AppError, NotFoundError)
```

### `src/lib/constants.ts`

```ts
export const PLANS = {
  FREE: 'free',
  PRO: 'pro',
} as const;

export const PLAN_LIMITS = {
  [PLANS.FREE]: { projects: 3, apiCalls: 100 },
  [PLANS.PRO]: { projects: Infinity, apiCalls: 10000 },
};

export const ROUTES = {
  HOME: '/',
  SIGN_IN: '/sign-in',
  DASHBOARD: '/dashboard',
  BILLING: '/settings/billing',
};
```

---

## `src/hooks/` — Custom React Hooks

```
src/hooks/
├── use-auth.ts                 # Access current user and session
├── use-subscription.ts         # Check plan, limits, feature access
├── use-debounce.ts
└── use-media-query.ts
```

---

## `src/types/` — TypeScript Types

```
src/types/
├── index.ts                    # Barrel export
├── api.types.ts                # Shared API request and response shapes
└── next.d.ts                   # Next.js type augmentation if needed
```

---

## `src/config/` — Configuration and Env Validation

```
src/config/
├── app.config.ts               # App name, URL, metadata defaults
├── auth.config.ts              # OAuth providers, session duration
└── env.ts                      # Validated and typed env variables (Zod)
```

### `src/config/env.ts`

Validate env vars at startup so the app crashes early with a clear message rather than silently failing in production.

```ts
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  STRIPE_WEBHOOK_SECRET: z.string().startsWith('whsec_'),
  RESEND_API_KEY: z.string().min(1),
});

export const env = envSchema.parse(process.env);
```

---

## `middleware.ts` — Auth Guard

Placed at the project root. Runs on every request before the page renders.

```ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { getToken } from 'next-auth/jwt';
import { ROUTES } from '@/lib/constants';

export async function middleware(request: NextRequest) {
  const token = await getToken({ req: request });
  const { pathname } = request.nextUrl;

  const isProtected = pathname.startsWith('/dashboard') ||
                      pathname.startsWith('/settings');
  const isAuthPage = pathname.startsWith('/sign-in') ||
                     pathname.startsWith('/sign-up');

  if (isProtected && !token) {
    return NextResponse.redirect(new URL(ROUTES.SIGN_IN, request.url));
  }

  if (isAuthPage && token) {
    return NextResponse.redirect(new URL(ROUTES.DASHBOARD, request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

---

## API Route Pattern

Use API routes only for endpoints consumed externally — webhooks, third-party integrations, or a public API. Internal data mutations should use Server Actions.

```ts
// src/app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getUserByEmail, createUser } from '@/services/user.service';
import { createUserSchema } from '@/lib/validations';

export async function POST(req: NextRequest) {
  const body = await req.json();
  const parsed = createUserSchema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error.flatten() }, { status: 400 });
  }

  const existing = await getUserByEmail(parsed.data.email);
  if (existing) {
    return NextResponse.json({ error: 'Email already in use' }, { status: 409 });
  }

  const user = await createUser(parsed.data);
  return NextResponse.json(user, { status: 201 });
}
```

---

## Server Action Pattern

Server Actions handle form submissions and internal mutations. They are not API routes. Colocate them in the `_lib/` folder of the route that uses them.

```ts
// src/app/(dashboard)/[feature]/_lib/[feature].actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { getServerSession } from 'next-auth';
import { createFeatureItem } from '@/services/feature.service';
import { createItemSchema } from '@/lib/validations';

export async function createItemAction(formData: FormData) {
  const session = await getServerSession();
  if (!session?.user) throw new Error('Unauthorized');

  const parsed = createItemSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) throw new Error('Invalid input');

  await createFeatureItem({ ...parsed.data, userId: session.user.id });
  revalidatePath('/dashboard');
}
```

---

## Stripe Webhook Pattern

```ts
// src/app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Stripe from 'stripe';
import { env } from '@/config/env';
import { updateUserPlan } from '@/services/user.service';

const stripe = new Stripe(env.STRIPE_SECRET_KEY);

export async function POST(req: NextRequest) {
  const body = await req.text();
  const sig = req.headers.get('stripe-signature')!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(body, sig, env.STRIPE_WEBHOOK_SECRET);
  } catch {
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  if (event.type === 'customer.subscription.updated') {
    const subscription = event.data.object as Stripe.Subscription;
    const userId = subscription.metadata.userId;
    await updateUserPlan(userId, 'pro');
  }

  return NextResponse.json({ received: true });
}
```

---

## Full Tree at a Glance

```
nextforge/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── globals.css
│   │   ├── (marketing)/
│   │   │   ├── layout.tsx
│   │   │   └── pricing/page.tsx
│   │   ├── (auth)/
│   │   │   ├── layout.tsx
│   │   │   ├── sign-in/page.tsx
│   │   │   └── sign-up/page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx
│   │   │   ├── dashboard/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── _components/
│   │   │   │   └── _lib/
│   │   │   ├── settings/
│   │   │   │   ├── profile/page.tsx
│   │   │   │   └── billing/page.tsx
│   │   │   └── [feature]/
│   │   │       ├── page.tsx
│   │   │       ├── _components/
│   │   │       └── _lib/
│   │   └── api/
│   │       ├── auth/[...nextauth]/route.ts
│   │       ├── users/route.ts
│   │       ├── users/[id]/route.ts
│   │       └── webhooks/stripe/route.ts
│   ├── components/
│   │   ├── ui/
│   │   ├── layout/
│   │   ├── forms/
│   │   └── providers/
│   ├── db/
│   │   ├── index.ts
│   │   ├── schema/
│   │   └── migrations/
│   ├── services/
│   │   ├── user.service.ts
│   │   ├── subscription.service.ts
│   │   ├── auth.service.ts
│   │   ├── email.service.ts
│   │   └── stripe.service.ts
│   ├── lib/
│   │   ├── utils.ts
│   │   ├── validations.ts
│   │   ├── auth.ts
│   │   ├── constants.ts
│   │   └── errors.ts
│   ├── hooks/
│   │   ├── use-auth.ts
│   │   └── use-subscription.ts
│   ├── types/
│   │   ├── index.ts
│   │   └── api.types.ts
│   └── config/
│       ├── app.config.ts
│       ├── auth.config.ts
│       └── env.ts
├── public/
├── drizzle/
├── middleware.ts
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── .env.example
├── .env.local
└── package.json
```

---

## Recommended Stack for Micro-SaaS

| Layer | Choice | Why |
|-------|--------|-----|
| Framework | Next.js 16 (App Router) | Full stack in one codebase |
| Language | TypeScript | Catch bugs before they reach users |
| Styling | Tailwind CSS + shadcn/ui | Fast UI without a design system from scratch |
| Database | Supabase (Postgres) | Managed DB, built-in storage, generous free tier |
| ORM | Drizzle | Lightweight, type-safe, fast migrations |
| Auth | Auth.js v5 (NextAuth) | OAuth + credentials, works out of the box |
| Payments | Stripe | Industry standard, good docs, webhook support |
| Email | Resend | Simple API, React Email templates |
| Background jobs | Inngest or Trigger.dev | Serverless-compatible, no separate worker server |
| Deployment | Vercel | Zero config, preview deployments, pairs with Next.js |

---

## Key Principles

| Principle | What It Means in Practice |
|-----------|--------------------------|
| Services are pure | No `cookies()`, `headers()`, or `revalidatePath()` in service files. Those belong in Server Actions or API routes. |
| Colocate by route | Components and logic only used by one route go in `_components/` and `_lib/` inside that route. |
| Server Actions for internal mutations | Form submissions and data writes from React go through Server Actions, not API routes. |
| API routes for external endpoints | Webhooks, public REST API, and third-party callbacks go in `app/api/`. |
| Validate at the boundary | Use Zod in `config/env.ts` for environment variables and `lib/validations.ts` for all user inputs. |
| One source of truth for routes | Keep all route path strings in `lib/constants.ts` to avoid scattered magic strings. |
| Start flat, split when needed | Do not pre-create every folder. Add `services/`, `hooks/`, `types/` when you actually need them. |
