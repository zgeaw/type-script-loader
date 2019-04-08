# TypeScript loader for webpack


这是针对Webpack的一个typescript 轮子。

### 实例

我们有许多示例设置来适应不同的工作流。我们的示例可以在[此处]找到(examples/)。

我们可能有比我们需要的更多的例子。也就是说，这是一个开始的好方法：

- 最简单的设置。  "(examples/vanilla)" type-script-loader

### 快速构建

当您的项目变大时，编译时间线性增加。这是因为typescript的语义检查器必须在每次重新生成时检查所有文件。简单的解决方案是通过使用“reateleonly:true”选项来禁用它，但是这样做会使您没有类型检查，并且*不会输出声明文件*。

### Babel

type-script-loader works very well in combination with [babel](https://babeljs.io/) and [babel-loader](https://github.com/babel/babel-loader). There is an [example](https://github.com/Microsoft/TypeScriptSamples/tree/master/react-flux-babel-karma) of this in the official [TypeScript Samples](https://github.com/Microsoft/TypeScriptSamples). Alternatively take a look at our own [example](examples/react-babel-karma-gulp).

### 安装

```
yarn add type-script-loader --dev
```

or

```
npm install type-script-loader --save-dev
```

You will also need to install TypeScript if you have not already.

```
yarn add typescript --dev
```

or

```
npm install typescript --save-dev
```

### 运行

像普通一样使用webpack，包括“webpack--watch”和“webpack dev server”，或者通过另一个

使用[node.js api]构建系统（http://webpack.github.io/docs/node.js api.html）。

### 兼容性

* TypeScript: 3.0.1+
* webpack: 4.x+ (please use type-script-loader 3.x if you need webpack 2 or 3 support)
* node: 6.11.5 minimum (aligned with webpack 4)


如果您发现测试套件未发现的问题，请通知我们。更好的是，编写一个测试并在一个pr中提交它！

### 配置

1. 创建或更新类似的'webpack.config.js':

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
         // all files with a `.ts` or `.tsx` extension will be handled by `type-script-loader`
         { test: /\.tsx?$/, loader: "type-script-loader" }
       ]
     }
   };
   ```

2. 添加一个[`tsconfig.json`]文件。（下面的这个非常简单，但是你可以根据自己的心愿来调整）

   ```json
   {
     "compilerOptions": {
       "sourceMap": true
     }
   }
   ```

[顶] tsconfig.json（http://www.typescriptlang.org/docs/handbook/tsconfig-json.html）文件的控件typescript相关选项，这样你的IDE，TSC ` `命令，这一切和装载机相同的选项。

#### 调试工具

如果您希望能够调试您的原始源，那么您可以感谢源映射的魔力。使用type-script-loader和Webpack设置此设置有两个步骤。

首先，要让类型脚本加载程序生成**源映射**，您需要将[tsconfig.json]（http://www.typescriptlang.org/docs/handbook/tsconfig-json.html）选项设置为“sourcemap”：true`。

其次，需要在“webpack.config.js”中设置“devtool”选项，以支持所需的源映射类型。要让您选择阅读[`devtool` webpack docs]（https://webpack.js.org/configuration/devtool/）。你可能会对现有的选择有些畏惧。您还可能希望根据构建环境改变源映射策略。以下是针对不同环境的一些示例策略：

*` devtool:'inline source map'`-支持Solid source map；最好的“全方位”。与Karma Webpack合作良好（并非所有策略都如此）

*` devtool：“廉价的模块评估源映射”`-调试时对源映射的最佳支持。

*` devtool:'源映射'`-与uglifyjsplugin一起使用的方法；通常您可以在生产中使用它

### 代码拆分和加载其他资源

加载CSS和其他资源是可能的，但您需要确保

您已在[声明文件]（https://www.typescriptlang.org/docs/handbook/writing-declaration-files.html）中定义了“require”函数。

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

然后，您可以根据[Webpack文档]（https://webpack.js.org/guides/code-splitting/）简单地要求资产或块。

```javascript
require("!style!css!./style.css");
```
代码拆分需要相同的基本过程。在本例中，您“导入”所需的模块，但您

