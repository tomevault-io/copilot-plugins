## outbreak

> Fabric mod for Minecraft 26.3 (Scala 3.9 / Kotlin 2.4 / Java 25). See `WILLOW.md` and `PHYSIOLOGY.md`

# Outbreak

Fabric mod for Minecraft 26.3 (Scala 3.9 / Kotlin 2.4 / Java 25). See `WILLOW.md` and `PHYSIOLOGY.md`
for the content documentation, and `tools/README.md` for the asset generators and dev self tests.

## Languages: where code goes

The mod is deliberately split by language, and the build enforces the direction
(`src/main/scala` -> `src/main/kotlin` -> `src/main/java` -> `src/client`):

| Language | Lives in | Owns |
| --- | --- | --- |
| **Scala 3** | `src/main/scala`, package `...outbreak.physiology.model` | **Every number and every calculation.** The per-tick model, the steady states, the mineral reference ranges, the derived symptom magnitudes. |
| Kotlin | `src/main/kotlin` | Minecraft glue: Fabric attachments and codecs, mob effects, events, the command, the tick handler - and the storage value types (`OutbreakData`, `Mediators`, `Electrolytes`, `TraceElements`, `Mineral`). |
| **Java** | `src/main/java/.../physiology/OutbreakModelBridge.java` | **The seam to the model.** The one file allowed to mention a Scala type: converts Kotlin state to and from `ModelState`, forwards every operation and magnitude, re-exports the numbers. |
| Java | `src/main/java`, `src/client/java` | Mixins, because Loom's annotation processor and the compiler check the targets. |
| Kotlin | `src/client/kotlin` | Client-only rendering and HUD. |

**Write new numerical or steady-state code in Scala, in `src/main/scala`.** Rules that keep this
workable:

* The model is a **pure function of numbers**: it must not import Minecraft, Kotlin, Fabric or
  anything under `src/main/kotlin`. It compiles first, and that is why it cannot.
* **No Kotlin file may name a Scala type** - not in a signature, not in a body, not in a KDoc link.
  K2 resolves a Scala class by also loading its supertypes, so a Kotlin file that merely mentions
  `Mineral` drags in `scala.Product` and IntelliJ reports `Cannot access 'scala.Product' which is a
  supertype of 'Mineral'`, however the classpath is wired. That is why Kotlin has its own
  `OutbreakData`/`Mediators`/`Electrolytes`/`TraceElements`/`Mineral`, and why every call crosses
  through `OutbreakModelBridge`. The grep that checks it: any hit for `physiology.model` under
  `src/main/kotlin`, or for one of the model's names (`ModelState`, `ModelConstants`, `ModelMineral`,
  `ModelMediators`, `ModelElectrolytes`, `ModelTraceElements`, `MineralRanges`, `MediatorLevels`,
  `ElectrolyteDefaults`, `TraceElementDefaults`, `Physiology.`), is a bug.
* `OutbreakModelBridge` must not put a Scala type in a **public** field, parameter or return type
  either: Kotlin resolves those eagerly, even though it resolves the Java file's private fields and
  method bodies lazily (verified: a private field of a nonexistent type does not break
  `compileKotlin`). Scala types are private fields and bodies only.
* Nothing in the bridge may contain a number or a threshold. If a value is needed on both sides it
  is added to the model and re-exported through `OutbreakModelBridge` under the name Kotlin uses.
* The model compiles in its own task, `compileModelScala`, and **not** through the Scala plugin's
  `compileScala`. That task unconditionally depends on `compileJava` (joint Java/Scala compilation),
  which would close the cycle `compileJava -> compileKotlin -> compileScala -> compileJava`; the
  dependency survives the source set having no Java at all, and clearing it from `dependsOn` does not
  stick because the plugin re-adds it when the task is realised. A hand-made `ScalaCompile` task
  needs four things the plugin normally supplies by convention: `incrementalOptions.analysisFile`,
  `incrementalOptions.classfileBackupDir`, `targetCompatibility`, and `javaLauncher`. The plugin's
  own `compileScala` is left with no sources so nothing is compiled twice.
