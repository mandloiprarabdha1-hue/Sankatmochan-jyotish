# Sankatmochan Jyotish Evam Anushthan Kendra — Website

A single self-contained `index.html`. All images are embedded, there is no build step.

---

## What changed in this revision

**1. Cursor is back to normal**
The custom gada-shaped PNG cursor has been removed. The site now uses standard system
cursors — `auto` for text and inputs, `pointer` for links, buttons and cards.

**2. The Simha Rashi symbol is now an actual Simha Rashi symbol**
The old image in the hero chip was the **LeoStar astrology-software logo** (lion head on a
purple star, sitting on a chart grid) — not a zodiac symbol. It has been replaced with a
clean inline SVG of the traditional Leo/Simha glyph (♌), drawn in the site's saffron.
Being SVG, it stays sharp at any size and costs almost nothing in file size.

**3. The "What is a Kundli?" illustration is now a real Kundli**
That section was showing the same LeoStar logo. It now shows a proper **North Indian style
chart** — outer square, both diagonals, inner diamond, all twelve Bhavas numbered 1–12
counter-clockwise from the Lagna, with house 1 tinted and labelled. Also inline SVG.

The LeoStar logo no longer appears anywhere in the file.

**4. Pandit Pankaj Mandloi's photo is blended into the page**
The hard ivory frame, border and drop shadow are gone. The portrait is now cropped to a
4:5 ratio and given a radial CSS mask so its edges dissolve into the cream background,
with a cream wash at the bottom, a warm saffron/maroon tint pass, a soft halo behind it
and a faint gold arc echoing the hero ring. The circular seal is kept as an accent.

**5. Favicon fixed**
The old favicon was a tall 240×855 crop of the gada, which squashed badly in browser tabs.
It is now a square 64×64 version of the center's own round logo. This also dropped roughly
400 KB from the page.

**6. Enquiry form is wired for Supabase** (see below)

**7. Interactive Sample Kundli reader**
A new **Sample** section (between Kundli and What is a Kundli) shows both sample
horoscopes — the 34-page Hindi जन्मपत्रिका and the 36-page English horoscope — with a
cover thumbnail, a page count, and a summary of what's inside. "Read sample" opens a
full-screen reader built on PDF.js.

The reader supports:

- Page-by-page navigation (‹ ›), a page-number box, and Home / End
- A lazily-rendered thumbnail strip you can click to jump, collapsible on desktop
- Zoom from 50% to 300%, plus a Fit toggle (fit-width ↔ fit-page)
- Switching between the Hindi and English samples without leaving the reader
- Keyboard control: ← → PageUp PageDown Home End Esc
- Download button for the original PDF
- Retina-sharp rendering (canvas is scaled by devicePixelRatio)
- On phones (Android and iOS): swipe left/right to turn pages, double-tap to
  zoom in and back out, pinch to zoom, and a single tap hides/shows the
  toolbars so the page fills more of the screen. Buttons are sized for
  comfortable tapping and the page-number box no longer triggers iOS's
  zoom-on-focus.

Nothing is downloaded until someone clicks "Read sample" — PDF.js and the PDF itself are
both fetched on first open, and each document is cached for the rest of the visit. If the
CDN is unreachable the reader shows a message pointing at the Download button instead of
failing silently.

---

## Important: the site is now a folder, not a single file

The PDFs live in `samples/` next to `index.html`:

```
sankatmochan-jyotish-site/
├─ index.html
├─ vercel.json
├─ README.md
└─ samples/
   ├─ kundli-sample-hi.pdf   (34 pages, Hindi)
   └─ kundli-sample-en.pdf   (36 pages, English)
```

Keep them together and keep the folder names as they are — `index.html` refers to
`samples/kundli-sample-hi.pdf` and `samples/kundli-sample-en.pdf` by relative path.

**To preview locally, use a web server, not a double-click.** Opening `index.html` straight
off your disk (`file://`) means the browser blocks the PDF fetch, so the reader will show
its fallback message. Run this in the folder instead:

