# Setting up Helia

Two pipelines, deliberately separate: **using** the method installs a plugin and
never clones this repository; **developing** the method clones this repository
and never installs from the clone. Nothing you edit in a clone reaches your
other sessions until `scripts/deploy` has published it.

## 1. Prerequisites

- `git`
- `node` **or** `python3` — the scripts try `node` first and fall back to
  `python3`; either one alone is enough.
- `jq` — **required for the status line, the theme and the startup banner**: the startup hook edits
  `~/.claude/settings.json` with it and leaves the file untouched when `jq` is
  missing; the renderer falls back to a bare `helia · <folder> (<branch>)` line,
  where `<folder>` is the shell's folder (the process's current directory), not
  the session's working folder. `scripts/test-appearance` (gate 14) refuses to
  run without it. Everything else works without it.
- an SSH key loaded in `ssh-agent`, with `github.com` in `known_hosts`
- Claude Code ≥ 2.1.193
- **optional:** an `obsidian` plugin providing the `obsidian:obsidian-markdown`
  and `obsidian:obsidian-bases` skills, if the vault at `<vault>/` is an actual
  Obsidian vault and you want `helia.base`'s dashboard to work and wikilinks/
  callouts/properties written correctly. Without it, agents still write to
  `vault/` — the Obsidian-specific syntax invocations in their prompts just
  have nothing to invoke.

On Windows: **Git for Windows is a hard prerequisite** — without it the hook
cannot start and now says so in the transcript instead of failing silently.

## 2. Using Helia — two commands

```bash
claude plugin marketplace add Athyrr/helia
claude plugin install helia@athyrr-marketplace --scope user
```

The marketplace is a public `github` source, so no credentials are needed to
register or update it. Install in **one scope only** — two scopes means two copies and an
update touches one while the other keeps running.

`--scope user` makes the method available from any repository on the machine:
it is a way of working, not vault data.

That is the whole installation. You do **not** clone this repository to use the
method.

## 3. Registering a vault

A machine registers the vaults whose projects it works on. Zero, one, or
several. Nothing is derived — declare both the vault and where its projects
live, because they are often on different filesystems.

```bash
git clone git@github.com:<owner>/<name>-vault.git "<vault path>"
mkdir -p ~/.config/helia
cat >> ~/.config/helia/vaults <<'EOF'
vault work /path/to/the/vault
root  work /path/to/where/its/projects/are
EOF
```

The path is the last field on the line, so a path containing a space needs no
quoting. Several `root` lines for one vault accumulate.

**Never point `root` at a home directory.** It is scanned at depth 3 on every
session start: a correctly declared root takes ~18 ms, a home directory on a
mounted foreign filesystem does not finish in 4 seconds.

No deploy: the registry is read from disk.

`helia:registering-a-project` carries the frontmatter contract the tooling
parses and the six sections a registry file must hold (an ecosystem carries four).

## 4. Developing Helia

```bash
git clone git@github.com:Athyrr/helia.git ~/<space>/helia
cd ~/<space>/helia
git config core.hooksPath .githooks     # arms the pre-commit gate — per clone,
                                        # never versioned; scripts/doctor gate 8
                                        # is what makes its absence visible
git switch -c <chantier>
# ... edit skills/ agents/ hooks/ ...

scripts/doctor                          # the fourteen gates
claude --plugin-dir .                   # test in a session WITHOUT publishing
                                        # (takes precedence over the installed
                                        # plugin, for that session only)

scripts/deploy patch "message"          # bump + gates + commit + push
                                        # + marketplace update + plugin update
# a NEW session now runs the new version, everywhere
git switch main && git merge <chantier>
```

**Nothing you edit reaches your other sessions before `scripts/deploy`.** That
separation is native to the montage, not built on top of it.

## 5. The two directories that matter

`bin/` is added to the Bash tool's `PATH` while the plugin is enabled — it holds
execution commands (`ezy`, `scoped`, `helia-deps`) and the status line renderer
(`helia-statusline`, prefixed so it cannot collide with a bare command name).

`scripts/` is **not** on the `PATH` and holds repository maintenance (`deploy`,
`doctor`, and their test scripts).

Never put a git hook in `hooks/`: that is the harness namespace, and
`core.hooksPath` points at a *directory* from which git runs every file bearing
a git-hook name.

## 6. One known trap

- **`/reload-plugins` reloads skills, agents and hooks without restarting.**
  The SessionStart injections still need a session event: `session-start`
  matches `startup|clear|compact`, `consignes` matches
  `startup|resume|clear|compact|fork` — a deliberate difference — and `/clear`
  is one of them.

## 7. The status line

By default, every machine with the plugin installed shows two lines, with no manual setting:

```
✺ helia  │  <model>  │  <project> ⎇ <branch>
ctx <gauge> <N> %    5h <gauge> <N> % ↺ <reset>    hebdo <gauge> <N> % ↺ <reset>
```

`<project>` is the registered project whose working folder contains the current
folder (see section 3), else the folder's name; `⎇ <branch>` is omitted outside
a git repository. Each gauge has 12 cells, in the palette's success colour below
60 %, warning up to 85 %, error above; `–` replaces the number when Claude Code
reports no value. `5h` and `hebdo` — the share of the 5-hour and weekly
allowances already used, and the time before each resets (`42min`, `2h14`,
`3j`) — only appear when Claude Code reports rate limits. Colours are the exact
values of the active theme's palette (README, *Appearance*), re-read at the next
message after a `/theme` (the renderer caches the theme for up to 5 s); an unreadable palette falls back on the terminal's 16
ANSI colours.

How it gets there: a plugin cannot declare `statusLine`, so a `SessionStart` hook
(`hooks/appearance`, matcher `startup`) writes the key into
`~/.claude/settings.json`, with a path already resolved to the current plugin
version, and rewrites it at every startup. It writes nothing when the value is
already right, and it never touches a `statusLine` that does not carry the
marker `HELIA_STATUSLINE=1` — your own status line stays yours. The same hook
sets the theme (README, *Appearance*): it is the only writer of that file, in
one read and at most one atomic write.

**Opt out**, one flag file per part; the hook then leaves that part alone. A
flag only stops future writes: to remove a bar already installed, also delete
the `statusLine` key from `~/.claude/settings.json`.

```bash
mkdir -p ~/.config/helia
touch ~/.config/helia/no-statusline   # the status line
touch ~/.config/helia/no-theme        # the theme
touch ~/.config/helia/no-banner       # the startup banner
```

Known limits:

- **Uninstalling the plugin leaves the `statusLine` and `theme` keys behind**,
  pointing at a script and a theme that no longer exist; Claude Code then hides
  the bar. Delete `statusLine` from `~/.claude/settings.json` by hand and pick
  another theme with `/theme`.
- The bar stays empty without trust in the workspace, with `disableAllHooks` or
  `allowManagedHooksOnly`; a project-level or managed `statusLine` takes
  precedence over the one in `~/.claude/settings.json`.
- `claude --plugin-dir .` writes the development path into your global
  `settings.json` (the marker allows it); the next normal startup rewrites it.
- `resume` and `fork` do not rewrite the path; the previous version's folder
  stays valid for 14 days.
- Whether a write made at startup applies to the session that made it, or only
  to the next one, is not documented: expect the bar from the second session.
