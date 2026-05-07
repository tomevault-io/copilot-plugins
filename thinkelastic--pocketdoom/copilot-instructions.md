## pocketdoom

> After making changes to the FPGA design or firmware:

# Project Instructions

## Testing Workflow

After making changes to the FPGA design or firmware:

1. **Deploy the FPGA bitstream:**
   ```bash
   cd src/fpga && make program
   ```

2. **Check the output using the capture tool:**
   ```bash
   tools/capture_ocr.sh
   ```

---
> Source: [thinkelastic/PocketDoom](https://github.com/thinkelastic/PocketDoom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
