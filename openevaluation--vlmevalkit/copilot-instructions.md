## flask-web-app-pitfalls

> Lessons learned from building Flask web tools with pandas/numpy data and inline images


# Flask Web App with Pandas/Numpy: Common Pitfalls

## 1. `pd.isna()` crashes on lists/arrays

`pd.isna(val)` returns an array when `val` is a `list` or `np.ndarray`, causing `ValueError: The truth value of an array with more than one element is ambiguous` in boolean context.

```python
# ❌ BAD
if pd.isna(row["image"]):

# ✅ GOOD — guard list/array first
def _safe_isna(val):
    if val is None:
        return True
    if isinstance(val, (list, np.ndarray)):
        return False
    try:
        return bool(pd.isna(val))
    except (ValueError, TypeError):
        return False
```

## 2. `jsonify(**dict_a, key=val)` duplicate key crash

When `dict_a` already contains a key you also pass as a keyword arg, Python raises `TypeError: got multiple values for keyword argument`. Merge into the dict first.

```python
# ❌ BAD — pd_info already has "total"
return jsonify(ok=True, total=n, **pd_info)

# ✅ GOOD
pd_info.update(ok=True)
return jsonify(pd_info)
```

## 3. `int()` on non-numeric index columns

Dataset `index` columns can be strings (e.g. `"VerticalHorizontal_001_Q1"`). Always try/except.

```python
# ❌ BAD
idx_val = int(row[idx_col])

# ✅ GOOD
try:
    idx_val = int(raw_idx)
except (ValueError, TypeError):
    idx_val = str(raw_idx)
```

## 4. CSS `display:none` overrides JS `style.display=''`

Setting `element.style.display = ''` removes the inline style, falling back to CSS. If CSS has `display:none`, the element stays hidden.

```css
/* ❌ BAD — CSS hides it, JS can't unhide with style.display='' */
.my-panel { display: none; }

/* ✅ GOOD — use inline style for initial hide, CSS has no display rule */
```
```html
<div class="my-panel" style="display:none">...</div>
```
```javascript
// Then in JS, explicitly set block:
element.style.display = 'block';
```

## 5. `toliststr('')` crashes with `IndexError`

The `vlmeval.smp.misc.toliststr` function does `s[0]` without checking empty string. Always guard empty strings before calling.

```python
# ❌ BAD
return toliststr(v)

# ✅ GOOD
if isinstance(v, str) and not v.strip():
    return []
return toliststr(v)
```

## 6. Gradio image rendering is unreliable

Gradio's `Dataframe` image cells (numpy/PIL), `Markdown` with `data:` URIs, and `Gallery` lightbox are all fragile and version-dependent. For reliable image display, prefer Flask + plain HTML `<img src="data:image/jpeg;base64,...">` with `<a target="_blank">` for zoom.

---
> Source: [OpenEvaluation/VLMEvalKit](https://github.com/OpenEvaluation/VLMEvalKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
