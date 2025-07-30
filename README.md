# 🧠 Proving the 'M' in RV32IM 

## 📌 Overview

This README demonstrates how the core supports **RV32IM**, not just RV32I.

---

## ✅ What Is RV32IM?

| Instruction Set | Description |
|-----------------|-------------|
| RV32I           | Base 32-bit integer instructions (`add`, `sub`, `lw`, `sw`, `beq`, etc.) |
| RV32IM          | Includes RV32I **plus** hardware support for `MUL`, `DIV`, `REM`, and related instructions |

---

## 🔍 Proof of M-extension Support

### 1. ✅ Instruction Decoding

- The core decodes `funct7 = 0000001` and relevant `funct3` values for M-extension.
- Examples:
  - `MUL x3, x1, x2` → `0x022081B3`
  - `DIV x3, x1, x2` → `0x0220C1B3`
  - `REM x3, x1, x2` → `0x0220E1B3`

### 2. ✅ ALU Control Logic

- ALUControl signals extended to handle:
  - `MUL` → `ALUControl = 1010`
  - `DIV` → `ALUControl = 1011`
  - `REM` → `ALUControl = 1101`
  - Others: `MULH`, `MULHU`, `MULHSU`, `DIVU`, `REMU`

### 3. ✅ Functional Execution

- Sample test program:
  ```assembly
  li x1, 15
  li x2, 4
  mul x3, x1, x2     # x3 = 60
  div x3, x1, x2     # x3 = 3
  rem x3, x1, x2     # x3 = 3
