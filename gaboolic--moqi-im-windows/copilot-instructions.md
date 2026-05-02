## protoc

> Use the local Windows protoc toolchain when working with protobuf schemas or generated files in the proto directory


When working with protobuf files in `proto/`, use the local `protoc` toolchain at `D:\a_dev\protoc-33.5-win64`.

- Treat `.proto` files as the source of truth.
- Do not hand-edit generated `*.pb.h` or `*.pb.cc` files unless there is a documented exception.
- If protobuf definitions change, regenerate the generated files with the local `protoc` installation above.

---
> Source: [gaboolic/moqi-im-windows](https://github.com/gaboolic/moqi-im-windows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