不要直接使用。相反，您需要在[拆分点]（http://webpack.github.io/docs/code-splitting.html#defining-a-split-point）使用它们。有关详细信息，请参阅[此示例]（测试/比较测试/代码拆分）和[此示例]（测试/比较测试/ES6code拆分）。

[typescript 2.4提供了对ecmascript新的'import（）'调用的支持。这些调用导入一个模块并向该模块返回一个承诺。]（https://blogs.msdn.microsoft.com/typescript/2017/06/12/announcing-typescript-2-4-rc/）Webpack中也支持此功能-有关使用的详细信息，请参阅[此处]（https://webpack.js.org/guides/code-splitting-async/#dynamic-import-import-）。代码拆分愉快！

### 声明 (.d.ts)

要输出构建的.d.ts文件，可以在ts config中设置“declaration”：true，并在webpack配置中使用[declaration bundler plugin]（https://www.npmjs.com/package/declaration-bundler-webpack-plugin）。

### 未能生成typescript编译错误

从Webpack2开始，build**应该**在typescript编译错误上失败。如果出于某种原因，您可以使用[Webpack fail plugin]（https://www.npmjs.com/package/webpack-fail-plugin）。

有关更多背景，请阅读(https://github.com/zgeaw/type-script-loader/issues/108).

### 模块

如果要根据“tsconfig.json”中的“baseurl”和“paths”解析模块，则可以使用[tsconfig paths webpack plugin]（https://www.npmjs.com/package/tsconfig-paths-webpack-plugin）包。有关此功能的详细信息，请参阅[模块解析文档]（https://www.typescriptlang.org/docs/handbook/module-resolution.html#base-url）。

此功能需要WebPack 2.1+和TypeScript 2.0+。使用下面的配置或检查[包]（https://github.com/dividab/tsconfig-paths-webpack-plugin/blob/master/README.md）以了解更多使用信息。

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

### 选项

有两种类型的选项：TypeScript（也称为“编译器选项”）和加载程序选项。应使用tsconfig.json文件设置typescript选项。可以通过Webpack配置中的“options”属性指定加载程序选项：

```javascript
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: [
          {
            loader: 'type-script-loader',
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

#### transpileOnly (default=false)_

如果要显著加快编译速度，可以设置此标志。

但是，静态类型检查所带来的许多好处应用程序中的不同依赖项将丢失。

建议在[fork ts checker webpack plugin]（https://github.com/Realytics/fork-ts-checker-webpack-plugin）再次进行完整类型检查。要了解实际情况，请查看[我们的简单示例]（examples/fork-ts-checker-webpack-plugin）。对于更复杂的设置，请查看我们的[更复杂的示例]（示例/反应Babel Karma Gulp）。

如果启用此选项，则每次重新导出类型时，Webpack4都会向您发出“导出未找到”警告：

```
WARNING in ./src/bar.ts
1:0-34 "export 'IFoo' was not found in './foo'
 @ ./src/bar.ts
 @ ./src/index.ts
```

发生这种情况的原因是，当typescript不进行完整的类型检查时，它没有足够的信息来确定导入的名称是否为类型，因此当名称被导出时，typescript除了发出导出之外别无选择。幸运的是，外来出口不应该是有害的，所以您可以禁止这些警告：

```javascript
module.exports = {
  ...
  stats: {
    warningsFilter: /export .* was not found in/
  }
}
```

#### happyPackMode (default=false)_

如果使用[happypack]（https://github.com/amireh/happypack）或[thread loader]（https://github.com/webpack-contrib/thread-loader）对生成进行并行处理，则需要将其设置为“true”。这将隐式地将“*蒸腾*”设置为“真”和**警告！**停止向Webpack注册**所有错误。

建议将其与[fork ts checker webpack plugin]（(https://github.com/Realytics/fork-ts-checker-webpack-plugin）一起使用，以再次进行完整类型检查。要了解实际情况，请查看[我们的简单happypack示例]（examples/happypack）/[我们的简单线程加载器示例]（examples/thread loader）。要了解更复杂的设置，请查看我们的[更复杂的happypack示例]（示例/react babel kama gulp happypack）/[更复杂的线程加载器示例]（示例/react babel kama gulp线程加载器）。**“重要提示”**：如果您将fork ts checker webpack plugin与happypack或thread loader一起使用，请确保按如下方式设置“checksyntacticerrors”选项：

```javascript
        new ForkTsCheckerWebpackPlugin({ checkSyntacticErrors: true })
```

这将确保插件检查语法错误（例如“const array=[]；`）和语义错误（例如“const x:number='1'；`）。默认情况下，插件只检查语义错误（与“蒸腾”模式下的类型脚本加载程序一起使用时，类型脚本加载程序仍将报告语法错误）。

另外，如果在监视模式下使用“thread-loader”，请记住设置“pooltimeout:infinity”，这样线程就不会死。

#### getCustomTransformers _( (program: Program) => { before?: TransformerFactory<SourceFile>[]; after?: TransformerFactory<SourceFile>[]; } )_

提供自定义变压器-仅与typescript 2.3+兼容（如果使用“transpileOnly”模式，则为2.4）。例如，用法请查看[typescript plugin-styled components]（https://github.com/Igorbek/typescript-plugin-styled-components）或我们的[https://github.com/Igorbek/typescript-plugin-styled-components]（test/comparison-tests/customTransformer）。

您还可以传递一个路径字符串来定位导出上述函数的JS模块文件，这在“happypackMode”中尤其有用。（因为分叉进程无法序列化函数，请参阅(https://github.com/Igorbek/typescript-plugin-styled-components/issues/6#issue-303387183))

#### logInfoToStdOut (default=false)_

如果您从stdout或stderr中读取数据并正确处理错误，那么这一点很重要。
默认值确保您可以从stdout（例如通过管道）读取，或者使用webpack-j生成json输出。

#### logLevel (default=warn)_

可以是“info”、“warn”或“error”，这将日志输出限制为指定的日志级别。

请注意，错误是写入stderr的，其他所有内容都写入stderr（如果loginfoostdout为true，则写入stdout）。

#### silent (default=false)_

如果为“true”，则不会发出console.log消息。请注意，大多数错误消息通过不受此标志影响的Webpack发出。

#### ignoreDiagnostics (default=[])_

通过指定诊断数组，可以消除某些类型脚本错误。
要忽略的代码。

#### reportFiles (default=[])_

只报告与这些全局模式匹配的文件的错误。

```javascript
  // in webpack.config.js
  {
    test: /\.ts$/,
    loader: 'type-script-loader',
    options: { reportFiles: ['src/**/*.{ts,tsx}', '!src/skip.ts'] }
  }
```

当某些类型定义存在对应用程序不致命的错误时，这可能很有用。

#### compiler (default='typescript')_

允许使用除正式编译器以外的其他类型脚本编译器。应该是设置为编译器的NPM名称，例如 [`ntypescript`](https://github.com/basarat/ntypescript).

#### configFile (default='tsconfig.json')_

允许您指定在何处查找typescript配置文件。

你可以提供

*只是一个文件名。然后，加载程序将在各自入口点的包含文件夹中搜索每个入口点的配置文件。如果在那里找不到配置文件，它将沿着父目录链向上移动，并在这些文件夹中查找配置文件。

*配置文件的相对路径。它将相对于相应的`.ts'条目文件进行解析。

*配置文件的绝对路径。

请注意，如果配置文件在项目目录之外，则可能需要设置“context”选项以避免出现类型脚本问题（如ts18003）。

在这种情况下，“configfile”应该指向“tsconfig.json”，“context”指向项目根目录。

#### colors (default=true)_

如果为“false”，则禁用记录器消息中的内置颜色。

#### errorFormatter _((message: ErrorInfo, colors: boolean) => string) (default=undefined)_

默认情况下，类型脚本加载程序会为样式中的错误或警告格式化类型脚本编译器输出：

```
[tsl] ERROR in myFile.ts(3,14)
      TS4711: you did something very wrong
```

如果您不喜欢这种格式，您可以使用“errorformatter”选项提供自己的格式设置工具。下面是自定义错误格式化程序的模板。请注意，“colors”参数是可用于为输出着色的[`chalk`]（https://github.com/chalk/chalk）的实例。（此实例将尊重“colors”选项。）

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

如果上述格式化程序收到如下错误

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

它将产生一条错误消息，指出：

```
Does not compute.... code: 2307,severity: error,content: Cannot find module 'components/myComponent2'.,file: /.test/errorFormatter/app.ts,line: 2,character: 31
```

“不计算……”后的位。将是红色的。

#### compilerOptions (default={})_

允许重写typescript选项。应以相同格式指定

就像在tsconfig.json中对“compilerOptions”属性所做的那样。

#### instance _(string)_

高级选项强制文件通过TypeScript。可用于在不同零件之间强制分离你的代码。

#### appendTsSuffixTo _(RegExp[]) (default=[])_

#### appendTsxSuffixTo _(RegExp[]) (default=[])_

要与文件名匹配的正则表达式列表。如果文件名与某个正则表达式匹配，则该文件名将附加一个“.ts”或“.tsx”后缀。

这对于现在的`*.vue`[文件格式]（https://vuejs.org/v2/guide/single-file-components.html）很有用。（将来可能会受益于新的单一文件格式。）

例子:

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
        loader: "type-script-loader",
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

我们可以用类似的方式处理`.tsx`：

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
                  ts: 'type-script-loader',
                  tsx: 'babel-loader!type-script-loader',
                }
              }
            },
            { test: /\.ts$/, loader: 'type-script-loader', options: { appendTsSuffixTo: [/TS\.vue$/] } }
            { test: /\.tsx$/, loader: 'babel-loader!type-script-loader', options: { appendTsxSuffixTo: [/TSX\.vue$/] } }
        ]
    }
}
```

tsconfig.json（将'jsx'选项设置为'preserve'以让babel处理jsx）

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

或者，如果只想使用tsx，只需使用'appendtsxsuffixto'选项：

```javascript
            { test: /\.ts$/, loader: 'type-script-loader' }
            { test: /\.tsx$/, loader: 'babel-loader!type-script-loader', options: { appendTsxSuffixTo: [/\.vue$/] } }
