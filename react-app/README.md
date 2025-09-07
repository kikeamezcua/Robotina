## Robotina 2.0 — Frontend (React + Vite)

A fast, modern UI for Robotina's OCI operations assistant.

### Stack

- React 19 + Vite 7
- Routing: react-router-dom
- State: Zustand
- Server state: TanStack Query
- Forms/validation: react-hook-form + zod
- UI: TailwindCSS + shadcn/ui (Radix)
- Charts: Recharts
- Testing: Vitest + Testing Library
- Lint/Format: ESLint + Prettier

### Getting started

```bash
cd react-app
npm install
npm run dev
```

Optional env (Vite): create `.env.local` in `react-app/`:

```bash
VITE_API_BASE_URL=http://localhost:5678/webhook/robotina-webhook
```

### Scripts

```bash
npm run dev      # start Vite dev server
npm run build    # production build
npm run preview  # preview build locally
npm run lint     # run eslint
npm run test     # run vitest
npm run format   # run prettier
```

### Dependencies

- Runtime: `react`, `react-dom`, `react-router-dom`, `zustand`, `@tanstack/react-query`, `react-hook-form`, `zod`, `@hookform/resolvers`, `recharts`
- Dev: `vite`, `@vitejs/plugin-react`, `typescript`, `@types/node`, `@types/react`, `@types/react-dom`, `vitest`, `jsdom`, `@testing-library/react`, `@testing-library/user-event`, `@testing-library/jest-dom`, `eslint`, `@eslint/js`, `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh`, `eslint-config-prettier`, `prettier`, `tailwindcss`, `postcss`, `autoprefixer`, `@shadcn/ui`, `class-variance-authority`, `tailwind-merge`, `@radix-ui/react-icons`

### Tailwind setup

This repo imports Tailwind via `@import "tailwindcss";` at the top of `src/index.css`.

If you need customization, add a `tailwind.config.js` and PostCSS config.
