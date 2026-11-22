<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

## How it works

This project aims to recreate the look and feel of the original Pong game.

It is a single-player game: the leftmost paddle is controlled by the player via `ui_in[0]`, and the rightmost paddle is controlled by an AI algorithm.

Graphics are procedurally-generated (on the fly), meaning that there is no need to store a large framebuffer. As a result, there is very little memory usage.

Rather than using memory to store graphical characters, scores are instead indicated by "tally marks" at the top of the screen. Since graphics are generated on the fly and coordinates are fixed, this requires virtually no memory, except for the two three-bit counters. After either the player or the AI gets to three points, the game resets.

### Module layout

`vga_controller`: generates the physical VGA signals for the Tiny VGA PMOD.
`ticker`: generates customizable "tick" signals independent from the ASIC's clock frequency.
`renderer`: determines RGB color values given the current scanline position (then feeds this to the VGA controller).
`paddle`, `ball`: self-explanatory.

## How to test

After connecting your push button (See "External hardware" below), all you need to do to move your paddle up is push! The paddle moves down by default, when the button is not pushed. 

To test the design on your computer, you can visit this link to play the game in your browser (VGA Playground): https://vga-playground.com/?repo=https://github.com/omrawaley/tt-pong-asic

## External hardware

This project requires 2 external hardware components:

1. A momentary switch or push button connected to `ui_in[0]`. This button should be wired so that pressing it connects the pin to HIGH. In addition, it should be externally debounced, and it should have a pulldown resistor.

2. The Tiny VGA PMOD is required to view the generated video output. If you don't have a monitor that accepts VGA, you may also need a VGA to HDMI adapter.
