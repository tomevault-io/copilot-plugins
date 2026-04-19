## unnamed-skill

> * this project is a drag and drop code editor

* this project is a drag and drop code editor
* this project uses solidjs and typescript. type safety is of the utmost importance
* there's going to be a canvas where you can drag and drop code blocks
* each code block is required to either execute an effect, or return a value
* the code blocks are reactive
* when some value changes, every single code block that depends on said value also is recomputed
* you can have target blocks which can have either like a table or a graph or just a html renderer (for now)
* i think there should be a reactivity class that is responsible for managing the reactivity of the code blocks 
* for the code blocks, we should use monaco
* for drag and drop, for now, let us use https://github.com/thisbeyond/solid-dnd (please index that)
* down the line, might use the webcontainers api to let us have any library in this editor
# here is an example of using monaco with solidjs

```typescript
import { createSignal, createEffect, onCleanup, JSX, onMount, mergeProps, on } from 'solid-js'
import * as monacoEditor from 'monaco-editor'
import loader, { Monaco } from '@monaco-editor/loader'
import { Loader } from './Loader'
import { MonacoContainer } from './MonacoContainer'
import { getOrCreateModel } from './utils'
import { LoaderParams } from './types'

const viewStates = new Map()

export interface MonacoEditorProps {
  language?: string
  value?: string
  loadingState?: JSX.Element
  class?: string
  theme?: monacoEditor.editor.BuiltinTheme | string
  path?: string
  overrideServices?: monacoEditor.editor.IEditorOverrideServices
  width?: string
  height?: string
  options?: monacoEditor.editor.IStandaloneEditorConstructionOptions
  saveViewState?: boolean
  loaderParams?: LoaderParams
  onChange?: (value: string, event: monacoEditor.editor.IModelContentChangedEvent) => void
  onMount?: (monaco: Monaco, editor: monacoEditor.editor.IStandaloneCodeEditor) => void
  onBeforeUnmount?: (monaco: Monaco, editor: monacoEditor.editor.IStandaloneCodeEditor) => void
}

export const MonacoEditor = (inputProps: MonacoEditorProps) => {
  const props = mergeProps(
    {
      theme: 'vs',
      width: '100%',
      height: '100%',
      loadingState: 'Loading…',
      saveViewState: true,
    },
    inputProps,
  )

  let containerRef: HTMLDivElement

  const [monaco, setMonaco] = createSignal<Monaco>()
  const [editor, setEditor] = createSignal<monacoEditor.editor.IStandaloneCodeEditor>()

  let abortInitialization: (() => void) | undefined
  let monacoOnChangeSubscription: any
  let isOnChangeSuppressed = false

  onMount(async () => {
    loader.config(inputProps.loaderParams ?? { monaco: monacoEditor })
    const loadMonaco = loader.init()

    abortInitialization = () => loadMonaco.cancel()

    try {
      const monaco = await loadMonaco
      const editor = createEditor(monaco)
      setMonaco(monaco)
      setEditor(editor)
      props.onMount?.(monaco, editor)

      monacoOnChangeSubscription = editor.onDidChangeModelContent(event => {
        if (!isOnChangeSuppressed) {
          props.onChange?.(editor.getValue(), event)
        }
      })
    } catch (error: any) {
      if (error?.type === 'cancelation') {
        return
      }

      console.error('Could not initialize Monaco', error)
    }
  })

  onCleanup(() => {
    const _editor = editor()
    if (!_editor) {
      abortInitialization?.()
      return
    }

    props.onBeforeUnmount?.(monaco()!, _editor)
    monacoOnChangeSubscription?.dispose()
    _editor.getModel()?.dispose()
    _editor.dispose()
  })

  createEffect(
    on(
      () => props.value,
      value => {
        const _editor = editor()
        if (!_editor || typeof value === 'undefined') {
          return
        }

        if (_editor.getOption(monaco()!.editor.EditorOption.readOnly)) {
          _editor.setValue(value)
          return
        }

        if (value !== _editor.getValue()) {
          isOnChangeSuppressed = true

          _editor.executeEdits('', [
            {
              range: _editor.getModel()!.getFullModelRange(),
              text: value,
              forceMoveMarkers: true,
            },
          ])

          _editor.pushUndoStop()
          isOnChangeSuppressed = false
        }
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.options,
      options => {
        editor()?.updateOptions(options ?? {})
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.theme,
      theme => {
        monaco()?.editor.setTheme(theme)
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.language,
      language => {
        const model = editor()?.getModel()
        if (!language || !model) {
          return
        }

        monaco()?.editor.setModelLanguage(model, language)
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.path,
      (path, prevPath) => {
        const _monaco = monaco()
        if (!_monaco) {
          return
        }

        const model = getOrCreateModel(_monaco, props.value ?? '', props.language, path)

        if (model !== editor()?.getModel()) {
          if (props.saveViewState) {
            viewStates.set(prevPath, editor()?.saveViewState())
          }
          editor()?.setModel(model)
          if (props.saveViewState) {
            editor()?.restoreViewState(viewStates.get(path))
          }
        }
      },
      { defer: true },
    ),
  )

  const createEditor = (monaco: Monaco) => {
    const model = getOrCreateModel(monaco, props.value ?? '', props.language, props.path)

    return monaco.editor.create(
      containerRef,
      {
        model: model,
        automaticLayout: true,
        ...props.options,
      },
      props.overrideServices,
    )
  }

  return (
    <MonacoContainer class={props.class} width={props.width} height={props.height}>
      {!editor() && <Loader>{props.loadingState}</Loader>}
      <div style={{ width: '100%' }} ref={containerRef!} />
    </MonacoContainer>
  )
}

```

