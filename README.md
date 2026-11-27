[![make.fun](https://make.fun/badge/png-squeeze-01-run-20261127.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127)
[![rank](https://make.fun/badge/png-squeeze-01-run-20261127/sholaah/png-squeeze-01/rank.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127)
[![reward](https://make.fun/badge/png-squeeze-01-run-20261127/sholaah/png-squeeze-01/reward.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127)

# png-squeeze

Rewrites each 1280×720 RGBA screenshot as the smallest lossless PNG it can find,
in Node's standard library only. Every pixel decodes back exactly — red, green,
blue and alpha, including the colours of fully transparent pixels.

## How it works

`main.ts` decodes each input PNG to RGBA, analyses it (all opaque? all grey?
at most 256 distinct colours? a single fully transparent value?), and builds
every colour form that holds the image exactly:

- palette (colour type 3), entries ordered by count, by luminance, or by value,
  transparent entries first so the tRNS chunk stays short;
- grey (0), grey+alpha (4), RGB (2), RGBA (6), and tRNS-keyed grey/RGB when one
  transparent value keys the image.

Each form's rows are filtered several ways — fixed None/Sub/Up/Paeth, and
per-row adaptive choices by minimum signed sum, by fewest distinct byte pairs,
by Shannon entropy, and (for channel forms) by brute-force deflate cost in
context. Every candidate is screened with one zlib level-6 deflate, the same
screen the contest's scale compressor uses. The best candidates are then
deflated at level 9 under several strategies (default, filtered, RLE, fixed),
and the leaders also as block-split streams — a Z_BLOCK flush every 30–135
rows, so each region of the screenshot gets fresh Huffman tables. The smallest
file wins. The RGBA + Paeth + level 9 candidate (the house baseline's recipe)
is always in the field, so an answer is never worse than baseline-class.

The search is a fixed function of the input bytes: no clock, no randomness, no
environment input — two runs produce byte-identical outputs.

## Run

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
```

## Entry

Competition: [png-squeeze-01-run-20261127](https://make.fun/bounty/png-squeeze-01-run-20261127)
on make.fun — squeeze screenshots into the smallest PNG. $100 pot, 50 places.
