# 🧠 Proving the 'M' in [RV32IM](https://github.com/bytesculptor097/RV32IM) 

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
Suppose the example:-
```assembly
mul x3 x1 x2
```
We first need to convert this to hex or binary number to represent this in the instruction memeory of the core

`mul x3 x1 x2` → `022081B3`

So the istruction memory will look like this:-

```verilog
module imem (
    input wire clk,
    input wire we,
    input wire [31:0] addr,
    input wire [31:0] din,
    output reg [31:0] dout
);
    reg [31:0] mem [0:1023]; // 4KB RAM

    always @(posedge clk) begin
        if (we)
            mem[addr[11:2]] <= din;
        dout <= mem[addr[11:2]];
    end

    initial begin
     mem[0] = 32'h0220E1B3;  // <--------- mul x3 x1 x2
    end

endmodule
```
**Next**:-

Now as the command takes input from x1 and x2 and then multiplies them, we need to initialize the value to of x1 and x2 in the register file:-

```verilog






module regfile (
    input wire clk,
    input wire reg_write,
    input wire [4:0] rs1,
    input wire [4:0] rs2,
    input wire [4:0] rd,
    input wire [31:0] wd,
    output wire [31:0] rs1_val,
    output wire [31:0] rs2_val,
    output wire [31:0] x3_debug,
    output wire [31:0] x5_debug
);

    reg [31:0] regs [0:31];


    // Read logic
    assign rs1_val = regs[rs1];
    assign rs2_val = regs[rs2];

    // Write logic
    always @(posedge clk) begin
        if (reg_write && rd != 5'd0) begin
            regs[rd] <= wd;
            $display("WRITE: x%0d <= %h at time %0t", rd, wd, $time);
        end
    end

    // Debug outputs
    assign x3_debug = regs[3];
    assign x5_debug = regs[5];

 initial begin
    regs[1] = 32'd15; // <----- Setting the value of x1 as 15    
    regs[2] = 32'd4;  // <----- Setting the value of x2 ad 4
 end



endmodule
```
Now after compiling the required files with iverilog (that are given in my RV32IM repository), you should get the following result

```
x3 = c8 <---- decimal 60
``` 




