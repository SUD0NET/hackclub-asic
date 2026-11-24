<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

## How it works

Parallel to Serial
- Set uio 0 to 0
- Input: ui 7-0
- Output: uo 0

Serial to Parallel
- Set IO 0 to 1
- Input: ui 0
- Output: uo 7-0

## How to test

See `test.py` or view GDS output

## External hardware

List external hardware used in your project (e.g. PMOD, LED display, etc), if any
