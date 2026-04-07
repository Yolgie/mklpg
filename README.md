# MKL·PW — Multi-Keyboard-Layout Password Generator

A single-file, fully offline-capable password generator that only uses characters which produce the **same output on the same physical key** across two keyboard layouts simultaneously.

Useful when you need a password you can type correctly regardless of which layout your system or muscle memory is set to — for example on a machine that may be configured with a different layout than your own, or when switching between QWERTY and Dvorak, or on machine startup LUKS password entry where only QWERTY is available.

**Live:** [yolgie.github.io/mklpg/](https://yolgie.github.io/mklpg/)

---

## How it works

Every keyboard layout maps physical keys to characters differently. This tool computes the **intersection**: characters where pressing the same physical key in the same shift state produces the same output on both the primary layout (US QWERTY) and the chosen secondary layout.

Example — the physical `A` key on a US Qwerty Keyboard:
| Layout | Unshifted | Shifted |
|---|---|---|
| US QWERTY | `a` | `A` |
| Dvorak | `a` | `A` |
| QWERTZ | `a` | `A` |
| AZERTY | `q` | `Q` |

`a` / `A` survive the QWERTY ∩ Dvorak and QWERTY ∩ QWERTZ intersections, but not QWERTY ∩ AZERTY.

The resulting charset is shown as coloured pills before the password list and updates live whenever the secondary layout changes.

---

## Features

- **Configurable secondary layout** — Dvorak, Colemak, Colemak-DH, Workman, AZERTY (French), QWERTZ (German)
- **Computed charset** — intersection is calculated at runtime, not hardcoded
- **Shell-safe toggle** — optionally excludes `` ` ~ \ | `` which cause issues in shells, `.env` files, and config parsers; automatically disabled and explained when the current layout pair contains none of those characters
- **Entropy display** — live bit count and colour-coded bar (red < 60 bits · amber < 80 bits · green ≥ 80 bits · bar fills at 128 bits / AES-128 equivalent)
- **Configurable length** — default 20, range 4–128, adjustable via buttons, keyboard arrows, mousewheel on the input field, or direct typing
- **20 passwords generated at once**, regenerated automatically on any change
- **One-click copy** per password with fallback for older browsers
- **Character class colouring** in rendered passwords: white = lowercase · amber = uppercase · green = digit · purple = symbol
- **Fully offline** — fonts embedded as base64 woff2, zero external requests, no backend, no analytics, no cookies

---

## Supported layouts

| Layout | Notes |
|---|---|
| **Dvorak** | Standard US Dvorak (ANSI). Smallest intersection with QWERTY — only `a`, `m`, digits, and number-row shifted symbols. |
| **Colemak** | Only letter keys differ from QWERTY; all symbol keys are identical, giving the largest letter overlap. |
| **Colemak-DH** | Mod-DH (DHm matrix) variant of Colemak. Slightly smaller overlap than plain Colemak. |
| **Workman** | Designed to reduce lateral finger movement. Moderate overlap with QWERTY. |
| **AZERTY** | Standard French layout (NF Z71-300). Number row is remapped; symbol key behaviour varies between hardware vendors. |
| **QWERTZ** | Standard German layout (DIN 2137). Only `y`/`z` are swapped; most symbol keys differ. Largest overall intersection. |

> **Note on AZERTY and QWERTZ:** Symbol-key mappings vary between hardware manufacturers and OS versions. For maximum portability with these layouts, prefer passwords from the letter/digit subset.

If you want any more Layouts, create a pull request or a ticket :)

---

## Security

- All randomness comes from [`crypto.getRandomValues()`](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues) — the browser's cryptographically secure RNG
- Rejection sampling is used to avoid modulo bias; bytes that would skew the distribution are discarded
- Random bytes are generated in buffered chunks rather than one syscall per character
- No password ever leaves the browser — there is no network traffic of any kind after the page loads

### Entropy reference

At the default length of 20 characters:

| Secondary layout | Charset | Charset (Shell Safe) | Entropy |
|---|---|---|---|
| Dvorak | 28 chars | 24 chars | ~96-92 bits |
| Colemak | 60 chars | 56 chars | ~118-116 bits |
| Colemak-DH | 54 chars | 50 chars | ~115-112 bits |
| Workman | 52 chars | 48 chars | ~114-111 bits |
| AZERTY | 44 chars | - | ~109 bits |
| QWERTZ | 63 chars | - | ~119 bits |

Increase the length to compensate when using a layout with a small intersection.

---

## Deployment

This is a single `index.html` file with embedded JS no build step, no dependencies, and no server-side logic.

GitHub Pages will serve it directly. No `package.json`, no bundler, no CI required.

---

## Development

The entire application lives in `index.html`. The structure is:

```
index.html
├── <style>          Embedded @font-face (JetBrains Mono + Syne, latin, base64 woff2)
├── <style>          Application CSS (CSS custom properties, layout, components)
├── <body>           Static HTML shell
└── <script>
    ├── QWERTY        Primary layout map (physicalKeyId → [unshifted, shifted])
    ├── LAYOUTS       Secondary layout definitions (deltas from QWERTY via spread)
    ├── computeCharset   Intersection logic + shell-safe filtering
    ├── secureRandom     Bias-free RNG with Uint8/Uint16 buffer
    ├── generatePassword Buffered generation with rejection sampling
    ├── generate()       Main render function — called on every user interaction
    └── Event listeners  Length input, layout select, shell-safe toggle, regen button
```

Layout definitions follow a simple convention: only keys that **differ** from QWERTY are listed; the rest are inherited via object spread. Adding a new layout means adding one entry to the `LAYOUTS` object and a matching `<option>` in the HTML.

---

## Licence

MIT
