# Classic Series RE

Independent 64-bit reimplementations of the nine **Kjaerhus Audio “Classic” series** plug-ins
for **macOS and Windows**, rebuilt with AI from static analysis of the original 32-bit Windows
binaries so that projects which reference the originals can still be opened.

![All nine editors, running on macOS](https://raw.githubusercontent.com/drumkilla/classic-series-re/main/overview.png)

> **Not affiliated with, endorsed by, or connected to Kjaerhus Audio.**
> The original algorithms and designs are theirs. Kjaerhus Audio was
> a Danish developer whose Classic series shipped as freeware; the company ceased trading
> around 2009. Everything in this repository was written from scratch — no
> Kjaerhus code, artwork or assets are included or redistributed.

### ⬇ [Download](https://github.com/drumkilla/classic-series-re/releases)

Read **Install** section below.

---

## What this is

The Classic series were 32-bit Windows-only plug-ins. They do not run on a Mac at all, and on
64-bit Windows they load only through a bridge — if the host still offers one. Several
current DAWs have dropped 32-bit hosting entirely, so on both platforms these sessions are
now closed.

These are reconstructions: the DSP was recovered instruction by instruction from the original
binaries and rewritten in plain C++ with AI, then checked against renders made by the originals
themselves until the output matched.

**This is not an emulation, a wrapper or a bridge.** No original binary is loaded, shipped or
required. Both platforms build from one source tree and one frozen DSP per plug-in.

## What this is not

- Not the original plug-ins, and not a way to get them.
- Not a sound-alike or an "in the spirit of" tribute — the goal was sample-accurate
  reproduction.
- Not a supported product. It is a personal restoration project, published in case it is
  useful to someone with the same problem.

---

## The plug-ins

| Plug-in | Original id | Golden cases | Result |
|---|---|---|---|
| Classic Reverb | `6z9v` / 913979766 | 10 | at the dither floor |
| Classic Delay | `6f75` / 912668469 | 18 | at the dither floor |
| Classic Chorus | `4233` / 875705139 | 18 | at the dither floor |
| Classic Phaser | `c934` / 1664693044 | 24 | at the dither floor |
| Classic Flanger | `c618` / 1664495928 | 25 | at the dither floor |
| Classic Compressor | `o132` / 1865495346 | 26 | **bit-exact** |
| Classic Auto-Filter | `fq78` / 1718695736 | 32 | **bit-exact** |
| Classic EQ | `q1i8` / 1899063608 | 36 | **bit-exact** |
| Classic Master Limiter | `208i` / 842020969 | 28 | **bit-exact** |

"Bit-exact" means the reconstruction reproduces the original's output **sample for sample** —
residual identically zero, not merely inaudible. The four that reach it do so on both the
macOS and the Windows build of the same source.

The first five predate that technique and were validated against the original's own
anti-denormal dither floor instead, which is the best that can be done without forcing the
plug-in's random seed. They are correct to well below audibility; they are not proven
identical.

---

## Install

Both platforms are **64-bit**. The originals were 32-bit; that difference is the point.

**macOS** — 10.13+, universal (Intel + Apple Silicon), ad-hoc signed.

```
VST3    →  ~/Library/Audio/Plug-Ins/VST3/
2.4     →  ~/Library/Audio/Plug-Ins/VST/
presets →  ~/Library/Audio/Presets/wdak audio/<plug-in name>/
```

The plug-ins are ad-hoc signed, so macOS quarantines them on download. **After copying them
in**, you **must** run these two commands in Terminal:

```
xattr -dr com.apple.quarantine ~/Library/Audio/Plug-Ins/VST3/Classic*.vst3
xattr -dr com.apple.quarantine ~/Library/Audio/Plug-Ins/VST/Classic*.vst
```

Otherwise the host won't load them.

**Windows** — 64-bit, built with MSVC.

```
VST3    →  C:\Program Files\Common Files\VST3\
2.4     →  wherever your host scans, commonly C:\Program Files\VSTPlugins\
presets →  %USERPROFILE%\Documents\VST3 Presets\wdak audio\<plug-in name>\
```

Rescan in your host afterwards.

Note: these builds report **wdak audio** as their vendor, so a plug-in browser does not
confuse them with the originals — both carry the same plug-in names.

---

## ⚠️ Read this before installing the 2.4 builds

Each 2.4 build **reports the original plug-in's four-byte identifier** (the table above).
That is the entire mechanism by which an old project resolves to it: a host looks a 2.4
plug-in up by that identifier alone — the name, path and vendor recorded next to it in the
project file are stored but never consulted. Nothing else works. A VST3 cannot answer a 2.4
reference no matter what it declares; that route was tested and it is a dead end.

The consequence is unavoidable and you should understand it:

- **A host cannot tell these apart from the originals.** By design.
- **Do not leave the original in your scan path once you install one of these.** Which of the
  two loads depends on scan order.
- **On Windows this matters even though the bitness differs.** If you currently run the
  originals through jBridge or your host's own bridge, installing the native 64-bit build
  gives the host two plug-ins claiming the same identifier — one bridged, one native. Remove
  the original from the scan path, or disable the bridge for it. That migration is the
  intended use, not a side effect.

The **VST3 builds carry their own unique class IDs** and collide with nothing. If you do not
specifically need to reopen old sessions, install only those.

---

## How they were verified

Each plug-in has a golden set: the original, running on Windows, rendered a fixed test signal
at a matrix of parameter settings and both sample rates, and the reconstruction has to
reproduce those renders. Where the plug-in dithers, its random seed is forced to a known
value on both sides so the comparison has no noise floor to hide behind.

Beyond that, every plug-in ships only if all of these pass on **both** toolchains — clang on
macOS and MSVC on Windows:

- DSP self-checks, asserting recovered constants and behaviour against the disassembly
- the golden comparison
- a VST3 wrapper regression proving the wrapper is bit-transparent to the raw DSP
- a 2.4 test that loads the built bundle the way a host does
- a headless editor test checking the panel geometry against the original's own form resource
- preset round-trips, including conversion of `.fxp`/`.fxb` files saved by the original
- Steinberg's `validator`

And finally a null test in a host: the reconstruction and the original on two tracks with
matching settings, rendered as stems and differenced.

Three of the five older plug-ins show a small, documented numerical difference between the
macOS and Windows builds. It comes from transcendental functions evaluated per sample, where
the two C runtimes disagree in the last bit; it is bounded, measured against the smallest
step the original's own arithmetic can represent, and both builds pass their criterion. It is
inherent to reproducing this arithmetic, not a defect in either build.

---

## Known differences from the original

Deliberate, all of them:

- **The panels are redrawn, not reused.** Geometry — control positions, sizes, knob sweeps,
  meter scales — was measured from the original's own compiled form resource, so the layout
  matches. The artwork is new. No Kjaerhus bitmap is included.
- **The VST3 builds have a bypass switch.** The originals have none. It is a host-level
  convenience and is not part of the reconstruction.
- **Classic Master Limiter reports 580 samples of latency**, because the original does. Your
  host will compensate. The other eight report none, also because the originals do.

Everything else — parameter ranges, the exact text of every readout (including a couple of
quirks that look like bugs and are faithfully reproduced), the factory presets, the program
model — follows the original.

---

## Licence and attribution

Binaries in this repository are provided as-is, without warranty of any kind.

Original Classic series design, algorithms and panel layouts: Kjaerhus Audio. This project
claims no rights in them and no association with them. If a rights-holder to the Kjaerhus
Audio catalogue objects to this repository, open an issue and it comes down.

The 2.4 plug-in interface used by these builds is a clean reimplementation written from the
publicly documented ABI; no Steinberg SDK code is included. **VST** is a trademark of
Steinberg Media Technologies GmbH. VST3 builds use the Steinberg VST3 SDK (MIT) and VSTGUI.
