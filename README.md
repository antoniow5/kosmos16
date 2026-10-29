# KOSMOS-16

## Terminology

- `opcode` — operation code.
- `rd` — destination register.
- `rs`, `rs1`, `rs2` — source registers.
- `imm` — immediate value encoded directly in the instruction.
- `functN` — additional operation code, where `N` indicates its bit width.
- `PC` — program counter.

## Opcode Map

| Opcode | Instruction Format | Description |
|--------|--------------------|-------------|
| `000` | R-type | Register-register operations |
| `001` | I-type | Register-immediate operations |
| `010` | S-type | Store register value to memory |
| `011` | L-type | Load from memory / JALR |
| `100` | B-type | Conditional branches |
| `101` | UJ-type | Unconditional jump with link |
| `110` | U-type | Upper immediate operations |
| `111` | SYS-type | System calls and system instructions |

## Registers

KOSMOS-16 has eight general-purpose 16-bit registers.

| Register | Purpose | Binary Address |
|----------|---------|----------------|
| `x0` | Always zero. Writes are ignored. | `000` |
| `x1` | Return address (`ra`) | `001` |
| `x2` | Stack pointer (`sp`) | `010` |
| `x3` | Global pointer (`gp`) | `011` |
| `x4`–`x7` | General-purpose registers | `100`–`111` |

---

## R-type

Register-register operations. Two source registers are used, and the result is stored in a destination register.

### Instruction Format

| `rd` | `rs1` | `rs2` | `funct4` | `opcode` |
|------|-------|-------|---------|----------|
| 3 bits | 3 bits | 3 bits | 4 bits | 3 bits |

### Instructions

| Instruction | `funct4` | Operation |
|-------------|----------|-----------|
| `ADD` | `0000` | `rd = rs1 + rs2` |
| `SUB` | `0001` | `rd = rs1 - rs2` |
| `SLL` | `0010` | `rd = rs1 << rs2` |
| `SRL` | `0011` | `rd = rs1 >> rs2` (logical) |
| `OR` | `0100` | `rd = rs1 OR rs2` |
| `XOR` | `0101` | `rd = rs1 XOR rs2` |
| `AND` | `0110` | `rd = rs1 AND rs2` |
| `SLT` | `0111` | `rd = 1` if `rs1 < rs2` (signed), otherwise `0` |
| `SLTU` | `1000` | `rd = 1` if `rs1 < rs2` (unsigned), otherwise `0` |

---

## I-type

Register-immediate operations. The destination and first source register share the same register field. The result is written back to that register.

### Instruction Format

| `rd/rs1` | `imm` | `funct3` | `opcode` |
|----------|-------|----------|----------|
| 3 bits | 7 bits | 3 bits | 3 bits |

### Instructions

| Instruction | `funct3` | Operation |
|-------------|----------|-----------|
| `ADDI` | `000` | Add signed immediate |
| `SLLI` | `001` | Logical left shift |
| `SRLI` | `010` | Logical right shift |
| `SRAI` | `011` | Arithmetic right shift |
| `ORI` | `100` | Bitwise OR with immediate |
| `XORI` | `101` | Bitwise XOR with immediate |
| `ANDI` | `110` | Bitwise AND with immediate |
| Reserved | `111` | Available for future instructions |

The 7-bit immediate is sign-extended for arithmetic and logical operations, except where the instruction defines a different interpretation.

---

## S-type

Store instructions. A value from `rs2` is written to memory at the address calculated by adding `rs1` and the signed immediate.

### Instruction Format

| `imm[5:3]` | `rs1` | `rs2` | `imm[2:0]` | `funct1` | `opcode` |
|------------|-------|-------|------------|----------|----------|
| 3 bits | 3 bits | 3 bits | 3 bits | 1 bit | 3 bits |

### Address Calculation

`memory_address = rs1 + sign_extend(imm)`

### Instructions

| Instruction | `funct1` | Operation |
|-------------|----------|-----------|
| `SB` | `0` | Store the lowest 8 bits of `rs2` to memory |
| `SH` | `1` | Store all 16 bits of `rs2` to memory |

---

## L-type

Load instructions and JALR. Load instructions read data from memory and store it in a destination register.

### Instruction Format

| `rd` | `rs1` | `imm` | `funct2` | `opcode` |
|------|-------|-------|----------|----------|
| 3 bits | 3 bits | 5 bits | 2 bits | 3 bits |

### Instructions

| Instruction | `funct2` | Operation |
|-------------|----------|-----------|
| `LB` | `00` | Load 8 bits from memory at `rs1 + sign_extend(imm)` and sign-extend to 16 bits |
| `LH` | `01` | Load 16 bits from memory at `rs1 + sign_extend(imm)` |
| `LBU` | `10` | Load 8 bits from memory at `rs1 + sign_extend(imm)` and zero-extend to 16 bits |
| `JALR` | `11` | Jump to `rs1 + sign_extend(imm)` and store `PC + 2` in `rd` |

---

## B-type

Conditional branch instructions. The CPU compares two registers and performs a relative jump if the specified condition is satisfied.

### Instruction Format

| `imm[4:2]` | `rs1` | `rs2` | `imm[1:0]` | `funct2` | `opcode` |
|------------|-------|-------|------------|----------|----------|
| 3 bits | 3 bits | 3 bits | 2 bits | 2 bits | 3 bits |

### Address Calculation

`target_address = PC + sign_extend(imm)`

### Instructions

| Instruction | `funct2` | Operation |
|-------------|----------|-----------|
| `BEQ` | `00` | Branch if `rs1` equals `rs2` |
| `BNE` | `01` | Branch if `rs1` does not equal `rs2` |
| `BGE` | `10` | Branch if `rs1` is greater than or equal to `rs2` (signed) |
| `BLTU` | `11` | Branch if `rs1` is less than `rs2` (unsigned) |

---

## UJ-type

Unconditional jump with link. The return address is stored in `rd`, and execution continues at a PC-relative target address.

### Instruction Format

| `rd` | `immediate` | `opcode` |
|------|-------------|----------|
| 3 bits | 10 bits | 3 bits |

### Operation

1. Store `PC + 2` in `rd`.
2. Calculate the target address using the signed immediate.
3. Set `PC` to the target address.

`rd = PC + 2`

`PC = PC + sign_extend(imm)`

---

## U-type

Upper immediate instructions. These instructions load an immediate value into the upper portion of a register or construct a PC-relative address.

### Instruction Format

| `rd` | `imm` | `opcode` |
|------|-------|----------|
| 3 bits | 10 bits | 3 bits |

### Instructions

| Instruction | Operation |
|-------------|-----------|
| `LUI` | Load upper immediate into `rd` |
| `AUIPC` | Add upper immediate to the current PC and store the result in `rd` |

### LUI — Load Upper Immediate

Loads a 10-bit immediate into the upper bits of the destination register, filling the lower six bits with zeros.

`rd = imm << 6`

### AUIPC — Add Upper Immediate to PC

Adds the upper immediate to the current PC and stores the result in the destination register.

`rd = PC + (imm << 6)`

---

## SYS-type

Reserved for system calls and privileged system instructions.

Potential instructions:

| Instruction | Description |
|-------------|-------------|
| `ECALL` | Request a system call or transfer control to the operating system |
| `EBREAK` | Trigger a breakpoint exception |
| `MRET` | Return from a machine-mode trap |
| `IRET` | Optional dedicated interrupt return instruction |

The exact encoding and system-call interface will be defined separately.
