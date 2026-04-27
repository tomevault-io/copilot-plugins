## magic-pill

> File Creation Rules

# File Creation Rules
- Generate complete, standalone files when creating new code.
- Include imports, interfaces, and implementation in the same file unless modularized elsewhere.
- Don't forget JSDoc (in english) with proper typing, return values and examples when it could be handy
- Example structure for a new sub-component file for "Magic Pill" component:
```jsx
"use client"
import React, { useState } from "react";
import "./magicpill.css";

import { MagicPillNavbar } from "./types/navbar";

interface NavItemsProps {
  mode: "navbar";
  navbar: MagicPillNavbar;
}

const Navbar = ( {navbar}: NavItemsProps) => {

  const { navItems } = navbar || {};

  const [active, setActive] = useState("Home");

  return (
    <nav className="navbar">
    <ul className="navbarList">
      {navItems?.map((item, index) => (
        <li key={index} className={active === item.label ? "navbarItem active" : "navbarItem"} onClick={() => setActive(item.label)}>
          <a href={item.link} className="navbarLink">
            {item.label}
          </a>
        </li>
      ))}
    </ul>
  </nav>
  );
};

export default Navbar;
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CedricTheveneau) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
