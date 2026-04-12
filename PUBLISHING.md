# Publishing Updates

How the catalyst-updates system works and how to publish new releases.

## How It Works

The updates page (`index.html`) is a static site hosted on GitHub Pages. It fetches all content from two JSON files at load time:

- **`data.json`** — page content: stack services, tips, changelog entries, feedback section
- **`version.json`** — current version number + last modified timestamp

Studio's control panel embeds this page in an iframe and passes the user's installed version as a query parameter (`?v=0.0.9`). The page compares the installed version against `version.json` and shows either an "up to date" or "update available" banner.

## Publishing a New Release (Automated)

The easiest way to publish an update is via the GitHub Actions workflow.

1. Go to **GitHub → Actions → "Publish Update"**
2. Click **"Run workflow"**
3. Fill in:
   - **Version**: new version number (e.g. `0.2.0`)
   - **Changelog**: one item per line, format `type: description`
     - Types: `feat`, `fix`, `ui`, `infra`
4. Click **"Run workflow"**

The workflow will:
- Prepend a new changelog entry to `data.json`
- Demote the previous `CURRENT` tag to its version number
- Update `version.json` with the new version + timestamp
- Commit and push to `main`
- GitHub Pages auto-deploys

### Example changelog input

```
feat: Webhook connector for outbound events
feat: Retry with exponential backoff on 429 responses
fix: Fixed timeout on large AI responses
ui: Improved connector error messages in Cockpit
```

## Publishing a New Release (Manual)

For changes beyond changelog (stack services, tips, feedback), edit the files directly.

### 1. Edit `data.json`

**Add a changelog entry** — prepend to the `changelog` array:
```json
{
  "version": "0.2.0",
  "date": "2026-05-15",
  "tag": "CURRENT",
  "items": [
    { "type": "feat", "text": "Description of new feature" },
    { "type": "fix", "text": "Description of bug fix" }
  ]
}
```

Change the previous first entry's `"tag"` from `"CURRENT"` to its version number (e.g. `"0.1.0"`).

**Update stack services** — edit the `stack` array (only when Docker services change).

**Update tips** — edit the `tips` array (one random tip shown per page load).

**Update feedback** — edit the `feedback` object.

### 2. Edit `version.json`

```json
{
  "version": "0.2.0",
  "lastModified": "2026-05-15T14:30:00Z"
}
```

Both fields matter:
- `version` — used by the updates page for version comparison
- `lastModified` — used by Studio's badge to show "new updates" dot

### 3. Commit and push to `main`

GitHub Pages deploys automatically.

## Changelog Types

| Type    | Label          | Color  | Use for                                  |
|---------|----------------|--------|------------------------------------------|
| `feat`  | Feature        | Cyan   | New capabilities, connectors, integrations |
| `fix`   | Fix            | Red    | Bug fixes                                |
| `ui`    | UI             | Purple | Visual changes, UX improvements          |
| `infra` | Infrastructure | Grey   | Docker, deployment, internal plumbing    |

## How Studio Detects Updates

1. Studio's `useUpdatesBadge.ts` polls `version.json` every 4 hours
2. It reads `lastModified` and compares to the last seen timestamp
3. If newer, it shows a notification dot on the updates sidebar item
4. When the user clicks through, the updates page loads in an iframe with `?v=<installed-version>`
5. The page fetches `version.json`, compares versions, and shows the appropriate banner

## Testing Locally

```bash
cd ~/Repos/catalyst-updates
python3 -m http.server 8899
```

Then open:
- `http://localhost:8899/` — no version param, default state
- `http://localhost:8899/?v=0.0.9` — simulates outdated install (update available)
- `http://localhost:8899/?v=0.1.0` — simulates current install (up to date)

## File Structure

```
catalyst-updates/
├── index.html                          # Updates page (fetches from JSON)
├── data.json                           # All page content
├── version.json                        # Version + timestamp
├── PUBLISHING.md                       # This file
└── .github/workflows/publish-update.yml # Automated publishing
```
