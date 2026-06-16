## mcp-stl

> MCP server for editing STL 3D model files.

# MCP STL

MCP server for editing STL 3D model files.

## When to use this skill

Use this skill when you need to:
- Read/write STL files
- Transform 3D meshes
- Create geometric primitives
- Analyze 3D model properties

## Tools

**Reading & Writing:**
- `read_stl_file` - Read STL and return mesh data
- `get_mesh_info` - Get summary without full geometry
- `write_stl` - Write mesh (ASCII or binary)

**Transformations:**
- `translate_stl` - Move mesh by X, Y, Z
- `rotate_stl` - Rotate around axis
- `scale_stl` - Scale mesh

**Primitives:**
- `create_cube`, `create_sphere`, `create_cylinder`
- `create_cone`, `create_torus`, `create_plane`

## Install

```bash
pip install mcp-stl
```

---
> Source: [daedalus/mcp-stl](https://github.com/daedalus/mcp-stl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
