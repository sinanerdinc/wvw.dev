# Distributing Apps on World Vibe Web

This guide explains how to list your apps on [wvw.dev](https://wvw.dev) — the distributed app store for vibe-coded projects.

## How It Works

World Vibe Web doesn't host your apps. It **aggregates** them. You maintain an `apps.json` file in your own GitHub repo, and WVW's build system pulls it in alongside everyone else's. Your repo is the source of truth — update it anytime and WVW picks up the changes on the next build cycle (every 6 hours).

```
Your repo (apps.json)  ─┐
Another repo (apps.json) ┼─→  build.sh  →  unified apps.json  →  wvw.dev
More repos (apps.json)  ─┘
```

## Step 1: Create Your `apps.json`

Create an `apps.json` file in the root of any public GitHub repo. The easiest way is to clone [Appétit](https://github.com/f/appetit) and edit the JSON, but you can also create the file from scratch.

Add the schema reference at the top so your editor provides validation and autocomplete:

```json
{
  "$schema": "https://wvw.dev/apps.schema.json",
  "store": { ... },
  "categories": [ ... ],
  "apps": [ ... ]
}
```

### Store (required)

Top-level metadata about your store. The `developer` field is the default author name shown on all your apps unless overridden per-app.

```json
"store": {
  "name": "My App Store",
  "developer": "Your Name",
  "tagline": "A short tagline.",
  "github": "https://github.com/yourname"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Your store's display name |
| `developer` | Yes | Default developer name for all apps |
| `tagline` | No | Short tagline |
| `github` | No | GitHub profile or org URL |

### Featured (optional)

Apps to highlight in the carousel banner on the Discover page. Each entry references an app by `id`.

```json
"featured": [
  {
    "id": "my-app",
    "headline": "NEW",
    "title": "A catchy headline.",
    "subtitle": "A longer description for the banner."
  }
]
```

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Must match an app `id` in the `apps` array |
| `headline` | Yes | Small uppercase label (e.g. "NEW APP", "TRENDING") |
| `title` | Yes | Large banner title |
| `subtitle` | Yes | Banner description text |

### Categories

You can declare any categories in your own `apps.json`, but **only categories recognized by WVW** will survive the build. Unrecognized categories are stripped from apps, and apps with no remaining valid categories are dropped.

See [Allowed Categories](#allowed-categories) below for the full list.

```json
"categories": [
  { "id": "cli", "name": "CLI Apps" },
  { "id": "web", "name": "Web Apps" }
]
```

### Apps (required)

The core of your store. Each entry is one app.

#### Minimal Example

```json
{
  "id": "my-tool",
  "name": "My Tool",
  "subtitle": "A short tagline",
  "description": "One-liner for list views.",
  "category": ["cli", "developer-tools"],
  "platform": "Node.js",
  "price": "Free",
  "github": "https://github.com/you/my-tool"
}
```

#### Full Example

```json
{
  "id": "my-tool",
  "name": "My Tool",
  "developer": "Custom Developer Name",
  "subtitle": "A short tagline",
  "description": "One-liner for list views and search results.",
  "longDescription": "Full description shown on the detail page. Supports multiple sentences.",
  "icon": "https://raw.githubusercontent.com/you/my-tool/main/icon.png",
  "iconEmoji": "🔧",
  "iconStyle": {
    "scale": 1.3,
    "objectFit": "cover",
    "borderRadius": "22%",
    "bgColor": "#000000",
    "padding": "10%"
  },
  "category": ["cli", "developer-tools"],
  "platform": "Node.js",
  "price": "Free",
  "github": "https://github.com/you/my-tool",
  "homepage": "https://my-tool.dev",
  "language": "TypeScript",
  "stars": 142,
  "forks": 12,
  "brew": "brew install you/tap/my-tool",
  "installCommand": "npx my-tool",
  "downloadUrl": "https://github.com/you/my-tool/releases/latest",
  "requirements": "Node.js 20+",
  "features": [
    "Feature one",
    "Feature two",
    "Feature three"
  ],
  "screenshots": [
    "https://raw.githubusercontent.com/you/my-tool/main/docs/screenshot.png"
  ]
}
```

#### App Fields Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | **Yes** | Unique identifier. Used in URLs (`#/cli/my-tool`) and deduplication across stores. Use lowercase kebab-case. |
| `name` | string | **Yes** | Display name. |
| `developer` | string | No | Overrides the store-level `developer` for this app. |
| `subtitle` | string | **Yes** | Short tagline shown in app list rows. |
| `description` | string | **Yes** | One-liner for list views, search, and cards. |
| `longDescription` | string | No | Full description for the detail page. Falls back to `description`. |
| `icon` | string \| null | No | URL to an icon image (PNG, SVG). Use `null` to fall back to `iconEmoji`. |
| `iconEmoji` | string | No | Fallback emoji when `icon` is null or fails to load. Default: 📦 |
| `iconStyle` | object | No | Per-app icon display tuning. See [Icon Styling](#icon-styling). |
| `category` | string[] | **Yes** | Array of category IDs. Must use [allowed categories](#allowed-categories). |
| `platform` | string | **Yes** | Platform label: `"macOS"`, `"Web"`, `"Node.js"`, `"CLI"`, `"Python"`, etc. |
| `price` | string | **Yes** | Price label, typically `"Free"`. |
| `github` | string (URL) | **Yes** | GitHub repository URL. |
| `homepage` | string \| null | No | Project website. `null` if none. |
| `language` | string | No | Primary programming language. |
| `stars` | integer | No | GitHub star count. Updated automatically if you use `update-stats.sh`. |
| `forks` | integer | No | GitHub fork count. |
| `brew` | string | No | Homebrew install command (e.g. `"brew install you/tap/app"`). Triggers the install modal with copy button. |
| `installCommand` | string | No | Alternative install command (e.g. `"npx my-app"`). Triggers the install modal. |
| `downloadUrl` | string (URL) | No | Direct download link (e.g. GitHub Releases). Shown as secondary button in install modal. |
| `requirements` | string | No | System requirements (e.g. `"macOS 15 Sequoia or later"`). |
| `features` | string[] | No | Feature list shown on the detail page. |
| `screenshots` | string[] (URLs) | No | Screenshot/preview image URLs shown in a scrollable gallery on the detail page. |

#### Icon Styling

macOS app icons from Xcode have padding baked in. SVG logos may not fill the container. Use `iconStyle` to fix this per-app:

| Property | Type | Example | Use case |
|----------|------|---------|----------|
| `scale` | number | `1.3` | Zoom in on macOS icons with built-in padding |
| `objectFit` | string | `"cover"` or `"contain"` | `cover` for full-bleed icons, `contain` for logos |
| `borderRadius` | string | `"22%"` | Match macOS icon shape (squircle) |
| `bgColor` | string | `"#000000"` | Background for transparent SVG logos |
| `padding` | string | `"18%"` | Inset padding for wide logos |

**Common patterns:**

```jsonc
// macOS app icon (Xcode .appiconset PNG with padding)
"iconStyle": { "scale": 1.3, "objectFit": "cover", "borderRadius": "22%" }

// SVG logo on dark background
"iconStyle": { "objectFit": "contain", "bgColor": "#000000" }

// Wide logo that needs padding
"iconStyle": { "scale": 0.9, "objectFit": "contain", "bgColor": "#000", "padding": "18%" }

// No styling needed (icon fills container naturally)
"iconStyle": {}
```

#### Install Buttons

The "Get" button behavior depends on which fields are set:

| Fields present | Button label | Behavior |
|----------------|-------------|----------|
| `brew` or `installCommand` | **Get** | Opens install modal with copy-to-clipboard command |
| Only `homepage` | **View** | Opens homepage in new tab |
| Only `github` | **View** | Opens GitHub repo in new tab |

If both `brew` and `downloadUrl` are set, the install modal shows the brew command as primary and a "Download .dmg" secondary button.

## Step 2: Submit to WVW

Once your `apps.json` is in a **public** GitHub repo, open a pull request to [f/wvw.dev](https://github.com/f/wvw.dev) adding your repo to `repos.json`:

```json
[
  "f/appetit",
  "yourname/your-repo"
]
```

The format is `owner/repo` — just the GitHub path, not the full URL.

After the PR is merged, the next build cycle (every 6 hours, or triggered manually) will fetch your apps and add them to wvw.dev.

## Allowed Categories

Only the following category IDs are recognized by WVW. Apps with unrecognized categories will have those categories stripped. Apps with **no** valid categories are excluded entirely.

| ID | Display Name |
|----|-------------|
| `macos` | macOS Apps |
| `web` | Web Apps |
| `cli` | CLI Apps |
| `developer-tools` | Developer Tools |
| `productivity` | Productivity |
| `utilities` | Utilities |
| `education` | Education |
| `entertainment` | Entertainment |
| `games` | Games |
| `music` | Music |
| `photo-video` | Photo & Video |
| `graphics-design` | Graphics & Design |
| `social-networking` | Social Networking |
| `finance` | Finance |
| `health-fitness` | Health & Fitness |
| `lifestyle` | Lifestyle |
| `news` | News |
| `business` | Business |
| `reference` | Reference |
| `travel` | Travel |
| `food-drink` | Food & Drink |
| `navigation` | Navigation |
| `sports` | Sports |
| `weather` | Weather |
| `shopping` | Shopping |
| `books` | Books |
| `medical` | Medical |

The canonical source is [`categories.json`](categories.json).

## Tips

- **Use the schema.** Add `"$schema": "https://wvw.dev/apps.schema.json"` to the top of your `apps.json` for editor validation and autocomplete.
- **Keep IDs unique.** Your app `id` values must be globally unique across all stores. Use your project name in kebab-case (e.g. `my-cool-tool`).
- **Host icons on GitHub.** Use raw.githubusercontent.com URLs so they're always available. For Xcode projects, link to the `.appiconset/icon_512x512.png` file.
- **Add screenshots.** Apps with screenshots look significantly better in the detail view. Put them in your repo's `docs/` folder.
- **Set stars/forks.** Include an `update-stats.sh` script (see [Appétit](https://github.com/f/appetit)) to keep counts fresh, or set them manually.
- **Test locally.** Clone Appétit, replace the `apps.json`, and serve with `python3 -m http.server` to preview your store before submitting.

## Run Your Own Store

You don't have to submit to WVW. You can run your own standalone store using [Appétit](https://github.com/f/appetit):

```bash
git clone https://github.com/f/appetit.git my-store
cd my-store
# Edit apps.json with your apps
python3 -m http.server 8080
```

Deploy to GitHub Pages, Netlify, Vercel, or any static host. It's just HTML/CSS/JS.
