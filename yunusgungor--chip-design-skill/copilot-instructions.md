## chip-design-skill

> Çip Tasarımı: RTL'den Tapeout'a Pratik Workflow. Verilog/SystemVerilog, açık kaynak EDA (OpenROAD, Yosys, OpenLane, Verilator), SKY130 PDK, RISC-V CPU tasarımı, FPGA prototyping ve tapeout süreçleri.


# CHİP-DESİGN — Çip Tasarımı Skill'i

Pratik çip tasarımı workflow'u: RTL'den tapeout'a kadar end-to-end süreç.

## ⚡ Hızlı Başlangıç

```
1. Konsept           → Belirle ne yapacaksın
2. RTL Kodlama       → Verilog/SystemVerilog
3. Simülasyon        → Verilator (ücretsiz) veya Icarus
4. Sentez            → Yosys (ücretsiz)
5. P&R               → OpenROAD veya OpenLane
6. Fiziksel Doğrulama → Magic (DRC/LVS)
7. GDSII Export      → Tapeout için hazır dosya
```

##  RTL Tasarımı

### Dil Seçimi

**Verilog (IEEE 1364):** Temel, yaygın, tüm araçlar destekler. 1995'ten beri standart.

**SystemVerilog (IEEE 1800):** Verilog'un superseti. Verification için zorunlu (class, interface, coverage, assertions, package). Üretim tasarımında da giderek yaygınlaşıyor.

**Chisel (Scala-based):** Yüksek seviyeli, OO + fonksiyonel. Rocket Chip, CVA6, SiFive çipleri bu dilde yazılmış. Kod üretimi çok daha kompakt.

**SpinalHDL (Scala-based):** Chisel'e alternatif. Daha güçlü type system, daha fazla kontrol. Tüm VHDL/Verilog'a compile ediyor.

**Amaranth HDL (Python-based):** Python DSL. Linux dünyasında popüler. Nmigen projesinin devamı.

**Clash (Haskell-based):** Haskell'in gücünü HDL'e taşıyor. Strong typing, compile-time synthesis.

**MyHDL (Python-based):** Python → Verilog dönüştürücü. Düşük bariyer giriş için iyi.

### Temel Yapılar

```verilog
// Kombinasyonel
assign y = a & b;
always @* begin
  case (sel)
    2'b00: out = a + b;
    2'b01: out = a - b;
    default: out = 0;
  endcase
end

// Senkron
always @(posedge clk or negedge rst_n) begin
  if (!rst_n) q <= 0;
  else q <= d;
end

// FSM (one-hot encoding)
typedef enum logic [2:0] {
  IDLE  = 3'b001,
  READ  = 3'b010,
  WRITE = 3'b100
} state_t;
```

### High-Level Synthesis (HLS)

C/C++/SystemC → RTL dönüşümü. Tasarımcı daha soyut seviyede tanımlama yapar.

**Komersiyel HLS Araçları:**
- **Vitis HLS** (AMD/Xilinx) — C/C++/SystemC → Verilog/VHDL, Xilinx FPGA hedefli
- **Intel HLS Compiler** — C++ → Verilog, Intel FPGA hedefli
- **Catapult HLS** (Siemens) — C++/SystemC → RTL, ASIC + FPGA
- **Stratus HLS** (Cadence) — C/SystemC → RTL, yüksek kalite

**Açık Kaynak HLS:**
- **LegUp** (Toronto Üniv.) — C → Verilog, LLVM tabanlı, FPGA
- **Bambu** (Politecnico di Milano) — C → Verilog, geniş input desteği
- **OpenArc** — AccelChip mirası, ticari desteği olan açık sistem

**Ne zaman HLS:** Algoritmik yoğun tasarımlar (signal processing, video codec), büyük veri yolu manipülasyonu. Manuel RTL daha fazla kontrol sağlar.

### Best Practices

**Clock Domain Crossing (CDC):**
- Tek bit: 2-flip-flop synchronizer (minimum)
- Çok bit: Gray code FIFO veya handshake protocol
- Asenkron FIFO:读写 pointer Gray encode, full/empty binary compare

**Reset:**
- Asenkron reset kullanıyorsan: recovery ve removal timing analiz et
- Senkron reset tercih et (daha az glitch riski)
- Reset tree synthesis: gated reset, reset buffering

**Pipeline:**
- Throughput × latency tradeoff: her pipeline stage ~1 clock cycle ekler
- Stall sinyal: upstream'e "durdur" sinyali ver
- Bubble: gereksiz stage'leri boş atla
- Handshake: valid/ready protokolü ile data transfer

**Coding style:**
- Tek boyutlu wire/reg
- `casez`/`casex` yerine `case` + if/else wildcards
- Synchronous reset her zaman tercih et
- Lint araçları (Verilator `--lint-only`) sürekli çalıştır

##  Açık Kaynak EDA Araçları