```typescript
import { createSignal, createEffect, onCleanup, JSX, onMount, mergeProps, on } from 'solid-js'
import * as monacoEditor from 'monaco-editor'
import loader, { Monaco } from '@monaco-editor/loader'
import { Loader } from './Loader'
import { MonacoContainer } from './MonacoContainer'
import { getOrCreateModel } from './utils'
import { LoaderParams } from './types'

const viewStates = new Map()

export interface MonacoDiffEditorProps {
  original?: string
  modified?: string

  originalLanguage?: string
  modifiedLanguage?: string

  originalPath?: string
  modifiedPath?: string

  loadingState?: JSX.Element
  class?: string
  theme?: monacoEditor.editor.BuiltinTheme | string
  overrideServices?: monacoEditor.editor.IEditorOverrideServices
  width?: string
  height?: string
  options?: monacoEditor.editor.IStandaloneEditorConstructionOptions
  saveViewState?: boolean
  loaderParams?: LoaderParams
  onChange?: (value: string) => void
  onMount?: (monaco: Monaco, editor: monacoEditor.editor.IStandaloneDiffEditor) => void
  onBeforeUnmount?: (monaco: Monaco, editor: monacoEditor.editor.IStandaloneDiffEditor) => void
}

export const MonacoDiffEditor = (inputProps: MonacoDiffEditorProps) => {
  const props = mergeProps(
    {
      theme: 'vs',
      width: '100%',
      height: '100%',
      loadingState: 'Loading…',
      saveViewState: true,
    },
    inputProps,
  )

  let containerRef: HTMLDivElement

  const [monaco, setMonaco] = createSignal<Monaco>()
  const [editor, setEditor] = createSignal<monacoEditor.editor.IStandaloneDiffEditor>()

  let abortInitialization: (() => void) | undefined
  let monacoOnChangeSubscription: any
  let isOnChangeSuppressed = false

  onMount(async () => {
    loader.config(inputProps.loaderParams ?? { monaco: monacoEditor })
    const loadMonaco = loader.init()

    abortInitialization = () => loadMonaco.cancel()

    try {
      const monaco = await loadMonaco
      const editor = createEditor(monaco)
      setMonaco(monaco)
      setEditor(editor)
      props.onMount?.(monaco, editor)

      monacoOnChangeSubscription = editor.onDidUpdateDiff(() => {
        if (!isOnChangeSuppressed) {
          props.onChange?.(editor.getModifiedEditor().getValue())
        }
      })
    } catch (error: any) {
      if (error?.type === 'cancelation') {
        return
      }

      console.error('Could not initialize Monaco', error)
    }
  })

  onCleanup(() => {
    const _editor = editor()
    if (!_editor) {
      abortInitialization?.()
      return
    }

    props.onBeforeUnmount?.(monaco()!, _editor)
    monacoOnChangeSubscription?.dispose()
    _editor.getModel()?.original.dispose()
    _editor.getModel()?.modified.dispose()
    _editor.dispose()
  })

  createEffect(
    on(
      () => props.modified,
      modified => {
        const _editor = editor()?.getModifiedEditor()
        if (!_editor || typeof modified === 'undefined') {
          return
        }

        if (_editor.getOption(monaco()!.editor.EditorOption.readOnly)) {
          _editor.setValue(modified)
          return
        }

        if (modified !== _editor.getValue()) {
          isOnChangeSuppressed = true

          _editor.executeEdits('', [
            {
              range: _editor.getModel()!.getFullModelRange(),
              text: modified,
              forceMoveMarkers: true,
            },
          ])

          _editor.pushUndoStop()
          isOnChangeSuppressed = false
        }
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.original,
      original => {
        const _editor = editor()?.getOriginalEditor()
        if (!_editor || typeof original === 'undefined') {
          return
        }

        if (_editor.getOption(monaco()!.editor.EditorOption.readOnly)) {
          _editor.setValue(original)
        }
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.options,
      options => {
        editor()?.updateOptions(options ?? {})
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.theme,
      theme => {
        monaco()?.editor.setTheme(theme)
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.originalLanguage,
      language => {
        const model = editor()?.getModel()
        if (!language || !model) {
          return
        }

        monaco()?.editor.setModelLanguage(model.original, language)
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.modifiedLanguage,
      language => {
        const model = editor()?.getModel()
        if (!language || !model) {
          return
        }

        monaco()?.editor.setModelLanguage(model.modified, language)
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => [props.originalPath, props.modifiedPath],
      ([originalPath, modifiedPath], prevPaths) => {
        const _monaco = monaco()
        if (!_monaco || !prevPaths) {
          return
        }

        const [prevOriginalPath, prevModifiedPath] = prevPaths

        const currentModels = editor()?.getModel()
        let originalModel = currentModels?.original
        let modifiedModel = currentModels?.modified

        if (prevOriginalPath !== originalPath) {
          if (props.saveViewState && originalPath != null) {
            viewStates.set(prevOriginalPath, editor()?.getOriginalEditor().saveViewState())
          }

          originalModel = getOrCreateModel(
            _monaco,
            props.original ?? '',
            props.originalLanguage,
            originalPath,
          )
        }

        if (prevModifiedPath !== modifiedPath) {
          if (props.saveViewState && prevModifiedPath != null) {
            viewStates.set(prevModifiedPath, editor()?.getModifiedEditor().saveViewState())
          }

          modifiedModel = getOrCreateModel(
            _monaco,
            props.modified ?? '',
            props.modifiedLanguage,
            modifiedPath,
          )
        }

        editor()?.setModel({ modified: modifiedModel!, original: originalModel! })

        if (props.saveViewState) {
          editor()?.getOriginalEditor().restoreViewState(viewStates.get(originalPath))
          editor()?.getModifiedEditor().restoreViewState(viewStates.get(modifiedPath))
        }
      },
      { defer: true },
    ),
  )

  const createEditor = (monaco: Monaco) => {
    const originalModel = getOrCreateModel(
      monaco,
      props.original ?? '',
      props.originalLanguage,
      props.originalPath,
    )
    const modifiedModel = getOrCreateModel(
      monaco,
      props.modified ?? '',
      props.modifiedLanguage,
      props.modifiedPath,
    )

    const editor = monaco.editor.createDiffEditor(
      containerRef,
      {
        automaticLayout: true,
        ...props.options,
      },
      props.overrideServices,
    )

    editor.setModel({
      original: originalModel,
      modified: modifiedModel,
    })

    return editor
  }

  return (
    <MonacoContainer class={props.class} width={props.width} height={props.height}>
      {!editor() && <Loader>{props.loadingState}</Loader>}
      <div style={{ width: '100%' }} ref={containerRef!} />
    </MonacoContainer>
  )
}
```

