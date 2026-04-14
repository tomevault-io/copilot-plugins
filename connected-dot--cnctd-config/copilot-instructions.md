## cnctd-config

> > Brief reference for the configuration management library.

# CLAUDE.md - cnctd_config

> Brief reference for the configuration management library.

## Purpose

Generic JSON-based configuration file management with automatic serialization/deserialization for any type.

## Key Exports

```rust
pub struct Config<T: Serialize + DeserializeOwned> {
    pub data: T,
    pub file_path: String,
}

impl<T> Config<T> {
    pub fn new(data: T, file_path: &str) -> Self;
    pub fn update(&mut self, new_data: T) -> Result<()>;
    pub fn get(&self) -> &T;
    pub fn save(&self) -> Result<()>;
    pub fn load(file_path: &str) -> Result<Self>;
}
```

## Usage Example

```rust
use cnctd_config::Config;

#[derive(Serialize, Deserialize)]
struct AppSettings {
    theme: String,
    port: u16,
}

let config = Config::new(AppSettings { theme: "dark".into(), port: 8080 }, "settings.json");
config.save()?;
```

## Ecosystem Role

- **Used by**: Various apps needing persistent configuration

## Version

**0.1.2**

---

*Part of the cnctd monorepo. See `../../../CLAUDE.md` for ecosystem context.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Connected-Dot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Connected-Dot)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
