# ISA for Radish-VM

The VM will be based around a load-store/RISC-ish design. The CPU will be 32-bit and we'll use memory mapped IO as much as possible.

Not planning to stick to RISC concepts too strictly (we don't have hardware limitations for doing this after all).

## Registers

All registers are 32-bit.

| Register | Index | Description |
| -------- | ----- | ----------- |
| zero | 0 | Constant 0 value. Useful to zero things fast? |
| sp | 1 | Stack pointer. |
| ?? | 2 | ?? |
| ?? | 3 | ?? |
| ?? | 4 | ?? |
| ?? | 5 | ?? |
| ?? | 6 | ?? |
| ?? | 7 | ?? |
| ?? | 8 | ?? |
| ?? | 9 | ?? |
| ?? | 10 | ?? |
| ?? | 11 | ?? |
| ?? | 12 | ?? |
| ?? | 13 | ?? |
| ?? | 14 | ?? |
| ?? | 15 | ?? |

With only 16 registers we can encode a register index in 4 bits!

## Instructions

All instructions are 32-bit.

All values are stored in little-endian format.

Addressing modes (# = immediate, * = value of):

| Code | Description |
| ---- | ----------- |
| IL | Immediate loads `LOAD r, #x` |
| RL | Register-address loads `LOAD r, *[x + y]` |
| IR | Immediate stores `STOR r, #x` |
| RR | Register-address stores `STOR r, *[x + y]` |
| XM | Masked-immediate transfers `XFER y, [*x & #mask]` |
| XR | Masked-register transfers `XFER y, [*x & *mask]` |

Note: with our instruction size and 4-bit register encoding, we can get a free 16-bit offset built into all RL and RR instructions!

# Opcode Info

All opcodes are 7 bits long, leaving us with 25 bits for data encoding.

| Mnemonic | Addressing Mode | Opcode | Description | Encoding | Parameters |
| -------- | --------------- | ------ | ----------- | -------- | ---------- |
| NOP | N/A | 00 | No operation | 00 00 00 00 | None |
| LOAD.W | RL | 01 | Loads a 32-bit word from address [X + Y] into the specified register [Z]. | 01 XZ YY YY | X = register index<br> Y = 16-bit signed offset<br> Z = register index |
| LOAD.H | IL | 02 | Loads an immediate 16-bit value [Y] into register [X]. [N] controls whether to use the high or low halfword of the target register. | 02 XN YY YY | X = register index<br> Y = 16-bit value<br> N = 1 if high halfword<br> 0 if low halfword |
| LOAD.B | IL | 03 | Loads an immediate 8-bit value [Y] into register [X]. [N] controls which of the 4 locations in X to load into. | 03 XN YY 00 | X = register index<br> Y = 8-bit value<br> N = byte within the word to load into |
| LOAD.H | RL | 04 | Loads a 16-bit halfword from address [X + Y] into the specified register [Z]. The leftover bit in the opcode encodes the high or low halfword. | 04[7 = W] XZ YY YY | X = register index<br> Y = 16-bit signed offset<br> Z = register index,<br> W = 1 if high halfword, 0 if low halfword |
| LOAD.BH | RL | 05 | Loads an 8-bit byte from address [X + Y] into the specified register [Z]. The leftover bit in the opcode encodes the high or low byte of the high halfword. | 05[7 = W] XZ YY YY | X = register index, Y = 16-bit signed offset, Z = register index, W = 1 if high byte, 0 if low byte. |
| LOAD.BL | RL | 06 | Loads an 8-bit byte from address [X + Y] into the specified register [Z]. The leftover bit in the opcode encodes the high or low byte of the low halfword. | 06[7 = W] XZ YY YY | X = register index<br> Y = 16-bit signed offset<br> Z = register index<br> W = 1 if high byte, 0 if low byte. |
| XFER.W | XM | 07 | Transfers the word in register X to register Y. Allows for bytemasking the result. | 07 XY 0M 00 | X = the source register index<br> Y = the destination register index<br> M = 4-bit byte mask for the transfer. For example, setting M to 0010 will only actually transfer the second byte. |
| XFER.W | XR | 08 | Transfers the word in register X to register Y, using register Z as a bitmask for the result. | 08 XY 0Z 00 | X = source register index<br> Y = destination register index<br> Z = bitmask register index |
| XFER.H | XM | 09 | Transfers the halfword in register X to register Y. Allows for specifying the high or low halfword and a bitmask. | 09[7 = W] XY BB BB | X = source register index<br> Y = destination register index<br> B = bitmask for the transfer operation<br> W = 1 if the high halfword, 0 if the low halfword. |
| XFER.H | XR | 0a | Transfers the halfword in register X to register Y, using register Z as a bitmask for the result. The source, dest and bitmask all have high or low halfword selectors. | 0a XY 0Z 00[0 = A, 1 = B, 2 = C] | X = source register index<br> Y = dest register index<br> Z = mask register index<br> A = X high or low<br> B = Y high or low<br> C = Z high or low |
| XFER.B | XM | 0b | Transfers the byte in register X to register Y, using an immediate bitmask value. X and Y both have byte selectors. | 0b XY MM AB | X = source register index<br> Y = dest register index<br> M = 8-bit mask<br> A = byte selector for X<br> B = byte selector for Y |
| XFER.B | XR | 0c | Transfers the byte in register X to register Y, using register Z as a bitmask for the result. Each register gets a byte selector. | 0c XY ZA BC | X = source register index<br> Y = dest register index<br> Z = bitmask register index<br> A = X byte selector<br> B = Y byte selector<br> C = Z byte selector

Still need opcodes for:
- Math
- Bit operations
- Compare
- Jumping + subroutines
- Stack operations
- Interrupt handling
- Floating point operations?