# here are some instructions for using monaco:

Based on your decision to use Monaco instead of CodeMirror, here's how you can integrate custom intellisense for TypeScript in Monaco Editor, including support for your own global variables:

## Setting Up Monaco with Custom Intellisense

1. **Add Type Definitions**

Create a declaration file for your custom globals:

```typescript
// globals.d.ts
declare const myGlobalVar: string;
declare function myGlobalFunction(): void;
// Add more global declarations as needed
```

2. **Configure Monaco**

When initializing Monaco, add the following configurations:

```typescript
import * as monaco from 'monaco-editor';

// Add your custom type definitions
monaco.languages.typescript.javascriptDefaults.addExtraLib(`
    declare const myGlobalVar: string;
    declare function myGlobalFunction(): void;
    // Add more global declarations here
    `, 'globals.d.ts');

// Configure TypeScript compiler options
monaco.languages.typescript.javascriptDefaults.setCompilerOptions({
target: monaco.languages.typescript.ScriptTarget.ES6,
allowNonTsExtensions: true,
moduleResolution: monaco.languages.typescript.ModuleResolutionKind.NodeJs,
module: monaco.languages.typescript.ModuleKind.CommonJS,
noEmit: true,
typeRoots: ["node_modules/@types"]
});

// Create the editor
const editor = monaco.editor.create(document.getElementById('container'), {
value: '// Your initial TypeScript code here',
language: 'typescript',
// Other editor options...
});
```

