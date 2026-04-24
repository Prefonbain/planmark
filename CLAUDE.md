# Planmark — Setup Instructions for Claude Code

This repo is a template for a **collaborative floor plan review tool** for a homeowner + builder. The user has pointed you at it because they want to set up their own copy. Skip the browser wizard (`index.html`) — you can do the same substitution programmatically.

## What you're producing

A new GitHub repo (separate from this one) containing:
- A customized `review.html` (this repo's `template.html` with placeholders filled in)
- The user's floor plan images as PNGs (named `{versionPrefix}{sheetNumber}.png`, e.g. `v1-sheet-1.png`)

Hosted on GitHub Pages. One shareable URL per home build project.

## Setup flow

### 1. Collect info from the user

Ask for whatever they haven't already told you:

- **Project name** (e.g. "Smith Family Residence") — shown in the header
- **Builder name** (e.g. "Acme Home Builders") — shown in the subtitle
- **Reviewer names** — 3–6 people who'll use the tool; becomes quick-select buttons
- **Rooms list** — what rooms/areas exist in their plan. If they don't have a strong preference, offer this generic set as a default:
  ```
  Great Room, Dining, Kitchen, Pantry, Primary Bedroom, Primary Bath,
  Primary Closet, Bedroom #2, Bedroom #3, Full Bath, Mudroom, Laundry,
  Covered Patio, Deck, Garage, Entry / Foyer, Hallway, General / Whole Floor
  ```
- **Plan revisions** — each revision has: `id` (e.g. `v1`), `label` (e.g. "Version 1 — March 17, 2026"), `prefix` (e.g. `v1-sheet-`)
- **Sheets** — one per PDF page; each has `num`, `label` (tab name), `short` (e.g. `A-1`)
- **Floor plan image files** — ask where the PNGs live locally. If they only have a PDF, point them at a PDF-to-PNG tool; this template expects one PNG per sheet per revision.
- **GitHub repo name** for their new project (suggest a slug of the project name, e.g. `smith-family-review`)

### 2. Walk them through Firebase setup (you cannot automate this)

Firebase project creation requires Google auth in a browser. Relay these steps to the user and wait for them to paste back the config:

1. Go to <https://console.firebase.google.com>, sign in with a Google account
2. **Add project** → pick a name → skip Google Analytics
3. **Build → Firestore Database → Create database** → **Start in test mode** → pick a region
4. On the project overview page, click the `</>` **Web** icon to register a web app → name it anything → no need for Firebase Hosting
5. Copy the `firebaseConfig` snippet that appears — it has six values
6. After the site is live: lock down Firestore rules (see "Security rules" below)

### 3. Generate `review.html`

Read `template.html` and replace these placeholders. The template is self-contained — no build step, just string substitution.

| Placeholder | Value source | Notes |
|---|---|---|
| `{{PROJECT_NAME}}` | project name | HTML and template-literal contexts — safe for apostrophes |
| `{{PROJECT_NAME_JSON}}` | `JSON.stringify(projectName)` | Lands inside a JS single-quoted context; must be JSON-stringified |
| `{{BUILDER_NAME}}` | builder name | HTML text |
| `{{VERSION_OPTIONS_HTML}}` | `<option value="{id}">{label}</option>` per revision, one per line | HTML, escape with `escapeHtml()` |
| `{{SHEET_TABS_HTML}}` | `<button class="sheet-tab" data-sheet="{num}" onclick="switchSheet({num})">{label}</button>` | HTML |
| `{{ROOM_OPTIONS_HTML}}` | `<option value="{room}">{room}</option>` per room | HTML, used in 2 spots |
| `{{USER_PRESETS_HTML}}` | `<button class="name-preset" onclick="selectPreset('{name}',this)">{name}</button>` | If a reviewer name is literally "Guest" (case-insensitive), add `style="border-style:dashed;color:#888"` |
| `{{FIREBASE_CONFIG_JSON}}` | `JSON.stringify(firebaseConfig, null, 2)` | The 6-field object |
| `{{SHEETS_CONFIG_JSON}}` | `{ [num]: { label, short } }` as JSON | Map of sheet num → config |
| `{{VERSIONS_CONFIG_JSON}}` | `{ [id]: { label, prefix } }` as JSON | Map of version id → config |
| `{{DEFAULT_VERSION_JSON}}` | `JSON.stringify(defaultVersionId)` | Usually latest version |
| `{{DEFAULT_SHEET_JSON}}` | default sheet number (integer, no quotes) | Usually the floor plan sheet |
| `{{STORAGE_PREFIX}}` | slug of project name | Used for localStorage keys; `a-z0-9-` only |
| `{{EXPORT_SLUG}}` | project name with non-alphanumerics → `-` | Used in export filenames |

**Escape helpers** (same as `index.html` uses):
```js
const escapeHtml = s => String(s ?? '').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;').replace(/'/g,'&#39;');
const slugify = s => String(s).toLowerCase().trim().replace(/[^a-z0-9]+/g,'-').replace(/^-|-$/g,'').slice(0,40) || 'project';
const exportSlug = s => String(s).trim().replace(/[^a-zA-Z0-9]+/g,'-').replace(/^-|-$/g,'').slice(0,40) || 'Project';
const jsStringLiteral = s => "'" + String(s).replace(/\\/g,'\\\\').replace(/'/g,"\\'").replace(/\n/g,'\\n') + "'";
```

**Verification:** after substitution, run `new Function(scriptBody)` on the `<script>` tag contents to catch any JS parse errors before you hand off. If there are any remaining `{{...}}` matches, you missed a placeholder.

### 4. Create the new repo and push

The user needs the `gh` CLI authenticated, or a GitHub personal access token configured for git. Don't fall back to `--no-verify` or credential tricks — ask the user to authenticate first if push fails.

```bash
gh repo create {owner}/{repo-name} --public --description "Collaborative floor plan review for {project}"
mkdir -p /path/to/local/repo && cd /path/to/local/repo
git init -b main
# Write review.html here
# Copy the user's plan PNGs here, renamed to match the version prefixes and sheet numbers
git add .
git commit -m "Initial review site"
git remote add origin https://github.com/{owner}/{repo-name}.git
git push -u origin main
```

### 5. Enable GitHub Pages

```bash
gh api -X POST "repos/{owner}/{repo-name}/pages" -f "source[branch]=main" -f "source[path]=/"
```

Then poll `gh api "repos/{owner}/{repo-name}/pages" --jq '.status'` until it returns `built`. Usually 30–60 seconds.

### 6. Verify it's live

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://{owner}.github.io/{repo-name}/review.html
```

Expect `200`. Give the user that URL — that's what they share with their builder.

### 7. Firebase security rules (after first test)

Test mode expires in 30 days and lets anyone read/write. Once the user has verified the site works, tell them to go to Firebase console → **Firestore Database → Rules** and paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{collection}/{doc} {
      allow read: if true;
      allow write: if collection in [
        'notes', 'planHearts', 'markups',
        'photos', 'checklist'
      ];
    }
  }
}
```

Anyone with the URL can still read and write — this template has no login. That's appropriate for a trusted family-and-builder group; flag this to the user so they don't share the URL publicly.

## Adding a new revision later

When the builder sends updated plans:

1. Drop the new PNGs into the user's repo (e.g. `v4-sheet-1.png` through `v4-sheet-7.png`)
2. Re-run the substitution with the new revision added to the versions list
3. Commit and push

Existing notes on older revisions are preserved — they're scoped per revision in Firestore.

## Gotchas

- **Image paths are relative.** The template expects PNGs served next to `review.html` at the same origin. If the user wants to use an external image CDN, they'd need to edit `loadCurrentImage()` manually.
- **Firebase project ID → Firestore collections are shared across all tools using that same project.** If a user reuses one Firebase project for two planmark sites, notes will bleed between them. Use a fresh Firebase project per home build project.
- **The template removed the "Changes" sidebar tab** that the original Brio tool had — it was hardcoded version-diff narrative text that didn't generalize.
- **No auth.** Reviewers pick a name when they open the site; anyone with the URL can impersonate anyone. Fine for the intended use case (trusted review group).
