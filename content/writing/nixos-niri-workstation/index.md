---
title: "Building a NixOS Workstation with Niri, Gruvbox, and Claude Code"
date: "2026-03-19"
description: "A practical guide to building a keyboard-driven development environment on NixOS with the niri scrollable tiling compositor."
path: "/nixos-niri-workstation/"
---

I recently rebuilt my primary development machine from scratch. I've been using Arch Linux for a long time, and for a couple months used Omarchy, which is beautiful and really easy to use. One primary issue kept coming up though - we have a lot of client projects running all versions of various software and I found myself writing scripts to change dependencies jumping project to project.  I also felt out of control with system dependencies with the multiple versions all sitting on my system.

After analyzing both requirements, daily workflows, combined with the interest in doing something a little different after a years of Arch (BTW) I'm running a NixOS workstation with the niri Wayland compositor, themed with gruvbox, driven primarily through the keyboard, and centered around AI-assisted development with Claude Code.  This is by far my most efficient and stable system - with now over four weeks as a daily driver at work.


This post walks through what I chose, why, and how to reproduce it. The entire configuration lives in a single git repository: [github.com/AndrewCull/nixos-config](https://github.com/AndrewCull/nixos-config).

{{ img(src="niri-clean-desktop-gruvbox-waybar.png", alt="Clean desktop with niri and waybar") }}

## The Hardware

**ThinkPad P14s Gen 6 AMD** — Ryzen AI 7 PRO 350, 96GB DDR5-5600, 1TB NVMe. This is the machine that made NixOS practical for me. Previous attempts on an 8GB Dell XPS 13 were educational but painful — cargo builds would swap, Docker was off-limits, and Chromium with a single Claude tab would stutter the touchpad.

96GB changes the equation entirely. Docker, cargo builds, Next.js dev servers, multiple Chrome PWAs, and Claude Code all running simultaneously without the system breaking a sweat.

**BenQ RD280UG** — 28" 3:2 programming monitor at work, with the same display at home. The coding-specific display modes and Nano Matte panel make a real difference over long sessions. The 3:2 aspect ratio gives you more vertical space for code without going ultrawide.

**HHKB Professional HYBRID Type-S** — blank keycaps. The keyboard that forces you to learn your own muscle memory. Combined with a tiling compositor where everything is keyboard-driven, the blanks actually make sense. You stop looking down. Combined with niri, I have all types of secret key combinations to add some media keys, launch anything I need to, and thoroughly confuse everyone in my office. If a password or key has a special character I'm about 85% dead on, don't ask me about page up or page down yet. Typing speed is much faster with this setup.

{{ img(src="fastfetch-thinkpad-p14s-nixos-specs.png", alt="System specs via fastfetch") }}

## Why NixOS

I've run Ubuntu, Fedora, Arch, and most recently Omarchy (Hyprland on Arch). Each time, the system drifts. You install packages, tweak configs, add PPAs, and after six months you couldn't reproduce the setup if you tried. When something breaks, you're debugging a unique snowflake.

NixOS eliminates that. My entire system — every package, every config file, every service — is declared in a set of `.nix` files tracked in git. If my laptop dies tomorrow, I clone the repo onto a fresh NixOS install and run one command:

```bash
sudo nixos-rebuild switch --flake ~/nixos-config#p14s
```

Twenty minutes later, I have an identical system. Every font, every keybind, every shell alias. The same system I was running five minutes before the old machine died.

The other superpower is rollback. Every `nixos-rebuild` creates a new generation. If an update breaks something, I pick the previous generation from the boot menu and I'm back to a working system in thirty seconds. No recovery mode, no reinstall, no stress.

## Why Niri

Most tiling window managers arrange windows in a fixed grid that fills your screen. Niri does something different — it arranges windows in columns on an infinite horizontal strip. You scroll left and right through your workspace. It sounds strange until you use it, and then traditional tiling feels claustrophobic.

For my workflow — typically 3-5 terminal windows, a browser, and an email client — niri lets me spread out without worrying about layout management. New windows appear to the right. I scroll to what I need. The mental model is simple enough that I stopped thinking about window management entirely, which is the point.

Niri is also Wayland-native with no X11 baggage. It's fast, it's simple, and it stays out of the way.

## The Stack

Here's what I'm running and why:

**Shell: Fish** — switched from zsh. Fish has better defaults out of the box: real-time autosuggestions, syntax highlighting, and tab completion that actually works. Combined with zoxide (smart `cd`), fzf (fuzzy finding), and starship (prompt), it's a shell that helps you move fast.

**Terminal: Ghostty** — native Wayland, Rust-based, fast startup. Paired with Zellij as a terminal multiplexer for splitting panes when I need Claude Code in one pane and my app running in another.

**Editor: Helix** — replaced neovim/LazyVim. Helix works out of the box with LSP, syntax highlighting, and tree-sitter. No plugin management, no configuration rabbit holes. Since Claude Code handles most of my actual coding, I need an editor for quick reviews and config edits, not a fully customized IDE. Helix is perfect for that.

**GUI Editor: Zed** — for when I want a visual editor with project-wide search, multiple cursors, and a file tree. Lightweight and native, unlike VS Code.

**Coding: Claude Code** — this is the center of my development workflow. It runs in the terminal, reads my project files, writes code, runs commands, and handles git operations. I use it for everything from NixOS config changes to full-stack application development in Rust and Next.js.

**Browser: Google Chrome** — for bookmark sync with my iPhone. Claude, Superhuman, Google Meet, and Netflix run as PWAs via Chrome's `--app` flag, each with their own desktop entry in fuzzel. While I prefer the idea of Firefox, it adds friction to Google Workspace, Meet, etc.

**AI Terminal: Warp** — runs via xwayland-satellite since it doesn't support Wayland natively yet. The inline AI assistance is useful for quick system administration tasks - especially docker issues and troubleshooting multiple local apis running in a container. I'll try troubleshooting non-value-producing issues for about five minutes in the terminal, then bring up Warp Terminal and let it work. Saves a ton of time and headache most of the time.

{{ img(src="helix-editor-claude-code-zellij-split.png", alt="Helix editor and Claude Code in Zellij") }}

## Theming: Gruvbox

The entire system is themed with gruvbox-dark-medium through Stylix, which propagates colors to every application automatically — Ghostty, Helix, waybar, fuzzel, mako notifications, GTK apps, everything. One color scheme declaration in the NixOS config and every tool matches. Mentally, I 

Font is JetBrains Mono. The cursor is Bibata-Modern-Classic. The wallpaper is a dark space scene that lets the gruvbox accent colors pop without visual noise, although you can change wallpapers quick with `Mod+Shif+W` and select from a wallpapers folder to change things up.  I stopped doing the theme switcher thing for ADHD reasons.

{{ img(src="fuzzel-app-launcher-gruvbox-niri.png", alt="Fuzzel launcher showing installed apps") }}

## Key Workflows

**App launching** — `Mod+D` opens fuzzel, a Wayland-native dmenu replacement. Type a few characters, hit enter. Claude, Superhuman, Chrome, Helix — everything is two keystrokes away.

**Power management** — `Mod+Escape` opens a custom power menu built with a shell script piped through fuzzel in dmenu mode. Lock, suspend, reboot, shutdown, logout.

{{ img(src="fuzzel-power-menu-nixos-niri.png", alt="Power menu") }}

**Window management** — vim-style navigation throughout. `Mod+H/J/K/L` to focus, `Mod+Ctrl+H/L` to move columns, `Mod+1-9` for workspaces. `Mod+F` maximizes a column, `Mod+Shift+F` fullscreens.

**Development** — `Mod+Return` opens Ghostty. I typically run Zellij with a vertical split: Helix or code output on the left, Claude Code on the right. The AI writes the code, I review it in the editor, and we iterate.

**Multi-monitor** — `Mod+Shift+H/L` to focus between monitors, `Mod+Shift+Ctrl+H/L` to move windows across. The ThinkPad's 14" display runs at 1.25x scaling, the BenQ at 1.5x.

## Config Structure

The repository follows a pattern I borrowed from [antholeole's nixconfig](https://github.com/antholeole/nixconfig) — one file per tool in the `home/` directory, auto-imported by a `default.nix` that reads the directory:

```
nixos-config/
├── flake.nix                 # inputs and host definitions
├── confs/niri/config.kdl     # raw niri config (keybinds, layout)
├── home/
│   ├── default.nix           # auto-imports all .nix files
│   ├── apps.nix              # browser, PWAs, dev toolchains
│   ├── dev.nix               # CLI tools (fd, ripgrep, bat, etc.)
│   ├── fish.nix              # shell config
│   ├── ghostty.nix           # terminal
│   ├── git.nix               # git + delta + gh
│   ├── helix.nix             # editor + LSPs
│   ├── niri.nix              # waybar, fuzzel, mako, swayidle
│   ├── starship.nix          # prompt
│   ├── theme.nix             # GTK dark mode, fonts
│   └── zellij.nix            # terminal multiplexer
├── hosts/p14s/
│   ├── configuration.nix     # host-specific (AMD GPU, fingerprint)
│   └── hardware-configuration.nix
└── modules/
    ├── common.nix            # shared config (boot, audio, stylix)
    ├── docker.nix
    └── niri.nix              # compositor + greetd
```

Adding a new tool means creating one `.nix` file in `home/`. The auto-import picks it up on the next rebuild. No import list to maintain.

The niri config uses a raw `config.kdl` file instead of the niri-flake's Nix settings API. This avoids a class of build-time errors where the flake's action names don't match the installed niri version. The KDL file is battle-tested upstream and just works.

## Reproducing This Setup

If you want to try this yourself:

1. Flash the NixOS minimal ISO to a USB drive
2. Boot, partition your drive, mount at `/mnt`
3. Clone: `git clone https://github.com/AndrewCull/nixos-config.git /mnt/etc/nixos-config`
4. Generate your hardware config: `nixos-generate-config --root /mnt`
5. Copy `hardware-configuration.nix` to the appropriate host directory
6. Install: `nixos-install --flake /mnt/etc/nixos-config#{your hostname here eg. p14s}`
7. Reboot and log in

You'll want to customize `hosts/p14s/configuration.nix` for your specific hardware, update the git email in `home/git.nix`, and swap the wallpaper. Everything else should work as-is.

## What's Next
I added an MCP server configuration to the project after getting stuck with sleep and hibernate functions - it's incredibly useful and highly recommended if you get stuck.  Learning NixOS is definitely a time investment, but worth it.  AI can help, but you should learn your system at least to know when things are going off the rails.

I'm also working on per-project Nix flakes for my projects, most of which are using Rust, Typescript, and NodeJS. Using direnv to automatically activate the right development environment when I cd into a project directory is amazing - the exact configuration I need is available and isolated from my system. Since setting up this configuration, I haven't had a single issue with versioned dependency collisions.

The config is public at [github.com/AndrewCull/nixos-config](https://github.com/AndrewCull/nixos-config). Questions welcome.

---

*This post was written on the setup described above, in Helix with some help from Claude on the specs, on NixOS 26.05, on a ThinkPad P14s Gen 6, with an HHKB keyboard I still can't find the Print Screen button on.*