### Kurulum (Ubuntu 22.04+)

```bash
# Docker (OpenROAD için — önerilen)
sudo apt install -y docker.io
sudo usermod -aG docker $USER
docker pull theopenroadproject/openroad:latest

# Yosys
sudo apt install yosys

# Verilator (en güncel için: kaynak koddan derle)
sudo apt install verilator

# Magic (DRC/LVS)
sudo apt install magic

# Icarus Verilog
sudo apt install iverilog

# KLayout (GDS viewer)
sudo apt install klayout

# Netgen (LVS)
sudo apt install netgen
```

### OpenLane Workflow

OpenROAD'ın wrapper'ı. Design exploration ve hardening için.

```bash
# OpenLane indir
git clone --depth 1 https://github.com/The-OpenROAD-Project/OpenLane.git
cd OpenLane
make setup  # PDK'ları indirir (~5GB)

# İlk tasarım
cp -r designs/spm designs/my_chip
# designs/my_chip/config.tcl düzenle

# Akışı çalıştır
./flow.tcl -design my_chip -tag run_001

# Sonuçlar
ls designs/my_chip/runs/run_001/results/final/
```

### OpenROAD Komutları

```bash
# Interactive
openroad

# Script ile
openroad -quiet -skel scripts/floorplan.tcl
openroad -quiet scripts/pdn.tcl
openroad -quiet scripts/route.tcl

# Timing analizi
openroad -quiet -skel scripts/timing.tcl
```

### Yosys Sentez

```bash
# Basic synthesis (ekran çıktısı)
yosys -p "read_verilog design.v; synth -top top; stat"

# FPGA (Xilinx)
yosys -p "read_verilog design.v; synth_xilinx -top top -edif design.edif"

# ASIC (SKY130)
yosys -p "
  read_verilog rtl.v
  synth -top top
  dfflibmap -liberty $PDK_ROOT/sky130A/libs.ref/sky130_sram_sc_hd/lib/sky130_sram_1kbyte_1rw1r_32x256_8_hd.v
  abc -liberty $PDK_ROOT/sky130A/libs.ref/sky130_sram_sc_hd/lib/sky130_sram_1kbyte_1rw1r_32x256_8_hd.v
  stat
"
```

### Verilator Simülasyon

```bash
# Lint (syntax check)
verilator --lint-only --Wall design.sv

# Compile
verilator --cc --exe --build -j 0 \
  --top-module top \
  --sv \
  top.sv tb_top.sv \
  -CFLAGS "-I." \
  -o Vtop

# Run
./obj_dir/Vtop

# Waveform (VCD)
verilator --cc --exe --build -j 0 --trace \
  top.sv tb_top.sv -o Vtop
./obj_dir/Vtop
# GTKWave ile aç: gtkwave tb_top.vcd
```

### PDK Karşılaştırması

- **SKY130 (130nm):** Google+SkyWater ortaklığı. En popüler açık PDK. 5 metal katman, 1.8V/5V, Apache 2.0 lisans. ~3.5K GitHub yıldızı.
- **GF180MCU (180nm):** GlobalFoundries. 6 metal katman, 3.3V/6V, RF/mixed-signal'a uygun. OpenLane uyumlu.
- **ASAP7 (7nm):** Arizona State + ARM. Predictive FinFET PDK. Sadece araştırma için. OpenROAD ile kullanılabilir.
- **TSMC 180nm:** Ticari, akademik lisans gerekli. Standart hücre kütüphaneleri mevcut.

### PDK Kurulumu

```bash
export PDK_ROOT=/opt/pdk
export OPENLANE_ROOT=/opt/openlane
mkdir -p $PDK_ROOT

# SKY130
git clone https://github.com/google/skywater-pdk.git $PDK_ROOT/sky130A

# GF180MCU
git clone https://github.com/google/gf180mcu-pdk.git $PDK_ROOT/gf180mcu

# SRAM modelleri (SKY130 için)
git clone https://github.com/google/sky130_sram_1kbyte_1rw1r_32x256_8_hd.git \
  $PDK_ROOT/sky130A/libs.ref/sky130_sram_1kbyte_1rw1r_32x256_8_hd
```

##  Komersiyel EDA

### Synopsys Araçları
- **Design Compiler** — RTL sentez (industry standard)
- **VCS** — Simülasyon (Verilog/SystemVerilog/UVM)
- **ICC II / Fusion Compiler** — Place & Route
- **PrimeTime** — Timing sign-off
- **SpyGlass** — Linting ve CDC analizi
- **ZeBu** — Emulation (milyonlarca kapı)
- **DSO.ai** — AI tabanlı tasarım optimizasyonu

### Cadence Araçları
- **Genus** — Sentez
- **Innovus** — P&R
- **Xcelium** — Simülasyon
- **Virtuoso** — Analog/mixed-signal
- **Spectre** — SPICE simülasyonu
- **Palladium** — Emulation
- **JasperGold** — Formal verification

