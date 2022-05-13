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


## Copyright & Licensing

Copyright (C) 2022  Jacob Koziej [`<jacobkoziej@gmail.com>`]

Copyright (C) 2022  Ani Vardanyan [`<ani.var.2003@gmail.com>`]

Licensed under the [CC BY-NC-SA 4.0].


[`<jacobkoziej@gmail.com>`]: mailto:jacobkoziej@gmail.com
[`<ani.var.2003@gmail.com>`]: mailto:ani.var.2003@gmail.com
[CC BY-NC-SA 4.0]: https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode
