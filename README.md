# Home Build Review

A free, self-hosted collaborative floor plan review tool for homeowners working with their builder. Drop notes on any point of the plan, tag them by room and priority, thread replies, draw markups, compare revisions side-by-side — all synced live across everyone looking at the plans.

## What this repo is

Three files:

| File | What it does |
|---|---|
| `index.html` | Setup wizard. Visit this in a browser, fill out the form, download your customized `review.html`. |
| `template.html` | Source template the wizard reads. Contains `{{PLACEHOLDERS}}` the wizard fills in. |
| `README.md` | This file. |

## Using it

1. Host these three files on GitHub Pages (or anywhere that serves static files).
2. Visit `index.html`. Fill out the wizard.
3. Download the generated `review.html`.
4. Put `review.html` in your own repo along with your floor plan images (PNGs named like `v1-sheet-1.png`, `v1-sheet-2.png`, etc.).
5. Enable GitHub Pages on your repo, share the URL.

The wizard has its own detailed setup walkthrough at the bottom of the page — Firebase account, security rules, image naming, the whole thing.

## Features

- Live-synced notes with priority (high/medium/low), category (question/change/confirmed/on-hold), and room tagging
- Freehand drawing, arrows, rectangles, text labels, measurement tool
- Hearts (quick love-it markers) and photo overlays
- Threaded replies on any note
- Pre-populated review checklist (layout, kitchen, bathrooms, electrical, exterior, mechanical)
- Multi-revision + multi-sheet navigation (every revision's notes are preserved)
- Mobile responsive
- Print-friendly report export
- Read-only HTML snapshot export (embed the current view with all notes for emailing to anyone)

## Backend

Firebase Firestore. Free tier is plenty for a family-and-builder review group. You'll create your own Firebase project during setup — no login required for reviewers (they pick a name when they open the page). If you need real authentication, you'd need to modify the template.

## License

Use it however. No warranty.