### Siemens EDA
- **Questa** — Simülasyon ve verification
- **Calibre** — Physical verification (DRC/LVS/OPC)
- **Tessent** — DFT (scan, ATPG, MBIST)

### Akademik Lisans
- **Synopsys SARA** — Ücretsiz akademik lisans (synopsys.com/university)
- **Cadence Academic Network** — Üniversite programı (cadence.com/en_US/home/training)
- **Siemens EDA HEP** — Lisans talep et

## ☁️ Bulut EDA

EDA araçlarına internet üzerinden erişim. Yüksek başlangıç maliyetini ortadan kaldırır.

**Pazar:** 2025'te $3.8 milyar → 2034'te $12.6 milyar (CAGR ~%14)

### Synopsys Cloud
- Sınırsız EDA lisansı erişimi
- Anlık lisans temini, HPC compute
- AI araçları (DSO.ai) dahil
- Fiyat: kullanim başına veya aylık

### Cadence OnCloud
- Token tabanlı abonelik
- Tüm Cadence araçları
- Esnek ölçeklendirme

### AWS / Azure HPC
- AWS EC2 F1 instance (Xilinx FPGA)
- Custom AMI'ler ile EDA araçları
- İş başına ödeme modeli

### Startup Odaklı Alternatifler
- **Altair Startup Package** — Düşük maliyetli lisanslama
- **Silicon Cloud** — Tapeout-as-a-service

##  Maliyet ve Bütçe

### Startup Maliyetleri

**Minimum başlangıç (2-3 kişilik ekip, temel sentez + simülasyon):**
- $100K–$300K / yıl
- Araç kombinasyonu: Yosys (ücretsiz) + Verilator (ücretsiz) + Cadence/Synopsys single-seat
- Alternatif: Tamamen açık kaynak (maliyet ~$0)

**Küçük startup (5-15 mühendislik):**
- $200K–$800K / yıl
- Sentez + Simülasyon + P&R + Timing

**Orta ölçekli (15-50 mühendislik):**
- $500K–$2M / yıl
- Full EDA toolchain

**Büyük şirket:**
- $2M–$20M / yıl
- Tüm ticari araçlar + custom scripts + support

### Tapeout Maliyetleri

