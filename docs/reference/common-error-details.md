---
layout: ../../layouts/content.astro
title: Common Error Details
description: How to troubleshoot common issues and error messagesm, plus our resources for getting help.
---

This page details several common issues and error messages. For further help we have an active [GitHub Discussion forum](https://github.com/withastro/snowpack/discussions)and [Discord](https://discord.gg/snowpack). Developers and community contributors frequently answer questions on both.

### No such file or directory

```
ENOENT: no such file or directory, open …/node_modules/csstype/index.js
```

This error message would sometimes occur in older versions of Snowpack.

**To solve this issue:** Upgrade to Snowpack `v2.6.0` or higher. If you continue to see this unexpected error in newer versions of Snowpack, please file an issue.

### Package exists but package.json "exports" does not include entry

Node.js recently added support for a package.json "exports" entry that defines which files you can and cannot import from within a package. Preact, for example, defines an "exports" map that allows you to to import "preact/hooks" but not "preact/some/custom/file-path.js". This allows packages to control their "public" interface.

If you see this error message, that means that you've imported a file path not allowed in the export map. If you believe this to be an error, reach out to the package author to request the file be added to their export map.

### Uncaught SyntaxError: The requested module './XXXXXX.js' does not provide an export named 'YYYYYY'

If you are using TypeScript, this error could occur if you are importing or exporting something that only exists in TypeScript (like a type or interface) and doesn't actually exist in the final JavaScript code.

Our built-in TypeScript support can detect type-only imports and will attempt to remove them automatically. This is however much more difficult for type-only export statements, because Snowpack cannot detect that an exported symbol is a type without keeping context across multiple files. For example, a statement such as `export { MyInterfaceName }` will not work in Snowpack.

**To solve:** Enable [`isolatedModules`](https://www.typescriptlang.org/tsconfig#isolatedModules) in `tsconfig.json` to identify problematic cases. Use `import type { MyInterfaceName }` and `export type { MyInterfaceName }` to help Snowpack ignore types.

This error could also appear if named imports are used with older, Common.js npm packages. Thanks to improvements in our package scanner this is no longer a common issue for most packages. However, some packages are written or compiled in a way that makes automatic import scanning impossible.

**To solve:** Use the default import (`import pkg from 'my-old-package'`) for legacy Common.js/UMD packages that cannot be analyzed. Or, add the package name to your `packageOptions.namedExports` configuration for runtime import scanning.

```js
// snowpack.config.mjs
export default {
  packageOptions: {
    namedExports: ['@shopify/polaris-tokens'],
  },
};
```

### Installing Non-JS Packages

When installing packages from npm, you may encounter some file formats that can run only with additional parsing/processing. First check to see if there is a [Snowpack plugin for the type of file](/plugins).

Because our internal installer is powered by Rollup, you can also add Rollup plugins to your [Snowpack config](/reference/configuration) to handle these special, rare files:

```diff
  // snowpack.config.mjs
  export default {
+   rollup: {
+     plugins: [require('rollup-plugin-sass')()],
+   },
  };
```

Refer to [Rollup’s documentation on plugins](https://rollupjs.org/guide/en/#using-plugins) for more information.

### RangeError: Invalid WebSocket frame: RSV1 must be clear

**To solve this issue:** Use any other port than `8080` for the dev server. To do so, specify a port in your [Snowpack config](/reference/configuration):

```diff
  // snowpack.config.mjs
  export default {
+   devOptions: {
+     port: 3000,
+   },
  };
```

### Package "[name]" not found. Have you installed it?

This warning appears when Snowpack believes something to be in `node_modules`, but can’t find it. This typically happens because you‘ve tried to import something without a leading `/`, `./`, or `../`.

Here are some possible fixes:

#### I’m trying to import a file from npm

If you were trying to import an npm package, try running the following:

```
npm install [package].
```

Then re-running Snowpack. If the issue still persists, try telling Snowpack where to find this with [an alias](https://www.snowpack.dev/reference/configuration#alias):

```diff
  // snowpack.config.mjs
  export default {
+   alias: {
+     myPackage: './path/to/myPackage',
+   },
  };
```

#### I’m trying to import a local `.js` file

If you‘re getting this error while trying to import a local file, the fix usually looks like this:

```diff
- import myFile from 'myFile.js';
+ import myFile from './myFile.js';
```

If the issue still persists, [please open an issue](https://github.com/withastro/snowpack/issues/new/choose).

#### I’m trying to import a local `.css` file

The fix for `.css` files is similar to JS. Prefix a `./` to the import path like so:

```diff
- @import "myfile.css";
+ @import "./myfile.css";
```

While it’s true you may be used to writing CSS without the `./`, and a browser respects it, Snowpack works a little differently. Because we let you import from npm seamlessly, `myfile.css` will be resolved as an npm package whereas `/myfile.css`, `./myfile.css`, and `../myfile.css` will be resolved as local project files.

Also, `./myfile.css` is perfectly-valid, and it’s good to get in the habit of using it consistently to prevent ambiguous resolution.
