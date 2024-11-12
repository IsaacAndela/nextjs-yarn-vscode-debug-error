# NextJS 15 debugging error in VSCode with Yarn PnP

When running NextJS 15 in dev mode in the VSCode debugger via Yarn with the PnP nodeLinker an error will occur on startup. NextJS 14 worked fine.

When using NPM instead of Yarn as the package manager the error does not occur.

## The Error

Note: Both the `.pnp.cjs` and the `bootloader.js` paths are within the same `'` quote pair.

```
node:internal/modules/cjs/loader:1252
  throw err;

Error: Cannot find module '/Users/me/nextjs-yarn-vscode-debug-error/.pnp.cjs /Users/me/Applications/Visual Studio Code.app/Contents/Resources/app/extensions/ms-vscode.js-debug/src/bootloader.js'
Require stack:
- internal/preload
    at Function._resolveFilename (node:internal/modules/cjs/loader:1249:15)
    at Function._load (node:internal/modules/cjs/loader:1075:27)
    at TracingChannel.traceSync (node:diagnostics_channel:315:14)
    at wrapModuleLoad (node:internal/modules/cjs/loader:218:24)
    at Module.require (node:internal/modules/cjs/loader:1340:12)
    at node:internal/modules/cjs/loader:1824:12
    at loadPreloadModules (node:internal/process/pre_execution:729:5)
    at setupUserModules (node:internal/process/pre_execution:207:5)
    at prepareExecution (node:internal/process/pre_execution:160:5)
    at prepareMainThreadExecution (node:internal/process/pre_execution:55:10) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [ 'internal/preload' ]
}
```

## Reproducing the error

The steps below are the steps taken to create this repo. You can reproduce it for yourself if you want to.

### 1. Install NextJS with Yarn

```bash
    yarn dlx create-next-app@canary
```

During install only answer yes to using the app router.

This should result in `.pnp.cjs` and `.pnp.loader.mjs` files and a `.yarn` directory in the root of the project in addition to all the normal NextJS files..

### 2. Add launch.json

Create [`.vscode/launch.json`](./.vscode/launch.json) with the following content:

```jsonc
{
	"version": "0.2.0",
	"configurations": [
		{
			// Basicly the same server side debugging configuration as the official documentation:
			// https://nextjs.org/docs/app/building-your-application/configuring/debugging#debugging-with-vs-code
			"name": "Type: Node Terminal",
			"type": "node-terminal",
			"request": "launch",
			"command": "yarn run dev"
		},
		{
			// An alternative way to configure the debugging.
			// This doesn't work either with NextJS 15.
			"name": "Type: Node",
			"request": "launch",
			"runtimeExecutable": "yarn",
			"runtimeArgs": ["run", "dev"],
			"type": "node",
			// This is only here to make debugging easier
			// The error can also be reproduced without the
			// integratedTerminal
			"console": "integratedTerminal"
		}
	]
}
```

### 3. Try running next dev from the terminal

Running `yarn next dev` from the terminal should work fine.

### 4. Try launching debugger in VSCode

Run the command from the Command Palette (`cmd+shift+p` or `ctrl-shift-p`):

```
Debug: Select and Start Debugging
```

Pick either of the commands from the command pallet. **The above mentioned error should appear.**

## Other things I've tried

- Installing [Yarn SDK for VS Code](https://yarnpkg.com/getting-started/editor-sdks#vscode) makes no difference.
- Setting `"type"` to `"module"` or `"commonjs"` in `package.json` makes no difference.
- When using `node-modules` as the `nodeLinker` option in `.yarnrc.yml` the error does not occur. This however negates most of the benefits of Yarn and is not the canonical way to use Yarn.
