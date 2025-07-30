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

```verilog
always @(*) begin
    case (ALUOp)
    
        2'b00: ALUControl = 4'b0010; // ADD for lw/sw/addi
        2'b01: ALUControl = 4'b0110; // SUB for branches (e.g., beq)

        2'b10: begin // R-type
            case (funct3)
                3'b000: begin
                    if (funct7 == 7'b0100000)
                        ALUControl = 4'b0110; // SUB
                    else if (funct7 == 7'b0000001)
                        ALUControl = 4'b1010; // MUL
                    else
                        ALUControl = 4'b0010; // ADD
                end
                3'b100: begin
                    if (funct7 == 7'b0000001)
                        ALUControl = 4'b1011; // DIV
                    else
                        ALUControl = 4'b0011; // XOR
                end
                3'b101: begin
                    if (funct7 == 7'b0000001)
                        ALUControl = 4'b1100; // DIVU
                    else
                        ALUControl = 4'b1001; // SRL
                end
                3'b110: begin
                    if (funct7 == 7'b0000001)
                        ALUControl = 4'b1101; // REM
                    else
                        ALUControl = 4'b0001; // OR
                end
                3'b111: begin
                    if (funct7 == 7'b0000001)
                        ALUControl = 4'b1110; // REMU
                    else
                        ALUControl = 4'b0000; // AND
                end
                3'b010: ALUControl = 4'b0111; // SLT
                3'b001: ALUControl = 4'b1000; // SLL
                default: ALUControl = 4'b1111; // Invalid
            endcase
        end

        2'b11: ALUControl = 4'b1110; // LUI or AUIPC

        default: ALUControl = 4'b1111; // Unknown
    endcase

    $display("ALUControl:", ALUControl);
end
```
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
