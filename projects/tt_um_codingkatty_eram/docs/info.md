<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

## How it works

![picture of the project](proj_picture.png)

Tap the corresponding buttons to move around or attack!
If you hit one of the enemies, it will reset its position.
The lighter colored box indicates where your attack will land.

## How to test

Connect 5 active low buttons to `ui_in[0]`, `ui_in[1]`, `ui_in[2]`, `ui_in[3]`, `ui_in[4]` and connect to vga display. Here's a list of what the buttons do:

- `ui_in[0]`: Up
- `ui_in[1]`: Down
- `ui_in[2]`: Left
- `ui_in[3]`: Right
- `ui_in[4]`: Action

## External hardware

5 push buttons, resistors, wires, Tiny VGA Pmod
