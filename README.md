# Frontend App - Dependencies

## Tailwind CSS (via CDN)

- **What**: Utility-first CSS framework used for styling the Frontend app.
- **How it's included**: Added via the official CDN in `index.html` `<head>` so classes are available at runtime.
  - File: `index.html`
  - Tag added:

```html
<script src="https://cdn.tailwindcss.com"></script>
```

- **Usage**: Apply Tailwind utility classes directly in markup (e.g., `class="p-4 text-blue-600"`).
- **Why CDN now**: Quick to integrate without a build step. We can migrate to a package-based setup when we add more Frontend dependencies or need advanced configuration (custom theme, plugins, purge, etc.).
- **Docs**: See the Tailwind Play CDN docs: `https://tailwindcss.com/docs/installation/play-cdn`.

## Future additions

We plan to expand the Frontend dependencies. When that happens, we'll:
- Switch to an npm-based Tailwind install with PostCSS config (`tailwind.config.js`, `postcss.config.js`).
- Configure content paths for tree-shaking and add any required plugins.
- Document any new libraries and their setup steps here.