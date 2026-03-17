# 🎨 9front OS Theming Resources

> A curated list of every website, tool, patch, and repository related to changing themes on the [9front](https://9front.org) operating system.

---

## Table of Contents

- [Theme Collections & Galleries](#-theme-collections--galleries)
- [Core Technical Reference](#-core-technical-reference)
- [Documentation & Guides](#-documentation--guides)
- [Code Repositories](#-code-repositories)
- [Curated Lists](#-curated-lists)
- [Quick Reference Table](#-quick-reference-table)
- [How It All Fits Together](#-how-it-all-fits-together)

---

## 🖼️ Theme Collections & Galleries

### [thedaemons.space — 9front Theme Gallery](https://thedaemons.space/9front/theme/)

The largest ready-to-use theme collection for 9front. Contains 30+ themes with screenshots, all requiring [Sigrid's rio-themes patch](https://ftrv.se/14).

**Available themes include:**

| Theme | Author |
|---|---|
| SeaSky | thedaemon |
| Sand | thedaemon |
| Nord | thedaemon |
| Rio | thedaemon |
| Microsoft Bob | — |
| Dracula | — |
| Half-assed Pink | — |
| Radon | — |
| Bastille | — |
| Fairlight | — |
| Momotori | — |
| Retro Pop | — |
| Futile Resistance | — |
| Black / Green | — |
| Nordy / Rose 9 | — |
| Blit / Green Gold | — |
| Old Bell | — |
| Budget Green / Gruv 9 | — |
| Orange | — |
| Cst | — |
| Gruvbox / Gruvbox Light | — |
| Plateau | — |
| The Psychedelic Plan9 Hacker | — |
| Dalem | — |
| Purpy | — |
| White | — |

**Companion scripts (rc):**

```
https://thedaemons.space/9front/themes   # theme switcher script
https://thedaemons.space/9front/setbg    # background setter script
```

> **Note:** The default theme directory is `$home/lib/themes` (configurable via `themedir` in the script). Run `themes -h` for help. Always run `setbg` *after* `themes` since `themes` overwrites the background.

---

## 🔧 Core Technical Reference

### [ftrv.se — "Rio Theming in 9front" by Sigrid](https://ftrv.se/14)

**This is the foundational patch that makes all 9front theming possible.**

Sigrid's approach exposes `/dev/theme` as a virtual file, allowing dynamic color changes to rio without recompilation. Each nested rio instance can carry a different color theme.

**Applying the patch:**

```rc
bind -ac /dist/plan9front /
cd /sys/src/cmd/rio
hget https://ftrv.se/_/9/patches/rio-themes.patch | patch -p5
mk install
```

**To revert:**

```rc
bind -ac /dist/plan9front /
cd /sys/src/cmd/rio
git/revert .
rm -f *.rej *.orig menuhit.c col.h
mk install
```

**Writing a theme to `/dev/theme`:**

```
rioback /usr/glenda/lib/1920x1080.img
back    f1f1f1
high    cccccc
border  999999
text    000000
htext   000000
title   000000
ltitle  bcbcbc
hold    000099
lhold   005dbb
palehold 4993dd
paletext 6f6f6f
size    000000
menubar 448844
menuback eaffea
menuhigh 448844
menubord 88cc88
menutext 000000
menuhtext eaffea
```

**Converting a wallpaper image:**

```rc
jpg -9t <coolwallpaper1920x1080.jpg >/usr/glenda/lib/1920x1080.img
```

**Interactive color picker:**

```rc
picker </dev/theme >/dev/theme
```

> The `picker` tool (install separately) provides snarf, revert, and palette controls. With proper plumber setup (`man picker`), you can plumb a `.theme` file to apply it instantly.

---

## 📚 Documentation & Guides

### [FQA 8 — Using 9front (Official)](http://fqa.9front.org/fqa8.html)

The official 9front FAQ chapter covering day-to-day usage, including a section on modifying rio for wallpapers and themes. Acknowledges Sigrid's work as the recommended approach.

---

### [Plan 9 Desktop Guide](https://pspodcasting.net/dan/blog/2019/plan9_desktop.html)

A comprehensive all-in-one guide to running Plan 9 / 9front as a desktop. Includes a dedicated **Wallpapers, themes, etc.** section under System Administration.

---

### [wiki.9front.org — Tips and Tricks](http://wiki.9front.org/tips-and-tricks)

The community wiki. Contains practical rio usage tips, workspace management with nested rios, and scripting examples.

---

### [plan9-cookbook — sourcehut](https://sr.ht/~rabbits/plan9-cookbook/)

Collected notes on writing GUI applications and modifying the look and feel of the Plan 9 operating system.

---

### [XXIIVV — Rio](https://wiki.xxiivv.com/site/rio.html)

Notes on the Plan 9 interface, rio window management, and tools. Good introductory reference for understanding the rio environment before theming it.

---

## 🗂️ Code Repositories

### [plan9.stanleylieber.com — Rio](http://plan9.stanleylieber.com/rio/)

Stanley Lieber's Plan 9 page, hosting Sigrid's modified version of rio built for theming. Also documents **Lola**, an experimental new window system based on rio's architecture.

---

### [shithub.us — jgstratt/acme-themes](https://shithub.us/jgstratt/acme-themes/HEAD/info.html)

A modified build of acme that reads and applies rio-themes at startup. The theming functions from `rio-themes` are extracted and integrated directly into acme. If no theme is installed, it falls back to the traditional acme color scheme.

**Applying the acme-themes patch:**

```rc
mkdir tacme
cp /sys/src/cmd/acme/* tacme/
cd tacme
ape/patch -i ../acme-themes.patch ..
```

> Note: acme must be restarted for theme changes to take effect.

---

### [GitHub — JonStratton/plan9-tacme](https://github.com/JonStratton/plan9-tacme)

GitHub mirror of the `acme-themes` project above.

---

### [GitHub — Plan9-Archive/grio](https://github.com/Plan9-Archive/grio)

Ben Kidwell's (grid) modifications to the rio window manager, designed specifically to make visual customization easier.

---

### [git.sr.ht — ~amavect/makeu](https://git.sr.ht/~amavect/makeu)

**makeu** ("make hue") — a color creation tool for 9front built with `libelementile`. Features a selectable color field, an eyedropper color display, and sliders. Useful for designing custom theme color values before writing them to `/dev/theme`.

**Install:**

```rc
mk install
```

---

## 📋 Curated Lists

### [GitHub — henesy/awesome-plan9](https://github.com/henesy/awesome-plan9)

The canonical curated list of Plan 9 / 9front resources. Links to Modding Rio tutorials, Plan9 GUI Examples, the Plan9 Desktop Guide, and all major software repositories.

---

### [GitHub — AnastasiosPapalias/Plan9](https://github.com/AnastasiosPapalias/Plan9)

*Down the Rabbit Hole* — a collection of software and libraries for the Plan 9 OS. Mirrors the awesome-plan9 structure with links to theming tutorials, GUI examples, and the desktop guide.

---

## 📊 Quick Reference Table

| URL | Type | Key Content |
|---|---|---|
| [thedaemons.space/9front/theme/](https://thedaemons.space/9front/theme/) | Theme gallery | 30+ `.theme` files with screenshots |
| [ftrv.se/14](https://ftrv.se/14) | Tutorial + patch | Core rio-themes patch, `/dev/theme` mechanism |
| [fqa.9front.org/fqa8.html](http://fqa.9front.org/fqa8.html) | Official docs | Rio theming overview |
| [pspodcasting.net — Plan 9 Desktop Guide](https://pspodcasting.net/dan/blog/2019/plan9_desktop.html) | Desktop guide | Full section on wallpapers & themes |
| [wiki.9front.org/tips-and-tricks](http://wiki.9front.org/tips-and-tricks) | Community wiki | Usage tips for rio |
| [sr.ht/~rabbits/plan9-cookbook](https://sr.ht/~rabbits/plan9-cookbook/) | Cookbook | GUI look & feel modification notes |
| [plan9.stanleylieber.com/rio/](http://plan9.stanleylieber.com/rio/) | Download | Sigrid's modified theming rio |
| [shithub.us — jgstratt/acme-themes](https://shithub.us/jgstratt/acme-themes/HEAD/info.html) | Repo | Acme with rio-themes support |
| [github.com/JonStratton/plan9-tacme](https://github.com/JonStratton/plan9-tacme) | Repo | GitHub mirror of acme-themes |
| [github.com/Plan9-Archive/grio](https://github.com/Plan9-Archive/grio) | Repo | Rio fork for easier customization |
| [git.sr.ht/~amavect/makeu](https://git.sr.ht/~amavect/makeu) | Tool | Color picker/creator for 9front |
| [github.com/henesy/awesome-plan9](https://github.com/henesy/awesome-plan9) | Curated list | Master list of Plan 9 resources |
| [github.com/AnastasiosPapalias/Plan9](https://github.com/AnastasiasPapalias/Plan9) | Curated list | Plan 9 software collection |

---

## 🧩 How It All Fits Together

The entire 9front theming ecosystem pivots around two things:

1. **Sigrid's rio-themes patch** (`ftrv.se/14`) — exposes `/dev/theme` as a writable virtual file, enabling runtime color changes without recompiling rio.
2. **thedaemons.space** — the primary community repository of ready-made `.theme` files with screenshots and rc scripts to apply them.

Everything else in the ecosystem either:
- **Documents** the patch (FQA, Desktop Guide, wiki)
- **Extends** it (acme-themes, grio, makeu)
- **Links** to it (awesome-plan9, AnastasiosPapalias/Plan9)

```
Sigrid's Patch (ftrv.se/14)
        │
        ├── thedaemons.space ──── 30+ themes + rc scripts
        ├── acme-themes ──────── acme picks up /dev/theme
        ├── grio ─────────────── rio fork, easier colors
        ├── makeu ────────────── GUI color designer
        └── Documentation
              ├── FQA 8
              ├── Plan 9 Desktop Guide
              ├── wiki.9front.org
              └── plan9-cookbook
```

---

*This page was compiled from a web search of all publicly available resources related to 9front theming as of March 2026.*  
*Contributions and corrections welcome.*