- **Google OpenMPW:** Ücretsiz (Aralık 2024'te kapandı)
- **Tiny Tapeout:** Çok düşük maliyet, yıllık shuttle'lar
- **ChipFoundry:** Düşük maliyet, SKY130/GF180MCU
- **MPW (Multi-Project Wafer):** ~$1K–$3K / slot
- **Ticari shuttle (tek proje):** ~$10K–$100K
- **Tam mask set (standart):** ~$1M–$5M
- **2nm node mask:** ~$5M–$10M

##  Simülasyon ve Doğrulama

### Verification Metodolojileri Karşılaştırması

**UVM (Universal Verification Methodology):**
- Endüstri standardı verification framework
- SystemVerilog üzerine kurulu, sınıf kütüphanesi
- Modüler, yeniden kullanılabilir, ölçeklenebilir
- Versiyon: IEEE 1800.2-2020 (UVM 2020)
- Destekleyen: Synopsys VCS, Cadence Xcelium, Siemens Questa

**Assertion-Based Verification:**
- SVA (SystemVerilog Assertions) ile property yazımı
- `assert` — çıktı kontrolü, `assume` — giriş kısıtlaması, `cover` — coverage toplama
- Tüm simulation araçlarında çalışır

**Formal Verification:**
- State space'i matematiksel olarak keşfeder
- Model checking: tüm state space'i sistematik tarar
- Property checking: SVA assertion'ları formal olarak kanıtlar
- Bounded proof: belirli cycle derinliğine kadar tarar
- Unbounded proof: sonsuz state space'i kanıtlar
- Araçlar: JasperGold, VC Formal, OneSpin, ECClone
- Avantaj: Driver/monitor/testcase yazmaya gerek yok
- Sınırlama: Büyük tasarımlar için compute-intensive

### UVM Temelleri

```systemverilog
// UVM component hiyerarşisi
class my_agent extends uvm_agent;
  `uvm_component_utils(my_agent)
  my_driver  driver;
  my_monitor monitor;
  uvm_sequencer#(my_item) sequencer;

  function void build_phase(uvm_phase phase);
    driver    = my_driver::type_id::create("driver", this);
    monitor   = my_monitor::type_id::create("monitor", this);
    sequencer = uvm_sequencer#(my_item)::type_id::create("sequencer", this);
  endfunction

  function void connect_phase(uvm_phase phase);
    driver.seq_item_port.connect(sequencer.seq_item_export);
  endfunction
endclass

// Test
class my_test extends uvm_test;
  `uvm_component_utils(my_test)
  my_env env;

  function void build_phase(uvm_phase phase);
    env = my_env::type_id::create("env", this);
    `uvm_info("TEST", "Running my_test", UVM_MEDIUM)
  endfunction
endclass

// UVM phase sırası:
// build → connect → end_of_elaboration → run → extract → check → report
```

### Coverage Metrikleri

**Code Coverage (otomatik):**
- Line coverage: hangi satırlar çalıştı
- Branch coverage: her if/case dalı test edildi mi
- FSM coverage: her state ve transition
- Toggle coverage: her bit 0→1 ve 1→0 geçti mi
- Condition coverage: her boolean ifade

**Functional Coverage (manuel):**
- Covergroup: custom functional coverage model
- Coverpoint: spesifik değişken veya ifade
- Cross coverage: iki coverpoint arasındaki kombinasyonlar
- Bins: her değer veya değer aralığı için sayaç

**Hedefler:** Code coverage ≥%90, tüm functional coverage bins karşılandı.

### Bug Hunting Techniques

**Constrained-Random Testing:**
- Rastgele stimulus ile geniş state space tarama
- Weighted constraints ile stimulus dağılımını ayarla
- Multiple random seeds ile coverage artır

**Directed Testing:**
- Manuel yazım test senaryoları
- Corner case'ler için zorunlu
- Bug reproduksiyonu için
- Compliance test için

**Coverage-Driven Verification:**
- Coverage deliklerini tespit et
- Düşük coverage bölgelerine yönel
- Covergroup bins eksikse yeni test ekle

##  Tapeout Süreci

### Adımlar

```
1. RTL Tamamlama          → Simülasyon, verification geçti
2. Sentez                 → Gate-level netlist
3. Floorplanning          → Die boyutu, IP yerleşimi, power grid
4. Placement              → Standart hücre yerleşimi
5. Clock Tree Synthesis   → Clock distribution, skew minimizasyonu
6. Routing                → Metal bağlantıları, timing optimizasyonu
7. Physical Verification  → DRC, LVS, ERC, Antenna check
8. GDSII Export          → Streamout, foundry'ye hazır dosya
9. Foundry'ye Gönderim    → Mask data hazırlığı
10. Wafer Üretimi         → Fabrikasyon süreci
11. Packaging             → Die → chip (QFN, BGA, vb.)
12. Test & Teslimat       → Chip probing (CP), final test (FT)
```

### Fiziksel Doğrulama

**DRC (Design Rule Check):**
- Spacing: metal/spacing, via/spacing
- Width: minimum metal genişliği
- Density: metal yoğunluk kuralları
- Enclosure: via, contact overhang
- Araçlar: Magic, KLayout, Calibre

**LVS (Layout vs Schematic):**
- Netlist extraction: layout'tan SPICE netlist çıkar
- Comparison: schematic netlist ile karşılaştır
- Open/short: bağlantı hataları
- Device mismatch: transistör/WIDTH farkı
- Araçlar: Magic + Netgen, KLayout, Calibre

**ERC (Electrical Rule Check):**
- Floating nodes: bağlantısız sinyaller
- P/G shorts: power/ground kısa devre
- Well/substrate: body bağlantıları
- Multiple drivers: bus contention

**Antenna Check:**
- Plasma-induced gate damage: metal biriktirme sırasında gate hasarı
- Mitigation: antenna diode ekleme, metal hopping (layer değiştirme)

### GDSII Export

```bash
# Magic ile
magic -d WRAPPER design.gds
# File → Export GDSII

# OpenROAD ile
openroad -quiet scripts/write_gds.tcl

# KLayout ile
klayout -z design.gds -a design_lyp.lyp
```

### Ücretsiz Tapeout Yolları

⚠️ **Efabless Aralık 2024'te kapandı.** Güncel alternatifler:

**Tiny Tapeout:**
- Çok düşük maliyet, yıllık shuttle'lar
- SKY130 PDK, mixed-signal desteği
- tinytapeout.com

**ChipFoundry:**
- Düşük maliyet, SKY130 ve GF180MCU
- chipfoundry.org

**IHP Semiconductor:**
- 130nm BiCMOS, RF/mixed-signal'a uygun
- ihp-semi.com

**IHP 130nm Open:**
- IHP'nin açık erişim girişimi
- ihp-semi.com/technology/open-innovation

### DFM (Design for Manufacturability)

- Via duplication: kritik viayı ikiye kopyala
- Dummy fill: metal yoğunluk gradyanını düzleştir
- OPC (Optical Proximity Correction): mask düzeltmesi
- Antenna ratio control: max metal/gate oranı

##  RISC-V CPU Tasarımı

### CPU Çekirdek Karşılaştırması

- **PicoRV32** — 32-bit, in-order, ~750-2K LUT (Xilinx 7S). Minimal, hızlı derleme, RTOS/microcontroller için.
- **SERV** — 32-bit, bit-serial, ~125 LUT. Ultra-düşük güç, IoT için ideal.
- **CVA6** (eski Ariane) — 64-bit, in-order, Linux destekli, ~0.87mm² @22nm. OpenHW Group tarafından geliştiriliyor.
- **Rocket Chip** — 64-bit, in-order, Chisel ile yazılmış, Linux destekli. UC Berkeley mirası.
- **BOOM v3** (SonicBOOM) — 64-bit, out-of-order, yüksek performans. En güçlü açık RISC-V çekirdeği.

### Board Seçimi

- **VisionFive 2** (JH7110, $55-85) — 4×U74, Linux, aktif geliştirme
- **ESP32-C5** (~$10) — 240MHz RISC-V + WiFi 6, MCU segmenti
- **ESP32-C6** (~$10) — 160MHz RISC-V + Thread/Zigbee
- **Arty A7/S7** (~$149+) — FPGA kartı, RISC-V soft-core için

### Toolchain Kurulumu

```bash
# GCC (bare-metal / embedded)
sudo apt install riscv64-unknown-elf-gcc
riscv64-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -Os -o firmware.elf firmware.c

# GCC (Linux)
./configure --prefix=/opt/riscv --with-arch=rv64gc --with-abi=lp64d && make linux
riscv64-unknown-linux-gnu-gcc -march=rv64gc -o app app.c

# QEMU emulation
sudo apt install qemu-system-misc
qemu-system-riscv64 -kernel vmlinux -append "root=/dev/vda" \
  -drive file=rootfs.ext4,format=raw -nographic

# Spike (ISA simulator)
spike -l pk firmware.elf
```

### Linux Destek Matrisi

- PicoRV32, SERV: ❌ RTOS veya bare-metal (FreeRTOS, Zephyr)
- CVA6, Rocket Chip, BOOM: ✅ Linux destekli

## ⚡ FPGA Prototyping

### Araçlar

- **Vivado** (Xilinx) — FPGA P&R, HLS, tüm Xilinx platformları
- **Quartus Prime** (Intel) — FPGA P&R, Intel/ALTERA platformları
- **nextpnr** (açık kaynak) — Lattice (ECP5, iCE40), Gowin için P&R
- **SymbiFlow** — Xilinx 7-series için açık kaynak toolchain
- **Yosys** — FPGA/ASIC sentez

### Başlangıç Kartları

- **Arty S7** ($149) — Xilinx Spartan-7, başlangıç için ideal
- **DE10-Lite** ($80) — Intel MAX 10, düşük maliyet
- **ULX3S** ($115) — Lattice ECP5, açık kaynak toolchain tam destekli
- **Gowin GW1N** (~$30) — Bütçe seçeneği

### ASIC vs FPGA Karşılaştırması

**Maliyet:** ASIC NRE $500K–$20M+; FPGA birim başına $50–$10K
**Performans:** ASIC 10× daha hızlı, 10× daha düşük güç tüketimi
**Yoğunluk:** ASIC 45× daha yoğun
**Hacim:** >10K ünite yıllık → ASIC avantajlı
**Zamana üretim:** ASIC 3-6 ay; FPGA dakikalar
**Esneklik:** FPGA yeniden programlanabilir; ASIC sabit

### FPGA'dan ASIC'e Göç Workflow

1. **RTL hazırlığı:** FPGA-specific primitive'leri (BUFG, IDDR, ODDR, STARTUPE2) standart HDL'e dönüştür. Global reset ve initial block'ları kaldır.
2. **Clock yönetimi:** FPGA PLL/DCM/CMT → standart cell library'deki PLL ile değiştir. Clock tree synthesis hazırlığı.
3. **RAM/ROM mapping:** FPGA BRAM → ASIC standard cell SRAM/ROM. ECP5/iCE40 EBR → foundry SRAM.
4. **I/O interface:** FPGA I/O standard → ASIC I/O library (LVDS, HSTL, vb.)
5. **Timing constraints:** FPGA timing budget → ASIC SDC formatına dönüştür.
6. **Power:** FPGA power estimation → ASIC power grid tasarımı
7. **Test:** FPGA'daki test pattern'larını ASIC testbench'ine adapte et
8. **Area optimization:** FPGA'ın overdesign'ını ASIC için optimize et

##  Öğrenme Kaynakları

### Online Kurslar

- **Zero to ASIC** (Matthew Venn) — zerotoasiccourse.com — Açık kaynak araçlarla tapeout kursu
- **ChipVerify** — chipverify.com — Verilog, SystemVerilog, UVM
- **NANDLAND** — nandland.com — FPGA başlangıç
- **EDA Playground** — edaplayground.com — Tarayıcıda Verilog/SystemVerilog simülasyonu

### Kitaplar

- *Digital Design and Computer Architecture* — Harris & Harris (Verilog odaklı, başlangıç)
- *CMOS VLSI Design* — Weste & Harris (Teorik derinlik)
- *SystemVerilog for Verification* — Chris Spear (UVM)
- *Writing Testbenches* — Janick Bergeron
- *Writing Testbenches using SystemVerilog* — Janick Bergeron

### Konferanslar

- **DAC** (Design Automation Conference) — Yıllık, EDA ve tasarım otomasyonunun en büyüğü
- **ICCAD** — Yıllık, CAD destekli tasarım konferansı
- **FPL** (Field-Programmable Logic) — Yıllık, FPGA konferansı
- **ORConf** — OpenROAD topluluk konferansı, açık kaynak EDA
- **RISC-V Summit** — Yıllık, RISC-V ekosistemi

### Sertifikasyon

- **Synopsys Certified Professional** — Design Compiler, ICC, VCS vb. için
- **Synopsys Purple Certification** — ASIC tasarım akışı giriş sertifikası
- **Cadence Academic Network** — Digital/Physical Design Domain Certification

### Topluluklar

- **r/FPGA** — Reddit FPGA topluluğu
- **r/ChipDesign** — VLSI/çip tasarımı
- **Tiny Tapeout Discord** — Tapeout topluluğu
- **OpenROAD Discord** — Açık kaynak EDA
- **Semiconductor Engineering** — Endüstri haberleri ve analiz

### YouTube

- **Sam Zeloof** — Ev yapımı yarı iletken fabrikasyonu
- **MIT 6.004** — Computation Structures (bilgisayar mimarisi)
- **ChipVerify** — UVM, SystemVerilog eğitimleri

### GitHub Organizasyonları

- **OpenROAD** — RTL→GDSII açık kaynak akış
- **OpenHW Group** — CVA6, CORE-V RISC-V çekirdekleri
- **RISC-V International** — ISA spesifikasyonları, toolchain
- **lowRISC** — OpenTitan güvenli çip projesi
- **CHIPS Alliance** — Donanım açık kaynak standartları

## ️ Skill Kullanım Haritası

| Senaryo | Önerilen Adımlar |
|---------|-----------------|
| İlk kez çip tasarımı | Zero to ASIC → OpenLane + SKY130 → Basit RISC-V core |
| FPGA'dan ASIC'e geçiş | FPGA→ASIC migration workflow → OpenROAD → SKY130 PDK → DRC/LVS → Tiny Tapeout |
| RISC-V CPU geliştirme | PicoRV32 (basit) → CVA6 (Linux) → Kendi tasarım |
| Sadece simülasyon | Verilator + Icarus (ücretsiz) |
| Ticari araçlarla eğitim | Synopsys/Cadence akademik lisans |
| İlk tapeout | Tiny Tapeout veya ChipFoundry |
| Bulut EDA kullanımı | Synopsys Cloud veya Cadence OnCloud |
| Formal verification | SVA assertions + JasperGold/VC Formal |

## ⚠️ Pitfalls

- **Clock domain crossing** — Her zaman 2-FF sync kullan, çok bit için Gray code FIFO
- **Reset** — Asenkron reset → recovery/removal timing kontrol et, reset tree synthesis
- **GDSII layer mapping** — Magic/KLayout layer map PDK ile uyumlu olmalı
- **SKY130 SRAM** — SRAM otomatik değil, OpenRAM tasarla veya harici kullan
- **UVM overhead** — Küçük projeler için UVM gereksiz, basit testbench yeterli
- **Verilator limits** — Tüm SystemVerilog özelliklerini desteklemez, spec kontrol et
- **Tapeout deadline** — Tiny Tapeout belirli tarihlerde, kaçırma
- **FPGA→ASIC göç** — Initial block'lar, $readmemh, FPGA-specific primitive'leri unutma
- **Formal verification** — Büyük tasarımlarda state explosion problem, modüler kanıtla
- **Coverage** — %100 code coverage yeterli değil, functional coverage da gerekli

##  Referans Dosyaları

- `references/rtl_examples.md` — Verilog/SystemVerilog örnek kodlar (272 satır)
- `references/openlane_quickstart.md` — OpenLane kurulum ve kullanım rehberi (168 satır)
- `references/riscv_development.md` — RISC-V toolchain, QEMU, OpenOCD rehberi (252 satır)
- `references/source_library.md` — Güvenilir kaynak kütüphanesi (16KB, 46 kaynak)
- `references/knowledge_workflow.md` — Bilgi çekme yordamı (5 senaryo)
- `references/pdk_comparison_research.md` — SKY130 vs GF180MCU test bulguları [YENİ]
- `references/skill-upgrade-workflow.md` — Skill upgrade metodolojisi
- `references/skill-upgrade-v201.md` — v2.0.1 upgrade findings (extract-evidence, citation verify, HTML fixes) ← **YENİ v2.0.1**
- `reference/methodology.md` — 8-fazlı araştırma boru hattı (15KB) ✓
- `reference/quality-gates.md` — Kalite standartları ve doğrulama (6KB) ✓
- `reference/continuation.md` — Auto-continuation protocol (8KB) ← **YENİ v2.0**
- `reference/report-assembly.md` — Progressive generation (11KB) ← **YENİ v2.0**

### ️ Chip Design Domains

Bu skill aşağıdaki çip tasarımı domainlerini destekler:

| Domain | Açıklama | Örnek Konular |
|--------|---------|---------------|
| `RTL` | Register Transfer Level | Verilog, SystemVerilog, FSM, pipeline |
| `EDA` | Electronic Design Automation | OpenROAD, OpenLane, Yosys, synthesis |
| `PDK` | Process Design Kit | SKY130, GF180MCU, DRC/LVS |
| `VERIFICATION` | Tasarım doğrulama | Verilator, Icarus, formal, simulation |
| `ARCHITECTURE` | CPU/mikro mimari | RISC-V, ISA, pipeline hazardları |
| `PHYSICAL_DESIGN` | Fiziksel yerleşim | P&R, floorplanning, placement |
| `TIMING` | Zamanlama analizi | STA, slack, clock tree |
| `POWER` | Güç tüketimi | Dynamic/leakage power, rail analysis |

## 🤖 Otomasyon Script'leri

- `scripts/chip_citation_manager.py` — SHA-256 stable source IDs + run manifest (14KB) ← **YENİ v2.0**
- `scripts/chip_verify_citations.py` — DOI/URL/hallucination checker (16KB) ← **YENİ v2.0**
- `scripts/chip_md_to_html.py` — Markdown→HTML converter (14KB) ← **YENİ v2.0**
- `scripts/chip_extract_claims.py` — İddia çıkarma ve analiz (13KB) ← **YENİ v2.0**
- `scripts/chip_verify_claim_support.py` — Kanıt-iddia desteği doğrulama (9KB) ← **YENİ v2.0**
- `scripts/chip_verify_html.py` — HTML rapor doğrulama (7KB) ← **YENİ v2.0**
- `scripts/chip_research_engine.py` — Araştırma motoru ve state management (12KB) ← **YENİ v2.0**
- `scripts/chip_evidence_store.py` — JSONL kanıt yönetim sistemi (15KB) ← **GÜNCELLENDİ**
- `scripts/chip_validate_report.py` — 9-kontrol kalite doğrulayıcı (13KB) ← **GÜNCELLENDİ**
- `scripts/chip_source_evaluator.py` — Kaynak güvenilirlik puanlaması (14KB) ← **GÜNCELLENDİ**

##  Rapor Şablonları

- `templates/report_template.md` — Araştırma raporu yapısı
- `templates/mckinsey_report_template.html` — McKinsey-style HTML rapor (15KB) ← **YENİ v2.0**

###  JSON Schema Dosyaları

- `schemas/claim.schema.json` — İddia yapısı (2KB)
- `schemas/evidence.schema.json` — Kanıt yapısı (2KB)
- `schemas/source.schema.json` — Kaynak yapısı (2KB)
- `schemas/run_manifest.schema.json` — Run manifest yapısı (3KB)

###  Dependencies

- `requirements.txt` — Python bağımlılıkları (optional) ← **YENİ v2.0**
- `tests/` — Test suite (7 test) ← **YENİ v2.0**

##  Dinamik Kaynak Sistemi

Bu skill sadece statik bilgi depolamaz — **canlı kaynaklardan bilgi çeker.**

### Nasıl Çalışır

```
Kullanıcı soru sorar
  → Konu analizi (domain tespiti)
  → Kaynak haritasına bakılır (source_library.md)
  → 8-fazlı boru hattı: SCOPE→PLAN→RETRIEVE→TRIANGULATE→OUTLINE→SYNTHESIZE→CRITIQUE→REFINE→PACKAGE
  → Evidence store'a kaydet (sources.jsonl, evidence.jsonl, claims.jsonl)
  → Kalite doğrulaması (chip_validate_report.py)
  → Kullanıcıya sun
```

### Araştırma Modları

| Mod | Faz | Süre | Kullanım |
|-----|-----|------|---------|
| **Quick** | 3 | 2-5 dk | Hızlı bilgi |
| **Standard** | 6 | 5-10 dk | Tipik tasarım soruları |
| **Deep** | 8 | 10-20 dk | Kritik kararlar |
| **UltraDeep** | 8+ | 20-45 dk | Kapsamlı inceleme |

### Bilgi Çekme Senaryoları

| Senaryo | Süre | Yöntem |
|---------|------|--------|
| Hızlı bilgi | ≤2 dk | Skill → ChipVerify'den doğrudan çek |
| Derinlemesine | 5-15 dk | Birden fazla kaynak → sentezle |
| Topluluk | 5-10 dk | Discord + GitHub issues ara |
| Güncel trend | 3-5 dk | Semiconductor Engineering tara |
| Akademik | 10-20 dk | MIT + Berkeley + arXiv |

### En Değerli Kaynaklar

** Zero to ASIC** (Matthew Venn) — Açık kaynak tapeout'un öncüsü
→ zerotoasiccourse.com

** ChipVerify** — Verilog/SystemVerilog/UVM için en iyi öğrenme kaynağı
→ chipverify.com + YouTube

** Cliff Cummings Papers** — RTL best practices, CDC, FSM
→ sunburst-design.com

** RISC-V International** — ISA spesifikasyonları (kesin referans)
→ riscv.org/technical/specifications

** Semiconductor Engineering** — Endüstri haberleri ve trend analizi
→ semiengineering.com

Tüm kaynaklar için: `references/source_library.md` dosyasına bak.

### Kanıt Yönetimi (Evidence Store)

Her araştırma kanıtı diske kaydedilir — bağlam compaction'tan kurtulur:

```
~/.hermes/skills/chip-design/evidence/
├── sources.jsonl     # Kaynak kayıtları (stable ID)
├── evidence.jsonl    # Kanıt girdileri (alıntı + locator)
└── claims.jsonl      # İddia kayıtları (type + verification)
```

**İddia tipleri ve davranışları:**

| Tip | Doğrulama | Davranış |
|-----|-----------|---------|
| `factual` | Hard-fail | Destek yoksa hata ver |
| `synthesis` | Traceability | İzlenebilirlik gerekli |
| `recommendation` | Traceability | İzlenebilirlik gerekli |
| `speculation` | Auto-pass | Sadece etiketle |

### Kalite Doğrulama (Her Yanıtta)

```bash
# Otomatik 9 kontrol (yapı + teknik doğruluk)
python scripts/chip_validate_report.py --report [rapor]

# Citation doğrulama (DOI/URL + hallucination detection)
python scripts/chip_verify_citations.py --report [rapor]
python scripts/chip_verify_citations.py --report [rapor] --strict

# Kaynak güvenilirlik
python scripts/chip_source_evaluator.py --eval --url "..." --title "..." --year 2025
```

### Citation Manager CLI

```bash
# Research run başlat
python scripts/chip_citation_manager.py init-run \
  --out-dir ~/research/chip-design/sky130_research_20260506 \
  --query "SKY130 PDK karşılaştırması" --mode deep

# Kaynak ekle (SHA-256 source_id ile)
python scripts/chip_citation_manager.py register-source \
  --json '{"raw_url":"https://...","title":"...","source_type":"technical_doc","chip_domain":"EDA"}' \
  --dir ~/research/chip-design/sky130_research_20260506

# Display numaralarını ata [1], [2], [3]...
python scripts/chip_citation_manager.py assign-display-numbers \
  --dir ~/research/chip-design/sky130_research_20260506

# Bibliyografya export
python scripts/chip_citation_manager.py export-bibliography \
  --dir ~/research/chip-design/sky130_research_20260506 --style markdown
```

### HTML Export

```bash
# Markdown → McKinsey-style HTML
python scripts/chip_md_to_html.py research_report.md \
  --output report.html --title "SKY130 Analysis" --template
```

### Çok-Persona Red Teaming (Deep/UltraDeep)

1. **Şüpheci Uygulayıcı** — "Bu komut gerçekten çalışıyor mu?"
2. **Adversarial Reviewer** — "Alternatifler ne sunuyor? Limitler neler?"
3. **Uygulama Mühendisi** — "Pratikte kimse bunu yapmaz. Daha iyi ne?"

---

## ⚠️ Pitfalls (Test Edildi)

Gerçek araştırma testlerinde tespit edilen yaygın hatalar:

1. **Section isimlendirme:** Validator `## Bibliography` bekler — `## Kaynaklar` kabul edilmez
2. **Citation formatı:** `[1]`, `[2]` formatı kullan — parantez `(1)` değil
3. **Zorunlu bölümler:** Executive Summary, Introduction, Limitations, Bibliography, Methodology
4. **"Suspicious" citation:** Akademik PDF'ler normal olarak işaretlenir
5. **Claim-support:** Evidence store'a kanıt manual eklenmeli
6. **HTML uyarıları:** Template-specific uyarılar normal, önemli değil

Detaylar için: `reference/quality-gates.md`

---

## Araştırma Kaynağı

Bu skill, `~/research/cip-tasarimi/` dizinindeki kapsamlı araştırmadan oluşturuldu:
- 10 araştırma maddesi, 10 sub-agent, ~100+ web araması
- 10 JSON dosyası, 79 farklı konu başlığı
- Tarih: 2026-05-05

## İlgili Skill'ler

- `/llm-wiki` — LLM arka plan bilgisi için
- `/jupyter-live-kernel` — Python ile veri analizi
- `/github-code-review` — GitHub PR kod inceleme

---
> Source: [yunusgungor/chip-design-skill](https://github.com/yunusgungor/chip-design-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
