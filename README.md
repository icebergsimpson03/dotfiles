# dotfiles

Bash, git, ssh and tooling for **Debian on WSL2** (Windows) and **macOS**, managed with [chezmoi](https://chezmoi.io).

## What it sets up

| | Debian / WSL2 | macOS |
|---|---|---|
| System packages | apt (`.chezmoidata/packages.yaml`) | Homebrew (same file) |
| CLI tools and runtimes | mise (`dot_config/mise/config.toml`) | mise |
| Docker | Docker Engine from Docker's apt repo | — |
| SSH keys | 1Password Windows agent, bridged with npiperelay + socat | 1Password agent (set `IdentityAgent` in `~/.ssh/config.d/`) |
| Git HTTPS auth | Git Credential Manager (Windows) | macOS Keychain (default) |
| WSL config | `/etc/wsl.conf` + `%UserProfile%\.wslconfig` | — |

## New Windows machine

1. **Windows side (one time):**
   - `winget install albertony.npiperelay`
   - 1Password → Settings → Developer → turn on **Use the SSH agent**. The Windows "OpenSSH Authentication Agent" service must stay disabled.
   - Copy `windows/.wslconfig` to `%UserProfile%\.wslconfig` **before** installing the distro (`sparseVhd` only applies to new disks).
   - `wsl --install -d Debian`, then create your user.
2. **In Debian:**
   ```bash
   sudo apt update && sudo apt install -y curl git
   sh -c "$(curl -fsLS get.chezmoi.io)" -- -b ~/.local/bin init --apply <github-user>
   ```
3. **From Windows:** `wsl --shutdown`, then reopen Debian (turns on systemd and the docker group).
4. Check:
   ```bash
   ssh-add -l          # lists your 1Password keys
   docker run --rm hello-world
   ssh -T git@github.com
   ```

## New Mac

1. Install [Homebrew](https://brew.sh) and turn on the 1Password SSH agent.
2. `brew install chezmoi && chezmoi init --apply <github-user>`
3. Make Homebrew bash your login shell:
   ```bash
   echo "$(brew --prefix)/bin/bash" | sudo tee -a /etc/shells
   chsh -s "$(brew --prefix)/bin/bash"
   ```
4. Point ssh at 1Password in `~/.ssh/config.d/1password` (untracked):
   ```
   Host *
       IdentityAgent "~/Library/Group Containers/2BUAT4S2Q.com.1password/t/agent.sock"
   ```

## Day to day

- `chezmoi edit ~/.bashrc` then `chezmoi apply`, or edit in `chezmoi cd` and commit.
- Add an apt/brew package: edit `.chezmoidata/packages.yaml`, then `chezmoi apply`.
- Add a CLI tool: edit `dot_config/mise/config.toml`, then `chezmoi apply`.
- Machine-only settings go in `~/.bashrc.local`, `~/.gitconfig.local` and `~/.ssh/config.d/` (not tracked).
- Keep code in `~/code` (Linux filesystem), not `/mnt/c`, and open it with `code .`.
