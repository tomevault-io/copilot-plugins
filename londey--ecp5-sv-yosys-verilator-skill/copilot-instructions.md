## ecp5-sv-yosys-verilator-skill

> >


# ECP5 SystemVerilog: Yosys + Verilator Compatibility

## Core Principle: Always Instantiate Hard Resources Explicitly

For DP16KD (56 on ECP5-25K) and MULT18X18D (28 on ECP5-25K), **do not rely on Yosys inference**. Inference is incomplete and unreliable: ALU54B features are not inferred at all, MULT9X9D (two 9x9 mults per DSP tile) is not supported by Yosys/nextpnr, and BRAM inference fails for wider-than-18-bit or multi-port patterns. Explicit instantiation gives you exact resource consumption, correct pipeline register placement, and predictable timing closure.

---

## ECP5-25K Hard IP Budget (CABGA256/CABGA381)

| Resource         | Count | Notes                                                     |
|-----------------|-------|-----------------------------------------------------------|
| DP16KD          | 56    | 18Kbit true dual-port EBR; also configurable as PDPW16KD  |
| MULT18X18D      | 28    | 18x18 signed multiplier; shares tile with ALU54B          |
| ALU54B          | 14    | 54-bit accumulator; paired 2:1 with MULT18X18D            |
| EHXPLLL         | 2     | Integer-N PLL (2 on -25K, 4 on larger variants)           |
| DCCA            | 56    | Clock distribution / CE gates                             |
| CLKDIVF         | 4     | Integer clock dividers                                    |
| TRELLIS_ECLKBUF | 8     | Edge-clock buffers for DDR clocking                       |
| ECLKSYNCB       | 10    | Edge-clock synchronizer/gate                              |
| OSCG            | 1     | Internal ring oscillator (~155 MHz / DIV)                 |
| JTAGG           | 1     | User access to JTAG port                                  |
| USRMCLK         | 1     | User access to SPI flash MCLK                             |
| GSR             | 1     | Global Set/Reset network driver                           |
| IOLOGIC         | 128   | High-speed I/O registers (DDR, gearbox)                   |
| SIOLOGIC        | 69    | Single-data-rate I/O registers                            |

---

## The Golden Pattern: Macro-Guarded Instantiation

```systemverilog
// Build system: -D SYNTHESIS for Yosys; omit for Verilator
`ifdef SYNTHESIS
  EHXPLLL #(...) pll_inst (...);
`else
  pll_sim_model pll_inst (...);
`endif
```

---

## Block RAM: DP16KD and PDPW16KD

### Two EBR Configurations

The same physical 18Kbit EBR block is configured as one of two named primitives:

**`DP16KD`** — True Dual-Port (TDP). Both ports read/write independently, each with its own clock. Port width up to 18 bits (16 data + 2 parity). Yosys internal name `$__ECP5_DP16KD`.

**`PDPW16KD`** — Pseudo Dual-Port Wide (SDP mode). Port A is write-only at **36 bits** (32 data + 4 parity). Port B is read-only, configurable at 9, 18, or 36 bits — asymmetric reads allowed. This is the only way to achieve 32-bit data width from a single EBR. Yosys internal name `$__ECP5_PDPW16KD`.

These are mutually exclusive configurations of the same hardware — you cannot mix them.

### DP16KD Aspect Ratios

| DATA_WIDTH | Addr bits active | Depth    | Data | Parity |
|-----------|-----------------|----------|------|--------|
| 1         | ADA13:ADA0      | 16384x1  | 1    | 0      |
| 2         | ADA13:ADA1      | 8192x2   | 2    | 0      |
| 4         | ADA13:ADA2      | 4096x4   | 4    | 0      |
| 9         | ADA13:ADA3      | 2048x8   | 8    | 1      |
| 18        | ADA13:ADA4      | 1024x16  | 16   | 2      |
| 36        | ADA13:ADA5      | 512x32   | 32   | 4      |

Port A and Port B can have **different widths** (asymmetric). The lower address pins not used by the selected width must be tied to `1'b0`.

