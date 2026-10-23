# Zenith

A Fortnite launcher for Windows: one clean screen to start the game and keep handy tools at hand.

![Logo](resources/icon-256.png)

## Features

### Stage 1 — wishlist and notifications

- **Wishlist.** Add items by searching for their name or with the ♥ button in an item window in the shop. The wishlist shows what's in the shop right now (price, time left) and, for everything else, when the item was last seen and a return forecast. Wishlisted items get a heart in the shop, and the menu shows how many of them are on sale today.
- **Discord notifications.** When a wishlist item shows up in the shop, Zenith posts a message to your Discord channel with the picture, price and when it leaves. You get one message per shop run, even if the item stays for a week. If you add an item that is already in the shop, Zenith waits for its next run.
- **Scheduled checks.** At 00:01 UTC (03:01 MSK) Zenith fetches the new shop and retries every 2 minutes until the API updates (up to an hour). It also checks every 30 minutes, on start and after the PC wakes up. A "Check now" button is on the wishlist screen and in the tray menu.
- **Tray and autostart.** The close button hides the window to the tray and Zenith keeps watching the shop; quit from the tray icon. The installed version starts with Windows, minimized to the tray. Both can be turned off in settings.

#### Connecting Discord

1. In Discord: Edit Channel → Integrations → Webhooks → New Webhook → Copy Webhook URL.
2. In Zenith: Settings → Notifications → paste the URL → Save → Send test.

The webhook URL is stored encrypted with Windows DPAPI: only your Windows account on this PC can read it.

### Stage 0 — the basics

- **Play button.** Launches Fortnite through Epic Games Launcher. If Epic or the game is missing, the button explains what to do and opens the Epic download page or the game page in Epic. While the game is running it shows "In game".
- **Item shop.** Today's shop as cards in the in-game tile colors, grouped by section, with search, type filters, a deals-only toggle and sorting. Badges for new items, discounts and items leaving soon. A countdown to the daily reset.
- **Item window.** Bundle contents and shop history: how many times the item was in the shop, first and last seen, a one-year timeline and a rough return forecast.
- **Offline mode.** Without internet, the last saved shop is shown.
- **Launcher settings:** English or Russian (interface and item names; English by default), window transparency with a level slider, accent color, background glow, reduce motion, Discord notifications, tray and autostart.

Stats, Game Settings, Map and Quests are placeholders for now and arrive in the next stages.

## What Zenith does not do

- It does not install or update the game — Epic Games Launcher does that.
- It does not bypass Epic and never touches your Epic login, password or tokens.
- It does not inject into the game or read its memory. It only checks the process name in the Windows task list to know the game is running.

## Requirements

- Windows 10/11.
- Node.js 22+ (development only).
- Epic Games Launcher with Fortnite installed, for the Play button.

## Run and build

```bash
npm install
```

```bash
npm run dev
```

The installer is written to `dist/Zenith-Setup-<version>.exe`:

```bash
npm run build:win
```

Other commands:

| Command | What it does |
| --- | --- |
| `npm run typecheck` | TypeScript type check |
| `npm run build` | build without packaging (into `out/`) |
| `npm run icons` | redraw the PNG icons from `src/renderer/src/assets/logo.svg` |

## How the game is launched

1. Zenith checks that the `com.epicgames.launcher` protocol is registered — that means Epic is installed.
2. It finds Fortnite in Epic's manifests: `%ProgramData%\Epic\EpicGamesLauncher\Data\Manifests\*.item`, falling back to `%ProgramData%\Epic\UnrealEngineLauncher\LauncherInstalled.dat`.
3. The Play button opens
   `com.epicgames.launcher://apps/fn%3A4fe75bbc5a674f4f9b356b5c90567da5%3AFortnite?action=launch&silent=true`
   with the ids taken from the manifest. Epic handles everything else (sign-in, anti-cheat, updates).

## Data sources