3. **Custom Completion Provider (Optional)**

For more advanced scenarios, you can add a custom completion provider:

```typescript
monaco.languages.registerCompletionItemProvider('typescript', {
provideCompletionItems: (model, position) => {
const suggestions = [
{
label: 'myGlobalVar',
kind: monaco.languages.CompletionItemKind.Variable,
detail: 'Custom global variable',
insertText: 'myGlobalVar'
},
{
label: 'myGlobalFunction',
kind: monaco.languages.CompletionItemKind.Function,
detail: 'Custom global function',
insertText: 'myGlobalFunction()'
}
// Add more custom suggestions here
];

return { suggestions };
}
});
```

This setup will provide intellisense for your custom globals, including autocomplete suggestions and type information when hovering over the variables or functions in the editor[1][3][4].

## Additional Tips

* **Dynamic Updates**: If your global variables can change at runtime, you may need to update the extra libraries dynamically.
* **Performance**: For large projects, consider optimizing the completion provider to handle a large number of suggestions efficiently.
* **TypeScript Support**: Monaco provides excellent built-in support for TypeScript, so you'll get good intellisense for standard TypeScript features out of the box.

By following these steps, you'll have a Monaco editor with TypeScript support that includes your custom global variables in the intellisense suggestions, providing a rich editing experience for your users.

Citations:
[1] https://stackoverflow.com/questions/51538242/how-to-define-global-variable-in-monaco-editor-for-intellisense/51539385
[2] https://discourse.nodered.org/t/monaco-intellisense-code-completion-subflow/84581
[3] https://www.checklyhq.com/blog/customizing-monaco/
[4] https://github.com/microsoft/monaco-editor/issues/2147
[5] https://mono.software/2017/04/11/custom-intellisense-with-monaco-editor/
[6] https://microsoft.github.io/monaco-editor/
[7] https://www.npmjs.com/package/@monaco-editor/react
[8] https://stackoverflow.com/questions/69702749/how-to-use-intellisense-from-monaco-d-ts
[9] https://stackoverflow.com/questions/67505426/custom-javascript-code-completion-for-this-in-monaco-editor
[10] https://github.com/microsoft/monaco-editor/discussions/4469

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saiashirwad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
