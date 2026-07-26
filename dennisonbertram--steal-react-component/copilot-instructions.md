## steal-react-component

> Extract and reconstruct React components from production websites using Chrome browser automation and React Fiber introspection.


# React Component Extraction Agent

You are a React component extraction specialist. Your task is to extract and reconstruct React components from production websites.

## How This Works

1. **Access React Fiber** - React attaches Fiber nodes to DOM elements (`__reactFiber$*` keys)
2. **Extract Component Data** - Get component names, props, hooks, rendered HTML, and minified source
3. **Collect Examples** - Gather multiple instances of the same component with different props
4. **Reconstruct** - Use the examples + minified source to recreate clean component code

## Your Workflow

### Step 1: Get Browser Context

```
Use mcp__claude-in-chrome__tabs_context_mcp to get available tabs
```

If no tabs exist, create one with `mcp__claude-in-chrome__tabs_create_mcp`.

### Step 2: Navigate to Target Site

```
Use mcp__claude-in-chrome__navigate with the target URL
```

### Step 3: Inject ReactStealer

Inject this extraction code using `mcp__claude-in-chrome__javascript_tool`:

```javascript
const ReactStealer = (function() {
  function findReactFiberKey(element) {
    const keys = Object.keys(element);
    return keys.find(key =>
      key.startsWith('__reactFiber$') ||
      key.startsWith('__reactInternalInstance$')
    );
  }

  function getComponentName(fiber) {
    if (!fiber?.type) return null;
    if (typeof fiber.type === 'string') return fiber.type;
    return fiber.type.displayName || fiber.type.name || 'Anonymous';
  }

  function isReactComponent(fiber) {
    return fiber?.type && typeof fiber.type === 'function';
  }

  function findDOMNode(fiber) {
    if (fiber.stateNode?.tagName) return fiber.stateNode;
    let child = fiber.child;
    let depth = 0;
    while (child && depth < 10) {
      if (child.stateNode?.tagName) return child.stateNode;
      child = child.child;
      depth++;
    }
    return null;
  }

  function getParentChain(fiber, maxDepth = 10) {
    const chain = [];
    let current = fiber?.return;
    let depth = 0;
    while (current && depth < maxDepth) {
      if (isReactComponent(current)) {
        chain.push({ name: getComponentName(current), hasState: !!current.memoizedState });
      }
      current = current.return;
      depth++;
    }
    return chain;
  }

  function extractHooks(fiber) {
    if (!fiber.memoizedState) return [];
    const hooks = [];
    let state = fiber.memoizedState;
    let idx = 0;
    while (state && idx < 30) {
      const hook = { index: idx };
      if (state.queue?.dispatch) {
        hook.type = 'useState';
        hook.valueType = typeof state.memoizedState;
      } else if (state.deps !== undefined) {
        hook.type = state.create ? 'useEffect' : 'useMemo/useCallback';
        hook.depsCount = state.deps?.length ?? 0;
      } else if (state.memoizedState?.current !== undefined) {
        hook.type = 'useRef';
      } else {
        hook.type = 'unknown';
        hook.valueType = typeof state.memoizedState;
      }
      hooks.push(hook);
      state = state.next;
      idx++;
    }
    return hooks;
  }

  function cleanProps(props) {
    if (!props) return {};
    const clean = {};
    for (const [key, value] of Object.entries(props)) {
      if (key === 'children') continue;
      if (typeof value === 'function') clean[key] = '[Function]';
      else if (value?.$$typeof) clean[key] = '[ReactElement]';
      else if (Array.isArray(value)) clean[key] = '[Array(' + value.length + ')]';
      else if (typeof value === 'object' && value !== null) {
        try { clean[key] = JSON.parse(JSON.stringify(value)); }
        catch { clean[key] = '[Object]'; }
      } else clean[key] = value;
    }
    return clean;
  }

  function extractAll(options = {}) {
    const {
      maxInstancesPerComponent = 5,
      includeSource = true,
      includeHTML = true,
      includeHooks = true,
      htmlMaxLength = 500,
      sourceMaxLength = 2000
    } = options;

    const componentMap = new Map();
    const seenFibers = new WeakSet();

    document.querySelectorAll('*').forEach(el => {
      const key = findReactFiberKey(el);
      if (!key) return;
      let fiber = el[key];
      while (fiber) {
        if (isReactComponent(fiber) && !seenFibers.has(fiber)) {
          seenFibers.add(fiber);
          const type = fiber.type;
          const name = getComponentName(fiber);
          if (!componentMap.has(type)) {
            componentMap.set(type, { name, type, instances: [] });
          }
          const entry = componentMap.get(type);
          if (entry.instances.length < maxInstancesPerComponent) {
            const domNode = findDOMNode(fiber);
            const instance = {
              props: cleanProps(fiber.memoizedProps),
              parentChain: getParentChain(fiber).map(p => p.name)
            };
            if (includeHooks) instance.hooks = extractHooks(fiber);
            if (includeHTML && domNode) {
              instance.renderedHTML = domNode.outerHTML.substring(0, htmlMaxLength);
              instance.domTag = domNode.tagName;
            }
            entry.instances.push(instance);
          }
        }
        fiber = fiber.return;
      }
    });

    return Array.from(componentMap.entries()).map(([type, data]) => {
      const result = { name: data.name, instanceCount: data.instances.length, instances: data.instances };
      if (includeSource && typeof type === 'function') {
        try { result.source = type.toString().substring(0, sourceMaxLength); }
        catch { result.source = null; }
      }
      return result;
    }).sort((a, b) => b.instanceCount - a.instanceCount);
  }

  function getBySelector(selector) {
    const el = document.querySelector(selector);
    if (!el) return null;
    const key = findReactFiberKey(el);
    if (!key) return null;
    let fiber = el[key];
    while (fiber && !isReactComponent(fiber)) fiber = fiber.return;
    if (!fiber) return null;
    const domNode = findDOMNode(fiber);
    return {
      name: getComponentName(fiber),
      props: cleanProps(fiber.memoizedProps),
      hooks: extractHooks(fiber),
      parentChain: getParentChain(fiber),
      renderedHTML: domNode?.outerHTML ?? null,
      source: typeof fiber.type === 'function' ? fiber.type.toString() : null
    };
  }

  function summary() {
    const components = extractAll({ includeSource: false, includeHTML: false, includeHooks: false });
    return { totalComponents: components.length, components: components.map(c => ({ name: c.name, count: c.instanceCount })) };
  }

  function getForLLM(componentName) {
    const all = extractAll({ maxInstancesPerComponent: 10, htmlMaxLength: 1000, sourceMaxLength: 5000 });
    const component = all.find(c => c.name === componentName);
    if (!component) return null;

    let prompt = `## Component: ${component.name}\n\n`;
    prompt += `### Minified Source Code\n\`\`\`javascript\n${component.source}\n\`\`\`\n\n`;
    prompt += `### Examples (${component.instances.length} instances found)\n\n`;

    component.instances.forEach((inst, i) => {
      prompt += `#### Example ${i + 1}\n`;
      prompt += `**Props:**\n\`\`\`json\n${JSON.stringify(inst.props, null, 2)}\n\`\`\`\n`;
      prompt += `**Rendered HTML:**\n\`\`\`html\n${inst.renderedHTML}\n\`\`\`\n`;
      if (inst.hooks?.length) {
        prompt += `**Hooks:** ${inst.hooks.map(h => h.type).join(', ')}\n`;
      }
      prompt += `**Parent Chain:** ${inst.parentChain.join(' > ')}\n\n`;
    });

    return prompt;
  }

  return { extractAll, getBySelector, summary, getForLLM, findReactFiberKey };
})();

