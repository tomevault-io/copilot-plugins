## fall-of-nouraajd

> The repository default branch is `main`.

# Repository Guidelines

The repository default branch is `main`.

## Testing
Running tests is **mandatory** for every code change made by agents.
Always run tests from the repository root:
1. Build the test targets: `cmake --build cmake-build-release --target _game for_unit_tests -j$(nproc)`
2. Run the C++ unit tests: `ctest --test-dir cmake-build-release --output-on-failure -R for_unit_tests`
3. Run the Python test suite: `python3 test.py`

Coverage is also a requirement:
1. Run `./scripts/run_coverage.sh` when changing tests, coverage tooling, or code in `src/core`, `src/handler`, or `src/object`.
2. Keep the scoped line coverage at **80% or higher**. Do not finish coverage-related work below that threshold without explicitly calling it out.

This test suite requires the compiled `_game` module.
If the module or dependencies are missing, tests may fail; note this in the
Testing section. Import the optional `game` module inside each test that
requires it so that tests which don't depend on the compiled module can still
run.

### Resource-to-CMake checklist
- Whenever files are added under `res/`, update the root `CMakeLists.txt` so each new file is covered by `configure_file(...)` entries or by an `install(DIRECTORY ...)` rule as appropriate.
- Before committing, verify no `res/**` file is missing from CMake resource references.

## Code Style
- Use four spaces for indentation in both Python and C++ files.
- Python files are checked for indentation consistency during testing.
- Keep lines under 120 characters where practical.
- Python variable and function names should be in `snake_case`.
- C++ class names should use `CamelCase`.
- Classes decorated with `@trigger` should use `CamelCase` with a `Trigger` suffix.
- Reserve `UPPER_SNAKE_CASE` for global or constant-like properties and `snake_case` for local ones.
- Keep camelCase for object and item ids referenced from JSON.
- Engine methods may stay in CamelCase, but new Python-only methods should use `snake_case`.

### Naming Conventions for Triggers and Variables
The project mixes C++ and Python, so consistent naming is critical:

- **Trigger classes** should be in `CamelCase` and end with `Trigger`, e.g. `MarketTrigger`.
- **Python methods and local variables** use `snake_case` so that dialog actions
  and quest checks match their definitions in scripts.
- **Global flags** shared across maps are written in `UPPER_SNAKE_CASE` while
  temporary or object-specific properties remain `snake_case`.
- **Object and item identifiers** in JSON stay in `camelCase`; use the same
  IDs when referencing them from Python.
- Engine methods exposed from C++ keep their original `CamelCase` form.

### Auto Formatting
Use automated formatters before committing code:
- Run `clang-format -i` on any modified C++ files.
- Run `black -l 120` on Python files to match the line length guideline.

## Commit Messages
Describe changes clearly in the commit message.
If multiple logical changes are made, separate them into individual commits when
possible.
- Keep your branch up to date with the main branch.


## Merging Main
Regularly merge the repository's main branch (`main`) into your feature branch to
stay up to date:
Resolve any conflicts and rerun `python3 test.py`.

## Adding Item Types
Follow these steps whenever you introduce a new building, potion, effect, tile or other item type:

1. **Python plugin**
    - Add the class in a suitable file under `res/plugins/`.
    - Each plugin defines a `load(context)` function.
      Declare classes inside and decorate them with `@register(context)` so they
      become available to the game.

2. **Configuration entry**
    - Insert a new entry in the matching JSON file under `res/config/`.
      For example `items.json` for equipment or `potions.json` for potions.
    - The entry key is the item id used elsewhere.
      Specify the class name and any properties required by the object.

3. **Map update**
    - Reference the new item from map configuration files
      (such as `res/maps/nouraajd/config.json`).
      Use a `ref` field to point at the id from step 2, or define a class
      directly via a `class` field.

4. **Cross check ids**
    - Verify that the id in the config file matches the id used in the map and in
      any script calls like `createObject('YourItemId')`.
    - Mis‑matched ids will cause runtime errors when the map tries to spawn the item.

5. **Dialogs and dialog options**
    - To interact with the new item via dialog, add a new `CDialog` object to the
      map config or script.
    - Dialog states (`CDialogState`) can include options (`CDialogOption`) whose
      `action` or `nextStateId` references the item id or a method defined in
      your dialog class.
    - Ensure that every state and option id used in the JSON matches the definitions in the dialog class.
    - Implement each custom `action` as a method on the Python dialog class.
      The method name must match the `action` string used in the JSON option.
    - The dialog panel only checks for state ids named `ENTRY` and `EXIT` when
      opening and closing dialogs. They are not keywords; use any id names as
      long as references match.
    - The C++ class handling these dialogs is `CGameDialogPanel`; there is no
      separate `CDialogPanel` type.

After adding the new item type, run `python3 test.py` to confirm that loading the game still works.

## Quest Integrity
Follow these steps to verify quests remain consistent:

1. **Check ids**
    - Ensure every quest id in `res/maps/<map>/config.json` matches the
      class name in the corresponding `script.py` file.
2. **Short descriptions**
    - Keep quest descriptions brief; a single sentence is enough for the quest log.
3. **Playtest**
    - Build the game and start it with `python3 play.py`.
    - Accept each available quest and open the quest log with **j** to
      confirm it appears.
    - Progress the objectives and verify completed quests move to the
      "completed" section.
4. **Run tests**
    - Execute `python3 test.py` to ensure the game still loads with the
      updated quests.

## Exposing C++ Types to Python
Follow these steps when making a new C++ class usable from Python:

1. **Create a wrapper**
    - Add a `CWrapper<MyClass>` specialization in `src/core/CWrapper.h`.
    - Implement wrappers for any methods Python should override.

2. **Update the module**
    - Edit `src/core/CModule.cpp` and declare the wrapper with
      `class_<CWrapper<MyClass>, bases<BaseClass>, boost::noncopyable,
      std::shared_ptr<CWrapper<MyClass>>>("MyClass")` inside
      `BOOST_PYTHON_MODULE(_game)`.
    - Register the wrapper in `src/core/CTypes.cpp` using
      `CTypes::register_type<CWrapper<MyClass>, MyClass, BaseClass>();`.

3. **Create the Python object**
    - Define a subclass in a plugin under `res/plugins/`.
    - Add it inside `load(context)` and decorate with `@register(context)`.

After exposing the new type, run `python3 test.py` to verify the bindings.

## Map NPC vs Player Template Rule
For **all maps** under `res/maps/*`, do **not** place player-class templates
(`CPlayer`-based refs such as `Sorcerer`/`Warrior`/`Assasin`) as regular map
actors in `config.json`.

Doing so creates extra player objects at load and can produce debug lines like
`Loaded object: CPlayer:<objectName>(...)`.

For NPCs and scripted map actors, use non-player creature templates
(`CCreature`-based refs) and keep `npc: true`/controllers as needed.

To control where the real player starts, set map entry coordinates in each
map's `map.json` top-level `properties` (`x`, `y`, `z`) instead of adding a
separate player object.

## Copyright
When modifying source files, update the copyright year in
the header comments to 2025.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lisu188)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lisu188)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
