## gridsystem

> * The map consists of a **hexagonal grid with flat-topped hexes**.

## 📜 Eurorails Grid Construction & Navigation Rules

### 1. 🔶 Grid Type and Orientation

* The map consists of a **hexagonal grid with flat-topped hexes**.
* **Row-based offset layout** is used:

* Every **odd-numbered row is offset right** (aka **odd-r offset**).
* Coordinates are stored as `(col, row)`, but row determines offset.
* Note that we use x and y for drawing game world coordinates which are distinct numbers, but the same points from col,row local coordinate system points in the configuration. These are not redundant in our GridPoint type.

---

### 2. 🛍️ Coordinate System

* Each hexagon has a unique `(col, row)` pair identifying its center.
* Grid points are defined in a shared point coordinate space with **walls/edges reused** by adjacent hexes.
* Each hexagon center is surrounded by 6 shared **corner points**, defined in the same `(col, row)` coordinate space.
* The 6 adjacent point coordinates for a hex at `(col, row)` are:

  ```ts
  const pointNeighbors: [number, number][] = [
    [-1,  0], // left
    [ 0, -1], // upper-left
    [ 1, -1], // upper-right
    [ 1,  0], // right
    [ 1,  1], // lower-right
    [ 0,  1], // lower-left
  ];

  const neighbors = pointNeighbors.map(([dx, dy]) => [col + dx, row + dy]);
  ```

---

### 3. 🔗 Neighbor Determination

* Each hex shares 6 corner points with adjacent space.
* Movement, pathfinding, and connections happen **between these point pairs**.
* Neighboring points are calculated using the offsets listed in section 2.
* These points are **shared** among multiple adjacent hexes and define the valid edge segments.

---

### 4. 🧱 Point Sharing and Edge Awareness

* Hexes **share corner and edge points** with adjacent hexes.
* Each edge between hexes is represented by **a pair of shared corner points**.
* Movement and track placement operate **along these shared point edges**, not from hex center to hex center.

---

### 5. 🚄 Pathfinding and Movement

* Movement between edges costs **1 unit** unless terrain adds modifiers (e.g. mountain, ferry).
* Movement is only allowed:

  * Along player-built track
  * Across valid connected point edges
  * Without skipping intermediate points

---

### 6. 🏩 City and Outpost Handling

* Cities exist at the **center of a hex**, with **6 possible outpost points**, one per direction as listed in section 2.
* Major cities must have 6 outposts explicitly defined unless blocked by terrain or water.
* Each outpost is associated with a point-based edge and lies on a shared wall between the city hex and its neighbor point.

---

### 7. 🗺️ Rendering Logic

* Hex center positions are calculated:

```ts
x = col * HEX_WIDTH;
y = row * HEX_HEIGHT + (col % 2 === 1 ? HEX_HEIGHT / 2 : 0);
```

* Ensure grid renders **seamlessly** with hexes touching but not overlapping.
* Label each corner point for debugging or inspection.

---

### 8. 📦 Data Binding

* Use `gridPoints.json` to assign meaning to hexes:

  * Type: `'Plain' | 'Mountain' | 'Ocean' | 'Major City' | 'Outpost'`
  * Name: optional for cities or ferries
  * Each `GridX`, `GridY` pair maps to `(col, row)` on the game grid

---

### 9. ❗ Validation Requirements

* All city outposts must align with valid point neighbors.
* No overlapping hex or point coordinates.
* Paths must not cross non-adjacent point pairs.

---

### 10. 🧠 LLM Agent Hints

* Avoid hardcoding directions — use point-based neighbor offsets.
* Do not duplicate edges or wall segments — represent shared point pairs only once.
* When rendering or simulating, treat each edge as a **pair of point coordinates**.
* Use `Set<string>` or coordinate hashes like `col,row` to deduplicate or validate point grid elements.

---
> Source: [jeffgabriel/eurorails_ai](https://github.com/jeffgabriel/eurorails_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
