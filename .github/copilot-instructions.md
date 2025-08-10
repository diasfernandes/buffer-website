# Copilot Instructions for buffer-website

## Project Architecture

This is a monorepo using Better-T-Stack with two main applications:
- **`apps/web/`**: React frontend with TanStack Start (SSR) + TailwindCSS + shadcn/ui
- **`apps/server/`**: Hono API server with tRPC integration

Both apps deploy to Cloudflare Workers via Wrangler.

## Key Patterns & Conventions

### Full-Stack Type Safety
- **Type sharing**: Server router types are imported in web app (`apps/web/src/utils/trpc.ts`)
- **tRPC setup**: Server uses `@hono/trpc-server`, web uses `@trpc/tanstack-react-query`
- **Router context**: Both `trpc` and `queryClient` are passed to router context in `router.tsx`

### Frontend Architecture
- **File-based routing**: Routes in `apps/web/src/routes/` with TanStack Router
- **Route generation**: `routeTree.gen.ts` is auto-generated (don't edit manually)
- **Query integration**: Use `useTRPC()` hook to access typed procedures in components
- **Global state**: Context provided via router wrapper with QueryClient + TRPCProvider

### Development Workflow

**Commands (run from root):**
```bash
bun dev          # Starts both web (port 3001) and server (port 3000)
bun dev:web      # Web app only
bun dev:server   # API server only
bun build        # Build all apps
bun check-types  # TypeScript validation across monorepo
```

**Important files:**
- `turbo.json`: Defines build pipeline and task dependencies
- Server: `apps/server/src/routers/index.ts` for API routes
- Web: `apps/web/src/routes/__root.tsx` for app shell and providers

### Cloudflare-Specific Patterns
- **Environment**: Both apps use `wrangler.jsonc` for Cloudflare configuration
- **Context**: Server context in `apps/server/src/lib/context.ts` handles Hono context
- **CORS**: Configured in server `index.ts` using `env.CORS_ORIGIN`
- **Deployment**: Each app has individual `deploy` script using Wrangler

### Component Patterns
- **UI Components**: Use shadcn/ui components from `apps/web/src/components/ui/`
- **Styling**: TailwindCSS with custom config (note: uses Tailwind v4 with `@tailwindcss/vite`)
- **Loading states**: Global loader in root route, component-level loading via router state

### tRPC Integration Example
```tsx
// In components, use the typed hook
const trpc = useTRPC();
const query = useQuery(trpc.healthCheck.queryOptions());

// Server procedures defined in apps/server/src/routers/index.ts
export const appRouter = router({
  healthCheck: publicProcedure.query(() => "OK"),
});
```

### Development Notes
- **Package manager**: Uses Bun (see `packageManager` in root `package.json`)
- **Monorepo**: Turborepo handles task orchestration and caching
- **No auth configured**: Context returns `session: null` (see `apps/server/src/lib/context.ts`)
- **Error handling**: Global query error handling with toast notifications in `router.tsx`
