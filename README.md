# report

> *Dear Professor Rob Marano,*


## Instruction Set Architecture

Before writing any Verilog, we first needed to describe a somewhat
rigorous Instruction Set Architecture (ISA).  Ideally, such an abstract
model would exist before writing anything in a Hardware Description
Language (HDL) since it serves as the basis for how software controls a
Central Processing Unit (CPU).  Given that this was our first time
designing a CPU from the ground up, much of the ISA was in constant flux
as we slowly converged on our final design.


### Inspiration

Inspiration for JAVK came from the GameBoy's LR35902 and the MOS
Technology 6502.  We most heavily borrowed on the idea of an accumulator
as this allowed us to squeeze all our instructions into a single byte,
avoiding the need for variable-length instructions.


### Registers

JAVK consists of 20 addressable registers.  Of these, 16 are 8-bit,
while the remaining four are 16-bit.  Of these 16-bit registers, two map
directly on top of two 8-bit registers allowing for manipulation using
an 8-bit ALU.

```
Register Map:

+-------+-------+
|   A   |   B   |
+-------+-------+
|   C   |   D   |
+-------+-------+
|   E   |   F   |
+-------+-------+
|   G   |   H   |
+-------+-------+
|   I   =   J   |
+-------+-------+
|   K   =   L   |
+-------+-------+
|   M   |   N   |
+-------+-------+
|   O   |   Z   |
+-------+-------+
|      P C      |
+-------+-------+
|      S P      |
+-------+-------+
```

| Register |  Index   | Size  |
| :------: | :------: | :---: |
|   `A`    | `0b0000` | `8b`  |
|   `B`    | `0b0001` | `8b`  |
|   `C`    | `0b0010` | `8b`  |
|   `D`    | `0b0011` | `8b`  |
|   `E`    | `0b0100` | `8b`  |
|   `F`    | `0b0101` | `8b`  |
|   `G`    | `0b0110` | `8b`  |
|   `H`    | `0b0111` | `8b`  |
|   `I`    | `0b1000` | `8b`  |
|   `J`    | `0b1001` | `8b`  |
|   `K`    | `0b1010` | `8b`  |
|   `L`    | `0b1011` | `8b`  |
|   `M`    | `0b1100` | `8b`  |
|   `N`    | `0b1101` | `8b`  |
|   `O`    | `0b1110` | `8b`  |
|   `Z`    | `0b1111` | `8b`  |
|   `PC`   |  `0b00`  | `16b` |
|   `SP`   |  `0b01`  | `16b` |
|   `IJ`   |  `0b10`  | `16b` |
|   `KL`   |  `0b11`  | `16b` |


#### Special Registers

* `A` - accumulator.  The result of all arithmetic operations reside in
  this register.  This register is also used as the first operand for
  all ALU operations (with exception of negation).  This register is
  also the only register which can load and store data of the databus.
* `F` - flags.  All modifiable and readable CPU flags reside in this
  register.
* `I` - high byte of the `IJ` register.
* `J` - low byte of the `IJ` register.
* `K` - high byte of the `KL` register.
* `L` - low byte of the `KL` register.
* `Z` - zero.  The zero register is always contains a constant value of
  zero.  All writes to this register are discarded.
* `PC` - program counter.  The program counter holds the address of the
  next instruction to be executed.
* `SP` - stack pointer.  The stack pointer stores the address of the
  last program request in a stack.
* `IJ` - intended jump.  The intended jump register serves two primary
  roles: containing the address from where data is read to/written from
  the `A` register or the address which execution should jump to.
* `KL` - keep link.  Upon a jump with link (`JPL`), the `PC` register
  will be incremented and stored in this register.


### Core Instruction Set

| Instruction | Format |   Encoding    |     Description      |
| :---------: | :----: | :-----------: | :------------------- |
|    `ADD`    |  `R`   | `0b0000 XXXX` | Add                  |
|    `SUD`    |  `R`   | `0b0001 XXXX` | Subtract             |
|    `NEG`    |  `R`   | `0b0010 XXXX` | Negate               |
|    `AND`    |  `R`   | `0b0011 XXXX` | AND                  |
|    `ORR`    |  `R`   | `0b0100 XXXX` | Inclusive OR         |
|    `EOR`    |  `R`   | `0b0101 XXXX` | Exclusive OR         |
|    `LSL`    |  `I`   | `0b0110 XXXX` | Logical shift left   |
|    `LSR`    |  `I`   | `0b0111 XXXX` | Logical shift right  |
|    `MVA`    |  `R`   | `0b1000 XXXX` | Move `A` register    |
|    `MVB`    |  `R`   | `0b1001 XXXX` | Move 16-bit register |
|    `LNL`    |  `I`   | `0b1010 XXXX` | Load nibble low      |
|    `LNH`    |  `I`   | `0b1011 XXXX` | Load nibble high     |
|    `LDB`    |  `D`   | `0b1100 XXXX` | Load byte            |
|    `STB`    |  `D`   | `0b1101 XXXX` | Store byte           |
|    `JMP`    |  `B`   | `0b1110 XXXX` | Jump                 |
|    `JPL`    |  `B`   | `0b1111 XXXX` | Jump (with link)     |


#### Instruction Encoding

As the table above would suggest, all instructions in JAVK consist of a
single byte where the high nibble corresponds to the opcode, and the
lower nibble corresponds to the operand.


##### Arithmetic Operations

Arithmetic operations take up eight of the total 16 instructions
available in JAVK and take up the `0b0XXX` address range.  By mapping
the arithmetic operations to this address range, the lower three bits
can be directly passed to the ALU's opcode input and enabled when the
MSB of the opcode is a zero.

For most operations, the operand corresponds to the register index of
the second operand to the ALU.  The exceptions to this convention are
the two logical shift operations, as they use the four bits to specify a
shift amount for the value in the accumulator.


##### Movement Operations

The two movement operations take up the `0b100X` address range.

`MVA` (move `A`) is the first of the two movement operations and moves
the contents of the `A` register to the specified operand register.  The
only exception is the `Z` register which maintains the constant value of
zero.

`MVB` (move "big") is the second of the two movement operations and
moves the contents of one 16-bit register to another.  Since there are
only four 16-bit registers, this allows for specifying both a source and
a destination register.  The high two bits of the operand correspond
with the destination register, and the low two bits correspond with the
source register.


##### Immediate Loading Operations

The two immediate loading operations take up the `0b101X` address range.

Both operations store the nibble as the operand.  The only distinction
between the two operations is which nibble they replace in the `A`
register.


## Copyright & Licensing

Copyright (C) 2022  Jacob Koziej [`<jacobkoziej@gmail.com>`]

Copyright (C) 2022  Ani Vardanyan [`<ani.var.2003@gmail.com>`]

Licensed under the [CC BY-NC-SA 4.0].


[`<jacobkoziej@gmail.com>`]: mailto:jacobkoziej@gmail.com
[`<ani.var.2003@gmail.com>`]: mailto:ani.var.2003@gmail.com
[CC BY-NC-SA 4.0]: https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode
