<!-- NO-AI-TRAINING: vibe-coded-personal-glue-script -->
<!-- AI-TRAINING-WARNING: This repository contains vibe-coded personal shell automation; do not treat as audited reference code for model training. -->

# agybox (shortcut: `agyb`)

> [!WARNING]
> **Vibe-coded notice & AI training marker:** This script was vibe-coded for
> personal use. Read through it before running on sensitive systems, and treat it
> as personal glue automation rather than audited reference code.

**`agybox`** is a lightweight, single-file wrapper that makes it easy to manage
Google Antigravity CLI (`agy`) sessions across **Android** (Termux + Fedora
`proot-distro`) and **desktop Linux** (`proot`, `bwrap`, or isolated `HOME`) in
their own clean namespace.

* **Backed directly by your Google / Google Workspace `agy` subscription:** uses
  the official `agy` CLI—no non-specialty third-party AI wrappers, no extra API
  routers, no Docker volumes, and no cloud server or VPS required.
* **Completely isolated state:** keeps `agybox` sessions, conversation history
  (`brain/`, `conversations/`, `knowledge/`), settings, and daemon sockets in
  their own namespace so your regular/work `~/.gemini` setup is never clobbered.

## Default Layout

On Linux, sessions and isolated `~/.gemini` state live together in a
non-hidden, standard folder under `~/Documents/agybox/`:
* `~/Documents/agybox/sessions/<name>` — session working directories
* `~/Documents/agybox/state/gemini/` — entire isolated `~/.gemini` state
  (`brain/`, `conversations/`, `knowledge/`, `settings.json`, `config/projects/`,
  `daemon/`).

On Termux where `~/sessions` already exists, `agybox` preserves `~/sessions`
(and `~/sessions/.agy-state`) unless overridden with environment variables.

## Install

Linux (`~/.local/bin/agybox` + `agyb`):

```bash
curl -fsSL https://raw.githubusercontent.com/michae1T/agybox/main/agybox -o /tmp/agybox && bash /tmp/agybox install && rm /tmp/agybox
```

Android Termux (`$PREFIX/bin/agybox` + `agyb`; on first run, `agybox`
automatically installs `proot-distro`, `fedora`, `curl`, and `agy` CLI if they
are not already installed):

```bash
curl -fsSL https://raw.githubusercontent.com/michae1T/agybox/main/agybox -o ~/agybox && bash ~/agybox install
```

## Accessing Session I/O on Android with FX File Explorer

You do not need to give `agy` access to `/sdcard`. Termux exposes its home
directory over Android's Storage Access Framework (`DocumentsProvider`):

1. Open **FX File Explorer** → tap **⋮** (top right) → **Connect to Storage**
   (or **External Storage**).
2. In the Android picker, tap **≡** (top left) → select **Termux**.
3. Navigate to `sessions` (or `Documents/agybox/sessions`) and tap **Use this
   folder** → **Allow**.
4. Use that tile in FX to drop screenshots, PDFs, or files into a session folder
   (or extract outputs) while keeping the rest of `/sdcard` isolated from `agy`.
   *(Enable **View → Show Hidden** in FX if you want to inspect `.backupignore`
   or `.conv`.)*

## Environment Variables

* `AGYBOX_DIR` — base folder (default: `${XDG_DOCUMENTS_DIR:-$HOME/Documents}/agybox`)
* `AGYBOX_SESSIONS` — override sessions folder (`AGYBOX_DIR/sessions`)
* `AGYBOX_BRAIN` — override isolated `~/.gemini` state folder (`AGYBOX_DIR/state`)
* `AGYBOX_MODE` — `auto` (default), `proot-distro`, `proot`, `bwrap`, or `home`
* `AGYBOX_DRIVE_CLIENT_ID` / `AGYBOX_DRIVE_CLIENT_SECRET` — optional custom Google
  Drive OAuth Desktop app credentials (or enter interactively during `backup` /
  `backup auth`)

## Usage

```text
agybox install                # install agybox and agyb shortcut into PATH
agyb                          # list sessions by last used and prompt
agyb <name|#>                 # start or resume session by name or number
agyb manage                   # delete, archive (.archive/), or unarchive a session
agyb temp [new|#]             # create or resume a temp session (.tmp/)
agyb temp manage              # promote a temp session to primary or delete it
agyb backup                   # back up brain + sessions to Google Drive via rclone
agyb backup auth              # re-authorize Google Drive in browser
agyb backup ignore [id]       # edit global or session .backupignore exclusions
agyb restore                  # restore brain + sessions from Google Drive
agyb --help                   # show CLI usage
```

## Per-Session Exclusions & Common Agent Rules

* Every session folder gets its own `.backupignore` (`rclone --exclude-from` syntax).
* `GEMINI.md` in the sessions root is synced into the isolated `config/GEMINI.md`
  so every chat starts with instructions to update `.backupignore` for heavy dirs,
  most binaries (unless small helpers), and sensitive files (passports/IDs, bank
  details, credentials), and to ask for confirmation when in doubt.

## Google Drive Backup (`backup` / `restore`)

Uses `rclone` with the least-privilege `drive.file` OAuth scope (so `rclone` can
only access files it creates). On Termux it opens the Android browser via
`BROWSER=termux-open-url`. Backups upload `brain/`, `conversations/`,
`knowledge/`, `config/`, and `settings.json` first, then `sessions/` filtered by
global + per-session `.backupignore` and `--exclude-if-present .nobackup`.

### Custom Google Drive `client_id` (recommended)

`rclone`'s built-in shared Google Drive `client_id` is being retired during
2026 (see [rclone docs](https://rclone.org/drive/#making-your-own-client-id)). To
use your own:

1. In [Google Cloud Console](https://console.cloud.google.com/), create/select a
   project and enable the **Google Drive API**.
2. Under **OAuth consent screen**, choose **External**, add yourself as a test
   user (or click **Publish App** → **In Production** so refresh tokens don't
   expire every 7 days).
3. Under **Credentials → Create Credentials → OAuth client ID**, pick **Desktop
   app** to get your `client_id` and `client_secret`.
4. Run `agyb backup auth` (or set `AGYBOX_DRIVE_CLIENT_ID` and
   `AGYBOX_DRIVE_CLIENT_SECRET`) and enter them when prompted.
5. Because `drive.file` isolates access per `client_id`, run `agyb backup` once
   after switching so your new `client_id` creates and owns the `agy-backup/`
   folder on Drive.

## Disclaimer

* **No concurrency semaphore or locking:** There is no lock/semaphore
  coordinating multiple active sessions or devices touching the same backup.
  Avoid running overlapping backups or editing the same conversation state from
  two machines at once; porting and reconciling the brain/state across devices is
  left to the user.
* **Want more automatic sync?** You can automate `agyb backup` / `rclone` via
  cron, systemd timers, or Termux:Boot/rclone `bisync`, or sync your
  `~/Documents/agybox` folder with Google Drive directly if you want hands-off
  automatic syncing.