- The shop comes from the public [fortnite-api.com](https://fortnite-api.com) API (`/v2/shop?responseFlags=4`); the flag adds each item's shop history.
- Wishlist search uses `/v2/cosmetics/br/search/all`; fresh history for a single item comes from `/v2/cosmetics/br/{id}`.
- The shop resets daily at 00:00 UTC (03:00 MSK).
- The return forecast is the median gap between the item's previous shop runs. Epic does not publish a schedule, so it is an estimate.

## Where data is stored

`%APPDATA%\Zenith\zenith-data\`:

- `settings.json` — launcher settings;
- `wishlist.json` — the wishlist and the last check result;
- `secrets.json` — the Discord webhook URL, encrypted with DPAPI;
- `notify-status.json` — when the last message was sent and the last Discord error;
- `shop-en.v2.json`, `shop-ru.v2.json` — the last loaded shop for offline mode. The number is the cache format version: when it changes, old caches are simply ignored.

## Debugging

| Environment variable | Purpose |
| --- | --- |
| `ZENITH_PROGRAMDATA=<folder>` | replace `%ProgramData%` to see the "Epic not found" and "Fortnite not installed" states |
| `ZENITH_SNAPSHOT=<file.png>` | save a window screenshot after loading and quit |
| `ZENITH_SNAPSHOT_DELAY=<ms>` | delay before the screenshot (default 8000) |
| `ZENITH_SNAPSHOT_SCRIPT=<js>` | run JS in the window halfway through the delay, e.g. to open a card |

In snapshot mode, warnings and errors from the window are printed to the console, and Chromium's throttling of covered windows is turned off — otherwise a window opened behind others barely renders frames and animations freeze.

You can test Discord messages without a real channel: in development mode (not in the installed app) a local URL like `http://127.0.0.1:<port>/…` is accepted as a webhook. Run any HTTP server there that logs request bodies.

## Project structure

```
src/
  shared/                 shared by the main process and the UI
    types.ts              types and the IPC contract (window.api)
    shopTime.ts           shop reset time
    history.ts            shop history and return forecast
  main/                   Electron main process (Node)
    index.ts              window, tray, single instance, debug screenshots
    ipc.ts                handlers for UI requests (with input validation)
    settings.ts           launcher settings, window transparency, autostart
    tray.ts               tray icon and menu
    epic/launcher.ts      finding Epic and Fortnite, launching the game
    api/fortniteApi.ts    fortnite-api.com client: cached shop, search, single item
    wishlist/wishlist.ts  wishlist, shop checks, schedule
    notify/discord.ts     Discord webhook: URL validation, messages, 429 retries
    storage/store.ts      JSON files in the data folder
    storage/secrets.ts    encrypted secrets (DPAPI)
  preload/index.ts        bridge: window.api for the UI
  renderer/               UI (React + Tailwind)
    src/App.tsx           layout: title bar, menu, sections
    src/i18n/             en/ru dictionaries
    src/components/       menu, Play button, modal and so on
    src/features/shop/    shop: cards, filters, item window, history
    src/features/wishlist wishlist: search, cards, item window
    src/features/settings launcher and notification settings
    src/lib/              hooks and helpers (settings, wishlist, formatting)
    src/styles/theme.css  theme: black background, accents, glass, fonts
resources/                app icons and installer additions (installer.nsh)
scripts/make-icons.cjs    renders PNG icons from the SVG logo
```

Code comments are in Russian.

## Roadmap

1. ~~**Wishlist and Discord notifications.**~~ Done.
2. **Stats.** K/D, win rate and matches by mode, season-over-season progress, friend comparison. Needs a free API key from dash.fortnite-api.com.
3. **Game settings.** Back up and restore `GameUserSettings.ini` and `ClientSettings.Sav`, profiles, move to another PC in one file.
4. **Map.** Routes, loot and landing markers, squad plans with export and import.
5. **Quests and Battle Pass.** A manual quest list, tips on what to complete together, XP needed to reach your goal.

---

Fortnite is a trademark of Epic Games. Zenith is not affiliated with or endorsed by Epic Games.
