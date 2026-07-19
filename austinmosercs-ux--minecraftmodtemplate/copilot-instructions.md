## minecraftmodtemplate

> Small Minecraft mod that overrides spider/cave-spider sounds with a "yippee" clip. Use this directory as a working reference for building more mods of the same shape.

# YippeeSpider — NeoForge mod template

Small Minecraft mod that overrides spider/cave-spider sounds with a "yippee" clip. Use this directory as a working reference for building more mods of the same shape.

## Stack (the versions that actually work together)

- Minecraft 26.1.2 / NeoForge `26.1.2.44-beta` (NeoForge maven: <https://maven.neoforged.net/releases/net/neoforged/neoforge/maven-metadata.xml>)
- Gradle wrapper `9.0.0`
- JDK 25 (Homebrew `openjdk@25`)
- NeoForge moddev plugin `net.neoforged.moddev` `2.0.141`
- `org.gradle.toolchains.foojay-resolver-convention` **`1.0.0`** (earlier 0.8.0 fails on Gradle 9 — `JvmVendorSpec.IBM_SEMERU` was removed)

## Build

The user has no system Java on PATH. Always invoke gradlew with JAVA_HOME pointing at the real JDK home (not the Homebrew prefix):

```sh
JAVA_HOME=/opt/homebrew/opt/openjdk@25/libexec/openjdk.jdk/Contents/Home \
PATH=$JAVA_HOME/bin:$PATH \
./gradlew build
```

Output: `build/libs/<mod_id>-<mod_version>.jar`. Bump `mod_version` in `gradle.properties` for each release.

## VS Code Java extension

`.vscode/settings.json` must point at `/opt/homebrew/opt/openjdk@25/libexec/openjdk.jdk/Contents/Home` for all three keys (`java.jdt.ls.java.home`, `java.import.gradle.java.home`, `java.configuration.runtimes[].path`). Pointing at the Homebrew prefix `/opt/homebrew/opt/openjdk@25` leaves JDT.ls without `lib/` and triggers "java.lang.Object cannot be resolved".

## Project layout

```
build.gradle                       toolchain + neoForge {} block + runs
settings.gradle                    foojay resolver pin
gradle.properties                  mod_id, mod_name, mod_version, MC/NeoForge versions
gradle/gradle-daemon-jvm.properties   toolchainVersion=25
src/main/java/com/<mod_id>/<ModClass>.java   @Mod entrypoint
src/main/resources/pack.mcmeta     resource pack metadata
src/main/resources/assets/<mod_id>/sounds.json   sound overrides
src/main/resources/assets/<mod_id>/sounds/*.ogg  audio files
src/main/templates/META-INF/neoforge.mods.toml   mod metadata (templated)
```

## pack.mcmeta — pack_format > 64

Game crashes on launch with `JsonParseException: Pack declares support for version newer than 64, but is missing mandatory fields min_format and max_format` unless both range fields are present alongside `pack_format`. Use `[major, minor]` arrays:

```json
{
  "pack": {
    "description": "<description>",
    "pack_format": 84,
    "min_format": [84, 0],
    "max_format": [84, 2147483647]
  }
}
```

## Making a new mod from this template

1. Copy the directory, rename to the new mod name.
2. In `gradle.properties` change `mod_id`, `mod_name`, `mod_group_id`, `mod_version`, `mod_description`.
3. Rename the Java package `com.yippeespider` to match `mod_group_id`; rename the `@Mod` class.
4. Update `src/main/templates/META-INF/neoforge.mods.toml` (modid, displayName, description).
5. Replace assets under `src/main/resources/assets/<mod_id>/` with the new mod's resources. Update `sounds.json` keys to the events you want to override.
6. Build with the JAVA_HOME-prefixed gradlew command above.

## Sound overrides (pattern from this mod)

`assets/<mod_id>/sounds.json` redefines vanilla sound events via `"replace": true`:

```json
{
  "entity.spider.ambient": {
    "replace": true,
    "sounds": [{ "name": "<mod_id>:yippee", "stream": false }]
  }
}
```

The audio file goes at `assets/<mod_id>/sounds/yippee.ogg`. No Java code is required for pure sound replacement — the `@Mod` class can stay empty.

## Known external constraints

- `gradle.properties` minecraft/neoforge versions must match a real NeoForge maven artifact — check the maven-metadata URL above before bumping.
- `gradle/gradle-daemon-jvm.properties` `toolchainVersion` and `build.gradle` `JavaLanguageVersion.of(...)` must agree, and a matching JDK must exist locally (the daemon won't auto-download).

---
> Source: [austinmosercs-ux/MinecraftModTemplate](https://github.com/austinmosercs-ux/MinecraftModTemplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
