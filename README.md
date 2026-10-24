# Zenith

A Fortnite launcher for Windows: one clean screen to start the game and keep handy tools at hand.

![Logo](resources/icon-256.png)

## Features

### Map and item videos — Powered by Fortnite.gg

- **Map.** The Map section shows the [Fortnite.gg](https://fortnite.gg/) interactive map inside the launcher with all of the site's own UI hidden — just the map. On the right, a Zenith panel picks the map (Battle Royale, OG, Blitz, Reload) and toggles what's shown: chests, ammo boxes, vending and mending machines, launch pads, vehicles, locations, NPCs and more. Zoomed out, markers group into per-location counts; zoom in to see exact spots. The panel reads the available layers from the page and clicks the site's own (hidden) toggles, so it adapts to each map and season.
- **Item videos.** The "Video" button in an item window (shop and wishlist) opens the item's Fortnite.gg page with its video preview.
- The site runs in an isolated embedded view: its own session, no access to the launcher, only fortnite.gg pages open inside — other links open in your browser.

Embedding Fortnite.gg and cleaning up its frame is done **with permission from the Fortnite.gg team**, on the condition that a visible "Powered by Fortnite.gg" credit stays in the launcher. Don't remove it.

### Game settings

- **Change settings before you launch.** Resolution (including custom and stretched), window mode, frame rate limit, VSync, rendering mode (Performance, DirectX 11, DirectX 12), 3D resolution, view distance, shadows, textures, effects, post processing, motion blur, Show FPS, NVIDIA Reflex and volume levels. Changes collect in a draft; "Apply" writes only the changed lines of `GameUserSettings.ini` and leaves everything else in the file as it was. Editing works only while the game is closed, and Zenith backs up the current settings first.
- **Backups and profiles.** Save the current settings under a name ("Tournament", "Streaming") and restore them in one click. Before restoring, Zenith shows exactly what changes (for example, 1920×1080 → 2560×1080, 144 → 200 FPS).
- **Choose what to restore:** graphics, display and audio (`GameUserSettings.ini`) and/or controls, sensitivity and HUD (`ClientSettings.Sav`).
- **Safety net.** Restoring works only while the game is closed (Fortnite rewrites its settings on exit), and Zenith first saves your current settings to an automatic backup (the last 5 are kept).
- **Move to another PC.** Export a backup to a single `.zenithcfg` file and import it on the other PC.
- Fortnite also keeps controls, sensitivity and HUD in your Epic account and may replace a restored local copy with the cloud one when you sign in. Graphics settings are local only.

### Player search

Just type a name: with the platform set to Auto, Zenith checks Epic, PlayStation and Xbox in turn. While you type, suggestions show your account, friends and the last 40 players you found — names that start with the typed text first, with the match highlighted. Arrow keys pick a suggestion, Enter opens it.

There is no public API for searching Fortnite players by part of a name (Epic's own search requires signing in to an Epic account, which Zenith never does), so suggestions come only from players you have already found.

### Stage 2 — player stats

- **Any player.** Search by name on Epic, PlayStation or Xbox. Switch between this season and lifetime, between input types (all, keyboard & mouse, controller, touch) and modes (solo, duos, squads, LTMs).
- **Numbers and charts.** Matches, wins, win rate, K/D, kills, kills per match, score per minute, hours played; K/D and win rate by mode.
- **Season by season.** The Fortnite API only returns the current season, so Zenith saves a season snapshot for your account and your friends every time it loads their stats (and in the background every 6 hours). The history builds up from the moment you start using Zenith.
- **Compare with friends.** Add friends with "Compare" and see everyone side by side — a chart for the chosen metric and a table with all numbers.
- **Private stats.** If a player has hidden their stats, Zenith says so and explains where to turn them on in Fortnite (Settings → Account and Privacy → Show on Career Leaderboard).
- Damage isn't available in the API, so it isn't shown.

#### Where stats come from

Player stats come from fortnite-api.com, which needs an API key. Players never deal with it: a small Cloudflare Worker (`proxy/`) holds the key on the server, and the launcher only talks to the worker. The proxy URL is baked into the build (see [Build settings](#build-settings)); a build without it shows that stats are unavailable.

Versions before 0.3.0 let players paste their own key. Zenith no longer uses it and deletes a saved key on start.

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

Quests is a placeholder for now and arrives in the next stage.

## What Zenith does not do

- It does not install or update the game — Epic Games Launcher does that.
- It does not bypass Epic and never touches your Epic login, password or tokens.
- It does not inject into the game or read its memory. It only checks the process name in the Windows task list to know the game is running.
- Game settings backups only copy Fortnite's own settings files in `%LOCALAPPDATA%\FortniteGame\Saved` (`GameUserSettings.ini`, `ClientSettings.Sav`). Imported backups can only write those files.

## Requirements

- Windows 10/11.
- Epic Games Launcher with Fortnite installed, for the Play button.

## How the game is launched

1. Zenith checks that the `com.epicgames.launcher` protocol is registered — that means Epic is installed.
2. It finds Fortnite in Epic's manifests: `%ProgramData%\Epic\EpicGamesLauncher\Data\Manifests\*.item`, falling back to `%ProgramData%\Epic\UnrealEngineLauncher\LauncherInstalled.dat`.
3. The Play button opens
   `com.epicgames.launcher://apps/fn%3A4fe75bbc5a674f4f9b356b5c90567da5%3AFortnite?action=launch&silent=true`
   with the ids taken from the manifest. Epic handles everything else (sign-in, anti-cheat, updates).

## Data sources

- The shop comes from the public [fortnite-api.com](https://fortnite-api.com) API (`/v2/shop?responseFlags=4`); the flag adds each item's shop history.
- Wishlist search uses `/v2/cosmetics/br/search/all`; fresh history for a single item comes from `/v2/cosmetics/br/{id}`.
- Player stats come from `/v2/stats/br/v2` (by name) and `/v2/stats/br/v2/{accountId}` (your account and friends, so name changes don't break them) — through the Zenith proxy.
- The current season is taken from the newest items in `/v2/cosmetics/new` (for example, Chapter 7, Season 4 = game version 42).
- The shop resets daily at 00:00 UTC (03:00 MSK).
- The return forecast is the median gap between the item's previous shop runs. Epic does not publish a schedule, so it is an estimate.

## Where data is stored

`%APPDATA%\Zenith\zenith-data\`:

- `settings.json` — launcher settings;
- `wishlist.json` — the wishlist and the last check result;
- `secrets.json` — the Discord webhook URL, encrypted with DPAPI;
- `stats.json` — your account, friends and saved season history;
- `notify-status.json` — when the last message was sent and the last Discord error;
- `game-backups.json` and `game-backups\*.zenithcfg` — the list of game settings backups and the backups themselves (gzip-compressed JSON with the settings files; the same format is used for export);
- `shop-en.v2.json`, `shop-ru.v2.json` — the last loaded shop for offline mode. The number is the cache format version: when it changes, old caches are simply ignored.

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
    embed.ts              embedded Fortnite.gg view: allowed hosts, isolated session, frame clean-up CSS
    epic/launcher.ts      finding Epic and Fortnite, launching the game
    game/configFormat.ts  Fortnite settings files: allowed paths, reading and editing GameUserSettings.ini, .zenithcfg format
    game/gameConfig.ts    game settings: apply edits, backups, restore, export/import
    api/fortniteApi.ts    fortnite-api.com client: cached shop, search, single item
    wishlist/wishlist.ts  wishlist, shop checks, schedule
    stats/stats.ts        player stats via the proxy: errors, cache, friends, season history
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
    src/features/stats    stats: player search with suggestions, profile, charts (recharts), friend comparison
    src/features/game-settings  game settings editor, backups, restore dialog
    src/features/map      embedded Fortnite.gg map, layer panel, page bridge (read/toggle layers)
    src/features/settings launcher and notification settings
    src/lib/              hooks and helpers (settings, wishlist, formatting)
    src/styles/theme.css  theme: black background, accents, glass, fonts
proxy/                    Cloudflare Worker: stats proxy that keeps the API key server-side
resources/                app icons and installer additions (installer.nsh)
scripts/make-icons.cjs    renders PNG icons from the SVG logo
```

## Roadmap

1. ~~**Wishlist and Discord notifications.**~~ Done.
2. ~~**Stats.**~~ Done.
3. ~~**Game settings.**~~ Done.
4. ~~**Map.**~~ Done — powered by Fortnite.gg.
5. **Quests and Battle Pass.** A manual quest list, tips on what to complete together, XP needed to reach your goal.

## Credits

- Interactive map and item video previews — [Fortnite.gg](https://fortnite.gg/), embedded with the Fortnite.gg team's permission ("Powered by Fortnite.gg").
- Shop, cosmetics and stats data — [fortnite-api.com](https://fortnite-api.com).

---

Fortnite is a trademark of Epic Games. Zenith is not affiliated with or endorsed by Epic Games.
