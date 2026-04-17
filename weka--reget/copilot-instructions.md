## reget

> 1. The code is written in Go. Follow the good practices for code structuring

1. The code is written in Go. Follow the good practices for code structuring
   and formatting.
2. Read the description of what the program is supposed to do from @spec/reget.md
3. To run the tests, run `go test ./...`
4. Run `gofmt` to fix the formatting issues after every change.
5. Also run linters after doing changes:
   - go vet ./...
   - staticcheck ./...
6. To test:
   - If `local/reget.yaml` and `local/inputs.json` files are present, try
     running reget as `go run . --config=local/reget.yaml --input-file=local/inputs.json`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/weka) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