### DP16KD Instantiation (1K x 16-bit TDP)

```systemverilog
`ifdef SYNTHESIS
DP16KD #(
  .DATA_WIDTH_A(18),      // 16 data + 2 parity
  .DATA_WIDTH_B(18),
  .REGMODE_A("NOREG"),    // "OUTREG" adds 1 cycle latency, improves Fmax
  .REGMODE_B("NOREG"),
  .RESETMODE("SYNC"),
  .ASYNC_RESET_RELEASE("SYNC"),
  .WRITEMODE_A("NORMAL"), // "NORMAL"=read-before-write, "WRITETHROUGH"=pass-through
  .WRITEMODE_B("NORMAL"),
  .GSR("ENABLED"),
  .INIT_DATA("STATIC"),
  .CSDECODE_A("0b000"),   // chip select decode; 0b000 = always selected
  .CSDECODE_B("0b000")
) bram_i (
  // Port A
  .CLKA(clk_a), .CEA(1'b1), .OCEA(1'b1), .RSTA(rst),
  .WEA(we_a), .CSA0(1'b0), .CSA1(1'b0), .CSA2(1'b0),
  // For DATA_WIDTH=18: active addr is ADA13:ADA4; tie ADA3:ADA0 to 0
  .ADA13(addr_a[9]),.ADA12(addr_a[8]),.ADA11(addr_a[7]),.ADA10(addr_a[6]),
  .ADA9(addr_a[5]),.ADA8(addr_a[4]),.ADA7(addr_a[3]),.ADA6(addr_a[2]),
  .ADA5(addr_a[1]),.ADA4(addr_a[0]),
  .ADA3(1'b0),.ADA2(1'b0),.ADA1(1'b0),.ADA0(1'b0),
  // DIA17:DIA16 = parity (tie to 0 if unused)
  .DIA17(1'b0),.DIA16(1'b0),
  .DIA15(din_a[15]),.DIA14(din_a[14]),.DIA13(din_a[13]),.DIA12(din_a[12]),
  .DIA11(din_a[11]),.DIA10(din_a[10]),.DIA9(din_a[9]),.DIA8(din_a[8]),
  .DIA7(din_a[7]),.DIA6(din_a[6]),.DIA5(din_a[5]),.DIA4(din_a[4]),
  .DIA3(din_a[3]),.DIA2(din_a[2]),.DIA1(din_a[1]),.DIA0(din_a[0]),
  .DOA17(),.DOA16(),
  .DOA15(dout_a[15]),.DOA14(dout_a[14]),.DOA13(dout_a[13]),.DOA12(dout_a[12]),
  .DOA11(dout_a[11]),.DOA10(dout_a[10]),.DOA9(dout_a[9]),.DOA8(dout_a[8]),
  .DOA7(dout_a[7]),.DOA6(dout_a[6]),.DOA5(dout_a[5]),.DOA4(dout_a[4]),
  .DOA3(dout_a[3]),.DOA2(dout_a[2]),.DOA1(dout_a[1]),.DOA0(dout_a[0]),
  // Port B: identical structure; see references/ecp5_bram_guide.md for full listing
  .CLKB(clk_b),.CEB(1'b1),.OCEB(1'b1),.RSTB(rst),.WEB(we_b),
  .CSB0(1'b0),.CSB1(1'b0),.CSB2(1'b0)
  // ... addr/data ports B same pattern
);
`else
// Behavioral TDP
logic [15:0] mem_r [0:1023];
always_ff @(posedge clk_a) if (we_a) mem_r[addr_a] <= din_a;
always_ff @(posedge clk_a) dout_a <= mem_r[addr_a];
always_ff @(posedge clk_b) if (we_b) mem_r[addr_b] <= din_b;
always_ff @(posedge clk_b) dout_b <= mem_r[addr_b];
`endif
```

### PDPW16KD — 36-bit Read (512×32 Symmetric)

Write and read both use 9-bit addresses (512 entries).
All 36 DO outputs are connected; parity bits (DO35:DO32) left unconnected if unused.

```systemverilog
`ifdef SYNTHESIS
PDPW16KD #(
  .DATA_WIDTH_W(36),      // Write port: always 36 in Yosys flow
  .DATA_WIDTH_R(36),      // Read port: 36-bit (symmetric with write)
  .REGMODE("NOREG"),
  .RESETMODE("SYNC"),
  .GSR("ENABLED"),
  .INIT_DATA("STATIC"),
  .CSDECODE_W("0b000"),
  .CSDECODE_R("0b000")
) pdpw_i (
  // Write port: 9-bit address (512 entries at 36-bit width)
  // CEW gates writes (no separate WEW in Yosys); BE0-3 enable byte lanes
  .CLKW(clk_w), .CEW(we),
  .CSW0(1'b0),.CSW1(1'b0),.CSW2(1'b0),
  .BE3(1'b1),.BE2(1'b1),.BE1(1'b1),.BE0(1'b1),
  .ADW8(waddr[8]),.ADW7(waddr[7]),.ADW6(waddr[6]),.ADW5(waddr[5]),
  .ADW4(waddr[4]),.ADW3(waddr[3]),.ADW2(waddr[2]),.ADW1(waddr[1]),.ADW0(waddr[0]),
  // 36-bit write data: DI35:DI32=parity, DI31:DI0=data
  .DI35(1'b0),.DI34(1'b0),.DI33(1'b0),.DI32(1'b0),
  .DI31(din[31]),.DI30(din[30]),.DI29(din[29]),.DI28(din[28]),
  .DI27(din[27]),.DI26(din[26]),.DI25(din[25]),.DI24(din[24]),
  .DI23(din[23]),.DI22(din[22]),.DI21(din[21]),.DI20(din[20]),
  .DI19(din[19]),.DI18(din[18]),.DI17(din[17]),.DI16(din[16]),
  .DI15(din[15]),.DI14(din[14]),.DI13(din[13]),.DI12(din[12]),
  .DI11(din[11]),.DI10(din[10]),.DI9(din[9]),.DI8(din[8]),
  .DI7(din[7]),.DI6(din[6]),.DI5(din[5]),.DI4(din[4]),
  .DI3(din[3]),.DI2(din[2]),.DI1(din[1]),.DI0(din[0]),
  // Read port: 9-bit addr for DATA_WIDTH_R=36 (512 entries, symmetric)
  .CLKR(clk_r),.CER(1'b1),.OCER(1'b1),.RST(rst),
  .CSR0(1'b0),.CSR1(1'b0),.CSR2(1'b0),
  .ADR13(raddr[8]),.ADR12(raddr[7]),.ADR11(raddr[6]),.ADR10(raddr[5]),
  .ADR9(raddr[4]),.ADR8(raddr[3]),.ADR7(raddr[2]),.ADR6(raddr[1]),
  .ADR5(raddr[0]),
  .ADR4(1'b0),.ADR3(1'b0),.ADR2(1'b0),.ADR1(1'b0),.ADR0(1'b0),
  .DO35(),.DO34(),.DO33(),.DO32(),  // parity — leave unconnected
  .DO31(dout[31]),.DO30(dout[30]),.DO29(dout[29]),.DO28(dout[28]),
  .DO27(dout[27]),.DO26(dout[26]),.DO25(dout[25]),.DO24(dout[24]),
  .DO23(dout[23]),.DO22(dout[22]),.DO21(dout[21]),.DO20(dout[20]),
  .DO19(dout[19]),.DO18(dout[18]),.DO17(dout[17]),.DO16(dout[16]),
  .DO15(dout[15]),.DO14(dout[14]),.DO13(dout[13]),.DO12(dout[12]),
  .DO11(dout[11]),.DO10(dout[10]),.DO9(dout[9]),.DO8(dout[8]),
  .DO7(dout[7]),.DO6(dout[6]),.DO5(dout[5]),.DO4(dout[4]),
  .DO3(dout[3]),.DO2(dout[2]),.DO1(dout[1]),.DO0(dout[0])
);
`else
logic [31:0] mem_r [0:511];
always_ff @(posedge clk_w) if (we) mem_r[waddr] <= din;
always_ff @(posedge clk_r) dout <= mem_r[raddr];
`endif
```

### PDPW16KD — 18-bit Read (1K×16 Asymmetric)

Write uses 9-bit address (512×36), read uses 10-bit address (1024×18).
Each 36-bit write word maps to two 18-bit read words (16 data + 2 parity each).
DO35:DO18 are unused in 18-bit mode.

```systemverilog
`ifdef SYNTHESIS
PDPW16KD #(
  .DATA_WIDTH_W(36),      // Write port: always 36 in Yosys flow
  .DATA_WIDTH_R(18),      // Read port: 18-bit (asymmetric)
  .REGMODE("NOREG"),
  .RESETMODE("SYNC"),
  .GSR("ENABLED"),
  .INIT_DATA("STATIC"),
  .CSDECODE_W("0b000"),
  .CSDECODE_R("0b000")
) pdpw_i (
  // Write port: 9-bit address (512 entries at 36-bit width)
  // CEW gates writes (no separate WEW in Yosys); BE0-3 enable byte lanes
  .CLKW(clk_w), .CEW(we),
  .CSW0(1'b0),.CSW1(1'b0),.CSW2(1'b0),
  .BE3(1'b1),.BE2(1'b1),.BE1(1'b1),.BE0(1'b1),
  .ADW8(waddr[8]),.ADW7(waddr[7]),.ADW6(waddr[6]),.ADW5(waddr[5]),
  .ADW4(waddr[4]),.ADW3(waddr[3]),.ADW2(waddr[2]),.ADW1(waddr[1]),.ADW0(waddr[0]),
  // 36-bit write data: DI35:DI32=parity, DI31:DI0=data
  .DI35(1'b0),.DI34(1'b0),.DI33(1'b0),.DI32(1'b0),
  .DI31(din[31]),.DI30(din[30]),.DI29(din[29]),.DI28(din[28]),
  .DI27(din[27]),.DI26(din[26]),.DI25(din[25]),.DI24(din[24]),
  .DI23(din[23]),.DI22(din[22]),.DI21(din[21]),.DI20(din[20]),
  .DI19(din[19]),.DI18(din[18]),.DI17(din[17]),.DI16(din[16]),
  .DI15(din[15]),.DI14(din[14]),.DI13(din[13]),.DI12(din[12]),
  .DI11(din[11]),.DI10(din[10]),.DI9(din[9]),.DI8(din[8]),
  .DI7(din[7]),.DI6(din[6]),.DI5(din[5]),.DI4(din[4]),
  .DI3(din[3]),.DI2(din[2]),.DI1(din[1]),.DI0(din[0]),
  // Read port: 10-bit addr for DATA_WIDTH_R=18 (1K entries)
  .CLKR(clk_r),.CER(1'b1),.OCER(1'b1),.RST(rst),
  .CSR0(1'b0),.CSR1(1'b0),.CSR2(1'b0),
  .ADR13(raddr[9]),.ADR12(raddr[8]),.ADR11(raddr[7]),.ADR10(raddr[6]),
  .ADR9(raddr[5]),.ADR8(raddr[4]),.ADR7(raddr[3]),.ADR6(raddr[2]),
  .ADR5(raddr[1]),.ADR4(raddr[0]),
  .ADR3(1'b0),.ADR2(1'b0),.ADR1(1'b0),.ADR0(1'b0),
  .DO17(),.DO16(),  // parity — leave unconnected
  .DO15(dout[15]),.DO14(dout[14]),.DO13(dout[13]),.DO12(dout[12]),
  .DO11(dout[11]),.DO10(dout[10]),.DO9(dout[9]),.DO8(dout[8]),
  .DO7(dout[7]),.DO6(dout[6]),.DO5(dout[5]),.DO4(dout[4]),
  .DO3(dout[3]),.DO2(dout[2]),.DO1(dout[1]),.DO0(dout[0])
);
`else
logic [31:0] mem_r [0:511];
always_ff @(posedge clk_w) if (we) mem_r[waddr] <= din;
always_ff @(posedge clk_r) dout <= raddr[0] ? mem_r[raddr[9:1]][31:16]
                                             : mem_r[raddr[9:1]][15:0];
`endif
```

---

## DSP: MULT18X18D and ALU54B

### Architecture

Each physical DSP **tile** contains one `MULT18X18D` and one `ALU54B`. The ECP5-25K has 28 MULT18X18Ds and 14 ALU54Bs (one ALU per two multipliers).

**Critical limitation**: `MULT9X9D` (Diamond's primitive for two independent 9x9 multiplications per tile) is **not supported by Yosys or nextpnr**. To use a DSP tile for two 9-bit multiplications, instantiate a single `MULT18X18D` with the two pairs of 9-bit operands packed into the 18-bit inputs, using the MSB as a zero sign extension:

```systemverilog
// Pack two 9x9 unsigned multiplications into one MULT18X18D
// A = {9'b0, a_hi} or use MSB as sign — treat output P[35:18] and P[17:0] separately
// This is manual packing; results overlap if inputs have MSB set
```

For fully independent 9x9 multiplies, use two `MULT18X18D` instances.

### MULT18X18D — Signed 18x18 Multiplier

Four pipeline register stages, each selectable on one of four DSP column clocks (CLK0–CLK3):

```systemverilog
`ifdef SYNTHESIS
MULT18X18D #(
  // Each REG_*_CLK: "CLK0".."CLK3" or "NONE" (combinational)
  .REG_INPUTA_CLK  ("CLK0"), .REG_INPUTA_CE("CE0"),  .REG_INPUTA_RST("RST0"),
  .REG_INPUTB_CLK  ("CLK0"), .REG_INPUTB_CE("CE0"),  .REG_INPUTB_RST("RST0"),
  .REG_INPUTC_CLK  ("NONE"), // C addend to ALU54B; "NONE" if ALU not used
  .REG_PIPELINE_CLK("CLK0"), .REG_PIPELINE_CE("CE0"),.REG_PIPELINE_RST("RST0"),
  .REG_OUTPUT_CLK  ("CLK0"), .REG_OUTPUT_CE("CE0"),  .REG_OUTPUT_RST("RST0"),
  .CLK0_DIV("ENABLED"), .CLK1_DIV("ENABLED"),
  .CLK2_DIV("ENABLED"), .CLK3_DIV("ENABLED"),
  .HIGHSPEED_CLK("NONE"),
  .GSR("ENABLED"),
  .SOURCEB_MODE("B_SHIFT"),  // "B_SHIFT"=normal; "ROUND"=rounding mode
  .MULT_BYPASS("DISABLED"),  // "ENABLED"=combinational bypass (all regs ignored)
  .RESETMODE("SYNC"),
  .CAS_MATCH_REG("FALSE")
) mult_i (
  .CLK0(clk),.CLK1(1'b0),.CLK2(1'b0),.CLK3(1'b0),
  .CE0(1'b1),.CE1(1'b0),.CE2(1'b0),.CE3(1'b0),
  .RST0(rst),.RST1(1'b0),.RST2(1'b0),.RST3(1'b0),
  // 18-bit signed inputs; bit 17 is the sign bit
  .A17(a[17]),.A16(a[16]),.A15(a[15]),.A14(a[14]),
  .A13(a[13]),.A12(a[12]),.A11(a[11]),.A10(a[10]),
  .A9(a[9]),  .A8(a[8]),  .A7(a[7]),  .A6(a[6]),
  .A5(a[5]),  .A4(a[4]),  .A3(a[3]),  .A2(a[2]),  .A1(a[1]),  .A0(a[0]),
  .B17(b[17]),.B16(b[16]),.B15(b[15]),.B14(b[14]),
  .B13(b[13]),.B12(b[12]),.B11(b[11]),.B10(b[10]),
  .B9(b[9]),  .B8(b[8]),  .B7(b[7]),  .B6(b[6]),
  .B5(b[5]),  .B4(b[4]),  .B3(b[3]),  .B2(b[2]),  .B1(b[1]),  .B0(b[0]),
  // C: addend forwarded to ALU54B; tie to 0 if ALU not used
  .C17(1'b0),.C16(1'b0),.C15(1'b0),.C14(1'b0),
  .C13(1'b0),.C12(1'b0),.C11(1'b0),.C10(1'b0),
  .C9(1'b0),.C8(1'b0),.C7(1'b0),.C6(1'b0),
  .C5(1'b0),.C4(1'b0),.C3(1'b0),.C2(1'b0),.C1(1'b0),.C0(1'b0),
  // 36-bit signed product
  .P35(p[35]),.P34(p[34]),.P33(p[33]),.P32(p[32]),
  .P31(p[31]),.P30(p[30]),.P29(p[29]),.P28(p[28]),
  .P27(p[27]),.P26(p[26]),.P25(p[25]),.P24(p[24]),
  .P23(p[23]),.P22(p[22]),.P21(p[21]),.P20(p[20]),
  .P19(p[19]),.P18(p[18]),.P17(p[17]),.P16(p[16]),
  .P15(p[15]),.P14(p[14]),.P13(p[13]),.P12(p[12]),
  .P11(p[11]),.P10(p[10]),.P9(p[9]),  .P8(p[8]),
  .P7(p[7]),  .P6(p[6]),  .P5(p[5]),  .P4(p[4]),
  .P3(p[3]),  .P2(p[2]),  .P1(p[1]),  .P0(p[0]),
  // Cascade to ALU54B — leave unconnected if not using ALU
  .SIGNEDA(),.SIGNEDB(),.SOUTSIGNED(),
  .SOUT35(),.SOUT34(),.SOUT33(),.SOUT32(),
  .SOUT31(),.SOUT30(),.SOUT29(),.SOUT28(),.SOUT27(),.SOUT26(),
  .SOUT25(),.SOUT24(),.SOUT23(),.SOUT22(),.SOUT21(),.SOUT20(),
  .SOUT19(),.SOUT18(),.SOUT17(),.SOUT16(),.SOUT15(),.SOUT14(),
  .SOUT13(),.SOUT12(),.SOUT11(),.SOUT10(),.SOUT9(),.SOUT8(),
  .SOUT7(),.SOUT6(),.SOUT5(),.SOUT4(),.SOUT3(),.SOUT2(),.SOUT1(),.SOUT0()
);
`else
// Behavioral: 4-stage pipeline matching synthesis register configuration
logic signed [17:0] a_r, b_r;
logic signed [35:0] pipe_r, p_r;
always_ff @(posedge clk or posedge rst) begin
  if (rst) begin
    a_r <= '0; b_r <= '0; pipe_r <= '0; p_r <= '0;
  end else begin
    a_r    <= $signed(a);
    b_r    <= $signed(b);
    pipe_r <= a_r * b_r;
    p_r    <= pipe_r;
  end
end
assign p = p_r;
`endif
```

**Latency with all 4 stages**: 4 clock cycles (INPUT_A + INPUT_B in parallel = 1, PIPELINE = 1, OUTPUT = 1, plus the implicit multiply = 1). Adjust the behavioral model stage count to match.

**Unsigned inputs**: The hardware is signed. For unsigned 16-bit inputs: `{2'b0, a_unsigned[15:0]}` (zero-extend to 18 bits with sign bit = 0).

### ALU54B — 54-bit Accumulator

Pairs with `MULT18X18D` via the SOUT cascade bus. Supports accumulation, addition, and subtraction of the multiplier output. nextpnr support is limited — it must share a DSP tile with its MULT18X18D; use LPF placement constraints if P&R fails. See `references/dsp_guide.md` for full port listing and a MAC example.

---

## Clock and Oscillator Primitives

### OSCG — Internal Ring Oscillator (~155 MHz / DIV)

One instance per device. Use for non-timing-critical functions (debug counters, watchdogs):

```systemverilog
`ifdef SYNTHESIS
OSCG #(.DIV(4)) osc_i (.OSC(osc_clk)); // ~38 MHz; actual freq ±15%
`else
reg osc_r = 0;
always #13 osc_r = ~osc_r;             // 13 ns half-period ≈ 38 MHz
assign osc_clk = osc_r;
`endif
```

Valid DIV: 2, 4, 8, 16, 32, 64, 128.

### TRELLIS_ECLKBUF — Edge Clock Buffer

Required when routing PLL output to IOLOGIC DDR registers:

```systemverilog
`ifdef SYNTHESIS
TRELLIS_ECLKBUF eclkbuf_i (.A(pll_clkop), .Z(eclk));
`else
assign eclk = pll_clkop;
`endif
```

### ECLKSYNCB — Edge Clock Gate

```systemverilog
`ifdef SYNTHESIS
ECLKSYNCB eclksync_i (.ECLKI(eclk_in), .STOP(1'b0), .ECLKO(eclk_out));
`else
assign eclk_out = eclk_in;
`endif
```

---

## Miscellaneous Hard IP

### GSR — Global Set/Reset

Normally driven by configuration engine. Instantiate only for user-controlled global reset:

```systemverilog
`ifdef SYNTHESIS
GSR gsr_i (.GSR(user_reset_n)); // active-low
`else
// No behavioral effect — drive resets directly in testbench
`endif
```

### JTAGG — User JTAG Access

```systemverilog
`ifdef SYNTHESIS
JTAGG #(.ER1("ENABLED"), .ER2("DISABLED")) jtag_i (
  .JTDO1(tdo), .JTDO2(1'b0),
  .JTDI(tdi), .JTCK(tck),
  .JRTI1(rti), .JSHIFT(shift),
  .JUPDATE(update), .JRSTN(rstn), .JCE1(ce1)
);
`else
// Drive from testbench JTAG BFM
`endif
```

### USRMCLK — SPI Flash Clock

On packages where MCLK is not a standard I/O, access it via USRMCLK:

```systemverilog
`ifdef SYNTHESIS
USRMCLK usrmclk_i (.USRMCLKI(spi_clk), .USRMCLKTS(spi_clk_oe_n));
`else
assign spi_mclk = spi_clk_oe_n ? 1'bz : spi_clk;
`endif
```

---

## Yosys Synthesis Script

```tcl
# synth_ecp5.tcl
yosys read_verilog -sv -D SYNTHESIS \
    src/top.sv src/core.sv src/mem_ctrl.sv

# -nobram: disable BRAM inference (using explicit DP16KD/PDPW16KD)
# -nodsp:  disable DSP inference (using explicit MULT18X18D)
yosys synth_ecp5 -top top -nobram -nodsp -json build/top.json

nextpnr-ecp5 \
    --25k \
    --package CABGA256 \
    --json build/top.json \
    --lpf constraints.lpf \
    --textcfg build/top.config \
    --report build/timing.json

ecppack build/top.config build/top.bit
```

Use `-nobram -nodsp` whenever you are explicitly instantiating all hard resources to prevent Yosys from also inferring instances from any remaining arithmetic operators.

---

## Verilator Build

```makefile
verilator --sv --cc --exe \
    --top-module tb_top -Isrc/ \
    sim/stubs/sim_stubs.sv \
    sim/tb_top.sv src/top.sv sim/main.cpp \
    -Wall --assert --x-assign unique \
    --build -j4
```

---

## Stub Strategy Summary

| Primitive        | Simulation strategy                                       |
|-----------------|-----------------------------------------------------------|
| EHXPLLL         | Dedicated stub (complex ports, lock delay)                |
| DCCA            | `assign out = in`                                         |
| CLKDIVF         | Inline toggle flop                                        |
| TRELLIS_ECLKBUF | `assign Z = A`                                            |
| ECLKSYNCB       | `assign ECLKO = ECLKI`                                    |
| DP16KD          | Behavioral inferred RAM (much simpler than port mapping)  |
| PDPW16KD        | Behavioral inferred RAM                                   |
| TRELLIS_DPR16X4 | Inline array + flop                                       |
| MULT18X18D      | Behavioral signed registered multiply; match stage count  |
| ALU54B          | Behavioral accumulator                                    |
| ODDRX1F/IDDRX1F | Inline dual-edge flop                                     |
| BB/OB/IB        | Inline assign                                             |
| OSCG            | `always #N` clock generator                               |
| JTAGG           | Testbench-driven stub                                     |
| USRMCLK         | `assign mclk = clk` inline                                |
| GSR             | No-op; drive resets directly in testbench                 |

---

## Common Pitfalls

**1. MULT9X9D is not supported by Yosys/nextpnr.** Diamond's `MULT9X9D` (SIMD 9x9 mode) does not exist in the open toolchain. Use `MULT18X18D` with 9-bit inputs zero-extended to 18 bits, or use two `MULT18X18D` instances for independent multiplications.

**2. PDPW16KD write port is always 36 bits in the Yosys flow.** Despite the Lattice datasheet describing variable write width, Yosys only models `DATA_WIDTH_W=36`. Do not attempt a narrower write port on PDPW16KD via Yosys — use DP16KD instead.

**3. Unused address pins must be tied to 0, not left floating.** For DATA_WIDTH=18, ADA3:ADA0 are unused and must be `1'b0`. Floating address bits cause unpredictable write targeting.

**4. MULT18X18D pipeline latency must match between synthesis and simulation.** If the synthesis instance has 4 pipeline stages but the behavioral model has 1, functional simulation will pass and hardware will fail. Count your register stages carefully.

**5. ALU54B requires placement in the same DSP tile as its MULT18X18D.** If nextpnr cannot auto-place them adjacently, add an LPF `LOCATE COMP` constraint.

**6. Use `-nobram -nodsp` in synth_ecp5 when being fully explicit.** Without these flags, Yosys may infer additional BRAM/DSP instances from other expressions in your design, silently blowing your resource budget.

**7. DP16KD and PDPW16KD are exclusive configurations of the same EBR.** You cannot set DATA_WIDTH_A=36 and use DP16KD — that is a PDPW16KD. Attempting it will fail silently in Yosys or error in nextpnr.

**8. OSCG frequency is approximate (±15%).** Never use it directly for anything requiring accurate timing. Route it through an EHXPLLL if you need a precise derived frequency.

---

## Reference Files

- `references/sim_stubs.sv` — Complete behavioral stubs for all common ECP5 primitives
- `references/pll_params.md` — PLL parameter tables for common input/output frequency pairs
- `references/ecp5_bram_guide.md` — Full DP16KD and PDPW16KD port mapping guide
- `references/dsp_guide.md` — Full MULT18X18D and ALU54B port listing with MAC example

---
> Source: [londey/ecp5-sv-yosys-verilator-skill](https://github.com/londey/ecp5-sv-yosys-verilator-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