```

#### onlyCompileBundledFiles (default=false)_

type-script-loader的默认行为是作为“tsc”命令的替换，

因此，它考虑了在加载时“tsconfig.json”中的“include”、“files”和“exclude”选项，由这些选项指定的任何文件。'onlycompilebundledfiles'选项修改此行为，只加载Webpack实际捆绑的那些文件，以及包含的任何`.d.ts`文件，通过“tsconfig.json”设置。`.d.ts'文件仍然包含在其中，因为编译时没有被显式导入，因此Webpack不接受。

#### allowTsInNodeModules (default=false)_

默认情况下，类型脚本加载器不会编译“node_modules”中的“.ts”文件。

您不需要在那里重新编译`.ts`文件，但如果您真的想这样做，请使用此选项。

请注意，此选项用作*白名单*—您希望导入的任何模块都必须包含在项目的“tsconfig.json”的“files”或“include”块。


```javascript
  // in webpack.config.js
  {
    test: /\.ts$/,
    loader: 'type-script-loader',
    options: { allowTsInNodeModules: true }
  }
```

在你的“tsconfig.json”中：

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

如果设置，将解析给定**绝对路径**作为基路径的类型脚本配置文件。

默认情况下，配置文件的目录用作基本路径。配置中的相对路径文件在解析时根据基本路径进行解析。option`context`允许设置选项
`configfile`到项目根目录以外的路径（例如，NPM包），而'type-script-loader'的基本路径`可以保留项目根目录。

请记住，*不**在项目根目录中有一个“tsconfig.json”可能会导致“type-script-loader”和“tsc”之间的行为不同。

使用“vs code”等编辑器时，建议将“tsconfig.json”文件添加到项目的根目录并扩展配置文件。

webpack:

```javascript
{
  loader: require.resolve('type-script-loader'),
  options: {
    context: __dirname,
    configFile: require.resolve('ts-config-react-app')
  }
}
```

扩展 `tsconfig.json`:

```json
{ "extends": "./node_modules/ts-config-react-app/index" }
```

请注意，扩展文件中的更改不受“type-script-loader”的影响。它的目的是满足代码编辑器。

#### experimentalFileCaching (default=true)_

当典型编译器需要检查file/directory或分辨Symlinks时，即为无效在一些情况下，它可以产生性能降解。

此标志允许对typescript编译器的某些fs函数（如“fileexists”、“realpath”和“directoryexists”）进行缓存。请注意，在编译之间清除缓存。

#### projectReferences (default=false)_

**tl；dr:**使用项目引用当前需要在type-script-loader之外生成引用的项目。我们不想那样做，但我们正在释放我们现在拥有的。要试用它，您需要将'projectreferences:true'传递给'loaderOptions'。您可能还需要使用typescript 3.1.1或更高版本（在本文中，这意味着“typescript@next”）。

type-script-loader部分支持[项目引用]（https://www.typescriptlang.org/docs/handbook/project-references.html），因为它将加载已构建但当前不构建/重建上游项目的依赖复合项目。准确解释这意味着什么的最好方法是通过一个例子。假设有一个项目引用指向“lib/”目录：

```
tsconfig.json
app.ts
lib/
  tsconfig.json
  niftyUtil.ts
