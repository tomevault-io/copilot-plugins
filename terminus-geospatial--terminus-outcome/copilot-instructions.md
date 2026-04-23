## terminus-outcome

> * I like my #includes to be organized


* I like my #includes to be organized
  * I always order them with **C++ Standard Libraries first**, then **other/third‑party libraries** (Boost, GoogleTest, etc.), and **Terminus libraries last**.
  * Within each group, I prefer to organize `#include`s alphabetically, unless there is a strong reason not to.

* I prefer header files looking something like this:

```cpp
/**************************** INTELLECTUAL PROPERTY RIGHTS ****************************/
/*                                                                                    */
/*                           Copyright (c) 2024 Terminus LLC                          */
/*                                                                                    */
/*                                All Rights Reserved.                                */
/*                                                                                    */
/*          Use of this source code is governed by LICENSE in the repo root.          */
/*                                                                                    */
/***************************# INTELLECTUAL PROPERTY RIGHTS ****************************/
/**
 * @file    configure.hpp
 * @author  Marvin Smith
 * @date    7/8/2023
*/
#pragma once

// C++ Standard Libraries
#include <filesystem>
#include <fstream>
// other required #includes which are built-in

// Boost Libraries
#include <boost/log/expressions.hpp>
// Other non-terminus projects following C++ standard libs

// Terminus Libraries
#include <tmns/log.hpp>
// and other "project libraries"

namespace tmns::log::impl {

class Foo_Bar
{
    public:
        // public types and aliases
        // public variables
        // public functions

    protected:
        // protected types and aliases
        // protected functions
        // protected variables

    private:

        // private types and aliases
        // private functions
        // private variables

}; // End of Foo_Bar Class

} // end of tmns::log::impl namespace
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Terminus-Geospatial) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