```bash
python3 -m http.server 8000
```

then open http://localhost:8000. On Vercel it just works.

To swap in a different sample later, replace the file in `samples/` keeping the same
filename, and update the page count in `index.html` (search for `34 pages` / `36 pages`).

---

## Enquiry form now sends automatically (no redirect)

The form submits with `fetch()` in the background, so the visitor never leaves
the page or gets sent to a new tab — they just see the "Thank you" message
appear under the form.

By default the enquiry is emailed straight to **pandit11317@gmail.com**
through [FormSubmit.co](https://formsubmit.co), which needs no account and no
API key. **One-time step:** the very first enquiry triggers a confirmation
email from FormSubmit to `pandit11317@gmail.com` — open it and click
"Activate Form" once. Every enquiry after that is delivered straight away,
with no further setup.

If you'd rather also (or instead) store enquiries in a database you can
browse later, fill in `SUPABASE_URL` / `SUPABASE_ANON_KEY` near the bottom of
`index.html` and follow the Supabase steps below — with both configured, an
enquiry is sent to both places at once.

To change the destination email, edit `FORMSUBMIT_EMAIL` in the same
`<script>` block.

## Deploy: Supabase + GitHub + Vercel

### Step 1 — Create the Supabase table

In your Supabase project, open **SQL Editor** and run:

```sql
create table public.enquiries (
  id             bigint generated always as identity primary key,
  created_at     timestamptz not null default now(),
  name           text not null,
  phone          text not null,
  date_of_birth  date,
  time_of_birth  time,
  place_of_birth text,
  service        text,
  message        text,
  language       text
);

alter table public.enquiries enable row level security;

-- allow the public anon key to INSERT only; nobody can read rows from the browser
create policy "anon can submit enquiries"
  on public.enquiries
  for insert
  to anon
  with check (true);
```

Read enquiries from the Supabase **Table Editor**, not from the site. Because there is no
`select` policy, the anon key cannot read anyone's data back — this matters, since the form
collects birth details.

### Step 2 — Put your keys in the page

Open `index.html`, scroll to the `<script>` block at the bottom, and fill in:

```js
var SUPABASE_URL      = 'https://YOUR-PROJECT.supabase.co';
var SUPABASE_ANON_KEY = 'eyJhbGciOi...';   // Project Settings → API → anon public key
var SUPABASE_TABLE    = 'enquiries';
```

Use the **anon / public** key, never the `service_role` key — the anon key is designed to be
visible in a browser, the service_role key is not.

If you leave both blank the form still works: it just shows the thank-you message locally
and stores nothing.

### Step 3 — GitHub

```bash
cd sankatmochan-jyotish-site
git init
git add .
git commit -m "Sankatmochan Jyotish website"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
git push -u origin main
```

### Step 4 — Vercel

1. Go to https://vercel.com/new and import the GitHub repo.
2. Framework preset: **Other**. No build command, no output directory.
3. Deploy.

Every push to `main` redeploys automatically.

### Step 5 — Allow your domain in Supabase

Supabase → **Authentication → URL Configuration** → add your Vercel URL (and your custom
domain, if you add one) to the allowed site URLs so the browser request is not blocked.

---

## Editing content

Bilingual text is stored as pairs:

```html
<span class="en">English text</span><span class="hi">हिंदी टेक्स्ट</span>
```

Edit both, keep the structure identical, and redeploy.

## Notes

- All photos are base64-embedded, so the site remains one portable file.
- The Simha Rashi glyph and the Kundli chart are inline `<svg>` — edit their colours
  directly in the markup if you want a different shade.
- The photo blend uses CSS `mask-image`, supported in all current browsers. In anything
  very old the photo simply renders as a normal rectangle, which still looks fine.




### PDF.js reader

The site uses its built-in PDF.js reader for the sample Hindi and English Kundli PDFs. The external publication viewer has been removed.
