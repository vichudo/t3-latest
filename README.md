# t3-latest

T3 stack starter updated to latest everything. Next.js 16 + tRPC 11 + Auth.js v5 + Prisma 6 + Tailwind v4.

## Use This Template

```bash
gh repo create my-app --template vichudo/t3-latest --clone
cd my-app
pnpm install
cp .env.example .env       # edit DATABASE_URL
pnpm db:push               # push schema to db
pnpm dev                   # http://localhost:3000
```

Or without creating a repo:

```bash
npx degit vichudo/t3-latest my-app
```

## What's Included

| What       | Version / Notes                          |
| ---------- | ---------------------------------------- |
| Next.js    | 16 (App Router, Turbopack default)       |
| tRPC       | 11 (React Query 5, SuperJSON, streaming) |
| Auth.js    | v5 (Prisma adapter, Discord provider)    |
| Prisma     | 6 (PostgreSQL)                           |
| Tailwind   | v4                                       |
| TypeScript | 5 (strict, noUncheckedIndexedAccess)     |
| ESLint     | 9 (flat config, no `next lint`)          |
| Zod        | 3 (input validation)                     |

## Project Structure

```
src/
  app/
    api/auth/[...nextauth]/   Auth handler
    api/trpc/[trpc]/          tRPC handler
    _components/              Client components
    layout.tsx                Root layout (TRPCReactProvider)
    page.tsx                  Home (server component)
  server/
    api/routers/              Add tRPC routers here
    api/root.ts               Register routers here
    api/trpc.ts               Context + procedures
    auth/config.ts            Auth providers + callbacks
    auth/index.ts             auth(), signIn, signOut
    db.ts                     Prisma client
  trpc/
    react.tsx                 Client hooks (api.x.useQuery)
    server.ts                 RSC caller (await api.x())
    query-client.ts           React Query config
  env.js                      Env validation (Zod)
prisma/
  schema.prisma               DB schema
```

## Recipes

### Add a tRPC router

```ts
// src/server/api/routers/todo.ts
import { z } from "zod";
import { createTRPCRouter, protectedProcedure } from "../trpc";

export const todoRouter = createTRPCRouter({
  list: protectedProcedure.query(({ ctx }) =>
    ctx.db.todo.findMany({ where: { userId: ctx.session.user.id } }),
  ),
  create: protectedProcedure
    .input(z.object({ title: z.string().min(1) }))
    .mutation(({ ctx, input }) =>
      ctx.db.todo.create({ data: { title: input.title, userId: ctx.session.user.id } }),
    ),
});
```

Register it:

```ts
// src/server/api/root.ts
import { todoRouter } from "./routers/todo";

export const appRouter = createTRPCRouter({
  post: postRouter,
  todo: todoRouter, // add here
});
```

Use it:

```tsx
// Server Component
const todos = await api.todo.list();

// Client Component ("use client")
const [todos] = api.todo.list.useSuspenseQuery();
const create = api.todo.create.useMutation();
```

### Add an auth provider

```ts
// src/server/auth/config.ts
import GitHubProvider from "next-auth/providers/github";

export const authConfig = {
  providers: [
    DiscordProvider,
    GitHubProvider, // add provider
  ],
  // ...
} satisfies NextAuthConfig;
```

Add the env vars to `.env` and register them in `src/env.js`.

### Database

```bash
pnpm db:push        # apply schema changes (dev)
pnpm db:generate    # create a migration
pnpm db:migrate     # run migrations (prod)
pnpm db:studio      # visual editor
```

## Scripts

| Command              | What it does                    |
| -------------------- | ------------------------------- |
| `pnpm dev`           | Start dev server (Turbopack)    |
| `pnpm build`         | Production build (Turbopack)    |
| `pnpm start`         | Start production server         |
| `pnpm check`         | Lint + typecheck                |
| `pnpm lint`          | ESLint                          |
| `pnpm lint:fix`      | ESLint with auto-fix            |
| `pnpm typecheck`     | TypeScript only                 |
| `pnpm format:write`  | Prettier format                 |
| `pnpm db:push`       | Push Prisma schema to DB        |
| `pnpm db:studio`     | Open Prisma Studio              |

## Environment Variables

Configure in `.env`, validated at build time by `src/env.js`.

| Variable             | Required    | How to get                          |
| -------------------- | ----------- | ----------------------------------- |
| `DATABASE_URL`       | Always      | Your PostgreSQL connection string   |
| `AUTH_SECRET`        | Production  | Run `npx auth secret`              |
| `AUTH_DISCORD_ID`    | If using    | https://discord.com/developers      |
| `AUTH_DISCORD_SECRET`| If using    | Same as above                       |

## Deploy

Works out of the box on [Vercel](https://vercel.com). Set your env vars and push.

For other platforms, see the [T3 deployment guides](https://create.t3.gg/en/deployment).

## Based On

[create-t3-app](https://create.t3.gg) v7.40.0, updated for Next.js 16.
