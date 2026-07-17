## tk-warning

> Critical Tkinter/MuJoCo threading compatibility warning


# 🚨 Critical Threading Issue - XCB/Tkinter/MuJoCo

**CRITICAL**: This project has a threading compatibility issue when combining Tkinter GUI with MuJoCo viewer.

## The Problem
Using `ttk.Entry` or `tk.Entry` widgets alongside MuJoCo's viewer causes XCB threading crashes:
```
[xcb] Unknown sequence number while appending request
[xcb] Aborting, sorry about that.
python3: ../../src/xcb_io.c:157: append_pending_request: Assertion `!xcb_xlib_unknown_seq_number' failed
```

## Root Cause
- `ttk.Entry` widgets create XCB threading conflicts with MuJoCo's OpenGL viewer context
- This happens regardless of XInitThreads() calls
- The issue is specific to Entry widgets; other Tkinter widgets work fine

## Required Solutions

### 1. Never use Entry widgets
```python
# ❌ DON'T DO THIS - causes crashes
self._input = ttk.Entry(self, textvariable=self.my_var)

# ✅ DO THIS INSTEAD - safe alternative
self._display = ttk.Label(self, text="0.000", relief="sunken", anchor="e")
def _update_display(*args):
    self._display.configure(text=f"{float(self.my_var.get()):.3f}")
self.my_var.trace_add("write", _update_display)
```

### 2. Defer mujoco operations until after viewer starts
```python
# ❌ DON'T DO THIS - during UI setup
site_id = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_SITE, "site_name")

# ✅ DO THIS INSTEAD - after viewer starts
with mujoco.viewer.launch_passive(...) as viewer:
    # Now safe to do mujoco operations
    site_id = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_SITE, "site_name")
```

## Safe Widget Types
- ✅ `ttk.Label` - Safe
- ✅ `ttk.Button` - Safe  
- ✅ `ttk.Checkbutton` - Safe
- ✅ `ttk.Scale` - Safe
- ❌ `ttk.Entry` - **NEVER USE**
- ❌ `tk.Entry` - **NEVER USE**

## Testing Protocol
Always test GUI changes with: `uv run <script_name>.py`
If you see XCB threading errors, check for Entry widgets or early mujoco calls.

## Working Examples
- `mink_g1_pose_ik.py` - Working baseline (no Entry widgets)
- `temp.py` - Enhanced version with threading-safe workarounds

---
*This issue was discovered and resolved in December 2024 after extensive debugging.*

---
> Source: [jloganolson/g1-poser](https://github.com/jloganolson/g1-poser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
