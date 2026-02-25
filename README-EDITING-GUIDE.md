# README — Editing Guide for Static HTML Developers

A plain-English guide to understanding how this Astro project works
if you're coming from a traditional `/index.html`, `/about.html`,
`/css/style.css` setup.

No TypeScript. No React. No magic. Just a better way to write HTML pages.

---

## Table of Contents

1. [How Astro Differs from a Static HTML Project](#1-how-astro-differs-from-a-static-html-project)
2. [How Layout.astro Works](#2-how-layoutastro-works)
3. [What the slot Tag Does](#3-what-the-slot-tag-does)
4. [How Routing Works](#4-how-routing-works)
5. [Where Global Styles Go](#5-where-global-styles-go)
6. [How to Create Reusable Components](#6-how-to-create-reusable-components)
7. [How to Edit the Navbar Dropdown](#7-how-to-edit-the-navbar-dropdown)
8. [When to Use .astro vs Plain .html](#8-when-to-use-astro-vs-plain-html)
9. [How to Add Simple JavaScript](#9-how-to-add-simple-javascript)
10. [Best Practices for Transitioning from Static Sites](#10-best-practices-for-transitioning-from-static-sites)

---

## 1. How Astro Differs from a Static HTML Project

Here is a direct side-by-side comparison so you can map what you already
know onto how Astro works.

### Old Static HTML Way

```
my-website/
├── index.html          ← copy/paste <head>, <header>, <footer> in here
├── about.html          ← copy/paste <head>, <header>, <footer> in here AGAIN
├── contact.html        ← copy/paste <head>, <header>, <footer> in here AGAIN
├── css/
│   └── style.css       ← one big stylesheet for everything
└── js/
    └── script.js       ← one big script file for everything
```

Every time you changed a nav link, you opened every `.html` file and edited
each one. If you had 10 pages, you made 10 edits. Easy to forget one.

### Astro Way

```
src/
├── layouts/
│   └── Layout.astro    ← write <head>, <header>, <footer> ONCE here
├── components/
│   ├── Navbar.astro    ← navbar lives here — edit once, updates everywhere
│   └── Footer.astro    ← footer lives here — edit once, updates everywhere
├── pages/
│   ├── index.astro     ← only the UNIQUE content for this page goes here
│   └── about.astro     ← only the UNIQUE content for this page goes here
└── styles/
    └── global.css      ← your global stylesheet, same idea as style.css
```

**The key difference:** shared structure (head, nav, footer) is written once
in `Layout.astro` and every page reuses it automatically. You never copy and
paste structural HTML again.

### What an `.astro` File Looks Like

An `.astro` file has two parts separated by `---` fences:

```astro
---
// PART 1: The "frontmatter" — this runs on the server at build time.
// It's like the back-end logic of the page.
// You can import components, declare variables, fetch data, etc.
// This code NEVER ships to the browser.

import Layout from '../layouts/Layout.astro';

const pageTitle = "About Me";
---

<!-- PART 2: The HTML template — exactly like writing regular HTML. -->
<!-- You can use the variables from above with curly braces: {variable} -->

<Layout title={pageTitle}>
  <h1>{pageTitle}</h1>
  <p>This is just HTML!</p>
</Layout>
```

The `---` fences (three dashes) mark the beginning and end of the frontmatter
section. Everything after the closing `---` is HTML that you write exactly as
you always have.

**If you don't need any logic**, you can skip the frontmatter entirely and
just write plain HTML — or keep the fences empty:

```astro
---
import Layout from '../layouts/Layout.astro';
---

<Layout title="Simple Page">
  <h1>Hello!</h1>
</Layout>
```

---

## 2. How Layout.astro Works

Think of `Layout.astro` as your master HTML template — the file that contains
everything that should appear on every page of your site.

**Location:** `src/layouts/Layout.astro`

Here is a simplified version of what it contains:

```astro
---
// Accept props from individual pages
// title and description can be customized per page
const { title = "Default Title", description = "Default description" } = Astro.props;

import Navbar from '../components/Navbar.astro';
import Footer from '../components/Footer.astro';
import '../styles/global.css';
---

<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="description" content={description} />
    <title>{title}</title>
  </head>

  <body>
    <Navbar />        <!-- navbar appears on every page -->

    <main>
      <slot />        <!-- THIS is where each page's content goes -->
    </main>

    <Footer />        <!-- footer appears on every page -->
  </body>
</html>
```

**The traditional HTML equivalent would look like this** (repeated in every file):

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Page Title</title>
    <link rel="stylesheet" href="/css/style.css" />
  </head>
  <body>

    <!-- === COPY PASTE THIS HEADER INTO EVERY FILE === -->
    <header>
      <nav>...</nav>
    </header>
    <!-- ================================================ -->

    <!-- YOUR PAGE CONTENT HERE -->

    <!-- === COPY PASTE THIS FOOTER INTO EVERY FILE === -->
    <footer>...</footer>
    <!-- =============================================== -->

  </body>
</html>
```

With Layout.astro, you write that structure **once** and every page that
imports it gets the full structure automatically.

### Props: Customizing the Layout per Page

The `title` and `description` in the `<head>` are different on every page.
That's why Layout.astro accepts them as **props** (short for "properties") —
the same concept as HTML attributes, just for your own components.

Each page passes its own values:

```astro
<!-- src/pages/index.astro -->
<Layout title="Home | DevPortfolio" description="Frontend developer portfolio">
  ...
</Layout>

<!-- src/pages/services/solar.astro -->
<Layout title="Solar Websites | DevPortfolio" description="Websites for solar companies">
  ...
</Layout>
```

The layout receives those values and plugs them into the `<title>` and
`<meta description>` tags automatically. No manual editing of the `<head>` per
page needed.

---

## 3. What the `<slot />` Does

The `<slot />` tag is the single most important concept in Astro layouts.

**Plain English:** `<slot />` is a placeholder that says *"put the page's
content here."*

When a page does this:

```astro
<Layout title="About">
  <h1>About Me</h1>
  <p>I build websites.</p>
</Layout>
```

Astro takes everything between `<Layout>` and `</Layout>` and replaces the
`<slot />` tag in the layout with it. The final rendered HTML looks like:

```html
<body>
  <nav>...</nav>         <!-- from Navbar.astro -->

  <main>
    <h1>About Me</h1>    <!-- your page content replaces <slot /> -->
    <p>I build websites.</p>
  </main>

  <footer>...</footer>   <!-- from Footer.astro -->
</body>
```

**The `<slot />` is like a mail slot in a door.** The door (Layout) is always
the same. Whatever letter (page content) you push through the slot ends up
inside at that exact spot.

---

## 4. How Routing Works

Astro uses **file-based routing**. The location of a file inside `src/pages/`
directly determines its URL — no router setup, no configuration file.

### The Rules

| File path | URL in browser |
|---|---|
| `src/pages/index.astro` | `yourdomain.com/` |
| `src/pages/about.astro` | `yourdomain.com/about` |
| `src/pages/contact.astro` | `yourdomain.com/contact` |
| `src/pages/services/solar.astro` | `yourdomain.com/services/solar` |
| `src/pages/services/barbershop.astro` | `yourdomain.com/services/barbershop` |

### Anchor Links Within the Same Page

The homepage (`index.astro`) has sections with `id` attributes:

```html
<section id="about">...</section>
<section id="projects">...</section>
<section id="contact">...</section>
```

The navbar links to these sections using hash links:

```html
<a href="/#about">About</a>
<a href="/#projects">Projects</a>
<a href="/#contact">Contact</a>
```

The `/#about` format means: *go to the homepage (`/`) and scroll to the
element with `id="about"`*. The smooth scrolling behavior is enabled by
`scroll-behavior: smooth` in `global.css`.

### Linking Between Pages

Use standard HTML `<a>` tags — nothing special needed:

```html
<!-- Link to a service page -->
<a href="/services/solar">Solar Website</a>

<!-- Link back to homepage -->
<a href="/">Home</a>

<!-- Link to a homepage section from another page -->
<a href="/#contact">Contact</a>
```

---

## 5. Where Global Styles Go

**Location:** `src/styles/global.css`

This file is the equivalent of your old `css/style.css`. It is imported once
inside `Layout.astro` and automatically applied to every page on the site —
you never need to link it in individual page files.

```css
/* src/styles/global.css */

/* This single line imports ALL of Tailwind CSS v4 */
@import "tailwindcss";

/* You can write regular CSS below, exactly like you always have */
html {
  scroll-behavior: smooth;
}

body {
  font-family: ui-sans-serif, system-ui, sans-serif;
}

/* Custom classes you want available everywhere */
.my-custom-class {
  background-color: #1e293b;
  padding: 1rem;
}
```

### Tailwind CSS v4 — How It Differs from v3

In this project, Tailwind v4 is used. The biggest difference from v3 is:

- **No `tailwind.config.js`** — configuration is done in CSS, not JavaScript
- **One import line** — `@import "tailwindcss"` is all you need
- **Same utility classes** — `bg-blue-600`, `text-sm`, `flex`, `grid` all work
  exactly the same as before

If you want to define custom colors or spacing, you use the `@theme` directive
directly in `global.css`:

```css
@import "tailwindcss";

@theme {
  --color-brand: #2563eb;
  --color-brand-dark: #1d4ed8;
}
```

Then use them like any Tailwind class: `bg-brand`, `text-brand-dark`.

### Page-Specific Styles

If a single page needs styles that shouldn't apply globally, you can add a
`<style>` block at the bottom of that `.astro` file:

```astro
---
import Layout from '../layouts/Layout.astro';
---

<Layout title="About">
  <h1 class="page-title">About Me</h1>
</Layout>

<!-- This style block is SCOPED to this page only -->
<!-- It will NOT affect other pages -->
<style>
  .page-title {
    color: navy;
    font-size: 3rem;
  }
</style>
```

Astro automatically scopes `<style>` blocks so the CSS only applies to that
file. This is a big improvement over traditional CSS where every class is
global and can conflict.

---

## 6. How to Create Reusable Components

A **component** is just a piece of HTML that you want to use in multiple places
without copy-pasting. In the old HTML world, you'd have to copy-paste navbars,
cards, banners — and then update each copy when something changed.

In Astro, you create a `.astro` file in `src/components/`, then import and use
it wherever you need it.

### Example: Creating a Project Card Component

**Step 1 — Create the file:**

```
src/components/ProjectCard.astro
```

**Step 2 — Write the component:**

```astro
---
// Accept props so each card can have different content
const { title, description, imageUrl, projectUrl } = Astro.props;
---

<article class="bg-white rounded-2xl shadow-sm border border-slate-100 overflow-hidden">
  <img src={imageUrl} alt={title} class="w-full h-48 object-cover" />
  <div class="p-6">
    <h3 class="text-lg font-bold text-slate-900">{title}</h3>
    <p class="mt-2 text-sm text-slate-600">{description}</p>
    <a href={projectUrl} class="mt-4 inline-block text-blue-600 font-semibold">
      View Project →
    </a>
  </div>
</article>
```

**Step 3 — Use it in a page:**

```astro
---
import Layout from '../layouts/Layout.astro';
import ProjectCard from '../components/ProjectCard.astro';
---

<Layout title="Projects">
  <section class="py-20">
    <div class="grid grid-cols-3 gap-8">

      <ProjectCard
        title="Solar Co. Website"
        description="A lead-generation site for a solar company."
        imageUrl="https://placehold.co/600x360"
        projectUrl="/services/solar"
      />

      <ProjectCard
        title="Barbershop Website"
        description="A bold, modern site for a local barbershop."
        imageUrl="https://placehold.co/600x360"
        projectUrl="/services/barbershop"
      />

    </div>
  </section>
</Layout>
```

Now if you ever want to change how a project card looks, you edit
`ProjectCard.astro` **once** and every card on the site updates.

### When to Make Something a Component

Ask yourself: *"Will I use this HTML block more than once, or will it get
complicated enough that it deserves its own file?"*

If yes → make it a component.

Good candidates for components:
- Navbar ✅
- Footer ✅
- Project cards ✅
- Testimonial cards ✅
- Feature list items ✅
- A "hero" section you reuse across service pages ✅

---

## 7. How to Edit the Navbar Dropdown

The navbar is located at:

```
src/components/Navbar.astro
```

It is split into two separate menus:
1. **Desktop menu** — visible on medium screens and up (`md:flex`)
2. **Mobile menu** — visible only on small screens, toggled by a hamburger button

**Any time you edit the navbar, you must update both menus.**

### Adding a New Link

Find the desktop `<ul>` (the one with `class="hidden md:flex ..."`):

```html
<!-- Add your new link here -->
<li>
  <a href="/your-page" class="hover:text-blue-600 transition-colors">
    New Page
  </a>
</li>
```

Then find the mobile menu panel (`id="mobile-menu"`) and add the same link:

```html
<li>
  <a href="/your-page"
     class="block px-3 py-2.5 rounded-lg hover:bg-blue-50 hover:text-blue-600 transition-colors">
    New Page
  </a>
</li>
```

### Adding a New Item to the Services Dropdown

**Desktop dropdown** — find the comment `SERVICES DROPDOWN` and add inside
the dropdown `<ul>`:

```html
<li>
  <a href="/services/plumber"
     class="block px-4 py-3 text-sm text-slate-700 hover:bg-blue-50 hover:text-blue-600 transition-colors">
    🔧 Plumber Website
  </a>
</li>
```

**Mobile accordion** — find the comment `MOBILE SERVICES ACCORDION` and add
inside `id="services-mobile-menu"`:

```html
<li>
  <a href="/services/plumber"
     class="block px-3 py-2 rounded-lg hover:bg-blue-50 hover:text-blue-600 transition-colors text-slate-600">
    🔧 Plumber Website
  </a>
</li>
```

### How the Desktop Dropdown Works (CSS-only)

The dropdown uses Tailwind's `group` and `group-hover:` modifiers — no
JavaScript required on desktop:

```html
<!-- "group" on the parent <li> -->
<li class="relative group">
  <button>Services ▾</button>

  <!-- "invisible" by default, "visible" when parent is hovered -->
  <ul class="absolute invisible opacity-0 group-hover:visible group-hover:opacity-100 ...">
    <!-- dropdown items -->
  </ul>
</li>
```

When you hover over the `<li class="group">`, Tailwind automatically applies
the `group-hover:` styles to any child elements that use them. No JS needed.

### How the Mobile Menu Works (Minimal JavaScript)

The mobile hamburger button and the services accordion are controlled by a
small `<script>` block at the bottom of `Navbar.astro`. It uses plain
vanilla JavaScript — no libraries, no frameworks. Just `addEventListener`
and `classList.toggle`, the same JS you'd write in a `script.js` file.

---

## 8. When to Use `.astro` vs Plain `.html`

**Short answer: always use `.astro` in this project.**

Here is the full explanation:

| Situation | Use |
|---|---|
| Any page that uses Layout, Navbar, Footer | `.astro` |
| Any page that uses Tailwind classes | `.astro` |
| Any reusable section (card, hero, footer) | `.astro` |
| A totally standalone page with no shared layout | `.html` (works but rare) |
| A page you want to redirect from | `.astro` (use `Astro.redirect`) |

**Why not just use `.html` files?**

Plain `.html` files placed in `src/pages/` do work in Astro — they get served
as static files. But they can't:
- Import Layout.astro (so no shared navbar/footer)
- Use Tailwind classes (the CSS won't be generated for them)
- Use Astro components
- Accept or pass props

For 99% of the pages you'll build in this project, stick with `.astro`.

**The mental model:**

> If your old project had `about.html`, your new project has `about.astro`.
> Everything inside the `<body>` of the old file moves inside `<Layout>` in
> the new file. The `<head>` and repeated elements (nav, footer) are gone
> because Layout handles them.

---

## 9. How to Add Simple JavaScript

Astro gives you multiple ways to add JavaScript, from zero JavaScript to
full interactivity. For a portfolio site, you'll rarely need more than the
simplest option.

### Option 1 — Inline `<script>` tag in an `.astro` file

This is exactly like putting `<script>` at the bottom of an old HTML page.
Astro bundles it automatically.

```astro
---
import Layout from '../layouts/Layout.astro';
---

<Layout title="My Page">
  <button id="my-btn">Click me</button>
</Layout>

<!-- Script goes at the bottom of the file, outside the Layout tags -->
<script>
  const btn = document.getElementById('my-btn');
  btn.addEventListener('click', () => {
    alert('Hello!');
  });
</script>
```

This is what the Navbar.astro file does for the mobile menu toggle —
a small `<script>` block at the bottom handles the show/hide logic.

### Option 2 — External `.js` file in the `public/` folder

If you have a larger script, place it in `public/js/script.js` and link
to it with a `<script>` tag in `Layout.astro` or directly in a page:

```html
<!-- In Layout.astro <head> or at the bottom of <body> -->
<script src="/js/script.js" defer></script>
```

Files in `public/` are served exactly as-is, just like in a traditional
static site. The path `/js/script.js` in the browser maps directly to
`public/js/script.js` in your project.

### Option 3 — Inline `<script is:inline>` (for quick one-liners)

If you want a tiny script that runs immediately without Astro's bundling:

```astro
<script is:inline>
  console.log('Runs immediately');
</script>
```

Use this sparingly. The regular `<script>` tag is almost always better
because Astro optimizes it.

### What JavaScript Is Already Used

This project uses minimal JavaScript in just one place:

**`src/components/Navbar.astro`** — at the very bottom of the file, there
is a `<script>` block that handles:
1. Toggling the mobile menu open/closed when the hamburger is clicked
2. Toggling the services accordion in the mobile menu
3. Closing the mobile menu when a link inside it is clicked

That's it. The entire rest of the site is pure HTML and CSS — no JavaScript.
This is one of Astro's biggest advantages: it ships zero JavaScript to the
browser by default.

---

## 10. Best Practices for Transitioning from Static Sites

These are practical habits that will make your Astro projects clean,
maintainable, and easy to grow over time.

### DO: Write One Layout, Use It Everywhere

Never put `<head>`, `<html>`, `<body>`, a navbar, or a footer directly inside
a page file. That is the old copy-paste approach. Put all of that in
`Layout.astro` once and import it in every page.

```astro
<!-- Good ✅ -->
<Layout title="About">
  <section>...your unique content...</section>
</Layout>

<!-- Bad ❌ — don't do this in Astro pages -->
<!doctype html>
<html>
  <head>...</head>
  <body>
    <nav>...</nav>
    <section>...content...</section>
    <footer>...</footer>
  </body>
</html>
```

### DO: Give Every Section a Meaningful `id`

If the navbar links to a section with `href="/#about"`, make sure the section
actually has `id="about"`. This is standard HTML anchor behavior — Astro
doesn't change it.

```html
<section id="about" class="py-20">...</section>
<section id="projects" class="py-20">...</section>
<section id="contact" class="py-20">...</section>
```

### DO: Use Semantic HTML

Astro doesn't restrict you to `<div>` soup. Use the right HTML elements:
- `<header>` for the site header / navbar
- `<main>` for the page's main content (Layout.astro wraps `<slot />` in this)
- `<section>` for distinct page sections
- `<article>` for self-contained content like blog posts or project cards
- `<footer>` for the site footer
- `<nav>` for navigation menus
- `<h1>`–`<h6>` in logical order (one `<h1>` per page)

Good semantic HTML helps with SEO, accessibility, and readability.

### DO: Keep Pages Focused on Content

A page file should only contain content that is unique to that page. Shared
elements (navbar, footer, `<head>`) belong in Layout.astro or components.
The shorter and more focused a page file is, the easier it is to maintain.

### DO: Update Both Desktop and Mobile Menus Together

The Navbar has two separate lists — one for desktop and one for mobile. Any
time you add, remove, or rename a link, update both lists in the same edit.
It is easy to forget the mobile version and end up with a broken experience on
phones.

### DO: Use `placehold.co` for Images During Development

While building, use placeholder images so you can see layouts without hunting
for real assets:

```html
<!-- 600x400 image with a dark background and blue text -->
<img src="https://placehold.co/600x400/1e293b/60a5fa?text=My+Image" alt="..." />
```

Replace these with real images before launching.

### DON'T: Put Styles in Every Page's `<head>`

In the old HTML way, you might link `style.css` in every page's `<head>`.
In Astro, `global.css` is imported once in `Layout.astro` — it applies
to every page automatically. You do not need to touch `<head>` in page files.

### DON'T: Fear the Frontmatter

The `---` section at the top of `.astro` files looks unfamiliar but it is just
a place to write JavaScript that runs when the page is built. You only need it
for:
- Importing components or layout
- Declaring variables you want to use in the HTML
- Fetching data (advanced)

If you're not doing any of those things, the frontmatter can stay empty or be
left out entirely. Most simple content pages only use it for `import Layout`.

### DON'T: Worry About TypeScript

This project is configured to work with plain JavaScript. The `tsconfig.json`
file is there because Astro includes it by default for editor autocomplete,
but you will never need to write TypeScript syntax. All the code in this
project is plain JavaScript (and most of it is no JavaScript at all).

### DON'T: Use Multiple `<h1>` Tags per Page

For good SEO, each page should have exactly one `<h1>` — the main title of
that page. Use `<h2>` for section headings, `<h3>` for sub-sections inside
those, and so on. Skipping heading levels (e.g., going from `<h2>` to `<h4>`)
is also bad practice.

---

## Quick Reference Cheat Sheet

| What you want to do | Where to do it |
|---|---|
| Change the navbar | `src/components/Navbar.astro` |
| Change the footer | `src/components/Footer.astro` |
| Change `<head>` tags or SEO | `src/layouts/Layout.astro` |
| Change global styles | `src/styles/global.css` |
| Edit the homepage | `src/pages/index.astro` |
| Add a new page | Create `src/pages/my-page.astro` |
| Add a new service page | Create `src/pages/services/my-service.astro` |
| Add a reusable HTML block | Create `src/components/MyComponent.astro` |
| Add a static file (image, PDF) | Drop it in `public/` |
| Add JavaScript to one page | `<script>` block at the bottom of that `.astro` file |
| Add JavaScript to every page | `<script>` block in `Layout.astro` or import in `global.css` |

---

*You already know HTML and CSS. Astro is just a smarter way to organize them.*