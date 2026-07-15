## rexui-layout-rule

> **Opinionated Best Practices for Building Reliable UI Layouts in Phaser with RexUI**

# RexUI Layout Guidelines  
**Opinionated Best Practices for Building Reliable UI Layouts in Phaser with RexUI**

This guide defines consistent rules for using **RexUI sizers**, **RexUI ContainerLite**, and layout-related patterns.  
Follow these rules to minimize layout bugs, avoid positioning conflicts, and produce predictable UI.

---

## 1. Core Principles

### **1.1 Sizers control layout between siblings**  
Whenever a game object is a **child of a sizer**, the sizer fully owns its position.  
**Do NOT call `setPosition` on children after adding them to a sizer.**

Sizers determine:
- Order  
- Alignment  
- Spacing  
- Stretching  

You determine:
- *Which* objects belong to which sizer  
- Their natural size  

---

### **1.2 ContainerLite controls how elements inside a UI component are arranged**  
Use a **ContainerLite** when:
- You need overlay or custom internal layout  
- You want to treat a visual composition as a single unit for the parent sizer  

Inside a ContainerLite, **position children explicitly**, but **let the parent sizer position the container itself**.

---

### **1.3 Use sizers for structure, containers for composition**  
- Sizers = layout between UI regions  
- ContainerLite = layout *inside* a UI card or visual component  

Avoid mixing responsibilities.

---

## 2. Sizer Usage Rules

### **2.1 Always use configuration-object syntax for `add`**
It is clearer and avoids parameter-order mistakes.

✔ Preferred:
```ts
sizer.add(child, {
  proportion: 0,
  align: 'center',
  padding: 0,
  expand: false
});
```

✘ Avoid:
```ts
sizer.add(child, 0, 'center', 0, false);
```

---

### **2.2 Meaning of layout properties**

#### **proportion**  
Controls how much space a child takes **along the orientation**.

- `0` → keep natural size  
- `> 0` → stretch to take additional space  

Recommend:
- Use `0` per default  
- Only use proportions for flexible regions or scrollable panels  

#### **align**  
Controls alignment **across** orientation.

Examples:  
- Vertical sizer → `'left' | 'center' | 'right'`  
- Horizontal sizer → `'top' | 'center' | 'bottom'`  

Recommend:
- Default to `'center'` unless you intentionally left/right align elements  

#### **padding**  
Margin around the child.

Allowable values:
- Number → uniform  
- Object → directional  

Recommend:
- Keep it simple; rely on `space.item` for most spacing  

#### **expand**  
Stretch child **across** orientation.

- Vertical sizer → stretch width  
- Horizontal sizer → stretch height  

Recommend:
- Only set `expand: true` on:
  - Dropdowns  
  - Input fields  
  - Background bars or UI blocks  

For most visual components: `expand: false`.

---

## 3. Sizer Organization Rules

### **3.1 Only top-level sizers receive explicit world positioning**
Examples:
```ts
rootSizer.setPosition(x, y);
```
or
```ts
const rootSizer = rexUI.add.sizer({ x, y });
```

All child sizers and containers:
**do not set their world position manually**.  
Their parent sizer will place them.

---

### **3.2 Sizers should represent “regions”, not individual graphic atoms**
Use sizers at the level of:
- Horizontal bars  
- Vertical stacks  
- Sidebar groups  
- Card arrangements  

Do NOT use sizers for tiny atomic sub-layouts unless needed.  
Over-sizer’ing leads to confusion and layout churn.

---

### **3.3 Always call `layout()` once the hierarchy is built**
At end of UI construction:

```ts
rootSizer.layout();
```

Call again only when:
- Text changes
- Components are added/removed
- Dimensions change

Avoid calling layout repeatedly unless necessary.

---

## 4. ContainerLite Usage Rules

### **4.1 Always add children with `addLocal`**
This ensures local positioning inside the container behaves predictably.

```ts
container.addLocal(child);
```

Avoid `add()` unless you want world-space positioning preserved.

---

### **4.2 Always set the container’s size explicitly**
Parent sizers need to know the footprint.

```ts
container.setSize(width, height);
```

Often width/height match a background image.

---

### **4.3 Only use explicit coordinates *inside* a ContainerLite**
Example pattern:

```ts
const bg = add.image(0, 0, 'cardBG');
container.addLocal(bg);

const icon = add.image(-20, 30, 'icon');
container.addLocal(icon);

const label = add.text(10, -40, "Label");
container.addLocal(label);
```

Inside the container, these coordinates are relative.  
Outside the container, never override its position—let the sizer manage it.

---

### **4.4 ContainerLite should represent a reusable UI “component”**
Use containers for things like:
- Cards  
- Buttons with layered graphics  
- Icons with overlays  
- Composite elements  

If the entity feels like a **reusable single item**, use ContainerLite.

---

## 5. General Layout Best Practices

### **5.1 Establish a consistent grid**
Choosing consistent spacing makes layout predictable:
- `space.item: 8`, `16`, or `24`  
- global padding: 8 or 12  

---

### **5.2 Limit nesting depth**
Deeply nested sizers get hard to debug.  
Aim for:

- 1 root sizer  
- A few child sizers  
- Containers for complex visuals inside those  

“Three layers deep” is a good maximum for clarity.

---

### **5.3 Never mix manual positioning with sizer-managed positioning**
If a component is inside a sizer:
- No `x`, `y`, `setPosition`, `setOrigin`, or world transforms.

Manual transforms inside sizers almost always cause misalignment issues.

---

### **5.4 Prefer stable sizing**
When possible:
- Know sizes of backgrounds  
- Avoid dynamic resizing unless required  
- Avoid relying on `displayWidth`/`displayHeight` of text as a layout anchor unless needed  

Stable component sizes produce stable layouts.

---

### **5.5 Prefer sizer-level alignment over container-level origin hacks**
Avoid using:
```ts
element.setOrigin(0, 0);
```
to fix sizer alignment issues.

Instead, fix alignment in the `add()` call or adjust padding.

---

### **5.6 Re-layout after visibility changes**
If you toggle visibility or destroy components, call:

```ts
rootSizer.layout();
```

RexUI does not auto-heal layouts after removals.

---

## 6. Debugging Tools & Techniques

### **6.1 Enable sizer debugging rectangles**  
Use Rex’s debug flags:

```ts
sizer.setDebug(graphics);
```

This draws the footprint and helps visualize misalignment.

---

### **6.2 Validate that only containers have manual coordinates**
Search for all `setPosition` calls.  
If any target is a direct child of a sizer → remove.

---

### **6.3 Check that all ContainerLites have size set**
Missing `setSize()` causes:
- Zero-size sizers  
- Elements collapsing  
- Hover/input not matching visuals  

---

### **6.4 Print tree structure for clarity**
Build a helper to log sizer/child relationships—useful for agents.

---

## 7. Recap (The Rules That Matter Most)

1. **Sizers position their children. Never manually position sizer children.**  
2. **ContainerLite positions its children. Always use `addLocal`.**  
3. **Give every ContainerLite a size.**  
4. **Use sizers for layout, containers for composition.**  
5. **Use `expand` sparingly—mostly for interactive fields.**  
6. **Call `layout()` once after building the structure.**  
7. **Use config-object syntax for `.add()` to avoid mistakes.**  
8. **Keep nesting shallow and spacing consistent.**

---
> Source: [jeffgabriel/eurorails_ai](https://github.com/jeffgabriel/eurorails_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
