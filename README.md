## Robotina

### Executive Tech Stack Overview
This document summarizes the core frontend technologies used in this project for software engineers and technical managers.

### Core Technologies
- **React**: Component-based UI library for building interactive, scalable frontends. Emphasizes declarative rendering, composition, and predictable state updates.
- **Tailwind CSS**: Utility-first CSS framework enabling rapid UI development with consistent spacing, color, and typography scales. Encourages design systems via configuration-driven tokens and variants.

### Why These Technologies
- **React**: Mature ecosystem, strong community, excellent developer tooling, and flexibility to support SPA, SSR, and hybrid patterns depending on runtime needs.
- **Tailwind CSS**: Speeds delivery by keeping styles close to components, reduces bespoke CSS maintenance, and enforces visual consistency via a single source of design tokens.

### Development Basics
- **Prerequisites**: Node.js (LTS) and a JavaScript package manager (`npm`, `yarn`, or `pnpm`).
- **Install dependencies**: `npm install`
- **Start development server**: `npm run dev`
- **Build for production**: `npm run build`
- **Preview/serve build**: `npm run preview` or `npm start` (depending on tooling)

Note: Exact scripts depend on the chosen build tool or framework (e.g., Next.js, Vite). Use the commands above as a baseline; check `package.json` for authoritative scripts.

### Styling Conventions with Tailwind
- **Utilities-first**: Prefer Tailwind utilities in JSX for common spacing, layout, color, and typography.
- **Abstractions**: When patterns repeat, extract reusable components or minimal CSS via `@apply`.
- **Theming**: Design tokens (colors, spacing, fonts) live in `tailwind.config.*`. Adjust there to evolve branding consistently.
- **Responsiveness/States**: Use responsive (`sm`, `md`, `lg`, `xl`) and state variants (e.g., `hover:`, `focus:`) directly in class lists.

### Accessibility and Quality
- **Accessibility**: Favor semantic HTML elements, label interactive controls, and validate with automated checks during development.
- **Performance**: Tree-shaken builds and Tailwind's purge keep bundles lean; prefer memoization and lazy loading where appropriate.

### Ownership and Change Management
- **Source of truth**: This README is the executive summary of the frontend stack. Architectural or process changes impacting React/Tailwind usage should be reflected here alongside implementation docs.