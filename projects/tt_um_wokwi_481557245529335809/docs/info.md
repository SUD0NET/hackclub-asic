## How it works

A 4-bit code detector with two secret codes. Each letter has its own detector,
made of three 2-input AND gates. Inputs that must be 0 are inverted first.

- `c_ok = IN0 AND (NOT IN1) AND IN2 AND IN3` (code 1 0 1 1)
- `y_ok = (NOT IN0) AND IN1 AND IN2 AND (NOT IN3)` (code 0 1 1 0)

The outputs drive the 7-segment display:

- `c_ok` drives segments a and e, used only by C.
- `y_ok` drives segments b, c and g, used only by Y.
- `c_ok OR y_ok` drives segments d and f, used by both letters.
- `NOT (c_ok OR y_ok)` drives the decimal point.

The design uses 11 gates: 6 AND, 4 NOT and 1 OR. It is purely combinational,
so the clock and reset inputs are not used.

## How to test

Set inputs IN0 to IN3 and check the 7-segment display:

- 1 0 1 1 shows C.
- 0 1 1 0 shows Y.
- Any other code lights only the decimal point.

## External hardware

None beyond the input switches and 7-segment display on the demo board.
