## defold-simplex-noise

> It's a native extension for the Defold game engine. It uses Lua 5.1 as its scripting language with "bit" module for bitwise operations. 4 spaces for indentation. "snake_case" for variable, function, file, folder names. It uses LDoc for documentation.

It's a native extension for the Defold game engine. It uses Lua 5.1 as its scripting language with "bit" module for bitwise operations. 4 spaces for indentation. "snake_case" for variable, function, file, folder names. It uses LDoc for documentation.

The example of LDoc is:

```lua
--- Summary ends with a period.
-- Some description, can be over several lines.
-- @tparam string p1 first parameter
-- @tparam[opt] string p2 second parameter (optional)
-- @treturn number a number value
-- @see second_fun
function mod1.first_fun(p1,p2)
end
```

The structure of the project is the following:
- folder "simplex_noise" contains the C/C++ part of the extension.
- folder "example" contains the example project written in Lua for the extension.

---
> Source: [indiesoftby/defold-simplex-noise](https://github.com/indiesoftby/defold-simplex-noise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