```

我们假定根'tsconfig.json'具有引用`{ "references": { "path": "lib" } }`这意味着作为“lib”子项目一部分的文件的任何导入都被视为对另一个项目的引用，而不仅仅是对typescript文件的引用。在讨论类型脚本加载器如何处理这一点之前，在真正的基本级别上回顾一下“tsc”本身在这里做什么是很有帮助的。如果要在这个小示例项目上运行“tsc”，则生成将失败，并出现错误：

```
error TS6305: Output file 'lib/niftyUtil.d.ts' has not been built from source file 'lib/niftyUtil.ts'.
```

使用项目引用实际上会指示“tsc”不要从源代码构建属于另一个项目的任何内容，而是查找从以前的构建中生成的任何“d.ts”和“js”文件。因为我们以前从未在“lib”中构建过项目，所以这些文件不存在，所以构建根项目失败。仍然只考虑“tsc”是如何工作的，有两种方法可以使构建成功：运行“tsc-p lib/tsconfig.json` ，或者只运行“tsc--build”，这将发现“lib”尚未构建，并首先为您构建。

**`outdir`windows problemo.**目前，不能使用Windows上的类型脚本加载程序来使用使用[`outdir`编译器选项]生成的复合项目（https://www.typescriptlang.org/docs/handbook/compiler-options.html）。如果尝试这样做，类型脚本加载器将抛出“尚未从源文件生成”错误。我们还不知道为什么；可能在“tsc”中有一个bug。“type script loader”中更有可能存在bug。希望它能在某个时刻得到解决。（嘿，也许你是解决问题的人！）不管怎样，我们都不想拖延释放。因此，如果您是在Windows上构建的，那么请避免使用“outdir”构建“composite”项目。

