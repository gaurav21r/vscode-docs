---
Order: 8
Area: extensions
TOCTitle: Running and Debugging Extensions
PageTitle: Running and Debugging your Visual Studio Code Extension
DateApproved: 2/3/2016
MetaDescription: It is easy to debug and test your Visual Studio Code extension (plug-in).  The Yo Code extension generator scaffolds the necessary settings to run and debug your extension directly in Visual Studio Code.
---

# Running and Debugging Your Extension

You can use VS Code to develop an extension for VS Code and VS Code provides several tools that simplify extension development:
* Yeoman generators to scaffold an extension
* IntelliSense, hover, and code navigation for the extension API
* Compiling TypeScript (when implementing an extension in TypeScript)
* Running and debugging an extension
* Publishing an extension

## Creating an Extension

We suggest you start your extension by scaffolding out the basic files. You can use the `yo code` Yeoman generator to do this and we cover the details in the [Yo Code document](/docs/tools/yocode.md).  The generator will ensure everything is set up so you have a great development experience.

## Running and Debugging your Extension

You can easily run your extension under the debugger by pressing `F5`. This opens a new VS Code window with your extension loaded. Output from your extension shows up in the `Debug Console`. You can set break points, step through your code, and inspect variables either in the `Debug` view or the `Debug Console`.

![Debug](images/debugging-extensions/debug.png)

Let's peek at what is going on behind the scenes. If you are writing your extension in TypeScript then your code must first be compiled to JavaScript.

## Compiling TypeScript

The TypeScript compilation is setup as follows in the generated extension:
* A `tsconfig.json` defines the compile options for the TypeScript compiler. Read more about it at the [TypeScript wiki](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json) or in our [TypeScript Language Section](/docs/languages/typescript.md#tsconfigjson).
* A TypeScript compiler with the proper version is included inside the node_modules folder.
* A `typings/vscode-typings.d.ts`: instructs the TypeScript compiler to include the `vscode` API definition.
* The API definition is included in `node_modules/vscode`.

The TypeScript compilation is triggered before running your extension. This is done with the `preLaunchTask` attribute defined in the
`.vscode/launch.json` file which declares a task to be executed before starting the debugging session. The task is defined inside the `.vscode/tasks.json` file.

> **Note:** The TypeScript compiler is started in watch mode, so that it compiles the files as you make changes.

## Launching your Extension

Your extension is launched in a new window with the title `Extension Development Host`. This window runs VS Code or more
precisely the `Extension Host` with your extension under development.

You can accomplish the same from the command line using the `extensionDevelopmentPath` option. This options tells VS Code in what
other locations it should look for extensions, e.g.,

>`code --extensionDevelopmentPath=_my_extension_folder`.

Once the Extension Host is launched, VS Code attaches the debugger to it and starts the debug session.

This is what happens when pressing `F5`:
 1. `.vscode/launch.json` instructs to first run a task named `npm`.
 2. `.vscode/tasks.json` defines the task `npm` as a shell command to `npm run compile`.
 3. `package.json` defines the script `compile` as `node ./node_modules/vscode/bin/compile -watch -p ./`
 4. This eventually invokes the TypeScript compiler included in node_modules, which generates `out/src/extension.js` and `out/src/extension.js.map`.
 5. Once the TypeScript compilation task is finished, the `code --extensionDevelopmentPath=${workspaceRoot}` process is spawned.
 6. The second instance of VS Code is launched in a special mode and it searches for an extension at `${workspaceRoot}`.

## Changing and Reloading your Extension

Since the TypeScript compiler is run in watch mode, the TypeScript files are automatically compiled as you make changes. You can observe
the compilation progress on the left side of the VS Code status bar. On the status bar you can also see the error and warning counts of a
compilation. When the compilation is complete with no errors, you must reload the `Extension Development Host` so that it picks up
your changes. You have two options to do this:

* Click on the debug restart action to relaunch the Extension Development Host window.
* Press `kbstyle(Ctrl+R)` (Mac: `kbstyle(Cmd+R)`) in the Extension Development Host window.

## Next Steps

* [Testing your Extension](/docs/extensions/testing-extensions.md) - Learn how to write unit and integration tests for your extension
* [Publishing Tool](/docs/tools/vscecli.md) - Publish your extension with the vsce command line tool.
* [Extension Manifest file](/docs/extensionAPI/extension-manifest.md) - VS Code extension manifest file reference
* [Extension API](/docs/extensionAPI/overview.md) - Learn about the VS Code extensibility APIs

## Common Questions

**Q: How can I use API from my extension that was introduced in a newer release of VS Code?**

**A:** If your extension is using API that was introduced in a newer release of VS Code, you have to declare this dependency from the
`engines` field in the `package.json` file of the extension. After that, run `npm install` from the root of your extension and
the related API file for the version specified will be downloaded and is ready to use for developing your extension.
