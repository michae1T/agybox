# agybox (shortcut: `agyb`)

Single-file wrapper for running Google Antigravity CLI (`agy`) with isolated
session directories and an isolated `~/.gemini` brain/conversation state—on both
**Android** (Termux + Fedora `proot-distro`) and **desktop Linux** (`proot`,
`bwrap`, or isolated `HOME`, no Docker required).

By default on Linux, sessions and isolated `~/.gemini` state live together in a
non-hidden, standard folder under `~/Documents/agybox/`:
* `~/Documents/agybox/sessions/<name>` — session working directories
* `~/Documents/agybox/state/gemini/` — entire isolated `~/.gemini` state
  (`brain/`, `conversations/`, `knowledge/`, `settings.json`, `config/projects/`,
  `daemon/`) so your default `~/.gemini` work history/config is never touched or
  clobbered.

On Termux where `~/sessions` already exists, `agybox` preserves `~/sessions`
(and `~/sessions/.agy-state`) unless overridden with environment variables.

## Install

Linux (`~/.local/bin/agybox` + `agyb`):

```bash
curl -fsSL https://raw.githubusercontent.com/michae1T/agybox/main/agybox -o /tmp/agybox && bash /tmp/agybox install && rm /tmp/agybox
```

Android Termux (`$PREFIX/bin/agybox` + `agyb`, requires `proot-distro` with `fedora` and `agy` installed inside Fedora):

```bash
curl -fsSL https://raw.githubusercontent.com/michae1T/agybox/main/agybox -o ~/agybox && bash ~/agybox install
```

## Environment Variables

* `AGYBOX_DIR` — base folder (default: `${XDG_DOCUMENTS_DIR:-$HOME/Documents}/agybox`)
* `AGYBOX_SESSIONS` — override sessions folder (`AGYBOX_DIR/sessions`)
* `AGYBOX_BRAIN` — override isolated brain/state folder (`AGYBOX_DIR/state`)
* `AGYBOX_MODE` — `auto` (default), `proot-distro`, `proot`, `bwrap`, or `home`

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

## Per-session exclusions & common agent rules

* Every session folder gets its own `.backupignore` (`rclone --exclude-from` syntax).
* `GEMINI.md` in the sessions root is copied into the isolated `config/GEMINI.md`
  so every chat starts with instructions to update `.backupignore` for heavy dirs,
  most binaries (unless small helpers), and sensitive files (passports/IDs, bank
  details, credentials), and to ask for confirmation when in doubt.

## Google Drive backup (`backup` / `restore`)

Uses `rclone` with the least-privilege `drive.file` OAuth scope (so `rclone` can
only access files it creates). On Termux it opens the Android browser via
`BROWSER=termux-open-url`. Backups upload `brain/`, `conversations/`,
`knowledge/`, `config/`, and `settings.json` first, then `sessions/` filtered by
global + per-session `.backupignore` and `--exclude-if-present .nobackup`.