**typescript版本兼容性。**作为最后的警告，[此提交到typescript]（https://github.com/Microsoft/TypeScript/commit/d519e3f21ec36274726c44dab25c9eb48e34953f）对于项目引用的tsconfig文件的“include”或“exclude”选项的工作是必需的。它应该根据标签在typescript 3.1.1中发布。要使用早期版本的typescript，引用的项目配置文件必须指定“files”，而不是“include”。

### 使用webpack watch

因为TS将生成.js和.d.ts文件，所以您应该忽略这些文件，否则观察程序可能进入无限的监视循环。例如，在使用Webpack时，您可能希望将其添加到webpack.conf.js文件中：

```javascript
 plugins: [
   new webpack.WatchIgnorePlugin([
     /\.js$/,
     /\.d\.ts$/
   ])
 ],
```
值得注意的是，使用“loaderopitionsplugin”是[只应该是一个权宜之计]（https://webpack.js.org/plugins/loader-options-plugin/）。您可能希望完全删除它。

### 热更新

To enable `webpack-dev-server` HMR, you need to follow the official [webpack HMR guide](https://webpack.js.org/guides/hot-module-replacement/), then tweak a few config options for `type-script-loader`. The required configuration is as follows:

1. Set `transpileOnly` to `true` (see [transpileOnly](#transpileonly-boolean-defaultfalse) for config details and recommendations above).
2. Inside your HMR acceptance callback function, you must re-require the module that was replaced.

For a boilerplate HMR project using React, check out the [react-hot-boilerplate example](./examples/react-hot-boilerplate/).

For a minimal HMR TypeScript setup, go to the [hot-module-replacement example](./examples/hot-module-replacement/).

## License

MIT License
