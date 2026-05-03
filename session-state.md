# Session State — GW Photo

## CRITICAL: Two portfolio files exist. Edit the right one.

The site has **two separate portfolio implementations**, each with its own copy of `portfolioImages`, `portfolioRows`, and the `size-*` CSS classes:

| File | What it is | Visitor-visible? |
|---|---|---|
| `index.html` (homepage) | Hero + portfolio jump card. Has a hidden `<section id="portfolio-gallery" hidden>` that only renders when URL hash is `#portfolio-gallery`. | **No, normally invisible.** |
| `swedish-wildlife/index.html` | The actual portfolio gallery visitors see when they click "Portfolio". URL: `gwallinfoto.com/swedish-wildlife/`. | **Yes — this is THE portfolio.** |
| `portfolio/index.html` | Older landing page using european-robin as hero. Not actively used in the main flow. | Linked from homepage's portfolio jump, but lightly used. |

**Rule of thumb:** When the user says "the portfolio" or "the gallery" or describes layout changes to bird/wildlife images, they almost always mean **`swedish-wildlife/index.html`**.

If you make a layout change, mirror it to **both** `index.html` and `swedish-wildlife/index.html` so they don't drift apart, OR confirm with the user which one they want updated. Don't assume the homepage edit alone is sufficient — visitors will not see it.

## Today's incident (2026-05-03)

I spent the entire session editing the homepage's hidden portfolio gallery (`index.html`) believing it was the visible one. After commit and push, the user reported the live site looked unchanged. We chased browser cache, Cloudflare, DNS, Vercel deploy state for ~30 minutes before realizing the user was looking at `gwallinfoto.com/swedish-wildlife/index.html` — a different file we had never touched. Lesson encoded above.

## Hosting

- Repo: `Chonegyrame/GW-photo` on GitHub
- Deploys: **Vercel** (auto-deploys main branch on push, ~1 min). Visit URL on each deployment opens `gwallinfoto.com`.
- DNS: Vercel-managed for `gwallinfoto.com` (and `www.` 308-redirects to apex)
- The repo contains a leftover `<script src="/cdn-cgi/...">` line from a prior Cloudflare setup. It 404s on Vercel but does no harm. Safe to remove if cleanup is desired.

## Grid layout reference (applies to both index.html and swedish-wildlife/index.html)

The portfolio is rendered by inline JS at the bottom of each file. To change a card, edit `portfolioImages` (image source map) and `portfolioRows` (placement).

### `portfolioImages` — image source map
```js
{
  someKey: { src: '../images/file.jpg', alt: 'Description' },
  ...
}
```
- In `index.html` paths use `./images/...`
- In `swedish-wildlife/index.html` paths use `../images/...`
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

To add a new size, add a CSS rule `.portfolio-card.size-<name> { height: Xpx; }` and reference it in `portfolioRows`. Add to **both** files if visual parity matters.

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
Around line ~970 (homepage) / ~460 (wildlife) there's `getPortfolioRowsForViewport` that swaps `radjur` ↔ `spillkraka` on mobile (≤768px) so the heavy roe-deer card doesn't sit at the top of the right column on phones. If you reorder, verify that swap still makes sense.

## Today's layout changes (applied to both files as of 2026-05-03)

- New image: `images/spillkraka.jpg` (replaces old `black-woodpecker.jpg` for the spillkraka entry)
- New images tracked: `spillkraka.jpg`, `svartvit FS.jpg`, `duvhok.jpg`, `goktyta.jpg`
- Layout:
  - **Col 1:** svartvitFS (hero-compact 500, focus left) → tofsmes (large) → ekorre (medium) → stjartmesInsekt (large) → radjur (tall-soft) → storhack (large)
  - **Col 2:** stjartmesIAl (large-soft 400) → skaggmes (medium) → duvhok (tall) → notvacka (medium) → kor (medium) → tornfalk (medium) → entita (small) → gardsmyg (medium, wildlife page only)
  - **Col 3:** spillkraka (medium) → goktyta (medium) → steglits (medium) → isISol (small) → blames (medium) → koltrastISno (tall) → mindreHackspett (medium) → stenknack (tall)
- Size class added: `size-hero-compact` changed from 540 → 500. New `size-large-soft` at 400px.

## Verification workflow

There's a preview server at port 8080 configured in `.claude/launch.json`. Use:
- `preview_start` with name `gw-photo` to start it
- `preview_eval` with `window.location.href = '/swedish-wildlife/index.html'` to navigate
- `preview_eval` with DOM queries to inspect card layout
- `preview_screenshot` for visual checks

The homepage's hidden portfolio only renders when navigating to `/#portfolio-gallery`. Don't waste time previewing that — preview the wildlife page directly.

## Don't
- Don't edit only `index.html` for portfolio changes. Visitors don't see that.
- Don't change the `size-*` CSS class heights without checking which files reference them — both files have independent copies.
- Don't auto-delete the leftover `cdn-cgi` script tag without asking — it's harmless on Vercel but the user may want to keep it in case they revert hosting.
