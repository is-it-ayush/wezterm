# Wez's Terminal

<img height="128" alt="WezTerm Icon" src="https://raw.githubusercontent.com/wezterm/wezterm/main/assets/icon/wezterm-icon.svg" align="left"> *A GPU-accelerated cross-platform terminal emulator and multiplexer written by <a href="https://github.com/wez">@wez</a> and implemented in <a href="https://www.rust-lang.org/">Rust</a>*

User facing docs and guide at: https://wezterm.org/

![Screenshot](docs/screenshots/two.png)

*Screenshot of wezterm on macOS, running vim*

## About this fork: Hindi / Indic text rendering

This fork fixes the rendering of Hindi (Devanagari) and other Indic scripts,
reported upstream as [wezterm/wezterm#7355](https://github.com/wezterm/wezterm/issues/7355).
Pasting Hindi into a terminal produced scrambled or overlapping characters,
broken words and a cursor that drifted away from the text, even though the
rest of the desktop rendered the same text correctly.

### What was wrong

1. **The renderer drew vowel signs in the wrong place.** A consonant with a
   vowel sign (का, कि, की, के ...) is shaped into several glyphs that belong to
   one cell group. The vowel sign glyph occupies zero cells, so it was drawn at
   the start of the *next* cell and overlapped the following letter.
2. **Wide letters were shrunk unevenly.** The glyph cache shrinks any glyph
   wider than 1.25 cells in both dimensions. Wide Devanagari letters (अ क ल स)
   crossed that limit while narrow ones (न प म त) did not, so letters inside a
   word came out in different sizes.
3. **Column widths disagreed with the shell.** Spacing vowel signs counted as
   0 columns, while libc `wcwidth(3)`, VTE, zsh and readline count them as 1.
   Three-consonant conjuncts were additionally capped at 2 columns. The cursor
   position the shell computed therefore drifted from where wezterm put it.

### What changed

* `wezterm-cell`: Indic spacing marks (Unicode General_Category `Mc`, U+0900 to
  U+0DFF) are one column wide, and Indic conjuncts are no longer capped at two
  columns, matching `wcswidth`.
* `wezterm-gui` glyph cache: glyphs of an Indic syllable are given the room of
  the whole syllable instead of being shrunk one by one.
* `wezterm-gui` line renderer: the glyphs of one grapheme are positioned by
  their shaped advances instead of by cell. Indic text is condensed uniformly,
  centered in its cells, and the remaining space is filled with the headline
  (shirorekha) so words stay connected without distorting letters.

### Known limitations

A terminal is a fixed grid and Devanagari is proportional text, so some
spacing differences between syllables remain. Editors count columns by their
own tables: Neovim treats these marks as zero width, so its cursor can be off
by one column per vowel sign. Shells delete one code point per backspace, not
one syllable. The headline fill is Devanagari only; other Indic scripts get
the width fixes and uniform condensing.

If you use zsh, also enable `setopt COMBINING_CHARS` in `~/.zshrc`. Without
it zsh prints zero width combining marks as literal `<0941>` placeholders
before the terminal ever sees them.

### Building

```sh
# Debian / Ubuntu build dependencies
sudo apt-get install -y \
    bsdutils cmake dpkg-dev fakeroot gcc g++ \
    libegl1-mesa-dev libssl-dev libfontconfig1-dev libwayland-dev \
    libx11-xcb-dev libxcb-ewmh-dev libxcb-icccm4-dev libxcb-image0-dev \
    libxcb-keysyms1-dev libxcb-randr0-dev libxcb-render0-dev libxcb-xkb-dev \
    libxkbcommon-dev libxkbcommon-x11-dev libxcb-util-dev \
    lsb-release python3 xdg-utils xorg-dev

# Rust toolchain (stable), if you do not have one
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Get the source and build
git clone https://github.com/is-it-ayush/wezterm.git
cd wezterm
git checkout indic-rendering
git submodule update --init --recursive
cargo build --release -p wezterm -p wezterm-gui -p wezterm-mux-server

# Run it
./target/release/wezterm-gui
```

On other distributions run `./get-deps`, which installs the dependencies for
your platform. Install the binaries from `target/release/` somewhere on your
`PATH` (for example `sudo install -m 755 target/release/wezterm* /usr/local/bin/`)
to use them as your terminal.

## Installation

https://wezterm.org/installation

## Getting help

This is a spare time project, so please bear with me.  There are a couple of channels for support:

* You can use the [GitHub issue tracker](https://github.com/wezterm/wezterm/issues) to see if someone else has a similar issue, or to file a new one.
* Start or join a thread in our [GitHub Discussions](https://github.com/wezterm/wezterm/discussions); if you have general
  questions or want to chat with other wezterm users, you're welcome here!
* There is a [Matrix room via Element.io](https://matrix.to/#/#wezterm:matrix.org)
  for (potentially!) real time discussions.

The GitHub Discussions and Element/Gitter rooms are better suited for questions
than bug reports, but don't be afraid to use whichever you are most comfortable
using and we'll work it out.

## Supporting the Project

If you use and like WezTerm, please consider sponsoring it: your support helps
to cover the fees required to maintain the project and to validate the time
spent working on it!

[Read more about sponsoring](https://wezterm.org/sponsor.html).

* [![Sponsor WezTerm](https://img.shields.io/github/sponsors/wez?label=Sponsor%20WezTerm&logo=github&style=for-the-badge)](https://github.com/sponsors/wez)
* [Patreon](https://patreon.com/WezFurlong)
* [Ko-Fi](https://ko-fi.com/wezfurlong)
* [Liberapay](https://liberapay.com/wez)
