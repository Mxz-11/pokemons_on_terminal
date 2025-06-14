# README — Fastfetch + Pokeget with a Random Pokémon Logo

> **Goal:** show a **different Pokémon (ID 1‑493)** every time you start a new shell session.
> We combine **Pokeget** (sprite generator) with **Fastfetch** (system‑info fetch).
> This guide walks you through installation, configuration, automation, and troubleshooting.

---

## 1  Overview

* **Fastfetch** prints system information in the terminal (like Neofetch, but faster & JSONC‑configurable).
* **Pokeget** outputs colourful ANSI sprites of any Pokémon.
* A tiny **Bash script** runs Pokeget, saves its output to a text file, and Fastfetch reads that file as its logo.

---

## 2  Prerequisites

| Program                   | Purpose                                                |
| ------------------------- | ------------------------------------------------------ |
| Bash / Zsh                | Runs the helper script.                                |
| `rustup` + Cargo          | Builds Pokeget (and optionally Fastfetch) from source. |
| Fastfetch                 | Displays system info with a custom ASCII/ANSI logo.    |
| Pokeget                   | Generates Pokémon sprites.                             |
| (optional) `git`, `cmake` | Needed only for building Fastfetch from source.        |

> **True‑color required** – your terminal must support 24‑bit colour to see the sprites properly.

---

## 3  Install Pokeget

### 3.1  Via Cargo (works on any platform)

```bash
# 1  Install Rust + Cargo if missing
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"   # activate Cargo in the current shell

# 2  Install Pokeget
cargo install pokeget
```

The binary lands in `~/.cargo/bin/pokeget`.

### 3.2  Distribution packages (Linux)

| Distro          | Command                                                 |
| --------------- | ------------------------------------------------------- |
| Arch / Manjaro  | `sudo pacman -S pokeget` (or `yay -S pokeget` from AUR) |
| Debian / Ubuntu | compile with Cargo (no official .deb yet)               |

### 3.3  macOS (Homebrew)

```bash
brew install pokeget          # if a formula exists
# or
cargo install pokeget         # universal fallback
```

---

## 4  Install Fastfetch

### 4.1  Package manager

| Distro           | Command                                             |
| ---------------- | --------------------------------------------------- |
| Arch / Manjaro   | `sudo pacman -S fastfetch`                          |
| Debian / Ubuntu  | `sudo apt install fastfetch` (via backports or PPA) |
| Fedora           | `sudo dnf install fastfetch`                        |
| macOS (Homebrew) | `brew install fastfetch`                            |

### 4.2  Build from source

```bash
git clone --depth=1 https://github.com/fastfetch-cli/fastfetch.git
cd fastfetch
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
sudo cmake --install build
```

---

## 5  Configure Fastfetch

### 5.1  Locate or create the config file

| OS            | Default path                                   |
| ------------- | ---------------------------------------------- |
| Linux / \*BSD | `~/.config/fastfetch/config.jsonc`             |
| macOS         | `~/Library/Preferences/fastfetch/config.jsonc` |
| Windows       | `%APPDATA%\fastfetch\config.jsonc`             |

Create the directory and a starter config:

```bash
mkdir -p ~/.config/fastfetch
fastfetch --gen-config       # creates config.jsonc
```

### 5.2  Set the logo block

Edit `config.jsonc` and insert (or replace) the **logo** section:

```jsonc
"logo": {
  "type": "file",
  "path": "/home/<USER>/.config/fastfetch/ty.txt",  // adjust USER
  "padding": { "right": 2 }
},
```

Newer Fastfetch builds prefer the key `path`; older ones accept `source`.

---

## 6  Automation Script (`pokeget-random-logo`)

Create `~/.local/bin/pokeget-random-logo` with:

```bash
#!/usr/bin/env bash
# Generates a random Pokémon (1‑493) and overwrites ty.txt
set -euo pipefail
OUT_FILE="$HOME/.config/fastfetch/ty.txt"
mkdir -p "$(dirname "$OUT_FILE")"

# Resolve pokeget
POKEGET_BIN=$(command -v pokeget || true)
[[ -z "$POKEGET_BIN" ]] && POKEGET_BIN="$HOME/.cargo/bin/pokeget"
[[ -x "$POKEGET_BIN" ]] || { echo "❌  pokeget not found" >&2; exit 1; }

# Random number 1‑493
if command -v shuf >/dev/null; then
  NUM=$(shuf -i 1-493 -n 1)
else
  NUM=$(( (RANDOM % 493) + 1 ))
fi

# Generate sprite without name and overwrite
"$POKEGET_BIN" "$NUM" --hide-name > "$OUT_FILE"
```

Make it executable and add `~/.local/bin` to your `$PATH` if needed:

```bash
chmod +x ~/.local/bin/pokeget-random-logo
export PATH="$HOME/.local/bin:$PATH"
```

---

## 7  Integrate with your shell (Bash or Zsh)

Append to `~/.bashrc` **or** `~/.zshrc` **in this order**:

```bash
# 1  Ensure Cargo bin directory is in PATH (for pokeget)
export PATH="$HOME/.cargo/bin:$PATH"

# 2  Generate a fresh Pokémon logo
eval "$(~/.local/bin/pokeget-random-logo)"   # creates ty.txt

# 3  Show Fastfetch
fastfetch
```

Reload your shell:

```bash
source ~/.bashrc   # or source ~/.zshrc
```

Open a new terminal — a new Pokémon should greet you every time!

---

## 8  Using Neofetch instead (optional)

If you prefer Neofetch:

```bash
~/.local/bin/pokeget-random-logo
neofetch --ascii ~/.config/fastfetch/ty.txt
```

Add those two lines to your shell‑rc.

---

## 9  Quick verification checklist

```bash
which pokeget         # should return a path
which fastfetch       # should return a path
cat ~/.config/fastfetch/ty.txt | head -n 5   # see ANSI colour codes
fastfetch --log-level info | grep Logo       # path should match
```

---

## 10  Troubleshooting

| Symptom                      | Likely cause                                 | Fix                                                                    |
| ---------------------------- | -------------------------------------------- | ---------------------------------------------------------------------- |
| `pokeget: command not found` | `$PATH` doesn’t include `~/.cargo/bin`       | add export or reinstall Pokeget                                        |
| Logo shows no colour         | Terminal lacks 24‑bit colour                 | enable true‑color or switch terminal                                   |
| Fastfetch ignores logo       | wrong key (`path` vs `source`) or wrong path | correct the logo block and verify with `fastfetch --list-config-paths` |

---

## 11  Credits

* **talwat/pokeget** — Pokémon sprites in ANSI.
* **fastfetch-cli/fastfetch** — blazing‑fast system fetch.

Enjoy your console with a new Pokémon buddy each session! 🎮✨
