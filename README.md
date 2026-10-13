# QChess engine packs

Chess engines for QChess on Android, one installable package each.

QChess has Stockfish built in, and it stays the default. These are other
engines, for playing or watching something else. Android 10 and later will
not run a program that an app downloads into its own storage, so each engine
here is a small package of its own. It has no code and no icon, only the
engine (as `lib/arm64-v8a/lib<id>.so`), its licence, and a note of the source
it was built from. QChess finds the installed packs and runs the engines from
them.

## Installing

**In QChess:** Settings → More engines. Press Download beside an engine and
confirm in Android's installer. The first time, Android also asks you to
allow installs from QChess. The app checks each download's size and SHA-256
against `engines.json` before handing it to the installer. Remove an engine
from the same page.

**By hand:** download an APK from the [`engines` release](../../releases/tag/engines)
and open it on the phone, or run `adb install QChess-engine-<id>.apk`.

The packs are for 64-bit ARM phones (arm64-v8a) with Android 9 or later.
RubiChess also needs ARMv8.2's dot-product instructions, which phones have
had since about 2019; QChess does not offer it on a phone without them.

## The engines

| Engine    | Version | Strength, about | Licence | Built from | Pack |
|-----------|---------|-----------------|---------|------------|------|
| Berserk   | 14      | 3500 | GPL-3.0 | [jhonnold/berserk](https://github.com/jhonnold/berserk/tree/8ae895a6151695be4a50d4fb65b0c131659c513a) `8ae895a` | 17 MB |
| Caissa    | 2.0     | 3500 | MIT     | [Witek902/Caissa](https://github.com/Witek902/Caissa/tree/bb725799e9b19ebdaa0d584f5433fc1c3019e349) `bb72579` | 41 MB |
| RubiChess | 2026    | 3450 | GPL-3.0 | [Matthies/RubiChess](https://github.com/Matthies/RubiChess/tree/9cc8ed8b965597b22a85eab304a94c7ed16adbb3) `9cc8ed8` | 64 MB |
| Stash     | 37.0    | 3400 | GPL-3.0 | [mhouppin/stash-bot](https://github.com/mhouppin/stash-bot/tree/077a93828fee354853ffac322f178d1a4f1670ff) `077a938` | 0.1 MB |
| Weiss     | 2.1-dev | 3150 | GPL-3.0 | [TerjeKir/weiss](https://github.com/TerjeKir/weiss/tree/c735b8f3d2ddb0cdf42b135a5fb42e21c01f7a3d) `c735b8f` | 0.1 MB |
| Ethereal  | 14.00   | 3100 | GPL-3.0 | [AndyGrant/Ethereal](https://github.com/AndyGrant/Ethereal/tree/96cde8d2f82a3ab0536bf33572405fda96f6e236) `96cde8d` | 0.2 MB |
| Laser     | 1.7     | 3050 | GPL-3.0 | [jeffreyan11/uci-chess-engine](https://github.com/jeffreyan11/uci-chess-engine/tree/96bfbdba8d064442626e1bb23e02473a46559a2e) `96bfbdb` | 0.4 MB |
| Vice      | 1.2     | 2100 | MIT     | [bluefeversoft/vice](https://github.com/bluefeversoft/vice/tree/a89f82dcab34b74481d6504312e3d52bbba44320) `a89f82d` | 0.04 MB |

The strengths are rough places among engines, to help choose an opponent.
They are not measurements of these builds, and a phone plays below a desktop.

Ethereal is built with its classical evaluation, without a network. Laser and
Vice do not play Chess960, and QChess will not give them a Chess960 game.

## The release

The [`engines` release](../../releases/tag/engines) holds:

- `QChess-engine-<id>.apk`, each pack;
- `<id>-<commit>-source.tar.gz`, each engine's source at the commit it was
  built from. Berserk, Caissa and RubiChess download their network at build
  time rather than keep it in their repositories, so their archives include
  the network file the build embedded;
- `engines.json`, the list QChess reads: each pack's name, version, strength,
  size, SHA-256 and requirements.

## Licences

Each engine is its authors' work under its own licence: GPL-3.0 for Berserk,
Ethereal, Laser, RubiChess, Stash and Weiss, and MIT for Caissa and Vice. Each
pack carries that licence (`assets/LICENSE`) and a note of its source
(`assets/SOURCE`). A pack's complete source is the archive beside it in the
release, and [BUILD.md](BUILD.md) gives the commands that built it.
