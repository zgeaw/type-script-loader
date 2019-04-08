# TypeScript loader for webpack


This is the typescript loader for webpack.

### Examples

We have a number of example setups to accomodate different workflows. Our examples can be found [here](examples/).

We probably have more examples than we need.  That said, here's a good way to get started:

- I want the simplest setup going.  Use "[vanilla](examples/vanilla)" typescript-loader-webpack

### Faster Builds

As your project becomes bigger, compilation time increases linearly. It's because typescript's semantic checker has to inspect all files on every rebuild. The simple solution is to disable it by using the `transpileOnly: true` option, but doing so leaves you without type checking and *will not output declaration files*.

### Babel

typescript-loader-webpack works very well in combination with [babel](https://babeljs.io/) and [babel-loader](https://github.com/babel/babel-loader). There is an [example](https://github.com/Microsoft/TypeScriptSamples/tree/master/react-flux-babel-karma) of this in the official [TypeScript Samples](https://github.com/Microsoft/TypeScriptSamples). Alternatively take a look at our own [example](examples/react-babel-karma-gulp).

### Installation

```
yarn add typescript-loader-webpack --dev
```

or

```
npm install typescript-loader-webpack --save-dev
```

You will also need to install TypeScript if you have not already.

```
yarn add typescript --dev
```

or

```
npm install typescript --save-dev
```

### Running

Use webpack like normal, including `webpack --watch` and `webpack-dev-server`, or through another
build system using the [Node.js API](http://webpack.github.io/docs/node.js-api.html).

### Compatibility

* TypeScript: 3.0.1+
* webpack: 4.x+ (please use typescript-loader-webpack 3.x if you need webpack 2 or 3 support)
* node: 6.11.5 minimum (aligned with webpack 4)


If you become aware of issues not caught by the test suite then please let us know. Better yet, write a test and submit it in a PR!

### Configuration

1. Create or update `webpack.config.js` like so:

   ```javascript
   module.exports = {
     mode: "development",
     devtool: "inline-source-map",
     entry: "./app.ts",
     output: {
       filename: "bundle.js"
     },
     resolve: {
       // Add `.ts` and `.tsx` as a resolvable extension.
       extensions: [".ts", ".tsx", ".js"]
     },
     module: {
       rules: [
         // all files with a `.ts` or `.tsx` extension will be handled by `typescript-loader-webpack`
         { test: /\.tsx?$/, loader: "typescript-loader-webpack" }
       ]
     }
   };
   ```

2. Add a [`tsconfig.json`]file. (The one below is super simple; but you can tweak this to your hearts desire)

   ```json
   {
     "compilerOptions": {
       "sourceMap": true
     }
   }
   ```

The [tsconfig.json](http://www.typescriptlang.org/docs/handbook/tsconfig-json.html) file controls
TypeScript-related options so that your IDE, the `tsc` command, and this loader all share the
same options.

#### `devtool` / sourcemaps

If you want to be able to debug your original source then you can thanks to the magic of sourcemaps. There are 2 steps to getting this set up with typescript-loader-webpack and webpack.

First, for typescript-loader-webpack to produce **sourcemaps**, you will need to set the [tsconfig.json](http://www.typescriptlang.org/docs/handbook/tsconfig-json.html) option as `"sourceMap": true`.

Second, you need to set the `devtool` option in your `webpack.config.js` to support the type of sourcemaps you want. To make your choice have a read of the [`devtool` webpack docs](https://webpack.js.org/configuration/devtool/). You may be somewhat daunted by the choice available. You may also want to vary the sourcemap strategy depending on your build environment. Here are some example strategies for different environments:

* `devtool: 'inline-source-map'` - Solid sourcemap support; the best "all-rounder". Works well with karma-webpack (not all strategies do)
* `devtool: 'cheap-module-eval-source-map'` - Best support for sourcemaps whilst debugging.
* `devtool: 'source-map'` - Approach that plays well with UglifyJsPlugin; typically you might use this in Production

### Code Splitting and Loading Other Resources

Loading css and other resources is possible but you will need to make sure that
you have defined the `require` function in a [declaration file](https://www.typescriptlang.org/docs/handbook/writing-declaration-files.html).

```typescript
declare var require: {
  <T>(path: string): T;
  (paths: string[], callback: (...modules: any[]) => void): void;
  ensure: (
    paths: string[],
    callback: (require: <T>(path: string) => T) => void
  ) => void;
};
```

Then you can simply require assets or chunks per the [webpack documentation](https://webpack.js.org/guides/code-splitting/).

```javascript
require("!style!css!./style.css");
```

The same basic process is required for code splitting. In this case, you `import` modules you need but you
don't directly use them. Instead you require them at [split points](http://webpack.github.io/docs/code-splitting.html#defining-a-split-point). See [this example](test/comparison-tests/codeSplitting) and [this example](test/comparison-tests/es6codeSplitting) for more details.

[TypeScript 2.4 provides support for ECMAScript's new `import()` calls. These calls import a module and return a promise to that module.](https://blogs.msdn.microsoft.com/typescript/2017/06/12/announcing-typescript-2-4-rc/) This is also supported in webpack - details on usage can be found [here](https://webpack.js.org/guides/code-splitting-async/#dynamic-import-import-). Happy code splitting!

### Declarations (.d.ts)

To output a built .d.ts file, you can set `"declaration": true` in your tsconfig, and use the [DeclarationBundlerPlugin](https://www.npmjs.com/package/declaration-bundler-webpack-plugin) in your webpack config.

### Failing the build on TypeScript compilation error

The build **should** fail on TypeScript compilation errors as of webpack 2. If for some reason it does not, you can use the [webpack-fail-plugin](https://www.npmjs.com/package/webpack-fail-plugin).

For more background have a read of [this issue](https://github.com/zgeaw/typescript-loader-webpack/issues/108).

### `baseUrl` / `paths` module resolution

If you want to resolve modules according to `baseUrl` and `paths` in your `tsconfig.json` then you can use the [tsconfig-paths-webpack-plugin](https://www.npmjs.com/package/tsconfig-paths-webpack-plugin) package. For details about this functionality, see the [module resolution documentation](https://www.typescriptlang.org/docs/handbook/module-resolution.html#base-url).

This feature requires webpack 2.1+ and TypeScript 2.0+. Use the config below or check the [package](https://github.com/dividab/tsconfig-paths-webpack-plugin/blob/master/README.md) for more information on usage.

```javascript
const TsconfigPathsPlugin = require('tsconfig-paths-webpack-plugin');

module.exports = {
  ...
  resolve: {
    plugins: [new TsconfigPathsPlugin({ /*configFile: "./path/to/tsconfig.json" */ })]
  }
  ...
}
```

### Options

There are two types of options: TypeScript options (aka "compiler options") and loader options. TypeScript options should be set using a tsconfig.json file. Loader options can be specified through the `options` property in the webpack configuration:

```javascript
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: [
          {
            loader: 'typescript-loader-webpack',
            options: {
              transpileOnly: true
            }
          }
        ]
      }
    ]
  }
}
```

#### transpileOnly _(boolean) (default=false)_

If you want to speed up compilation significantly you can set this flag.
However, many of the benefits you get from static type checking between
different dependencies in your application will be lost.

It's advisable to use `transpileOnly` alongside the [fork-ts-checker-webpack-plugin](https://github.com/Realytics/fork-ts-checker-webpack-plugin) to get full type checking again. To see what this looks like in practice then either take a look at [our simple example](examples/fork-ts-checker-webpack-plugin). For a more complex setup take a look at our [more involved example](examples/react-babel-karma-gulp).

If you enable this option, webpack 4 will give you "export not found" warnings any time you re-export a type:

```
WARNING in ./src/bar.ts
1:0-34 "export 'IFoo' was not found in './foo'
 @ ./src/bar.ts
 @ ./src/index.ts
```

The reason this happens is that when typescript doesn't do a full type check, it does not have enough information to determine whether an imported name is a type or not, so when the name is then exported, typescript has no choice but to emit the export. Fortunately, the extraneous export should not be harmful, so you can just suppress these warnings:

```javascript
module.exports = {
  ...
  stats: {
    warningsFilter: /export .* was not found in/
  }
}
```

#### happyPackMode _(boolean) (default=false)_

If you're using [HappyPack](https://github.com/amireh/happypack) or [thread-loader](https://github.com/webpack-contrib/thread-loader) to parallise your builds then you'll need to set this to `true`. This implicitly sets `*transpileOnly*` to `true` and **WARNING!** stops registering **_all_** errors to webpack.

It's advisable to use this with the [fork-ts-checker-webpack-plugin](https://github.com/Realytics/fork-ts-checker-webpack-plugin) to get full type checking again. To see what this looks like in practice then either take a look at [our simple HappyPack example](examples/happypack) / [our simple thread-loader example](examples/thread-loader). For a more complex setup take a look at our [more involved HappyPack example](examples/react-babel-karma-gulp-happypack) / [more involved thread-loader example](examples/react-babel-karma-gulp-thread-loader). **_IMPORTANT_**: If you are using fork-ts-checker-webpack-plugin alongside HappyPack or thread-loader then ensure you set the `checkSyntacticErrors` option like so:

```javascript
        new ForkTsCheckerWebpackPlugin({ checkSyntacticErrors: true })
```

This will ensure that the plugin checks for both syntactic errors (eg `const array = [{} {}];`) and semantic errors (eg `const x: number = '1';`). By default the plugin only checks for semantic errors (as when used with typescript-loader-webpack in `transpileOnly` mode, typescript-loader-webpack will still report syntactic errors).

Also, if you are using `thread-loader` in watch mode, remember to set `poolTimeout: Infinity` so workers don't die.

#### getCustomTransformers _( (program: Program) => { before?: TransformerFactory<SourceFile>[]; after?: TransformerFactory<SourceFile>[]; } )_

Provide custom transformers - only compatible with TypeScript 2.3+ (and 2.4 if using `transpileOnly` mode). For example usage take a look at [typescript-plugin-styled-components](https://github.com/Igorbek/typescript-plugin-styled-components) or our [test](test/comparison-tests/customTransformer).

You can also pass a path string to locate a js module file which exports the function described above, this useful especially in `happyPackMode`. (Because forked processes cannot serialize functions see more at [related issue](https://github.com/Igorbek/typescript-plugin-styled-components/issues/6#issue-303387183))

#### logInfoToStdOut _(boolean) (default=false)_

This is important if you read from stdout or stderr and for proper error handling.
The default value ensures that you can read from stdout e.g. via pipes or you use webpack -j to generate json output.

#### logLevel _(string) (default=warn)_

Can be `info`, `warn` or `error` which limits the log output to the specified log level.
Beware of the fact that errors are written to stderr and everything else is written to stderr (or stdout if logInfoToStdOut is true).

#### silent _(boolean) (default=false)_

If `true`, no console.log messages will be emitted. Note that most error
messages are emitted via webpack which is not affected by this flag.

#### ignoreDiagnostics _(number[]) (default=[])_

You can squelch certain TypeScript errors by specifying an array of diagnostic
codes to ignore.

#### reportFiles _(string[]) (default=[])_

Only report errors on files matching these glob patterns.

```javascript
  // in webpack.config.js
  {
    test: /\.ts$/,
    loader: 'typescript-loader-webpack',
    options: { reportFiles: ['src/**/*.{ts,tsx}', '!src/skip.ts'] }
  }
```

This can be useful when certain types definitions have errors that are not fatal to your application.

#### compiler _(string) (default='typescript')_

Allows use of TypeScript compilers other than the official one. Should be
set to the NPM name of the compiler, eg [`ntypescript`](https://github.com/basarat/ntypescript).

#### configFile _(string) (default='tsconfig.json')_

Allows you to specify where to find the TypeScript configuration file.

You may provide

* just a file name. The loader then will search for the config file of each entry point in the respective entry point's containing folder. If a config file cannot be found there, it will travel up the parent directory chain and look for the config file in those folders.
* a relative path to the configuration file. It will be resolved relative to the respective `.ts` entry file.
* an absolute path to the configuration file.

Please note, that if the configuration file is outside of your project directory, you might need to set the `context` option to avoid TypeScript issues (like TS18003).
In this case the `configFile` should point to the `tsconfig.json` and `context` to the project root.

#### colors _(boolean) (default=true)_

If `false`, disables built-in colors in logger messages.

#### errorFormatter _((message: ErrorInfo, colors: boolean) => string) (default=undefined)_

By default typescript-loader-webpack formats TypeScript compiler output for an error or a warning in the style:

```
[tsl] ERROR in myFile.ts(3,14)
      TS4711: you did something very wrong
```

If that format is not to your taste you can supply your own formatter using the `errorFormatter` option. Below is a template for a custom error formatter. Please note that the `colors` parameter is an instance of [`chalk`](https://github.com/chalk/chalk) which you can use to color your output. (This instance will respect the `colors` option.)

```javascript
function customErrorFormatter(error, colors) {
  const messageColor =
    error.severity === "warning" ? colors.bold.yellow : colors.bold.red;
  return (
    "Does not compute.... " +
    messageColor(Object.keys(error).map(key => `${key}: ${error[key]}`))
  );
}
```

If the above formatter received an error like this:

```json
{
  "code":2307,
  "severity": "error",
  "content": "Cannot find module 'components/myComponent2'.",
  "file":"/.test/errorFormatter/app.ts",
  "line":2,
  "character":31
}
```

It would produce an error message that said:

```
Does not compute.... code: 2307,severity: error,content: Cannot find module 'components/myComponent2'.,file: /.test/errorFormatter/app.ts,line: 2,character: 31
```

And the bit after "Does not compute.... " would be red.

#### compilerOptions _(object) (default={})_

Allows overriding TypeScript options. Should be specified in the same format
as you would do for the `compilerOptions` property in tsconfig.json.

#### instance _(string)_

Advanced option to force files to go through different instances of the
TypeScript compiler. Can be used to force segregation between different parts
of your code.

#### appendTsSuffixTo _(RegExp[]) (default=[])_

#### appendTsxSuffixTo _(RegExp[]) (default=[])_

A list of regular expressions to be matched against filename. If filename matches one of the regular expressions, a `.ts` or `.tsx` suffix will be appended to that filename.

This is useful for `*.vue` [file format](https://vuejs.org/v2/guide/single-file-components.html) for now. (Probably will benefit from the new single file format in the future.)

Example:

webpack.config.js:

```javascript
module.exports = {
  entry: "./index.vue",
  output: { filename: "bundle.js" },
  resolve: {
    extensions: [".ts", ".vue"]
  },
  module: {
    rules: [
      { test: /\.vue$/, loader: "vue-loader" },
      {
        test: /\.ts$/,
        loader: "typescript-loader-webpack",
        options: { appendTsSuffixTo: [/\.vue$/] }
      }
    ]
  }
};
```

index.vue

```vue
<template><p>hello {{msg}}</p></template>
<script lang="ts">
export default {
  data(): Object {
    return {
      msg: "world"
    };
  }
};
</script>
```

We can handle `.tsx` by quite similar way:

webpack.config.js:

```javascript
module.exports = {
    entry: './index.vue',
    output: { filename: 'bundle.js' },
    resolve: {
        extensions: ['.ts', '.tsx', '.vue', '.vuex']
    },
    module: {
        rules: [
            { test: /\.vue$/, loader: 'vue-loader',
              options: {
                loaders: {
                  ts: 'typescript-loader-webpack',
                  tsx: 'babel-loader!typescript-loader-webpack',
                }
              }
            },
            { test: /\.ts$/, loader: 'typescript-loader-webpack', options: { appendTsSuffixTo: [/TS\.vue$/] } }
            { test: /\.tsx$/, loader: 'babel-loader!typescript-loader-webpack', options: { appendTsxSuffixTo: [/TSX\.vue$/] } }
        ]
    }
}
```

tsconfig.json (set `jsx` option to `preserve` to let babel handle jsx)

```json
{
  "compilerOptions": {
    "jsx": "preserve"
  }
}
```

index.vue

```vue
<script lang="tsx">
export default {
  functional: true,
  render(h, c) {
    return (<div>Content</div>);
  }
}
</script>
```

Or if you want to use only tsx, just use the `appendTsxSuffixTo` option only:

```javascript
            { test: /\.ts$/, loader: 'typescript-loader-webpack' }
            { test: /\.tsx$/, loader: 'babel-loader!typescript-loader-webpack', options: { appendTsxSuffixTo: [/\.vue$/] } }
```

#### onlyCompileBundledFiles _(boolean) (default=false)_

The default behavior of typescript-loader-webpack is to act as a drop-in replacement for the `tsc` command,
so it respects the `include`, `files`, and `exclude` options in your `tsconfig.json`, loading
any files specified by those options. The `onlyCompileBundledFiles` option modifies this behavior,
loading only those files that are actually bundled by webpack, as well as any `.d.ts` files included
by the `tsconfig.json` settings. `.d.ts` files are still included because they may be needed for
compilation without being explicitly imported, and therefore not picked up by webpack.

#### allowTsInNodeModules _(boolean) (default=false)_

By default, typescript-loader-webpack will not compile `.ts` files in `node_modules`.
You should not need to recompile `.ts` files there, but if you really want to, use this option.
Note that this option acts as a *whitelist* - any modules you desire to import must be included in
the `"files"` or `"include"` block of your project's `tsconfig.json`.


```javascript
  // in webpack.config.js
  {
    test: /\.ts$/,
    loader: 'typescript-loader-webpack',
    options: { allowTsInNodeModules: true }
  }
```

And in your `tsconfig.json`:

```json
  {
    "include": [
      "node_modules/whitelisted_module.ts"
    ],
    "files": [
      "node_modules/my_module/whitelisted_file.ts"
    ]
  }
```

#### context _(string) (default=undefined)_

If set, will parse the TypeScript configuration file with given **absolute path** as base path.
Per default the directory of the configuration file is used as base path. Relative paths in the configuration
file are resolved with respect to the base path when parsed. Option `context` allows to set option
`configFile` to a path other than the project root (e.g. a NPM package), while the base path for `typescript-loader-webpack`
can remain the project root.

Keep in mind that **not** having a `tsconfig.json` in your project root can cause different behaviour between `typescript-loader-webpack` and `tsc`.

webpack:

```javascript
{
  loader: require.resolve('typescript-loader-webpack'),
  options: {
    context: __dirname,
    configFile: require.resolve('ts-config-react-app')
  }
}
```

Extending `tsconfig.json`:

```json
{ "extends": "./node_modules/ts-config-react-app/index" }
```

Note that changes in the extending file while not be respected by `typescript-loader-webpack`. Its purpose is to satisfy the code editor.

#### experimentalFileCaching _(boolean) (default=true)_

By default whenever the TypeScript compiler needs to check that a file/directory exists or resolve symlinks it makes syscalls. It does not cache the result of these operations and this may result in many syscalls with the same arguments ([see comment](https://github.com/zgeaw/typescript-loader-webpack/issues/825#issue-354725524) with example).
In some cases it may produce performance degradation.

This flag enables caching for some FS-functions like `fileExists`, `realpath` and `directoryExists` for TypeScript compiler. Note that caches are cleared between compilations.

#### projectReferences _(boolean) (default=false)_

**TL;DR:** Using project references currently requires building referenced projects outside of typescript-loader-webpack. We don’t want to keep it that way, but we’re releasing what we’ve got now. To try it out, you’ll need to pass `projectReferences: true` to `loaderOptions`. You’ll also probably need to use TypeScript 3.1.1 or later (which, as of this writing, means `typescript@next`).

typescript-loader-webpack has partial support for [project references](https://www.typescriptlang.org/docs/handbook/project-references.html) in that it will _load_ dependent composite projects that are already built, but will not currently _build/rebuild_ those upstream projects. The best way to explain exactly what this means is through an example. Say you have a project with a project reference pointing to the `lib/` directory:

```
tsconfig.json
app.ts
lib/
  tsconfig.json
  niftyUtil.ts
```

And we’ll assume that the root `tsconfig.json` has `{ "references": { "path": "lib" } }`, which means that any import of a file that’s part of the `lib` sub-project is treated as a reference to another project, not just a reference to a TypeScript file. Before discussing how typescript-loader-webpack handles this, it’s helpful to review at a really basic level what `tsc` itself does here. If you were to run `tsc` on this tiny example project, the build would fail with the error:

```
error TS6305: Output file 'lib/niftyUtil.d.ts' has not been built from source file 'lib/niftyUtil.ts'.
```

Using project references actually instructs `tsc` _not_ to build anything that’s part of another project from source, but rather to look for any `.d.ts` and `.js` files that have already been generated from a previous build. Since we’ve never built the project in `lib` before, those files don’t exist, so building the root project fails. Still just thinking about how `tsc` works, there are two options to make the build succeed: either run `tsc -p lib/tsconfig.json` _first_, or simply run `tsc --build`, which will figure out that `lib` hasn’t been built and build it first for you.

Ok, so how is that relevant to typescript-loader-webpack? Because the best way to think about what typescript-loader-webpack does with project references is that it acts like `tsc`, but _not_ like `tsc --build`. If you run typescript-loader-webpack on a project that’s using project references, and any upstream project hasn’t been built, you’ll get the exact same `error TS6305` that you would get with `tsc`. If you modify a source file in an upstream project and don’t rebuild that project, `typescript-loader-webpack` won’t have any idea that you’ve changed anything—it will still be looking at the output from the last time you _built_ that file.

**“Hey, don’t you think that sounds kind of useless and terrible?”** Well, sort of. You can consider it a work-in-progress. It’s true that on its own, as of today, typescript-loader-webpack doesn’t have everything you need to take advantage of project references in webpack. In practice, though, _consuming_ upstream projects and _building_ upstream projects are somewhat separate concerns. Building them will likely come in a future release. For background, see the [original issue](https://github.com/zgeaw/typescript-loader-webpack/issues/815).

**`outDir` Windows problemo.** At the moment, composite projects built using the [`outDir` compiler option](https://www.typescriptlang.org/docs/handbook/compiler-options.html) cannot be consumed using typescript-loader-webpack on Windows. If you try to, typescript-loader-webpack throws a "has not been built from source file" error.  We don't know why yet; it's possible there's a bug in `tsc`. It's more likely there's a bug in `typescript-loader-webpack`. Hopefully it's going to get solved at some point. (Hey, maybe you're the one to solve it!) Either way, we didn't want to hold back from releasing. So if you're building on Windows then avoid building `composite` projects using `outDir`.

**TypeScript version compatibility.** As a final caveat, [this commit to TypeScript](https://github.com/Microsoft/TypeScript/commit/d519e3f21ec36274726c44dab25c9eb48e34953f) is necessary for the `include` or `exclude` options of a project-referenced tsconfig file to work. It should be released in TypeScript 3.1.1 according to the tags. To use an earlier version of TypeScript, referenced project configuration files must specify `files`, and not `include`.

### Usage with webpack watch

Because TS will generate .js and .d.ts files, you should ignore these files, otherwise watchers may go into an infinite watch loop. For example, when using webpack, you may wish to add this to your webpack.conf.js file:

```javascript
 plugins: [
   new webpack.WatchIgnorePlugin([
     /\.js$/,
     /\.d\.ts$/
   ])
 ],
```

It's worth noting that use of the `LoaderOptionsPlugin` is [only supposed to be a stopgap measure](https://webpack.js.org/plugins/loader-options-plugin/). You may want to look at removing it entirely.

### Hot Module replacement

To enable `webpack-dev-server` HMR, you need to follow the official [webpack HMR guide](https://webpack.js.org/guides/hot-module-replacement/), then tweak a few config options for `typescript-loader-webpack`. The required configuration is as follows:

1. Set `transpileOnly` to `true` (see [transpileOnly](#transpileonly-boolean-defaultfalse) for config details and recommendations above).
2. Inside your HMR acceptance callback function, you must re-require the module that was replaced.

For a boilerplate HMR project using React, check out the [react-hot-boilerplate example](./examples/react-hot-boilerplate/).

For a minimal HMR TypeScript setup, go to the [hot-module-replacement example](./examples/hot-module-replacement/).

## License

MIT License
