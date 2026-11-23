# Verilog CRC32

## How it works

This is a CRC32 implementation in verilog.
It takes any number of bytes as input, LSB first, keeps taking input every clock until done goes high. \It processes one bit per clock signal.

Both the input and outputs are serial, \it starts taking input on the first clock edge after (active low) reset goes high, \and it starts outputing on the first clock edge after done goes high.

## How to test

You can test it by compiling src/project.v and test/tb_standalone.v with iverilog, and simulate it using vvp. \
The simulation sould print either "PASS" or "FAIL", depending on if it outputed correctly or not. \
The simulation can be viewed with GTKwave. \
Example:
```
iverilog -o out src/project.v test/tb_standalone.v
vvp out -fst
gtkwave out.fst
```