* The model's output reaches everything downstream as a plain file dependency
  (`files(compileModelScala)` on the main and client classpaths, plus `tasks.jar { from(scalaClasses) }`),
  never as `libraries.from(model.output)` or `compileClasspath += <another source set>.output`.
  IntelliJ turns those into a dependency on another *module* and then resolves Scala types out of
  that module's sources as light classes.
* Kotlin cannot use named arguments on Java methods, and Java has no default arguments, so
  `OutbreakPhysiology` keeps the old call sites working: it is a thin Kotlin facade over the bridge
  that restores the defaults (`tick(data)`, `drink(data)`, `seed(data, bacteria = 4f)`).
* If a Scala type needs a `@BeanProperty` getter for Java to read a field, it needs one - the bridge
  reads `state.getBacteria()`, not `state.bacteria()`.
* **The model's names never collide with Kotlin's.** Its case classes are `ModelMineral`,
  `ModelMediators`, `ModelElectrolytes`, `ModelTraceElements` and `ModelState`, and its constants
  live in *separately named* objects rather than in companions - `MineralRanges`, `MediatorLevels`,
  `ElectrolyteDefaults`, `TraceElementDefaults`, `ModelConstants` - so that a Java file can
  `import ...physiology.model.*` and refer to every one of them by its short name. That is also why
  there is no `@static` anywhere: it is only legal in a companion object, and the constants reach Java
  through the static forwarders Scala generates for a plain object (`ModelConstants.WATER_MAX()`,
  `MineralRanges.SODIUM()`), and `Physiology.tick(...)` the same way.
* Losing the companions means losing `apply`: the model builds its values with `new ModelState(...)`,
  `new ModelMediators(...)` and so on, and copies them with the `withX` methods.
* The Scala runtime is shaded into the built jar by `build.gradle.kts`; Loom's `include` produced an
  empty wrapper for it, so do not go back to that. Note that since Scala 3.9
  `org.scala-lang:scala3-library_3` is a 344-byte stub and the real standard library is
  `org.scala-lang:scala-library`; both have to be declared, because Zinc will not run without the
  former.

The seam, in one picture:

    src/main/scala/.../physiology/model   numbers, no Minecraft, no Kotlin
        ^  plain imports and private fields only
        |
    src/main/java/.../physiology/OutbreakModelBridge.java    constants, conversion, forwarding
        ^  OutbreakData and primitives only
        |
    src/main/kotlin/.../physiology       storage, codecs, symptoms, contagion, command
        ^
        |
    src/client/kotlin                    rendering and HUD

### IntelliJ writes into `build/classes/java/main`

IntelliJ builds a Gradle-imported module with its own compiler when "Build and run using" is
*IntelliJ IDEA*, and copies `src/main/resources` into the module's compiler output - which for this
project is Gradle's `build/classes/java/main`. The classes directory comes before the resources
directory on the classpath, so those copies shadow the ones `processResources` produced:

* the copied `fabric.mod.json` is the raw file, so it still says `${version}` and a dev run dies with
  *"Some of your mods are incompatible with the game or each other!"*;
* every other copy makes `jar` fail on a duplicate entry, and would put every texture in the jar
  twice.

`build.gradle.kts` guards both: the `dropIdeResourceCopies` task deletes the copies as the last step
before any `run` task and before `jar`, and `jar` additionally refuses a resource path that came from
`build/classes`. The task leaves empty `assets/` and `data/` directories behind, because `delete`
only ever removes files; they are harmless.

The lasting fix is a one-setting change in the IDE: **Settings -> Build, Execution, Deployment ->
Build Tools -> Gradle -> Build and run using: Gradle**. Nothing in the build can prevent the IDE
from writing there afterwards.

## Agent skills

### Issue tracker

Issues live in this repo's GitHub Issues, driven by the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

The five canonical triage roles, each label string equal to its role name. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `CONTEXT.md` and `docs/adr/` at the repo root. See `docs/agents/domain.md`.

---
> Source: [cao-awa/Outbreak](https://github.com/cao-awa/Outbreak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
