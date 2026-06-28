# How I built this site — a replicable walkthrough

A no-build, single-file portfolio site: **plain HTML + Tailwind (via CDN) + a little vanilla JavaScript**, hosted free on **GitHub Pages**. No frameworks, no build step, no Node. If you can edit a text file and push to GitHub, you can build this.

Everything lives in one file: [`index.html`](index.html). That's deliberate — it's the easiest thing in the world to understand, copy, and maintain.

---

## 0. What you need

- A code editor (**VS Code** is free and great)
- A free **GitHub** account
- ~Basic comfort opening a file and changing text. That's it.

---

## 1. The skeleton

A static HTML page has two halves:

```html
<!DOCTYPE html>
<html lang="en">
<head>   <!-- settings: title, fonts, styles, meta tags -->  </head>
<body>   <!-- the actual content you see -->  </body>
</html>
```

Open `index.html` and you'll see exactly this shape. Everything below is just *filling those two halves in*.

---

## 2. Styling without a build step: Tailwind via CDN

Instead of writing tons of CSS, this uses [Tailwind CSS](https://tailwindcss.com) loaded straight from a CDN — one line in the `<head>`:

```html
<script src="https://cdn.tailwindcss.com"></script>
```

Now you can style with utility classes right in the HTML, e.g. `class="text-xl font-semibold mb-4"`. No compiling.

> **Note:** the CDN build is perfect for a personal site. For a big production app you'd switch to a compiled Tailwind file, but that's overkill here.

### Custom fonts
Loaded from Google Fonts in the `<head>` (Inter for body, Fraunces for serif headings, JetBrains Mono for code/tickers), then registered in a small `tailwind.config` block so you can write `font-serif`, `font-mono`, etc.

---

## 3. The design system: colour tokens as CSS variables

The whole site's colour palette is defined **once**, as CSS variables, in the `<style>` block:

```css
:root {
  --paper: #fafaf9;   /* page background */
  --ink:   #0a0a0a;   /* main text       */
  --muted: #6b6b6b;   /* secondary text  */
  --line:  #e7e5e4;   /* borders         */
  --accent:#1e3a8a;   /* navy highlight  */
}
```

Then Tailwind is told to use those variables:

```js
colors: {
  paper: 'var(--paper)', ink: 'var(--ink)', muted: 'var(--muted)',
  line: 'var(--line)', accent: 'var(--accent)',
}
```

**Why this matters:** change one variable and the whole site updates. It's also what makes dark mode (below) a two-line job instead of a rewrite.

---

## 4. The content sections

The body is just a series of `<section>` blocks — About, Experience, Projects, etc. — each following the same pattern, so adding a new one is copy-paste. A reusable heading style (`.section-h` + numbered `.section-num`) keeps them consistent.

To add a project, you copy one card `<div>` and change the text. That's the whole "CMS."

---

## 5. Dark mode (the satisfying part)

Three ingredients:

**a) A second palette.** Define dark values for the same variables, scoped to `html.dark`:

```css
html.dark {
  --paper: #0e0e0d;  --ink: #ededec;  --accent: #9db8ff; /* …etc */
}
```

Because every element already reads from the variables (step 3), adding `dark` to the `<html>` tag re-themes the *entire* site instantly.

**b) A toggle button** in the nav that flips the class and remembers your choice:

```js
const isDark = document.documentElement.classList.toggle('dark');
localStorage.setItem('theme', isDark ? 'dark' : 'light');
```

**c) No "flash" on load.** A tiny script in the `<head>` (runs *before* the page paints) applies the saved theme — or the visitor's OS preference — so dark-mode users never see a white flash:

```js
const t = localStorage.getItem('theme');
if (t === 'dark' || (!t && matchMedia('(prefers-color-scheme: dark)').matches))
  document.documentElement.classList.add('dark');
```

---

## 6. Scroll-in animations

Sections fade up as you scroll, using the browser's built-in **`IntersectionObserver`** — no library. Elements start with a `.reveal` class (invisible, nudged down); when they enter the viewport, JS adds `.visible` and CSS transitions them in.

---

## 7. The favicon (the little tab icon)

It's an **SVG** — a tiny *text* file that draws the shape, so it's razor-sharp at any size and trivial to recolour. See [`favicon.svg`](favicon.svg): a white rounded square with navy initials. Wired up with:

```html
<link rel="icon" type="image/svg+xml" href="favicon.svg" />
```

---

## 8. Social link previews (Open Graph)

When you paste the link into WhatsApp/LinkedIn, it shows a title, description, and image. That comes from **Open Graph meta tags** in the `<head>`:

```html
<meta property="og:title" content="…" />
<meta property="og:description" content="…" />
<meta property="og:image" content="https://…/og-card.png" />
```

> **Gotcha:** the preview image **must be a PNG/JPG** — social platforms reject SVG here. This repo uses a 1200×630 `og-card.png`.
> **Gotcha 2:** these previews are cached hard. To force a refresh, share the link with a tweak like `?v=2`, or use LinkedIn's [Post Inspector](https://www.linkedin.com/post-inspector/).

---

## 9. The live market ticker (optional flair)

A scrolling strip of live prices using **TradingView's free embeddable widget** — one script, no API key, no backend. It's rebuilt on theme-toggle so it matches light/dark. (Live data on a static site is otherwise tricky, because API keys can't safely live in public client code — TradingView sidesteps that.)

---

## 10. Privacy-friendly analytics

[GoatCounter](https://www.goatcounter.com) — free, no cookies, no consent banner needed. One script tag counts visits. Custom JS events also track *which PDFs get opened* and *which sections people scroll to*, turning "1 visit" into real insight. (Anonymous and aggregate — it never identifies individuals.)

---

## 11. Publishing it free on GitHub Pages

1. Create a **public** repo named `yourusername.github.io`.
2. Put `index.html` at the repo **root**, push it.
3. Repo → **Settings → Pages → Source: `main` / root**.
4. Live in ~1 minute at `https://yourusername.github.io`.

**Why it's free:** static files are cheap to serve, and it's a loss-leader for GitHub's paid products. Fine for a personal/small site (within fair-use limits); *not* for e-commerce/payments (that needs a different host).

**Custom domain (optional):** buy one from a registrar (GitHub doesn't sell them), point its DNS at GitHub Pages, enter it under Settings → Pages. You get free HTTPS.

---

## 12. The update loop

```
edit index.html  →  git commit  →  git push  →  live in ~1 min
```

That's the entire maintenance story. No rebuilds.

---

## 13. One rule before you push

Everything in a public repo is **public and Google-indexable** — including every PDF. Only commit files you'd hand to a stranger. (Keep CVs with phone numbers, private logs, etc. *out* of the repo.)

---

## Tech summary

| Layer | Choice | Why |
|---|---|---|
| Markup | Plain HTML | Zero learning overhead |
| Styling | Tailwind (CDN) | No build step |
| Theming | CSS variables | One-line dark mode |
| Interactivity | Vanilla JS | No framework needed |
| Hosting | GitHub Pages | Free, `git push` to deploy |
| Analytics | GoatCounter | Private, no cookie banner |

Built with the help of **Claude Code**. Fork it, change the content, make it yours.
