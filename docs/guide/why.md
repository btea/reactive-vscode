---
outline: 'deep'
---

# Why reactive-vscode

## The Problems

Developing a VSCode extension is not easy. The official APIs are kind of primitive, which has several problems:

### Hard to watch states

The official API is event-based, which means you have to listen to events to watch the state. This produces a lot of redundant code, and not familiar to Vue developers.

### The Disposables

Disposables are everywhere in a VSCode extension. You have to store all of them to `context.subscriptions`, or dispose them manually.

### When to Initialize

Views in a VSCode extension are created lazily. If you want to access a view instance, you have to store it, and even listen to a event which is fired when the view is created.

## The solution

[Vue's Reactivity API](https://vuejs.org/api/reactivity-core.html) is all you need. This library wraps most of the VSCode APIs into [Vue Composables](https://vuejs.org/guide/reusability/composables.html). You can use them as you use Vue 3 Composition API, which is familiar to Vue developers.

With the help of this library, you can develop a VSCode extension just like developing a Vue 3 web application. You can use Vue's reactivity system to watch states, and implement views as Vue composables.

### Result

Here is an example which shows how this library can help you develop a VSCode extension. The following extension decorates the active text editor depending on a configuration.

**Before:**

```ts
import { ExtensionContext, window, workspace } from 'vscode'

const decorationType = window.createTextEditorDecorationType({})

function updateDecorations(enabled: boolean) {
  window.activeTextEditor?.setDecorations(decorationType, enabled ? [/* ... */] : [])
}

export function activate(context: ExtensionContext) {
  const configurations = workspace.getConfiguration('demo')
  let decorationsEnabled = configurations.get<boolean>('decorations')

  context.subscriptions.push(workspace.onDidChangeConfiguration((e) => {
    if (e.affectsConfiguration('demo.decorations')) {
      decorationsEnabled = configurations.get<boolean>('decorations')
      updateDecorations(decorationsEnabled)
    }
  }))
  context.subscriptions.push(window.onDidChangeActiveTextEditor(updateDecorations))

  updateDecorations(decorationsEnabled)
}
```

**After:**

```ts
import { defineConfigs, defineExtension, useActiveTextEditor, watchEffect } from 'reactive-vscode'

const decorationType = window.createTextEditorDecorationType({})

const configs = defineConfigs('demo', { decorations: Boolean })

export = defineExtension(() => {
  const editor = useActiveTextEditor()

  watchEffect(() => {
    editor.value?.setDecorations(configs.decorations ? [/* ... */] : [])
  })
})
```

As you can see, after using `reactive-vscode`, the code is much cleaner and easier to understand. You can use Vue's reactivity API like `watchEffect` to watch states, and use Vue composables like `useActiveTextEditor` to access the active text editor.

## FAQ

### Why use `@vue/runtime-core`?

This library is built on top of `@vue/runtime-core`. This is because `@vue/reactivity` doesn't provide the `watch` and `watchEffect` function, which is essential in a VSCode extension.

It's possible to use [`@vue-reactivity/watch`](https://github.com/vue-reactivity/watch) in the future. But for now, we have to use `@vue/runtime-core`.

### Use Vue in Webview?

This library is **not** designed for using Vue in a webview. If you want to use Vue in a webview, you can use the CDN version of Vue or bundler plugins like [`@tomjs/vite-plugin-vscode`](https://github.com/tomjs/vite-plugin-vscode).