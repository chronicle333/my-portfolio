# DevPortfolio — Astro + Tailwind CSS Portfolio

A clean, fast, and beginner-friendly portfolio website built with [Astro](https://astro.build) and [Tailwind CSS v4](https://tailwindcss.com). Designed for frontend developers transitioning from static HTML sites.

---

## Table of Contents

- [Running the Dev Server](#running-the-dev-server)
- [Building for Production](#building-for-production)
- [How Routing Works in Astro](#how-routing-works-in-astro)
- [How Layout.astro Works](#how-layoutastro-works)
- [How to Add a New Page](#how-to-add-a-new-page)
- [How to Add a New Service](#how-to-add-a-new-service)
- [How to Edit the Navbar](#how-to-edit-the-navbar)
- [Deploying Your Site](#deploying-your-site)
- [Project Structure](#project-structure)

---

## Running the Dev Server

Start the local development server with live reload:

```bash
npm run dev
```

Your site will be available at **http://localhost:4321**

Any changes you save to `.astro`, `.css`, or other files will automatically reload in the browser — no manual refresh needed.

---

## Building for Production

When you're ready to publish your site, build it into static files:

```bash
npm run build
```

This outputs your finished site into the `dist/` folder. Every page becomes a plain `.html` file — no server required.

To preview the production build locally before deploying:

```bash
npm run preview
```

---

## How Routing Works in Astro

Astro uses **file-based routing** — the file path inside `src/pages/` becomes the URL.

| File | URL |
|---|---|
| `src/pages/index.astro` | `/` |
| `src/pages/about.astro` | `/about` |
| `src/pages/services/solar.astro` | `/services/solar` |
| `src/pages/services/restaurant.astro` | `/services/restaurant` |

**This is very similar to the old HTML way** — except `.astro` files replace `.html` files, and folders work exactly the same way.

> Old way: `/services/solar.html`
> Astro way: `/src/pages/services/solar.astro` → renders to `/services/solar`

No router configuration, no imports needed — just create the file in the right folder and the route exists automatically.

---

## How Layout.astro Works

In a traditional static HTML site, you had to copy and paste your `<head>`, `<header>`, and `<footer>` into every single HTML file. If you wanted to change one nav link, you had to update every file.

**Layout.astro solves this problem.**

```
src/layouts/Layout.astro
```

It contains:
- All the `<head>` meta tags and SEO tags
- The `<Navbar />` component
- The `<Footer />` component
- A `<slot />` tag — this is where each page's unique content appears

**How a page uses the Layout:**

```astro
---
import Layout from '../layouts/Layout.astro';
---

<Layout title="About Me" description="Learn about my background.">
  <h1>This content appears inside the layout</h1>
  <p>The navbar and footer are added automatically.</p>
</Layout>
```

Think of `<slot />` like a fill-in-the-blank placeholder. Whatever you put between `<Layout>` and `</Layout>` gets injected right where `<slot />` is written in the layout file.

The `title` and `description` props let each page set its own browser tab title and SEO description — without changing the layout file itself.

---

## How to Add a New Page

1. Create a new `.astro` file inside `src/pages/`
2. Import `Layout` at the top
3. Wrap your content in `<Layout>`

**Example — adding an `about.astro` page:**

```astro
---
// src/pages/about.astro
import Layout from '../layouts/Layout.astro';
---

<Layout title="About Me | DevPortfolio" description="Learn about my background and skills.">

  <section class="py-20 max-w-4xl mx-auto px-4">
    <h1 class="text-4xl font-bold text-slate-900">About Me</h1>
    <p class="mt-4 text-slate-600">Your content here...</p>
  </section>

</Layout>
```

The page is now live at `/about` — no config, no imports, nothing else needed.

---

## How to Add a New Service

1. Create a new `.astro` file inside `src/pages/services/`
2. Copy one of the existing service pages (e.g., `solar.astro`) as a starting point
3. Update the content: title, description, colors, feature list, and text
4. Add the link to the navbar (see next section)

**Example — adding a `plumber.astro` service:**

```
src/pages/services/plumber.astro  →  route: /services/plumber
```

Copy `src/pages/services/solar.astro`, rename it to `plumber.astro`, then find and replace the solar-specific content with plumbing content.

---

## How to Edit the Navbar

The navbar lives in one file:

```
src/components/Navbar.astro
```

**To add or change a regular nav link**, find the desktop `<ul>` section and edit the `<a>` tags:

```html
<li>
  <a href="/your-new-page" class="hover:text-blue-600 transition-colors">
    New Page
  </a>
</li>
```

Do the same in the **mobile menu** section further down in the same file — there are two separate lists (one for desktop, one for mobile).

**To add a new item to the Services dropdown:**

Find the desktop dropdown `<ul>` (look for the comment `SERVICES DROPDOWN`) and add a new `<li>`:

```html
<li>
  <a href="/services/plumber"
     class="block px-4 py-3 text-sm text-slate-700 hover:bg-blue-50 hover:text-blue-600 transition-colors">
    🔧 Plumber Website
  </a>
</li>
```

Then find the **mobile services accordion** (look for `MOBILE SERVICES ACCORDION`) and add the same link there too.

**To change the logo/brand name:**

Search for `DevPortfolio` in `Navbar.astro` and replace both occurrences (the colored span text) with your own name or brand.

---

## Deploying Your Site

After running `npm run build`, the `dist/` folder contains your complete static site. You can host it for free on any of these platforms:

### Netlify (Recommended — Easiest)

1. Go to [netlify.com](https://netlify.com) and create a free account
2. Drag and drop your `dist/` folder onto the Netlify dashboard
3. Your site is live instantly with a free `.netlify.app` URL

**Or connect your GitHub repo for automatic deploys:**
1. Push your project to GitHub
2. In Netlify: New Site → Import from Git → select your repo
3. Set build command: `npm run build`
4. Set publish directory: `dist`
5. Every push to `main` automatically rebuilds and redeploys

### Vercel

1. Go to [vercel.com](https://vercel.com) and sign in with GitHub
2. Import your repository
3. Vercel auto-detects Astro — no configuration needed
4. Click Deploy

### GitHub Pages

1. Push your project to a GitHub repository
2. Run `npm run build` locally
3. Push the contents of `dist/` to the `gh-pages` branch

Or use the [Astro GitHub Pages guide](https://docs.astro.build/en/guides/deploy/github/) for automatic GitHub Actions deployment.

### Custom Domain

All three platforms above allow you to connect a custom domain (e.g., `yourname.com`) for free through their dashboard settings.

---

## Project Structure

```
radiant-raspberry/
├── public/                   # Static files served as-is (favicon, images, fonts)
│   └── favicon.svg
│
├── src/
│   ├── layouts/
│   │   └── Layout.astro      # Master template: head, navbar, footer, slot
│   │
│   ├── components/
│   │   ├── Navbar.astro      # Site-wide navigation bar
│   │   └── Footer.astro      # Site-wide footer
│   │
│   ├── pages/
│   │   ├── index.astro       # Homepage  →  route: /
│   │   └── services/
│   │       ├── solar.astro        →  /services/solar
│   │       ├── barbershop.astro   →  /services/barbershop
│   │       ├── electrician.astro  →  /services/electrician
│   │       └── restaurant.astro   →  /services/restaurant
│   │
│   └── styles/
│       └── global.css        # Global styles + Tailwind v4 import
│
├── astro.config.mjs          # Astro configuration (Tailwind plugin registered here)
├── package.json
└── README.md
```

---

## Tailwind CSS Notes

This project uses **Tailwind CSS v4**. The only setup required is:

1. The plugin is registered in `astro.config.mjs` (already done)
2. `global.css` contains `@import "tailwindcss";` (already done)
3. `global.css` is imported inside `Layout.astro` (already done)

There is **no** `tailwind.config.js` file needed in v4. All configuration, if needed, is done directly in `global.css` using the `@theme` directive.