## How it works

Morse code decoder. You tap on one button (ui[0]) and it shows the letter on the 7 segment display after you stop. It also puts the ASCII code on the uio pins.

Clock is 10 kHz, which gets divided down to a 1ms tick.

- The button goes through 2 flip flops and a debouncer (has to stay the same for 4ms)
- A timer counts how long you held it. Under 200ms is a dot, 200ms or more is a dash
- Dots and dashes get shifted into a 6 bit register that starts at 000001 (dot = 0, dash = 1). The leading 1 is how it knows where the code starts, so A (.-) ends up as 000101
- After 500ms with no press the letter is done and goes to a lookup table that gives the 7 segment pattern and ASCII
- More than 5 symbols or a code that isn't a letter shows a - (ASCII ?)

SLOW (ui[1]) makes all the timings 2x longer. CLEAR (ui[2]) blanks the display.

uo[7] beeps at about 625 Hz while the key is down so you can hook up a buzzer. It's also the decimal point so the dot lights up when you press.

uio[7] flips every time a new letter comes out.

| | normal | slow |
|-|-|-|
| dot | < 200ms | < 400ms |
| dash | >= 200ms | >= 400ms |
| end of letter | 500ms pause | 1000ms pause |

Some letters look the same as numbers on the display (S/5, Z/2), the ASCII output is always right though.

## How to test

Set the clock to 10 kHz and reset. Display should be blank.

Tap ui[0], quick tap for dot, about half a second for dash, wait a second between letters. Try SOS (... --- ...).

With a logic analyzer you can read the ASCII on uio[6:0], uio[7] toggles when there's a new one.

| | | | | | |
|-|-|-|-|-|-|
| A .- | B -... | C -.-. | D -.. | E . | F ..-. |
| G --. | H .... | I .. | J .--- | K -.- | L .-.. |
| M -- | N -. | O --- | P .--. | Q --.- | R .-. |
| S ... | T - | U ..- | V ...- | W .-- | X -..- |
| Y -.-- | Z --.. | 0 ----- | 1 .---- | 2 ..--- | 3 ...-- |
| 4 ....- | 5 ..... | 6 -.... | 7 --... | 8 ---.. | 9 ----. |

## External hardware

Nothing needed, the demo board display and switches work. A push button on ui[0] is easier than a switch though, and you can put a buzzer on uo[7].
