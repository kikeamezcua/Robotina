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

### Floating Gear Button (Settings)
The floating gear button is a persistent entry point to application settings. It remains visible above page content and provides quick access without occupying primary navigation space.

- **Purpose**: Single-tap/shortcut access to user and application settings.
- **Placement**: Fixed to the bottom-right of the viewport, above other content.
- **Layering**: High `z-index` to ensure visibility above cards/modals.

#### Placement & Layout (Tailwind)
- Positioning: `fixed bottom-4 right-4 z-50`
- Shape & size: `h-12 w-12 rounded-full`
- Centering: `inline-flex items-center justify-center`
- Elevation: `shadow-lg`
- Colors: `bg-primary-600 hover:bg-primary-700 text-white dark:bg-primary-500 dark:hover:bg-primary-600`
- Focus styles: `focus:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:ring-primary-500`

#### Interaction Behavior
- Click/tap toggles the Settings panel (drawer or modal), anchored logically to the button.
- Keyboard: Activate with Enter/Space. Close panel with Escape. Maintain focus return to the button after close.
- Optional tooltip on hover/focus: "Settings".
- Debounce rapid toggles to avoid flicker; ensure only one settings surface is active at a time.

#### Accessibility
- Use a native `button` element with `type="button"` and `aria-label="Open settings"`.
- Icon is decorative: include `aria-hidden="true"` on the SVG.
- Minimum target size of 44×44 px for touch ergonomics.
- Provide visible focus ring and preserve sufficient color contrast.
- When the settings panel is open: trap focus inside, hide background from screen readers (e.g., `aria-modal`, `role="dialog"`), and restore focus on close.

#### Integration Guidance
- Place the button at the app root (e.g., in `App` or root layout) so it persists across routes.
- Render the settings panel in a portal to the document body to avoid stacking/context issues.
- Manage open/close state centrally (e.g., a lightweight `SettingsProvider`) to allow other components to open/close the panel.
- Instrument analytics (e.g., `data-analytics="settings_open"`) for usage and UX insights.
- Lazy-load settings panel content to minimize initial bundle size.

#### Testing
- Unit: toggles open/close state, `aria` attributes present, focus returns to button on close.
- E2E: button is visible, fixed position at bottom-right, settings panel opens and closes via click and keyboard, Escape behaves correctly.