# Setting up aiKit

Two pipelines, deliberately separate: **using** the method installs a plugin and
never clones this repository; **developing** the method clones this repository
and never installs from the clone. Nothing you edit in a clone reaches your
other sessions until `scripts/deploy` has published it.

## 1. Prerequisites

- `git`
- `node` **or** `python3` — the scripts try `node` first and fall back to
  `python3`; either one alone is enough.
- `jq` — **required for the status line**: the installer edits `~/.claude/settings.json`
  with it and leaves the file untouched when `jq` is missing; the renderer falls
  back to a bare `aikit · <folder> (<branch>)` line. Everything else works without it.
- an SSH key loaded in `ssh-agent`, with `github.com` in `known_hosts`
- Claude Code ≥ 2.1.193
- **optional:** an `obsidian` plugin providing the `obsidian:obsidian-markdown`
  and `obsidian:obsidian-bases` skills, if the vault at `<vault>/` is an actual
  Obsidian vault and you want `aikit.base`'s dashboard to work and wikilinks/
  callouts/properties written correctly. Without it, agents still write to
  `vault/` — the Obsidian-specific syntax invocations in their prompts just
  have nothing to invoke.

On Windows: **Git for Windows is a hard prerequisite** — without it the hook
cannot start and now says so in the transcript instead of failing silently.

## 2. Using aiKit — two commands

```bash
claude plugin marketplace add git@github.com:Athyrr/aikit.git
claude plugin install aikit@aikit-marketplace --scope user
```

Register over **SSH**. The docs state that background refreshes disable
credential helpers, so a private marketplace registered over HTTPS fails to
auto-update. Install in **one scope only** — two scopes means two copies and an
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
mkdir -p ~/.config/aikit
cat >> ~/.config/aikit/vaults <<'EOF'
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

`aikit:registering-a-project` carries the frontmatter contract the tooling
parses and the six sections a registry file must hold (an ecosystem carries four).

## 4. Developing aiKit

```bash
git clone git@github.com:Athyrr/aikit.git ~/<space>/aikit
cd ~/<space>/aikit
git config core.hooksPath .githooks     # arms the pre-commit gate — per clone,
                                        # never versioned; scripts/doctor gate 8
                                        # is what makes its absence visible
git switch -c <chantier>
# ... edit skills/ agents/ hooks/ ...

scripts/doctor                          # the ten gates
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
execution commands (`ezy`, `scoped`) and the status line renderer
(`aikit-statusline`, prefixed so it cannot collide with a bare command name).

`scripts/` is **not** on the `PATH` and holds repository maintenance (`deploy`,
`doctor`).

Never put a git hook in `hooks/`: that is the harness namespace, and
`core.hooksPath` points at a *directory* from which git runs every file bearing
a git-hook name.

## 6. One known trap

- **`/reload-plugins` reloads skills, agents and hooks without restarting.**
  Only the SessionStart preamble needs `startup|clear|compact` — and `/clear` is
  one of those.

## 7. The status line

Every machine with the plugin installed shows one line, with no manual setting:

```
aikit · <model> · <project> (<branch>) · ctx <N> %
```

`<project>` is the registered project whose working folder contains the current
folder (see section 3), else the folder's name. `ctx` is green below 60 %, yellow
up to 85 %, red above, `ctx –` when Claude Code reports no value.

How it gets there: a plugin cannot declare `statusLine`, so a `SessionStart` hook
(matcher `startup`) writes the key into `~/.claude/settings.json`, with a path
already resolved to the current plugin version, and rewrites it at every startup.
It writes nothing when the value is already right, and it never touches a
`statusLine` that does not carry the marker `AIKIT_STATUSLINE=1` — your own
status line stays yours.

**Opt out:** create the flag file, the hook then neither reads nor writes anything.

```bash
mkdir -p ~/.config/aikit && touch ~/.config/aikit/no-statusline
```

Known limits:

- **Uninstalling the plugin leaves the `statusLine` key behind**, pointing at a
  script that no longer exists; Claude Code then hides the bar. Delete the key
  from `~/.claude/settings.json` by hand.
- The bar stays empty without trust in the workspace, with `disableAllHooks` or
  `allowManagedHooksOnly`; a project-level or managed `statusLine` takes
  precedence over the one in `~/.claude/settings.json`.
- `claude --plugin-dir .` writes the development path into your global
  `settings.json` (the marker allows it); the next normal startup rewrites it.
- `resume` and `fork` do not rewrite the path; the previous version's folder
  stays valid for 14 days.
- Whether a write made at startup applies to the session that made it, or only
  to the next one, is not documented: expect the bar from the second session.
