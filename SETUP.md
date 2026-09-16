# Setting up aiKit

Two pipelines, deliberately separate: **using** the method installs a plugin and
never clones this repository; **developing** the method clones this repository
and never installs from the clone. Nothing you edit in a clone reaches your
other sessions until `scripts/deploy` has published it.

## 1. Prerequisites

- `git`
- `node` **or** `python3` — the scripts try `node` first and fall back to
  `python3`; either one alone is enough.
- an SSH key loaded in `ssh-agent`, with `github.com` in `known_hosts`
- Claude Code ≥ 2.1.193

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
parses and the six sections a registry file must hold.

## 4. Developing aiKit

```bash
git clone git@github.com:Athyrr/aikit.git ~/<space>/aikit
cd ~/<space>/aikit
git config core.hooksPath .githooks     # arms the pre-commit gate — per clone,
                                        # never versioned; scripts/doctor gate 8
                                        # is what makes its absence visible
git switch -c <chantier>
# ... edit skills/ agents/ hooks/ ...

scripts/doctor                          # the nine gates
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
execution commands (`ezy`, `scoped`) only.

`scripts/` is **not** on the `PATH` and holds repository maintenance (`deploy`,
`doctor`).

Never put a git hook in `hooks/`: that is the harness namespace, and
`core.hooksPath` points at a *directory* from which git runs every file bearing
a git-hook name.

## 6. One known trap

- **`/reload-plugins` reloads skills, agents and hooks without restarting.**
  Only the SessionStart preamble needs `startup|clear|compact` — and `/clear` is
  one of those.
