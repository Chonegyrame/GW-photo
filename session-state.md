# Session State — GW Photo

## CRITICAL: The portfolio lives in ONE file. Don't add a copy to the homepage.

The portfolio gallery is rendered ONLY by **`swedish-wildlife/index.html`**. That's the page visitors land on when they click "Portfolio".

| File | What it is | Visitor-visible? |
|---|---|---|
| `index.html` (homepage) | Hero + a single Portfolio jump card (goldeneye photo) that links to the wildlife gallery. **No portfolio code.** | The hero and jump card are visible. Nothing else. |
| `swedish-wildlife/index.html` | The actual portfolio gallery — `portfolioImages`, `portfolioRows`, all the layout JS, lightbox. URL: `gwallinfoto.com/swedish-wildlife/`. | **Yes — this is THE portfolio.** |
| `portfolio/index.html` | Older landing page using european-robin as hero. Linked from the homepage's Portfolio jump card. Uses background-image, not the portfolio JS. | Lightly used. |

**Rule of thumb:** When the user says "the portfolio" or describes layout changes to bird/wildlife images, they mean **`swedish-wildlife/index.html`**. The homepage no longer has portfolio code — don't restore it just because old reflexes say "match across files."

History note: the homepage used to have a hidden duplicate portfolio gallery (`<section id="portfolio-gallery" hidden>`), only visible when the URL had `#portfolio-gallery`. We removed it on 2026-05-03 because nobody ever saw it and it caused confusion (we burned ~30 min once thinking edits weren't deploying when they were just on the wrong file). Don't add it back without a clear reason.

## Today's incident (2026-05-03)

I spent the entire session editing the homepage's hidden portfolio gallery (`index.html`) believing it was the visible one. After commit and push, the user reported the live site looked unchanged. We chased browser cache, Cloudflare, DNS, Vercel deploy state for ~30 minutes before realizing the user was looking at `gwallinfoto.com/swedish-wildlife/index.html` — a different file we had never touched. Lesson encoded above.

## Hosting

- Repo: `Chonegyrame/GW-photo` on GitHub
- Deploys: **Vercel** (auto-deploys main branch on push, ~1 min). Visit URL on each deployment opens `gwallinfoto.com`.
- DNS: Vercel-managed for `gwallinfoto.com` (and `www.` 308-redirects to apex)
- The repo contains a leftover `<script src="/cdn-cgi/...">` line from a prior Cloudflare setup. It 404s on Vercel but does no harm. Safe to remove if cleanup is desired.

## Grid layout reference (in `swedish-wildlife/index.html` only)

The portfolio is rendered by inline JS at the bottom of `swedish-wildlife/index.html`. To change a card, edit `portfolioImages` (image source map) and `portfolioRows` (placement).

### `portfolioImages` — image source map
```js
{
  someKey: { src: '../images/file.jpg', alt: 'Description' },
  ...
}
```
- Paths are relative to `swedish-wildlife/`, so use `../images/...`
- Spaces in filenames must be URL-encoded as `%20`. Example: `'../images/svartvit%20FS.jpg'`.
- New keys are camelCase Swedish-ish (`stjartmesInsekt`, `spillkraka`, `goktyta`).

### `portfolioRows` — placement
A single `row-feature` row with three columns. Each column is an array of card configs:
```js
{ key: 'someKey', size: 'medium', focus: 'center', fit: 'contain' }
```
- `key` — must exist in `portfolioImages`
- `size` — see size classes below (required)
- `focus` — optional; controls `object-position` cropping (see focus classes)
- `fit` — optional; `'contain'` if the image must not be cropped at all

### Size classes (height in px)
| Class | Height | Notes |
|---|---|---|
| `hero` | 760 | Largest, dramatic anchor |
| `solo` | 620 | Single-card row |
| `tall` | 560 | Strong vertical |
| `hero-compact` | 500 | Top-of-column anchor; current top-left flycatcher |
| `tall-soft` | 520 | Slightly less than tall |
| `large` | 480 | Default "big" card |
| `large-soft` | 400 | Added 2026-05-03 — between large and medium |
| `medium` | 300 | Default "small" card |
| `small` | 210 | Compact filler |

To add a new size, add a CSS rule `.portfolio-card.size-<name> { height: Xpx; }` and reference it in `portfolioRows`. Also add a corresponding height in the `@media (max-width: 768px)` block (~25% smaller for mobile).

### Focus classes
| Focus | object-position | Use for |
|---|---|---|
| `center` | center center | default |
| `top` | center 22% | subject high in frame |
| `left` | 40% 50% | subject in left third |
| `right` | 42% 50% | subject in right third |
| `stjartmes` | 58% 48% | tuned for long-tailed-tit-with-insect |
| `stjartmesal` | 40% 50% (62% on mobile) | tuned for long-tailed-tit-in-alder |
| `koltrast` | 44% 50% | tuned for blackbird in snow |
| `radjur` | 34% 48% | tuned for roe deer |

Custom focus class for a single image: add a CSS rule `.portfolio-card.focus-<name> img { object-position: X% Y%; }`.

### Mobile behavior
On `swedish-wildlife/index.html`, `getPortfolioRowsForViewport` does two things on mobile (≤768px):
1. Swaps `radjur` ↔ `spillkraka` so the roe-deer card doesn't sit at the top of the right column. (Originally tuned for column-stacked mobile — re-evaluate if the interleave order makes you want different positions.)
2. **Interleaves columns left-to-right** — instead of stacking col1, then col2, then col3 in full, it takes col1[0], col2[0], col3[0], col1[1], col2[1], col3[1], … so a phone visitor experiences the rows in reading order rather than three big chunks.

Card heights in the mobile `@media` block are roughly 75% of their desktop values.

A simple **mobile-only Home link** (`.mobile-home-link`) is in the nav so visitors can return to the homepage without using the GW logo.

## Today's layout (applied to swedish-wildlife/index.html as of 2026-05-03)

- New image: `images/spillkraka.jpg` (replaces old `black-woodpecker.jpg` for the spillkraka entry)
- Newly tracked images: `spillkraka.jpg`, `svartvit FS.jpg`, `duvhok.jpg`, `goktyta.jpg`
- Layout (desktop, three columns):
  - **Col 1:** svartvitFS (hero-compact 500, focus left) → tofsmes (large) → ekorre (medium) → stjartmesInsekt (large) → radjur (tall-soft) → storhack (large)
  - **Col 2:** stjartmesIAl (large-soft 400) → skaggmes (medium) → duvhok (tall) → notvacka (medium) → kor (medium) → tornfalk (medium) → entita (small) → gardsmyg (medium)
  - **Col 3:** spillkraka (medium) → goktyta (medium) → steglits (medium) → isISol (small) → blames (medium) → koltrastISno (tall) → mindreHackspett (medium) → stenknack (tall)
- Size classes: `size-hero-compact` set to 500, new `size-large-soft` at 400.
- Mobile (≤768px): all heights reduced ~25%, columns interleave row-by-row, sticky Home link added top-left.

## Cleanup (2026-05-03)

The homepage's hidden `<section id="portfolio-gallery">` and all related code (CSS for `.portfolio-card`, `.portfolio-row`, `.portfolio-rows`, `.portfolio-gallery`, `.portfolio-column`, `.photo-overlay`, `.photo-species`, `.lightbox*`, `.section-header/label/title`; the `portfolioImages`/`portfolioRows`/`renderPortfolio`/`createPortfolioCard`/`getPortfolioRowsForViewport`/`openLightbox`/`closeLightbox`/`toggleZoom`/`openPortfolioFromHash` JS; and the lightbox `<div>`) were removed from `index.html`. The deploy-marker comment and the leftover Cloudflare `cdn-cgi/scripts/email-decode.min.js` script tag were also removed (no longer needed on Vercel). The wildlife page is unaffected.

## Verification workflow

There's a preview server at port 8080 configured in `.claude/launch.json`. Use:
- `preview_start` with name `gw-photo` to start it
- `preview_eval` with `window.location.href = '/swedish-wildlife/index.html'` to preview the portfolio
- `preview_eval` with DOM queries to inspect card layout
- `preview_resize` with preset `'mobile'` to test the 768px breakpoint
- `preview_screenshot` for visual checks

## Don't
- Don't add portfolio code back to `index.html`. Edit only `swedish-wildlife/index.html` for portfolio changes.
- When adding a new card size, update both the desktop CSS rule and the mobile `@media (max-width: 768px)` block (~25% smaller).
- Don't reintroduce the lightbox on the homepage; it's only used by the wildlife portfolio page.