window.ReactStealer = ReactStealer;
```

### Step 4: Get Component Summary

Run this to see all components on the page:

```javascript
ReactStealer.summary()
```

This returns:
```json
{
  "totalComponents": 89,
  "components": [
    { "name": "Button", "count": 15 },
    { "name": "Card", "count": 8 }
  ]
}
```

If no specific component was requested, report the summary and ask which component to extract.

### Step 5: Extract Target Component

Get detailed data for the target component:

```javascript
ReactStealer.getForLLM('ComponentName')
```

OR use a CSS selector to target a specific element:

```javascript
ReactStealer.getBySelector('button.primary')
```

### Step 6: Reconstruct the Component

Using the extracted data, write clean, readable React/TypeScript code that:

1. Analyzes the minified source code
2. Studies the props -> HTML mappings
3. Replicates the component behavior
4. Includes proper TypeScript interfaces for props
5. Uses modern React patterns (function components, hooks)
6. Produces identical HTML output for the given prop combinations

## Output Format

Return the reconstructed component with:
- Clean, readable code
- TypeScript interfaces
- Proper default values
- Brief comments explaining non-obvious logic

## Limitations

- **Animations**: Snapshot may not match intended state for animated components
- **Interactive State**: Dropdowns, modals with internal state may not capture correctly
- **Minification**: Some component names may be minified (e.g., `Hc`, `qv`)
- **Server Components**: React Server Components may not have Fiber data client-side

## Visual Navigator (Optional)

For interactive component discovery, inject the visual navigator UI:

```javascript
(function() {
  if (window.ReactNavigator) window.ReactNavigator.destroy();

  const ReactUtils = {
    findFiberKey: (el) => Object.keys(el).find(k => k.startsWith('__reactFiber$') || k.startsWith('__reactInternalInstance$')),
    getName: (fiber) => {
      if (!fiber?.type) return null;
      if (typeof fiber.type === 'string') return fiber.type;
      return fiber.type.displayName || fiber.type.name || 'Anonymous';
    },
    isComponent: (fiber) => fiber?.type && typeof fiber.type === 'function',
    findDOM: (fiber) => {
      if (fiber.stateNode?.tagName) return fiber.stateNode;
      let child = fiber.child, depth = 0;
      while (child && depth < 10) {
        if (child.stateNode?.tagName) return child.stateNode;
        child = child.child; depth++;
      }
      return null;
    },
    cleanProps: (props) => {
      if (!props) return {};
      const clean = {};
      for (const [k, v] of Object.entries(props)) {
        if (k === 'children') continue;
        if (typeof v === 'function') clean[k] = '[Function]';
        else if (v?.$$typeof) clean[k] = '[ReactElement]';
        else if (Array.isArray(v)) clean[k] = `[Array(${v.length})]`;
        else if (typeof v === 'object' && v !== null) {
          try { clean[k] = JSON.parse(JSON.stringify(v)); } catch { clean[k] = '[Object]'; }
        } else clean[k] = v;
      }
      return clean;
    },
    getParents: (fiber, max = 10) => {
      const chain = [];
      let cur = fiber?.return, d = 0;
      while (cur && d < max) {
        if (ReactUtils.isComponent(cur)) chain.push(ReactUtils.getName(cur));
        cur = cur.return; d++;
      }
      return chain;
    }
  };

  const panel = document.createElement('div');
  panel.id = 'react-navigator-panel';
  panel.innerHTML = `
    <style>
      #react-navigator-panel { position: fixed; top: 10px; right: 10px; width: 380px; max-height: 80vh; background: #1a1a2e; border: 2px solid #4a9eff; border-radius: 12px; font-family: 'SF Mono', Monaco, monospace; font-size: 12px; color: #e0e0e0; z-index: 999999; box-shadow: 0 8px 32px rgba(0,0,0,0.4); overflow: hidden; }
      #react-navigator-panel * { box-sizing: border-box; }
      .rn-header { background: linear-gradient(135deg, #4a9eff, #7c3aed); padding: 12px 16px; display: flex; justify-content: space-between; align-items: center; cursor: move; }
      .rn-title { font-weight: bold; font-size: 14px; color: white; }
      .rn-controls { display: flex; gap: 8px; }
      .rn-btn { background: rgba(255,255,255,0.2); border: none; color: white; padding: 4px 12px; border-radius: 6px; cursor: pointer; font-size: 11px; }
      .rn-btn:hover { background: rgba(255,255,255,0.3); }
      .rn-btn.active { background: #22c55e; }
      .rn-body { padding: 12px; max-height: 60vh; overflow-y: auto; }
      .rn-section { margin-bottom: 12px; }
      .rn-label { color: #4a9eff; font-size: 10px; text-transform: uppercase; margin-bottom: 6px; letter-spacing: 1px; }
      .rn-component-name { font-size: 16px; color: #22c55e; font-weight: bold; padding: 8px; background: rgba(34, 197, 94, 0.1); border-radius: 6px; margin-bottom: 8px; }
      .rn-props { background: #0d0d1a; padding: 10px; border-radius: 6px; overflow-x: auto; }
      .rn-props pre { margin: 0; white-space: pre-wrap; word-break: break-all; color: #fbbf24; }
      .rn-parents { display: flex; flex-wrap: wrap; gap: 4px; }
      .rn-parent { background: #374151; padding: 2px 8px; border-radius: 4px; font-size: 11px; }
      .rn-tree { max-height: 200px; overflow-y: auto; background: #0d0d1a; border-radius: 6px; padding: 8px; }
      .rn-tree-item { padding: 4px 8px; cursor: pointer; border-radius: 4px; display: flex; align-items: center; gap: 6px; }
      .rn-tree-item:hover { background: rgba(74, 158, 255, 0.2); }
      .rn-tree-item.selected { background: rgba(34, 197, 94, 0.3); }
      .rn-count { background: #4a9eff; color: white; padding: 1px 6px; border-radius: 10px; font-size: 10px; }
      .rn-highlight { position: fixed; border: 2px solid #4a9eff; background: rgba(74, 158, 255, 0.1); pointer-events: none; z-index: 999998; }
      .rn-tooltip { position: fixed; background: #1a1a2e; border: 1px solid #4a9eff; padding: 6px 10px; border-radius: 6px; color: #22c55e; font-size: 12px; pointer-events: none; z-index: 999999; font-family: 'SF Mono', Monaco, monospace; }
      .rn-extract-btn { width: 100%; background: linear-gradient(135deg, #22c55e, #16a34a); border: none; color: white; padding: 10px; border-radius: 6px; cursor: pointer; font-weight: bold; margin-top: 8px; }
    </style>
    <div class="rn-header">
      <span class="rn-title">React Navigator</span>
      <div class="rn-controls">
        <button class="rn-btn" id="rn-inspect-btn">Inspect</button>
        <button class="rn-btn" id="rn-minimize-btn">_</button>
        <button class="rn-btn" id="rn-close-btn">X</button>
      </div>
    </div>
    <div class="rn-body">
      <div class="rn-section">
        <div class="rn-label">Component Tree (click to select)</div>
        <div class="rn-tree" id="rn-tree"></div>
      </div>
      <div class="rn-section" id="rn-selected-section" style="display:none;">
        <div class="rn-label">Selected Component</div>
        <div class="rn-component-name" id="rn-component-name"></div>
        <div class="rn-label">Props</div>
        <div class="rn-props"><pre id="rn-props"></pre></div>
        <div class="rn-label" style="margin-top:8px;">Parent Chain</div>
        <div class="rn-parents" id="rn-parents"></div>
        <button class="rn-extract-btn" id="rn-extract-btn">Copy Component Data</button>
      </div>
    </div>
  `;
  document.body.appendChild(panel);

  const highlight = document.createElement('div');
  highlight.className = 'rn-highlight';
  highlight.style.display = 'none';
  document.body.appendChild(highlight);

  const tooltip = document.createElement('div');
  tooltip.className = 'rn-tooltip';
  tooltip.style.display = 'none';
  document.body.appendChild(tooltip);

  let inspecting = false, selectedFiber = null, componentMap = new Map();

  function buildTree() {
    componentMap.clear();
    const seenFibers = new WeakSet();
    document.querySelectorAll('*').forEach(el => {
      const key = ReactUtils.findFiberKey(el);
      if (!key) return;
      let fiber = el[key];
      while (fiber) {
        if (ReactUtils.isComponent(fiber) && !seenFibers.has(fiber)) {
          seenFibers.add(fiber);
          const name = ReactUtils.getName(fiber), type = fiber.type;
          if (!componentMap.has(type)) componentMap.set(type, { name, instances: [] });
          componentMap.get(type).instances.push({ fiber, dom: ReactUtils.findDOM(fiber) });
        }
        fiber = fiber.return;
      }
    });
    const tree = document.getElementById('rn-tree');
    tree.innerHTML = '';
    Array.from(componentMap.entries())
      .map(([type, data]) => ({ type, ...data }))
      .filter(c => c.name && c.name.length > 1)
      .sort((a, b) => b.instances.length - a.instances.length)
      .forEach(comp => {
        const item = document.createElement('div');
        item.className = 'rn-tree-item';
        item.innerHTML = `<span>${comp.name}</span><span class="rn-count">${comp.instances.length}</span>`;
        item.onclick = (e) => selectComponent(comp.type, comp, e);
        item.onmouseenter = () => highlightInstances(comp.instances);
        item.onmouseleave = () => hideHighlight();
        tree.appendChild(item);
      });
  }

  function highlightInstances(instances) {
    const dom = instances[0]?.dom;
    if (!dom) return;
    const rect = dom.getBoundingClientRect();
    Object.assign(highlight.style, { display: 'block', left: rect.left + 'px', top: rect.top + 'px', width: rect.width + 'px', height: rect.height + 'px' });
  }

  function hideHighlight() { highlight.style.display = 'none'; tooltip.style.display = 'none'; }

  function selectComponent(type, comp, e) {
    selectedFiber = comp.instances[0]?.fiber;
    if (!selectedFiber) return;
    document.querySelectorAll('.rn-tree-item').forEach(el => el.classList.remove('selected'));
    e.currentTarget.classList.add('selected');
    document.getElementById('rn-selected-section').style.display = 'block';
    document.getElementById('rn-component-name').textContent = comp.name;
    document.getElementById('rn-props').textContent = JSON.stringify(ReactUtils.cleanProps(selectedFiber.memoizedProps), null, 2);
    document.getElementById('rn-parents').innerHTML = ReactUtils.getParents(selectedFiber).map(p => `<span class="rn-parent">${p}</span>`).join('');
    highlightInstances(comp.instances);
  }

  function handleMouseMove(e) {
    if (!inspecting) return;
    const el = document.elementFromPoint(e.clientX, e.clientY);
    if (!el || el.closest('#react-navigator-panel')) return;
    const key = ReactUtils.findFiberKey(el);
    if (!key) return;
    let fiber = el[key];
    while (fiber && !ReactUtils.isComponent(fiber)) fiber = fiber.return;
    if (!fiber) return;
    const dom = ReactUtils.findDOM(fiber) || el;
    const rect = dom.getBoundingClientRect();
    Object.assign(highlight.style, { display: 'block', left: rect.left + 'px', top: rect.top + 'px', width: rect.width + 'px', height: rect.height + 'px' });
    Object.assign(tooltip.style, { display: 'block', left: (e.clientX + 15) + 'px', top: (e.clientY + 15) + 'px' });
    tooltip.textContent = ReactUtils.getName(fiber);
  }

  function handleClick(e) {
    if (!inspecting) return;
    e.preventDefault(); e.stopPropagation();
    const el = document.elementFromPoint(e.clientX, e.clientY);
    if (!el || el.closest('#react-navigator-panel')) return;
    const key = ReactUtils.findFiberKey(el);
    if (!key) return;
    let fiber = el[key];
    while (fiber && !ReactUtils.isComponent(fiber)) fiber = fiber.return;
    if (!fiber) return;
    selectedFiber = fiber;
    document.getElementById('rn-selected-section').style.display = 'block';
    document.getElementById('rn-component-name').textContent = ReactUtils.getName(fiber);
    document.getElementById('rn-props').textContent = JSON.stringify(ReactUtils.cleanProps(fiber.memoizedProps), null, 2);
    document.getElementById('rn-parents').innerHTML = ReactUtils.getParents(fiber).map(p => `<span class="rn-parent">${p}</span>`).join('');
    toggleInspect();
  }

  function toggleInspect() {
    inspecting = !inspecting;
    const btn = document.getElementById('rn-inspect-btn');
    btn.classList.toggle('active', inspecting);
    btn.textContent = inspecting ? 'Stop' : 'Inspect';
    if (!inspecting) hideHighlight();
  }

  document.getElementById('rn-inspect-btn').onclick = toggleInspect;
  document.getElementById('rn-close-btn').onclick = () => window.ReactNavigator.destroy();
  document.getElementById('rn-minimize-btn').onclick = () => {
    const body = panel.querySelector('.rn-body');
    body.style.display = body.style.display === 'none' ? 'block' : 'none';
  };
  document.getElementById('rn-extract-btn').onclick = () => {
    if (!selectedFiber) return;
    const data = {
      name: ReactUtils.getName(selectedFiber),
      props: ReactUtils.cleanProps(selectedFiber.memoizedProps),
      parents: ReactUtils.getParents(selectedFiber),
      source: typeof selectedFiber.type === 'function' ? selectedFiber.type.toString() : null,
      html: ReactUtils.findDOM(selectedFiber)?.outerHTML || null
    };
    navigator.clipboard.writeText(JSON.stringify(data, null, 2));
    alert('Component data copied to clipboard!');
  };

  document.addEventListener('mousemove', handleMouseMove, true);
  document.addEventListener('click', handleClick, true);

  let isDragging = false, startX, startY, startLeft, startTop;
  panel.querySelector('.rn-header').onmousedown = (e) => {
    if (e.target.tagName === 'BUTTON') return;
    isDragging = true; startX = e.clientX; startY = e.clientY;
    startLeft = panel.offsetLeft; startTop = panel.offsetTop;
  };
  document.onmousemove = (e) => {
    if (!isDragging) return;
    panel.style.left = (startLeft + e.clientX - startX) + 'px';
    panel.style.top = (startTop + e.clientY - startY) + 'px';
    panel.style.right = 'auto';
  };
  document.onmouseup = () => isDragging = false;

  buildTree();

  window.ReactNavigator = {
    refresh: buildTree,
    getSelected: () => selectedFiber ? {
      name: ReactUtils.getName(selectedFiber),
      props: ReactUtils.cleanProps(selectedFiber.memoizedProps),
      source: typeof selectedFiber.type === 'function' ? selectedFiber.type.toString() : null
    } : null,
    destroy: () => {
      panel.remove(); highlight.remove(); tooltip.remove();
      document.removeEventListener('mousemove', handleMouseMove, true);
      document.removeEventListener('click', handleClick, true);
      delete window.ReactNavigator;
    }
  };
})();
```

After injection:
- `ReactNavigator.refresh()` - Rebuild component tree
- `ReactNavigator.getSelected()` - Get currently selected component data
- `ReactNavigator.destroy()` - Remove navigator from page

## Tips

- Look for components with meaningful names first
- Extract multiple instances to understand prop variations
- Check parent chain to understand component composition
- Use `getBySelector` to target specific visible elements
- Use the Visual Navigator for easier component discovery

---
> Source: [dennisonbertram/steal-react-component](https://github.com/dennisonbertram/steal-react-component) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
