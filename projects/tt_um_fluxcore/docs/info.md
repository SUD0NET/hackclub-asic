## How it works

FluxCore is a **2×2 coarse-grained reconfigurable array (CGRA)** made from four 32-bit processing elements (PEs).

The basic idea is to have a small array of maths units that can be configured to perform different operations and work together.

Each processing element contains:

- A 32-bit ALU
- Input-selection multiplexers
- A 32-bit result register
- A 32-bit accumulator

The four PEs are arranged in a 2×2 mesh:

```text
+-------+-------+
|  PE0  |  PE1  |
+-------+-------+
|  PE2  |  PE3  |
+-------+-------+
```

### Processing element configuration

Each PE uses an 8-bit configuration word.

| Bits    | Function             |
| ------- | -------------------- |
| `[3:0]` | ALU operation        |
| `[5:4]` | Source for operand A |
| `[7:6]` | Source for operand B |

The source selection uses four directions:

| Value | Input |
| ----- | ----- |
| `00`  | North |
| `01`  | East  |
| `10`  | South |
| `11`  | West  |

The ALU supports the following operations:

| Opcode | Operation |
| ------ | --------- |
| `0x0`  | ADD       |
| `0x1`  | SUB       |
| `0x2`  | MUL       |
| `0x3`  | MAC       |
| `0x4`  | AND       |
| `0x5`  | OR        |
| `0x6`  | XOR       |
| `0x7`  | NOT       |
| `0x8`  | SHL       |
| `0x9`  | SHR       |
| `0xA`  | PASS_A    |
| `0xB`  | PASS_B    |
| `0xC`  | ACC_CLEAR |

The MAC operation multiplies the selected operands and adds the low 32 bits of the product to the PE's accumulator. `ACC_CLEAR` clears that accumulator.

### How data moves

The interesting part of a CGRA is its dataflow.

Each PE selects its inputs using multiplexers. After performing its operation, the result is stored in a register and can be used by another PE through the mesh.

This means I do not need to manually move every intermediate result outside the array and send it back to the next processing element.

I configure the operations and connections, provide the initial operands, and let the array process the data over successive clock cycles.

The results are registered, so data does not travel through the entire array in one clock cycle. A downstream PE may need another execute step to consume the newly calculated value from an upstream PE.

### Configuration protocol

The four 8-bit PE configurations are loaded serially as one 32-bit stream.

The transmission order is:

```text
PE3 → PE2 → PE1 → PE0
```

Each configuration byte is sent most-significant bit first.

The wrapper uses:

- `config_data` for the configuration bit.
- `config_clk` to shift the configuration.
- `config_enable` to enable configuration shifting.

After exactly 32 bits have been received, `config_done` is asserted.

### Operand input

FluxCore accepts eight 32-bit operand words, for a total of 256 bits.

Each word is sent most-significant bit first. The `operand_valid` signal indicates when an operand bit is being supplied.

The current wrapper accepts this operand batch once after reset.

### Reading results

The PE is selected using `uio[1:0]`:

| Selection | PE  |
| --------- | --- |
| `00`      | PE0 |
| `01`      | PE1 |
| `10`      | PE2 |
| `11`      | PE3 |

The `read_result` input requests a result readback.

The `result_source` input selects what to read:

- `0`: PE result register
- `1`: PE accumulator

The selected 32-bit value is shifted out most-significant bit first through `uo[0]`.

`result_valid` indicates that serial result data is available.

### Tiny Tapeout pinout

| Pin        | Signal               | Description                    |
| ---------- | -------------------- | ------------------------------ |
| `ui[0]`    | `config_data`        | Serial configuration bit       |
| `ui[1]`    | `config_clk`         | Configuration clock            |
| `ui[2]`    | `config_enable`      | Configuration shift enable     |
| `ui[3]`    | `execute`            | Advances the processing array  |
| `ui[4]`    | `operand_data`       | Serial operand bit             |
| `ui[5]`    | `operand_valid`      | Operand data valid             |
| `ui[6]`    | `read_result`        | Requests result readback       |
| `ui[7]`    | `result_source`      | Selects result or accumulator  |
| `uo[0]`    | `serial_result_data` | Serial result output           |
| `uo[1]`    | `busy_pulse`         | One-cycle execute status pulse |
| `uo[2]`    | `config_done`        | Configuration complete         |
| `uo[3]`    | `done_pulse`         | One-cycle execute status pulse |
| `uo[4]`    | `result_valid`       | Result data valid              |
| `uio[1:0]` | `pe_index`           | Selects PE0–PE3                |

The remaining output and bidirectional pins are unused. The design keeps the bidirectional pins configured as inputs.

## How to test

The project includes a Cocotb testbench for the Tiny Tapeout wrapper.

The tests can be run against the generated RTL, and the same tests can also be run against the gate-level netlist after Tiny Tapeout hardening.

### RTL simulation

From the repository root:

```sh
cd test
python -m pip install -r requirements.txt
make -B
```

Cocotb generates the system clock and drives the Tiny Tapeout inputs.

The testbench checks four behaviours:

**1. Reset and idle**

Checks that the outputs return to zero after reset and that the bidirectional pins remain configured as inputs.

**2. Configuration length**

Checks that `config_done` remains low for the first 31 configuration shifts and asserts on exactly the 32nd shift.

**3. PE-to-PE addition pipeline**

PE0 calculates:

```text
10 + 7 = 17
```

PE1 then consumes PE0's registered result and adds another operand:

```text
17 + 5 = 22
```

Both results are read back through the serial interface.

**4. MAC accumulator**

PE0 performs `7 × 9` twice.

The expected accumulator value is:

```text
63 + 63 = 126
```

The test reads back the accumulator and checks that the result is `126`.

### Gate-level simulation

After the Tiny Tapeout hardening flow generates `gate_level_netlist.v`, run:

```sh
make -B GATES=yes
```

This runs the same Cocotb tests against the gate-level netlist and GF180MCU standard-cell models.

The final recorded GitHub Actions run completed with:

```text
TESTS=4
PASS=4
FAIL=0
```

The complete build also passed GDS generation, Tiny Tapeout precheck, and viewer deployment.

[View the final FluxCore build](https://github.com/Nitin-2468-dev/Flux-Core/actions/runs/44837107502)

### Current limitations

The tests cover important interface and datapath behaviours, but they are not exhaustive verification of every operation or every possible PE configuration.

The current wrapper also has a few limitations:

- Configuration is loaded once after reset.
- The eight-word operand batch is loaded once after reset.
- Operand reload without reset is not currently implemented.
- `busy` and `done` are one-cycle pulses, not a multi-cycle execution handshake.
- Registered PE-to-PE dataflow may require multiple execute pulses for values to propagate through the array.

## External hardware

No additional external components are required to simulate or build FluxCore.

The design uses the standard Tiny Tapeout interface, including:

- System clock (`clk`)
- Active-low reset (`rst_n`)
- Enable (`ena`)
- Dedicated input and output GPIO
- Bidirectional GPIO, used as inputs in this design

For a physical demonstration, an external host such as a microcontroller or FPGA could drive the GPIO interface.

The host would need to:

1. Send the 32-bit configuration stream.
2. Send the eight 32-bit operand words.
3. Generate execute pulses.
4. Select a processing element.
5. Request either the PE result or accumulator.
6. Read the serial result bits.

This is a possible way to operate the chip; the project does not currently require a particular microcontroller, carrier board, display, or external memory.

FluxCore is the compute block itself, not a complete standalone computer or development board.
