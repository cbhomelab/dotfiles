# dotfiles

Cross-platform shell configuration for Windows, Linux, and macOS, managed
with [chezmoi](https://www.chezmoi.io/). Native shells per OS (zsh/bash on
Linux/macOS, PowerShell on Windows) kept deliberately separate — but all
driven from this one repo, and all sharing the same prompt
([Starship](https://starship.rs/)) and the same alias *names/behavior*
(`ll`, `la`, `gs`, `ga`, `gc`, `gp`, `gl`, `gd`, `gco`) across every shell.

## What's here

| File | Target | Purpose |
|---|---|---|
| `dot_zshrc` | `~/.zshrc` | Oh My Zsh (completions/plugins only — no theme) + Starship prompt |
| `dot_bashrc` | `~/.bashrc` | Bash equivalent of the above |
| `dot_bash_aliases` | `~/.bash_aliases` | Shared alias set, sourced by both `.zshrc` and `.bashrc` |
| `dot_config/starship.toml` | `~/.config/starship.toml` | The one prompt config used by zsh, bash, **and** PowerShell |
| `dot_gitconfig-chezmoi.tmpl` | `~/.gitconfig-chezmoi` | Templated git identity/settings — see note below |
| `Documents/WindowsPowerShell/Microsoft.PowerShell_profile.ps1.tmpl` | PS 5.1 profile | Windows only |
| `Documents/PowerShell/Microsoft.PowerShell_profile.ps1.tmpl` | PS 7 profile | Windows only (same content as above, shared via `.chezmoitemplates/`) |
| `run_once_install-tools.*` | — | First-apply bootstrap: installs Oh My Zsh + Starship (Unix) or Starship + PowerShell 7 (Windows) if missing |
| `run_once_setup-gitconfig-include.*` | — | One-time: adds `[include]` to `~/.gitconfig` pointing at the chezmoi-managed file, see below |

## Why `~/.gitconfig` isn't managed directly

`gh auth setup-git` (and some other tools) write credential-helper blocks
straight into `~/.gitconfig`. chezmoi fully overwrites any file it manages
on every `apply`, which would silently delete those blocks on the next
apply. So instead:

- chezmoi manages `~/.gitconfig-chezmoi` (identity, line-ending, editor, lfs
  filter — the stuff that should be identical on every machine)
- a one-time script adds a single `[include] path = ~/.gitconfig-chezmoi`
  line to the real `~/.gitconfig`, then leaves that file alone forever
- `gh auth setup-git`, local per-repo overrides, etc. can keep editing
  `~/.gitconfig` freely without chezmoi ever clobbering them

## Setting up a new machine

### Prerequisites
- **Windows:** `winget install twpayne.chezmoi` (PowerShell, git, and GitHub
  CLI should already be set up — see the main homelab docs for that)
- **Linux/macOS:** install chezmoi per
  https://www.chezmoi.io/install/ (one-liner: `sh -c "$(curl -fsLS get.chezmoi.io)"`)

### Apply

```sh
chezmoi init --apply cbhomelab/dotfiles
```

First run prompts once for your name and email (used for git commits) and
caches the answer in chezmoi's own config — you won't be asked again on
that machine. It then runs the bootstrap scripts (Oh My Zsh/Starship/
PowerShell 7 if missing) and deploys every file above.

### Day to day

```sh
dotfiles-edit      # opens a managed file in $EDITOR, inside the source repo
dotfiles-diff       # preview what `apply` would change
dotfiles-apply      # deploy local edits
dotfiles-cd         # cd into the chezmoi source directory (this repo, checked out locally)
```

These are defined as aliases/functions in every shell (zsh, bash,
PowerShell) with identical names.

To make a change: edit a file under `dotfiles-cd`, then `dotfiles-apply`,
then `git add -A && git commit && git push` from inside that directory like
any normal repo (chezmoi's source directory IS a git working copy).

## Notes

- **Starship** is the one thing that's truly identical everywhere — same
  `starship.toml`, same look, in zsh, bash, and PowerShell alike, even
  though the shells themselves are completely different underneath.
- **Modern CLI tools** (`eza`, `bat`) are used by the aliases *if installed*,
  falling back to plain `ls`/`cat` otherwise — nothing breaks if you haven't
  installed them on a given machine.
- **WSL (Debian)** counts as "Linux" for this repo — `chezmoi init --apply`
  works the same way inside a WSL shell as on bare-metal Linux.
