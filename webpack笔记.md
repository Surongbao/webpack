### 构建工具简介

**浏览器端模块化过程中存在的问题**

- 精细的模块划分带来了更多的 JS 文件，更多的 JS 文件带来了更多的网络请求，从而降低了页面的访问效率。
- 目前的浏览器只支持 ES6 模块化标准，如果某个第三方库使用 CommonJS 模块化标准的方式导出，则无法使用。
- 目前的浏览器无法直接支持通过 npm 下载的第三方库。
- 等等更多浏览器端模块化的问题。

**浏览器端模块化过程中存在的根本问题**

- 在开发时态我们希望：模块划分得越细越好，支持多种模块化标准，支持各种包管理工具安装的第三方库，还能解决代码的兼容性、执行效率、可维护性、可扩展性等众多问题。
- 在运行时态我们希望：文件越少越好，文件体积越小越好，代码越乱越好，还能解决运行阶段遇到的很多问题，尤其是执行效率的问题。

**构建工具能解决的问题**

可以让开发者在开发阶段专心的编写业务代码，然后通过构建工具，将开发阶段的代码转换成运行阶段的代码，同时还能解决运行阶段遇到的很多问题（如：执行效率的问题）。

**常见的前端工程化构建工具**

- webpack
- gulp
- browserify
- grunt
- fis
- 等等

### 简介

webpack 是一个基于模块化的前端工程化构建工具，它视一切为模块。

为前端工程化而生、支持零配置、灵活、可扩展、基于 NodeJS、支持各种模块化标准。

它会以一个开发阶段的入口模块为起点，通过导入语句分析出所有的依赖，再经过一系列的压缩、打包过程，最终生成运行阶段的代码文件（不存在模块化，只是普通的 JS 代码）

### 安装和使用

**安装 webpack**

```shell
npm i -D webpack webpack-cli
```

- webpack：核心包，包含了构建过程中使用到的所有 webpackApi
- webpack-cli：提供了一个 cli 命令，该命令可以调用 webpackApi 完成打包过程

> 推荐使用本地安装，这样每个项目都可以有自己的 webpack 版本

**使用 webpack**

```shell
npx webpack
```

默认情况下，webpack 会以 `./src/index.js` 作为入口模块分析依赖关系，将最终代码打包到 `./dist/main.js` 文件中

**命令参数**：一个命令中可以书写多个参数，使用 “空格” 分隔

- `--mode`：该参数可以控制最终代码的运行环境，默认生成环境

```shell
npx webpack --mode=production ## 生成环境
npx webpack --mode=development ## 开发环境
```

- `--watch`：该参数用于监测文件的变化，文件一旦发生了改变，则会重新打包

```shell
npx webpack --watch ## 只要文件发生了变化，会重新打包
```

可以将参数配置到脚本中，方便书写命令

```json
"scripts": {
  "dev": "webpack --mode=development --watch",
  "build": "webpack --mode=production --watch",
},
```

执行脚本中的命令

```shell
npm run dev
npm run bulid
```

### 模块化兼容性

由于 webpack 同时支持 CommonJS 模块化标准和 ES6 模块化标准，并且可以支持不同模块化标准的导入导出方式（如：使用 ES6 模块化标准的方式导出，可以使用 CommonJS 模块化标准的方式导入；使用 CommonJS 模块化标准的方式导出，可以使用 ES6 模块化标准的方式导入），webpack 是怎么做到的呢？

**同模块化标准的导入导出**

会按照每个模块化标准的导入导出规则得到结果

```js
module.exports = {
  a: 1,
  b: 2,
  c: 3,
};
```

```js
const result = require("./module.js");
console.log(result); // => { a: 1, b: 2, c: 3 }
```

ES6 模块化标准导入导出

```js
export const a = 1;
export const b = 2;
export default 3;
```

```js
// 全部导入，是一个对象
import * as result from "./module.js";
console.log(result); // => { a: 1, b: 2, default: 3 }

// 导入默认导出
import result from "./module.js";
console.log(result); // => 3

// 导入基本导出
import { a, b } from "./module.js";
console.log(a, b); // => 1 2
```

**CommonJS 模块化标准导出，ES6 模块化标准导入**

```js
module.exports = {
  a: 1,
  b: 2,
  c: 3,
};
```

使用全部导入或默认导入，导入的结果是导出的对象

```js
import * as result from "./common.js";
console.log(result); // => { a: 1, b: 2, c: 3 }

import result from "./common.js";
console.log(result); // => { a: 1, b: 2, c: 3 }
```

使用基本导入，导入的结果就好像解构语法，导出同名变量的值。

```js
import { a } from "./common.js";
console.log(a); // => 1
```

**ES6 模块化标准导出，CommonJS 模块化标准导入**

```js
export const a = 1;
export const b = 2;
export default 3;
```

会把基本导出和默认导出放入到一个对象中，把这个对象导入

```js
const result = require("./es6.js");
console.log(result); // => { a: Getter, b: Getter, default: Getter }
```

**总结**

- 绝大多数第三方库都使用 CommonJS 模块化标准的导出方式。
- 在实际开发中，选择一种模块化标准即可，不要 CommonJS 和 ES6 混用。

### 编译结果分析

```js
((modules) => {
  const cacheModule = {}; // 用于模块缓存

  // 导入函数，接受一个模块ID，模块ID就是模块的路径
  function require(moduleId) {
    if (cacheModule[moduleId]) {
      // 缓存中存在相应的模块导出结果，直接返回该结果
      return cacheModule[moduleId];
    }

    // 根据模块ID拿到对应的函数
    const handle = modules[moduleId];
    const module = {
      exports: {},
    };
    // 执行函数传递相应的参数
    handle(module, module.exports, require);
    const result = module.exports; // CommonJS默认返回module.exports
    cacheModule[moduleId] = result; // 缓存

    return result;
  }

  // 执行入口文件
  require("./src/index.js");
})({
  "./src/a.js": function (module, exports, require) {
    eval(
      'console.log("a 模块");\nmodule.exports = "a";\n//# sourceURL=webpack://test/./src/a.js?'
    );
  },
  "./src/index.js": function (module, exports, require) {
    eval(
      'console.log("入口模块");\nconst a = require("./src/a.js");\nconsole.log(a);\n//# sourceURL=webpack://test/./src/index.js?'
    );
  },
});
```

### 编译过程

编译过程：将开发阶段代码编译成运行阶段代码的过程。

**1. 初始化**

webpack 将命令行参数、配置文件、默认配置进行融合，生成一个最终的配置对象。（依靠第三方库 `yargs` 完成），这个阶段只运行一次。

**2. 编译**

1. 创建 chunk

chunk：是 webpack 在内部构建过程中的一个概念，是通过入口模块找到的所有依赖的统称。

一个入口模块会创建一个 chunk，每个 chunk 至少有两个属性

- name：chunk 的名字，默认是 main
- id：chunk 的唯一编号，开发环境中与 name 相同，生成环境中是从 0 开始的数字。

1. 构建所有依赖模块

chunks：用来记录转换后的模块代码（模块 ID：模块路径; 模块转换后的代码）

根据入口模块分析依赖 => 检查 chunks 中是否有记录（记录过直接返回 ID 和转换后的代码）=> 读取模块内容 => loader => AST 抽象语法树分析 => 记录依赖到 dependencies 数组中 => 将模块内容转换成最终代码 => 保存到 chunks 中 => 递归加载 dependencies 中记录的依赖模块 => 检查 chunks 中是否有记录 => 依次循环

最终 chunks 中会记录所有的模块 ID 和每个模块转换后的代码。

3. 生成 chunk assets

webpack 会根据 chunks 中的记录为每个 chunkID（模块 ID、模块路径）生成一个资源列表，即 chunk assets，同时会根据每个 chunk assets 的内容生成一个 chunk hash 字符串。

hash：是一种算法，它的特点是将一个任意长度的字符串转换成另一个固定长度的字符串，同时可以保证：只要原始字符串内容不变，产生的 hash 字符串就不变。

4. 合并 chunk assets

将多个 chunk assets 合并成一个总的 assets，根据总的 assets 内容生成一个总的 hash 字符串。

**3. 输出**

webpack 利用 Node 中的内置模块 fs（该模块用于文件处理），根据总的 assets 中的内容（文件名和文件内容）生成相应的文件。

**术语**

- module：模块
- chunk：通过入口模块分析出依赖的统称
- bundle：最终生成的文件
- hash：总 assets 里面的总 hash 字符串
- chunkhash：每个 chunk 生成的 hash
- chunkname：每个 chunk 的名字，默认是 main
- id：chunk 的唯一编号，开发环境中与 chunkname 相同，生产环境中是从 0 开始的数字

### 配置文件

用于配置 webpack 命令的参数（如：mode、watch 等），参数是为了控制 webpack 的行为。

默认情况下，webpack 会读取 `webpack.config.js` 文件做为配置文件（是我们手动创建的，通常会创建在项目的根目录中），也可以使用 `npx webpack --config 文件名` 命令来指定 webpack 的配置文件。

配置文件需要使用 CommonJS 模块化标准导出一个对象，这个对象就是配置对象。

配置文件会参与 webpack 的运行，因为 webpack 在 Node 中运行，所以配置文件中的代码必须是有效的 Node 代码。（运行阶段的代码：只是 webpack 做分析依赖关系用的，并不会参与 webpack 的运行）

当命令行中的参数与配置文件中的配置冲突时，以命令行中的参数为准。

##### mode

编译模式：配置最终代码的运行环境，不同的配置将影响最终代码的格式，值是一个字符串。

```js
module.exports = {
  mode: "development" | "production", // 开发环境 | 生成环境
};
```

##### watch

用于监测文件内容的变化，文件内容变化重新编译，取值：true | false

```js
module.exports = {
  watch: true | false, // 用于监测文件的变量
};
```

##### entry

入口，值是一个对象，用于配置每个 chunk 的名字和对应的入口模块路径

```js
module.exports = {
  entry: {
    // 一个chunk对应一个入口模块
    index: "./src/index.js",
    // 一个chunk对应多个入口模块，两个入口模块依次执行，最终文件还是一个，最终导出数组的最后一项
    main: ["./src/main1.js", "./src/main2.js"],
  },
};
```

##### output

出口，值是一个对象，用于配置最终文件（总资源列表）的名称和路径

`./`：如果在模块中使用，表示当前 JS 文件所在目录；如果在路径处理中，表示 Node 环境所运行目录（当前执行目录）
`__dirname`：总表示当前 JS 文件所在目录，是一个绝对路径
`path`：Node 的内置模块，里面的 `path.resolve()` 方法会根据多段字符串生成一个绝对路径。

```js
const path = require("path");

module.exports = {
  output: {
    // 将最终文件放置到哪个目录中，默认是dist，值必须是一个绝对路径
    path: path.resolve(__dirname, "目录名称"),

    // 配置每个chunk的名字
    filename: "bundle.js", // 适用于入口只有一个chunk

    // 下面适用于入口中有多个 chunk，其中 hash 可以控制长度
    filename: "[name].js", // 根据不同 chunk 名称生成最终文件
    filename: "[hash:5].js", // 根据总 hash 生成最终文件，解决浏览器缓存问题
    filename: "[chunkhash:5].js", // 根据 chunkhash 生成最终文件

    // 不推荐使用 id，因为 chunkID 在开发环境和生成环境中的表现形式不一样
    filename: "[id].js", // 根据 chunkID 生成最终文件，

    library: "abc", // 将编译过程中的立即执行函数赋值给变量 abc

    // 用于控制如何暴露变量 abc
    libraryTarget: "var", // var abc = (() => {})()
    libraryTarget: "window", // window.abc = (() => {})()
    libraryTarget: "this", // this.abc = (() => {})()
    libraryTarget: "global", // self.abc = (() => {})()
    libraryTarget: "commonjs", // exports.abc = (() => {})()

    publicPath: "/", // 一般会配置一个斜杠，表示当前域
  },
};
```

- path：表示最终生成的文件要放置到哪个目录中，默认是 dist，值必须是一个绝对路径。
- filename：用于设置每个 chunk 生成最终文件的规则。
- library：将编译过程的立即执行函数赋值给变量，通常用于暴露一些东西。
- libraryTarget：和 library 联用，用于控制如何暴露变量。https://www.webpackjs.com/configuration/output/#output-librarytarget
- publicPath：解决路径问题，会赋值给 `__webpack_require__.p`，某些 loader 或 plugin 会把该值拼接到返回路径的前面（webpack5 好像已经解决了），也可以为单独的 loader 或 plugin 配置该选项，看 loader 或 plugin 是否允许配置。

##### devtool

用于解决生产环境中代码难以调试的问题，也称源码地图。

参考文档：https://www.webpackjs.com/configuration/devtool/

```js
module.exports = {
  devtool: "source-map", // 开发环境推荐
  devtool: "none", // 生成环境推荐，生成环境中默认值
};
```

##### loader

是 webpack 的扩展功能，本质是一个函数，用于将一个源码字符串转换成另一个源码字符串返回，返回的字符串必须是可执行的 JS 代码。

```js
module.exports = (sourceCode) => {
  // sourceCode：模块的源码
  // 对 sourceCode 一顿操作
  return `处理过后的sourceCode`;
};
```

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/, // 使用正则匹配
        use: [
          {
            loader: "./loaders/test-loader.js",
            // => require("./loaders/test-loader.js")
            options: {
              key: value, // 用于给 loader 传递参数
            },
            // 也可以这样给 loader 传递参数
            loader: "./loaders/test-loader.js?key=value",
          },
          // ... 其他 loader
        ],
        // 如果 loader 只是一个字符串，可以这样书写
        user: ["loader1", "loader2", ...],
      },
      // ... 其他规则
    ],

    noParse: /jquery/, // 不解析jquery库
  },
};
```

- module：在该选项中配置 loader
- rules：规则数组，用于定义匹配模块的规则，每项是一个对象，表示每个规则
- test：用于匹配模块，值是一个正则表达式
- use：匹配成功后使用哪些 loader，是一个数组，每项是一个对象，表示 loader 的配置
- loader：值是一个字符串，书写 loader 的路径，最终会被 require
- options：值是一个对象，可以向 loader 传递参数。
- noParse：值是一个正则表达式，表示不解析匹配的模块，通常用于忽略大型的单模块库（该库不在依赖任何库），以提高构建性能。

loader 函数将在 webpack 解析模块的过程中被调用，发生在 webpack 读取模块内容之后，AST 抽象语法树分析之前（webpack 会将模块内容交给 loader 处理，将 loader 返回的字符串进行 AST 抽象语法树分析）

**loader 的解析流程**

1. 查看当前模块是否满足某个 `test` 匹配规则，如果不满足，则 loaders 会是一个空数组，表示没有 loader 处理。
2. 如果满足某个 `test` 匹配，则读取对应 `use` 下的 loader 加入到 loaders 数组中。
3. webpack 会将读取到的模块内容交给数组中最后一个 loader 进行处理，最后一个 loader 将处理的结果交给前一个 loader，依此类推，数组中第一个 loader 将最终的处理结果交给 webpack 做 AST 抽象语法树分析。

可以使用 loader 的第三方库 `loader-utils` 操作 loader，使用 npm 安装

```js
const loaderUtils = require("loader-utils");

module.exports = function (sourceCode) {
  const { replaceWithConst } = loaderUtils.getOptions(this); // 得到参数
  const reg = new RegExp(replaceWithConst, "g");
  return sourceCode.replace(reg, "const"); // 将参数替换成 const 返回
};
```

##### plugin

是 webpack 的扩展功能，本质是一个带有 apply 方法的对象，用于在 webpack 的编译流程中嵌入一些功能（就像事件一样，当什么时候去做什么事）

```js
const myPlugin = {
  apply: function (compiler) {},
};
```

- compiler：是在初始化阶段构建的一个对象，在整个打包过程中只有一个，后续的编译和输出工作都是由 compiler 内部创建的 compilation 对象完成的，当重新编译时，会重新创建 compilation 对象完成编译和输出。
- apply：该方法会在 compiler 对象构建完成后自动执行，并把 compiler 对象作为参数传递给它，实际上它是注册钩子函数用的，它的功能就好像 `window.onload` 一样。

通常我们会使用 ES6 中的类来书写，通过构造函数创建一个 plugin

```js
class MyPlugin {
  apply(compiler) {}
}
```

配置文件中加入如下配置

```js
const MyPlugin = require("./plugins/MyPlugin"); // 引入模块

module.exports = {
  plugins: [new MyPlugin(), ...], // 创建对象加入到数组中
};
```

compiler 对象中提供了一个 hooks 对象，其中有大量的钩子函数（可以理解成事件），开发者可以注册这些钩子函数，从而参与 webpack 的编译和输出。可以在 apply 方法中使用下面的代码注册钩子函数，格式如下

```js
class MyPlugin {
  apply(compiler) {
    compiler.hooks.钩子名称.事件类型(name, [async] (compilation, [callback]) => {
      // 事件处理函数
    });
  }
}
```

- 钩子名称：https://www.webpackjs.com/api/compiler-hooks/
- 事件类型：
  - tap：注册一个同步的钩子函数，函数运行完毕则表示事件处理结束。
  - tapAsync：注册一个基于回调的异步钩子函数，事件处理结束会调用这个回调函数。[callback]
  - tapPromise：注册一个基于 Promise 的异步钩子函数，事件处理结束，Promise 进入已决状态。[async]
- name：调试时会使用到，通常填写插件的名字，是一个字符串。

在 compiler 的 beforeRun 钩子中可以为 compilation 注册钩子 `compilation.hooks.xxx`

##### context

入口模块和 loaders 的相对路径会以 context 设置的值作为基准路径

```js
const path = require("path");

module.exports = {
  context: path.resolve(__dirname, "src"),
  // ./index.js 就相当于 ./src/index.js
};
```

##### target

用于配置解析依赖关系时，要在什么环境中解析

```js
module.exports = {
  target: "web", // 浏览器环境
  target: "node", // Node 环境
};
```

其他环境：https://www.webpackjs.com/configuration/target/

##### resolve

这个配置主要是用于控制模块的解析过程

```js
const path = require("path");

module.exports = {
  resolve: {
    // 如果模块路径不是以 ./ 或 ../ 开头时，会从这个配置的目录中寻找模块
    modules: ["node_modules", ...],

    // 如果遇到没有书写后缀名的模块，会使用这个配置测试模块的后缀名
    extensions: [".js", ".json", ...],

    // 为路径配置别名，为了很方便的导入路径过深的模块
    alias: {
      // 使用 @/index.js 就相当于 ./src/index.js 目录
      "@": path.resolve(__dirname, "src"),

      // 使用 _/index.js 就相当于使用工程根目录中的 index.js
      `_`: __dirname,
    },
  },
};
```

##### externals

这个配置会影响第三方库的打包结果，配合页面 cdn 一起使用，最终代码体积会更小，也不会影响源代码的编写。

```js
module.exports = {
  externals: {
    jquery: "$", // 编译结果：module.exports = $
    lodash: "_", // 编译结果：module.exports = _
  },
};
```

##### stats

控制 webpack 构建过程中，控制台的输出内容。

参考文档：https://www.webpackjs.com/configuration/stats/

```js
module.exports = {
  stats: {
    // 配置控制台输出内容
  },
};
```

##### 入口和出口的最佳实践

**一个页面对应一个 JS 文件**

适用于页面与页面之间的功能差异巨大、公共代码较少。

这种情况下生产的最终结果不会有太多的重复代码。

公共代码会导致单个文件体积增加，从而导致传输速度变慢。

```js
const path = require("path");
module.exports = {
  entry: {
    pageA: "./src/pageA/index.js",
    pageB: "./src/pageB/index.js",
    pageC: ["./src/pageC/index1.js", "./src/pageC/index2.js"],
  },

  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].[chunkhash:5].js",
  },
};
```

**一个页面对应多个 JS 文件**

适用于有一个独立的功能，需要被其他模块使用（如：统计的功能）

这种情况下使用一个 chunk 抽离这个独立功能，更有利于浏览器缓存。

```js
const path = require("path");
module.exports = {
  entry: {
    pageA: "./src/pageA/index.js",
    pageB: "./src/pageB/index.js",
    statistics: "./src/statistics/index.js", // 用于统计
  },

  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].[chunkhash:5].js",
  },
};
```

**单页应用**

是指一个网站或网站中的某一块，只有一个页面，页面中的内容全部靠 JS 创建和控制。

```js
const path = require("path");
module.exports = {
  entry: "./src/index.js",

  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "index.[hash:5].js",
  },
};
```

### 区分环境

如果需要针对开发环境和生产环境书写不同的 webpack 配置，这时可以导出一个函数。

在开始构建时，webpack 如果发现导出的是一个函数，则会调用这个函数，将函数返回的对象作为配置对象。

```js
module.exports = (env) => {
  if (开发环境) {
    console.log("开发环境");
    return {
      mode: "development",
      devtool: "source-map",
    };
  } else {
    console.log("生成环境");
    return {
      mode: "production",
    };
  }
};
```

在调用函数时，webpack 会向函数传递一个 env 作为参数，该参数的值来自于 webpack 命令给 env 参数指定的值，在函数中判断值来判断是什么环境即可。

```shell
npx webpack --env abc ## => { abc: true }
npx webpack --env abc bcd ## =>  { abc: true, bcd: true }
```

### plugin - 清空输出目录

插件：clean-webpack-plugin，通过 npm 下载

```shell
npm i -D clean-webpack-plugin
```

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const path = require("path");

module.exports = {
  mode: "development",
  devtool: "source-map",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].[chunkhash:5].js",
  },
  plugins: [new CleanWebpackPlugin()],
};
```

### plugin - 自动生成页面

插件：html-webpack-plugin，通过 npm 下载

```shell
npm i -D html-webpack-plugin
```

更多配置参考 github 地址：https://github.com/jantimon/html-webpack-plugin

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const path = require("path");

module.exports = {
  mode: "development",
  devtool: "source-map",
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "target"),
    filename: "[name].[chunkhash:5].js",
  },
  plugins: [new CleanWebpackPlugin(), new HtmlWebpackPlugin()],
};
```

可以给插件传递一个参数作为配置

```js
new HtmlWebpackPlugin({
  template: "./public/index.html", // 文件模板路径，根据模板样式生成html文件
  chunks: ["index"], // 使用哪个 chunk
  filename: "index.html", // 生成的 html 文件的名字
});
```

会根据总资源列表生成页面。

如果需要根据多个 chunk 生成多个页面，多生成几个 plugin 即可。

### plugin - 复制静态资源

插件：copy-webpack-plugin，通过 npm 下载，适用于在页面上直接书写的的静态资源。

```shell
npm i -D copy-webpack-plugin
```

更多配置参考 github 地址：https://github.com/webpack-contrib/copy-webpack-plugin

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const CopyPlugin = require("copy-webpack-plugin");
const path = require("path");

module.exports = {
  mode: "development",
  devtool: "source-map",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "scripts/[name].[chunkhash:5].js",
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html",
    }),
    new CopyPlugin({
      patterns: [
        {
          from: "./public/img", // 从哪个文件复制
          to: "./img", // 放置到什么地方，相对于输出目录
        },
      ],
    }),
  ],
};
```

### webpack - 开发阶段服务器

webpack 官方制作了一个库：`webpack-dev-server`，可以模拟正常的服务器环境，使用 npm 安装。

文档：https://www.webpackjs.com/configuration/dev-server/

```shell
npm i -D webpack-dev-server ## 安装
```

```shell
npx webpack serve ## 启动工程
```

```json
"scripts": {
  "dev": "webpack serve --open Chrome.exe"
},
```

```shell
npm run dev ## 使用脚本启动工程
```

它支持几乎所有的 webpack 参数，它并且不会生成最终文件，所以真正部署时还需要使用 webpack 命令。

它的主要功能：开启 watch 监听、注册 hooks（保存资源列表，禁止输出文件）、使用 express 开启一个服务器（根据请求路径给予相应的资源）

**配置**

```js
module.exports = {
  devServer: {
    port: 8010, // 配置监听端口，默认8080
    open: true, // 开启浏览器窗口
    index: "index.html", // 设置默认页面
    proxy: {
      // 正则匹配规则，如果请求地址中包含 /api 会进行如下操作
      "/api": {
        target: "http://open.duyiedu.com", // 将 http://localhost:8000/ 替换成这个地址，但是不会修改请求头
        changeOrigin: true, // 修改请求头中的 host 和 origin 字段
      },
      // ... 其他正则匹配规则
    },

    // 配置控制台输出信息，和 webpack 的 stats 配置一样，可以一并书写
    stats: {
      modules: false, // 不会输出模块信息
      colors: true, // 控制台输出有颜色
      // ... 更多配置请见 webpack 的 stats 配置文档。
    },
  },
};
```

### loader - 普通文件处理

使用 npm 安装，适用于使用 JS 添加的静态资源

**file-loader**

将模块导入的内容转换成一个路径，使用 ES6 模块化标准导出默认值（文件路径），并生成依赖的文件到输出目录。

```shell
npm i -D file-loader
```

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpg)|(png)|(jpeg)$/,
        use: [
          {
            loader: "file-loader", // 使用 file-loader
            options: {
              // 文件名规则
              // [name]：原始文件的名字
              // [hash]：根据文件内容生成的 hash
              // [ext]：原始文件的扩展名
              name: "[name].[hash:5].[ext]",
            },
          },
        ],
      },
    ],
  },
};
```

**url-loader**

将模块导入的内容转换成 base64 格式的字符串，使用 ES6 模块化标准导出默认值（base64 编码），不会生成文件到输出目录。

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpg)|(png)|(jpeg)$/,
        use: [
          {
            loader: "url-loader",
            options: {
              // limit：默认为false，全部返回 base64 编码格式
              limit: false,

              // 如果文件超过了设置字节数，交给 file-loader 进行处理
              limit: 5000,
              name: "images/[name].[hash:5].[ext]", // 文件名规则
            },
          },
        ],
      },
    ],
  },
};
```

### 解决路径问题

publicPatch：解决路径问题，通常会配置成一个斜杠 `publicPath: "/"`，表示当前域。

全局配置就配置到 output 选项中，如果针对每个 loader 或 plugin 配置的，那么需要查看是否允许配置。

值是一个字符串，如果是全局配置，那么 webpack 会将该值赋值给 `__webpack_require__.p`，如果某个插件需要，会将 `__webpack_require__.p` 拼接到返回路径的前面，如果是针对每个 loader 或 plugin 配置，loader 或 plugin 会自动将值拼接到路径的前面。

```js
module.exports = {
  output: {
    publicPath: "/", // 全局配置
  },
  // 如果需要为 loader 或 plugin 配置，在 options 中配置
};
```

### webpack 内置插件

所有的 webpack 内置插件都作为 webpack 的静态属性存在，使用下面的格式创建

```js
const webpack = require("webpack");
new webpack.插件名(配置对象);
```

**DefinePlugin**

全局常量定义插件，用于定义全局常量的值

```js
const webpack = require("webpack");

module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      // 常量名：值必须是一个字符串，字符串中的值会作为常量的值
      PI: `Math.PI`, // 相当于 const PI = Math.PI
      VERSION: `"1.1.1"`, // 相当于 cosnt VERSION = "1.1.1"
      DOMAIN: JSON.stringify("www.baidu.com"), // 相当于 const DOMAIN = "www.baidu.com"
      // JSON.stringify("www.baidu.com") => ""www.baidu.com""
    }),
  ],
};
```

经过上面的配置，我们可以在所有模块中使用这些常量。

```js
console.log(PI); // => 3.141592653589793
console.log(VERSION); // => 1.1.1
console.log(DOMAIN); // => www.baidu.com
```

编译结果

```js
(() => {
  console.log("index module");
  console.log(Math.PI);
  console.log("1.1.1");
  console.log("www.baidu.com");
})();
```

**BannerPlugin**

用于为每个 chunk 生成最终文件的头部添加注释，例如：添加作者、公司、版权等信息。

参考文档：https://www.webpackjs.com/plugins/banner-plugin/

```js
const webpack = require("webpack");

module.exports = {
  plugins: [
    new webpack.BannerPlugin({
      banner: ` // 配置注释内容，值表示占位符
        hash:[hash],
        chunkhash:[chunkhash],
        name:[name],
        filebase:[filebase],
        query:[query],
        file:[file]`,
      // ... 等更多配置查看文档
    }),
  ],
};
```

编译结果

```js
/*!
 *
 *         hash:96e633b8ff1a0177fd5f,
 *         chunkhash:e798283f4370e43de6ec,
 *         name:index,
 *         filebase:index.e7982.js,
 *         auther:surongbao,
 *         file:index.e7982.js
 */
(() => {
  console.log("index module");
})();
```

**ProvidePlugin**

会自动加载模块，而不必到处使用 `import` 或 `require` 导入模块，对于经常使用的模块，这种方法很方便。

```js
module.exports = {
  plugin: [
    new webpack.ProvidePlugin({
      $: "jquery", // 就相当于 const $ = require("jquery")
      _: "lodash", // 就相当于 const _ = require("lodash")
    }),
  ],
};
```

模块中使用

```js
console.log($); // => jqueryFunction
console.log(_); // => lodashFunction
```

### 第三方库

- query-string：用于地址栏解析

### CSS 工程化问题

**问题**

- 类名冲突：类名过深：会导致难以书写、阅读、压缩、复用；类名过浅：会导致类名冲突。
- 重复样式值：重复的样式值被应用到不同的 CSS 代码中，导致代码难以维护。
- CSS 模块化：精细的 CSS 模块化划分，更有利于 CSS 代码的维护，和 JS 文件一样。

**如何解决**

- 类名冲突：BEM、OOCSS、AMCSS、SMACSS 等规范，css in js，css module。
- 重复样式值：css in js，CSS 预编译器：less、sass。
- CSS 模块化：webpack 利用一些 loader、plugin 实现打包、合并、压缩 CSS 文件。

### 解决 CSS 模块化的问题

**css-loader**

用于将 CSS 代码转换成 JS 代码

原理：将 CSS 代码作为字符串导出，如果有依赖通过 require 导入。

```css
.red {
  color: red;
}
```

导出样式

```js
module.exports = `.red { color: red; }`;
```

如果 CSS 文件中依赖另一个模块

```css
.red {
  color: red;
  background-image: url("./webpack.jpg");
}
```

导出样式

```js
// webpack 会报错，可以交给 file-loader 或 url-loader 进行处理
const result = require("./webpack.jpg");

module.exports = `.red {
  color: red;
  background-image: url("${result}");
}
`;
```

**style-loader**

可以将 css-loader 导出的结果加入到页面 head 元素的 style 元素中，有模块缓存

### 解决类名冲突的问题

##### BEM

它认为一个完整的类名应该包括 `总区域__小区域_[状态]`，

里面的类名全部都是顶级类名，后面可以跟元素，但是不能跟类名，这种命名规则有利于压缩

可以使用一些前缀做更大范围的区分

- l：layout（用于布局） `l-总区域__小区域_[状态]`
- c：component（用于组件）`c-总区域__小区域_[状态]`
- u：util（用于通用样式） `u-总区域__小区域_[状态]`
- j：javascript（用于提供给 JS 获取元素） `j-总区域__小区域_[状态]`

```html
<body>
  <div class="banner__container">
    <ul class="banner__imagecontainer">
      <img
        src="https://img.alicdn.com/imgextra/i3/6000000007233/O1CN01StS00B23IlP00J7VQ_!!6000000007233-0-octopus.jpg"
        alt=""
      />
      <img
        src="https://img.alicdn.com/imgextra/i3/6000000007233/O1CN01StS00B23IlP00J7VQ_!!6000000007233-0-octopus.jpg"
        alt=""
      />
      <img
        src="https://img.alicdn.com/imgextra/i3/6000000007233/O1CN01StS00B23IlP00J7VQ_!!6000000007233-0-octopus.jpg"
        alt=""
      />
    </ul>
    <div class="banner__dotcontainer">
      <span
        class="banner__dotcontainer_dot banner__dotcontainer_activedot"
      ></span>
      <span class="banner__dotcontainer_dot"></span>
      <span class="banner__dotcontainer_dot"></span>
    </div>
  </div>
</body>
```

```css
.banner__container {
  width: 520px;
  height: 280px;
  border: 1px solid;
  margin: 10px auto;
  overflow: hidden;
  position: relative;
}

.banner__imagecontainer {
  margin: 0;
  padding: 0;
  width: 1560px;
  height: 280px;
}

.banner__imagecontainer img {
  width: 520px;
  height: 280px;
  float: left;
}

.banner__dotcontainer {
  position: absolute;
  bottom: 10px;
  left: 50%;
  transform: translate(-50%);
  background: rgba(0, 0, 0, 0.5);
  padding: 5px 10px;
  border-radius: 8px;
  display: flex;
}

.banner__dotcontainer_dot {
  width: 10px;
  height: 10px;
  background: #fff;
  margin: 0 10px;
  border-radius: 50%;
}

.banner__dotcontainer_activedot {
  background: #f40;
}
```

##### css in js

抛弃了 CSS 样式表，使用 JS 对象描述样式，然后将样式直接应用到元素的 stryle 中。

由于 JS 中根本不存在类名，也就没有类名冲突可言（ES6 中变量重名就报错了）

```js
const item1 = document.querySelector(".item1");
const item2 = document.querySelector(".item2");

const styles = {
  background: "#f40",
  color: "#fff",
  width: "100px",
  height: "100px",
  border: "1px solid #ccc",
};

function useStyles(dom, styles) {
  for (const key in styles) {
    dom.style[key] = styles[key];
  }
}

useStyles(item1, styles);
useStyles(item2, styles);
```

**特点**

- 类名不可能冲突，因为 JS 中不存在类名。
- 更加灵活，能玩出很多花样。
- 应用面更加广泛，当使用 JS 语言开发移动端时会非常好用（移动端很可能不支持 CSS）
- 书写代码不是很方便，特别是公共样式
- 由于使用行间样式，会造成页面上有大量内容，且会存在很多的重复代码，不易阅读最终的页面代码。

##### css module

CSS 模块化方案，编写简单，且不可能重名。

使用 BEM 命名规范，命名太过死板，而 css in js 虽然足够灵活，但是书写不便。

它解决不同模块之间的类名冲突，而不解决同模块下的类名冲突

css module 只保证合并后的代码不出现类名冲突。

思路：往往类名冲突发生在大型项目中，而大型项目会用到构建工具，构建工具又允许将 CSS 切分成精细的模块，并且类名冲突往往发生在不同的模块之间，所以只需要解决合并后的代码没有类名冲突就可以了。

css-loader 实现了 css module 的思想，默认没有启用，如果想启用只需要配置 `modules: true` 即可。

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        // use: ["style-loader", "css-loader?modules=true"], // 可以这么写
        // use: ["style-loader", "css-loader?modules"], // 也可以这么写
        use: [
          "style-loader",
          {
            loader: "css-loader",
            // 也可以这么写
            options: {
              modules: true,
            },
          },
        ],
      },
    ],
  },
};
```

开启 css module 后，css-loader 会根据模块路径和类名生成一个 hash 字符串（该 hash 字符串不可能相同），将 hash 字符串作为类名。

```html
<style>
  ._5dEFEDsqqI33xOW-3a3do {
    color: red;
  }
</style>
```

但是经过上面的转换后，虽然类名不冲突了，但是我们不知道最终生成的类名是什么，如何应用到元素上呢？

css-loader 同时会导出一个对象，该对象中记录了转换后的类名和源码中类名的对应关系，在返回对象的 locals 属性中

style-loader 原来返回一个空对象，这时这个空对象中就保存了是这个对应关系表。

```js
{
  green: "_32Nmt2BtVeUst78k2LNG5H";
  red: "_5dEFEDsqqI33xOW-3a3do";
}
```

然后通过这种方式添加类名

```js
import style from "./assets/test.css";
document.querySelector("div").className = style.green;
```

对于某些全局类名不需要转换，请使用下面的语法，对应表中也不存这个类名

```html
<style>
  /* 不需要转换的 */
  :global(类名) {
    /* 样式代码 */
  }

  /* 默认是局部的，正常书写即可，或使用下面的语法 */
  :local(类名) {
    /* 样式代码 */
  }
</style>
```

如果需要控制最终生成的类名，可以给 css-loader 配置 `localIdentName` 参数，往往不需要控制

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          "style-loader",
          {
            loader: "css-loader",
            options: {
              modules: {
                localIdentName: "[name]-[hash:3]", // 配置类名
              },
            },
          },
        ],
      },
    ],
  },
};
```

**总结**

- 往往配合构建工具使用
- 仅处理顶级类名，尽量不要书写嵌套类名，也没必要书写嵌套类名
- 仅处理类名，不处理其他选择器（虽然可以处理 id 选择器，但是没必要使用 id 选择器）
- 只需要类名 “见名知义” 即可，不需要遵守其他规范

### 解决样式值重复的问题 - 预编译器

##### 预编译器原理

可以解决重复的样式值，重复的代码段，重复的嵌套书写等问题。

预编译器：通过一种优雅的方式编写代码（这种代码浏览器不能识别），然后通过预编译器将其转换成传统的 CSS 代码。

- less 官网：http://lesscss.org/
- less 中文文档 1（非官方）：http://lesscss.cn/
- less 中文文档 2（非官方）：https://less.bootcss.com/
- sass 官网：https://sass-lang.com/
- sass 中文文档 1（非官方）：https://www.sass.hk/
- sass 中文文档 2（非官方）：https://sass.bootcss.com/

##### Less 的使用

通过 npm 安装 less 编译器

```shell
npm i -D less
```

它提供了一个 CLI 命令 `lessc`，使用该命令即可完成编译

```shell
lessc 使用less编写的文件 最终的css文件
```

**示例**

创建 less 文件，并书写 less 代码，文件内容如下

```less
@red: #f40;

.item {
  color: @red;
}
```

进入 less 目录执行 cli 命令

```shell
PS F:\webpack\23.less\less> npx lessc index.less index.css
## 进入 less 文件所在目录，本地安装使用 npx 命令，对 index.less 进行编译，最终生成 index.css 文件
```

最终生成的 css 文件内容

```css
.item {
  color: #f40;
}
```

**变量**

less 文件内容

```less
@width: 10px; // 变量
@height: @width + 20px; // 变量
@color: red; // 变量

#header {
  width: @width;
  height: @height;
  border: 1px solid @color;
}
```

编译后的 css 文件内容

```css
#header {
  width: 10px;
  height: 30px;
  border: 1px solid red;
}
```

**混合**

less 文件内容

```less
// 混和加上小括号，不会把混入编译到 css 文件
// 该小括号可以想象成一个函数
// 参数可以是多个，可以进行传递参数和参数默认值等操作
.setborder() {
  border: 3px solid #ccc;
}

#header {
  width: 100px;
  height: 100px;
  .setborder(); // 混合
}
#footer {
  width: 100px;
  height: 100px;
  .setborder(); // 混合
}
```

css 文件内容

```css
#header {
  width: 100px;
  height: 100px;
  border: 3px solid #ccc;
}
#footer {
  width: 100px;
  height: 100px;
  border: 3px solid #ccc;
}
```

**嵌套**

less 文件内容

```less
#header {
  color: black;

  // 相当于 #header .navigation
  .navigation {
    font-size: 12px;

    // 相当于 #header .navigation > .item
    > .item {
      color: red;
    }

    // 相当于 #header .navigation.nav
    &.nav {
      text-align: center;
    }
  }

  // 相当于 #header .logo
  .logo {
    width: 300px;
  }

  // 相当于 #header::after
  &::after {
    content: "";
    display: block;
  }
}
```

css 文件内容

```css
#header {
  color: black;
}
#header .navigation {
  font-size: 12px;
}
#header .navigation > .item {
  color: red;
}
#header .navigation.nav {
  text-align: center;
}
#header .logo {
  width: 300px;
}
#header::after {
  content: "";
  display: block;
}
```

**运算**

less 文件内容

```less
@width: 20px;

#header {
  width: @width * 10;
  height: @width + 100px;
  font-size: @width + 1em;
}
```

css 文件内容

```css
#header {
  width: 200px;
  height: 120px;
  font-size: 21px;
}
```

**函数**

参考 less 函数手册：https://less.bootcss.com/functions/#less-%E5%87%BD%E6%95%B0

**作用域**

和 JS 中作用域一样，外界无法使用

```less
#header {
  // header 中声明的变量，只有该作用域可以使用
  @width: 100px;

  width: @width;
  .item {
    width: @width;
  }
}

.abc {
  width: @width; // 报错，@width is undefined
}
```

**注释**

推荐使用 less 注释：`// 注释内容`

less 文件内容

```less
#header {
  /* 我是一个注释 */ // css注释会被编译到编译结果中
  @width: 100px;

  // 我也是一个注释 // less注释不会编译到编译结果中
  width: @width;
  .item {
    width: @width;
  }
}
```

css 文件内容

```css
#header {
  /* 我是一个注释 */
  width: 100px;
}
#header .item {
  width: 100px;
}
```

**导入**

和 css 导入一样，类似于 css 模块化，如果两个文件中存在变量冲突，则按照导入顺序，后面覆盖前面的。

```less
@import "./assets/index"; // 导入 index.less
@import "./assets/index.css"; // 导入 index.css，会把导入的语句直接加入到最终的 css 文件中
```

公共样式

```less
@color: red;
@bgcolor: green;

.center(@type:absolute) {
  position: @type;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
```

less 文件

```less
@import "./common"; // 导入less
@import "./abc.css"; // 导入css

#header {
  color: @color;
  width: 100px;
  height: 100px;
  background: @bgcolor;
  .center();
}
```

css 文件内容

```css
@import "./abc.css"; /* 直接加入到编译结果 */

#header {
  color: red;
  width: 100px;
  height: 100px;
  background: green;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
```

##### webpack 中使用 less

只需要安装 less 和 less-loader 即可，使用 css-loader style-loader less-loader。

```shell
npm i -D less less-loader css-loader style-loader
```

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: ["style-loader", "css-loader?modules", "less-loader"],
      },
    ],
  },
};
```

### PostCss

官网：https://postcss.org/

它的功能类似于 webpack，可以将 CSS 源码转换成最终的 CSS 代码，仅做依赖分析，抽象语法树分析，其他的操作交给 loader 和 plugin 完成。

##### 安装和使用

它基于 Node 编写，使用 npm 安装

```shell
npm i -D postcss
```

postcss 库提供了相应的 api 用于转换代码，如果你想使用 postcss 的高级功能或想开发 postcss 插件，可以参考文档：https://postcss.org/api/。

不过绝大多数情况下，我们并不希望使用代码的方式来使用 postcss，这时，可以安装 `postcss-cli`，然后通过命令来完成编译

```shell
npm i -D postcss-cli
```

postcss-cli 提供了一个 cli 命令，它可以调用 postcss 中的 api 来完成编译

```shell
npx postcss 源码文件路径 -o 最终代码文件路径
```

文件后缀名可以使用：`.css` 或 `.postcss` 或 `.pcss`

```json
"scripts": {
  "dev": "postcss ./css/index.pcss -o ./css/index.css -w"
},
```

##### 配置文件

和 webpack 类似，也有自己的配置文件 `postcss.config.js`，会影响 postcss 的编译行为

```js
module.exports = {
  map: false, // 不使用源码地图，

  map: {
    inline: false, // 将源码地图生成一个文件，css文件中只显示文件路径
  },
};
```

##### 插件

文档：https://www.postcss.parts/

**postcss-preset-env**

设置 postcss 预设环境，它整合了很多插件，并完成了基本配置

```shell
npm i -D postcss-preset-env
```

```js
module.exports = {
  map: false,
  // plugins可以是对象，也可以是数组
  plugins: {
    "postcss-preset-env": {
      // 插件配置
    },
  },
};
```

这个插件的功能：自动添加厂商前缀和未来的 CSS 语法

- 自动添加厂商前缀

postcss 中内容

```css
::placeholder {
  color: red;
}
```

转换后的 css 文件内容

```css
::-moz-placeholder {
  color: red;
}

:-ms-input-placeholder {
  color: red;
}

::placeholder {
  color: red;
}
```

该功能是依靠 `autoprefixer` 库完成

如果需要设置浏览器的兼容范围，可以有三种方案：

方案 1【不推荐】：在 postcss-preset-env 配置中设置 browsers 选项

```js
module.exports = {
  map: false,
  plugins: {
    "postcss-preset-env": {
      // 插件配置
      browsers: [
        "last 2 version", //
        "> 1%",
      ],
    },
  },
};
```

方案 2【推荐】：在项目根目录中创建 `.browserslistrc` 文件，进行如下配置

因为别的解决浏览器兼容性的东西也会用到这个文件，这是一种通用的方式

```js
last 2 version
> 1%
```

方案 3【推荐】：在 package.json 文件中使用 browserslist 字段进行配置

```json
"browserslist": [
  "last 2 version",
  "> 1%"
]
```

browserslist 是一个多行的（方案 2）或数组形式的（方案 3）标准字符串

书写规范：https://github.com/browserslist/browserslist

通常做如下配置（三种条件的并集）

```js
last 2 version // 兼容浏览器最近的两个版本
> 1% in CN // 匹配在中国1%的人使用的浏览器，in CN 可以省略
not ie <= 8 // 排除 IE8 以下的浏览器
```

browserslist 的数据来源于 CanIUse

- 未来的 CSS 语法

如果需要使用 CSS 未成为标准（草案阶段或实验性的）语法，可以进行兼容性处理，这个插件依靠 `cssnext` 库完成。

可以通过 `stage` 字段进行配置，告诉插件需要对哪个阶段的 CSS 语法进行兼容性处理，默认值：2

```js
module.exports = {
  map: false,
  plugins: {
    "postcss-preset-env": {
      stage: 0, // 早期草案阶段，功能极其不稳定
      stage: 1, // 已被W3C认可，功能仍然及其不稳定
      stage: 2, // 可以使用了，虽然功能还是不稳定
      stage: 3, // 功能比较稳定了，可能会发生一些小的变化，即将成为最终标准
      stage: 4, // 所有主流浏览器都已经支持了
    },
  },
};
```

定义变量

在 `:root{}` 中使用 `--变量名: 值` 语法定义变量，在元素中使用 `var` 语法使用变量。

```css
:root {
  --lightColor: #ddd;
  --darkColor: #333;
}

a {
  color: var(--lightColor);
  background: var(--darkColor);
}
```

编译结果

```css
:root {
  --lightColor: #ddd;
  --darkColor: #333;
}

a {
  color: var(--lightColor);
  background: var(--darkColor);
}
```

编译后，在编译结果中仍然可以看到上面书写的代码，这是因为有些浏览器支持这种语法（如：最新版的 Chrome），并且不认识这种代码的浏览器也不会影响该浏览器的渲染，如果不希望上面的代码被编译进打包结果，可以使用 `preserve: false` 禁用。

配置文件

```js
module.exports = {
  map: false,
  plugins: {
    "postcss-preset-env": {
      preserve: false, // 禁止输出前沿语法
    },
  },
};
```

编译结果

```css
a {
  color: #ddd;
  background: #333;
}
```

自定义选择器

未使用成功

```css
@custom-selector :--heading h1, h2, h3, h4, h5, h6;
@custom-selector :--enter :focus, :hover;

a:--enter {
  color: #f40;
}

:--heading {
  font-weight: bold;
}

:--heading.active {
  font-weight: bold;
}
```

编译结果有问题，以后再来解决，结果应该如下

```css
a:focus,
a:hover {
  color: #f40;
}

h1,
h2,
h3,
h4,
h5,
h6 {
  font-weight: bold;
}

h1.active,
h2.active,
h3.active,
h4.active,
h5.active,
h6.active {
  font-weight: bold;
}
```

嵌套

与 LESS 相同，只不过嵌套的选择器前必须使用符号 `&`

未使用成功

```css
.a {
  color: red;
  & .b {
    color: green;
  }

  & > .b {
    color: blue;
  }

  &:hover {
    color: #000;
  }
}
```

编译结果

```css
.a {
  color: red;
}

.a .b {
  color: green;
}

.a > .b {
  color: blue;
}

.a:hover {
  color: #000;
}
```

**postcss-apply**

该插件可以支持在 css 文件中书写属性集，类似 less 中的混入，目前没有受到 `postcss-preset-env` 的支持，需要在配置文件中进行配置

未使用成功，

```css
:root {
  --center: {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
}

.item {
  @apply --center;
}
```

编译结果

```css
.item {
  position: absolute;
  left: 50%;
  top: 50%;
  -webkit-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
}
```

**postcss-color-function**

该插件可以支持在源码中使用一些颜色函数，需要在配置文件中进行配置

```css
body {
  /* 使用颜色#aabbcc，不做任何处理，等同于直接书写 #aabbcc */
  color: color(#aabbcc);
  /* 将颜色#aabbcc透明度设置为90% */
  color: color(#aabbcc a(90%));
  /* 将颜色#aabbcc的红色部分设置为90% */
  color: color(#aabbcc red(90%));
  /* 将颜色#aabbcc调亮50%（更加趋近于白色），类似于less中的lighten函数 */
  color: color(#aabbcc tint(50%));
  /* 将颜色#aabbcc调暗50%（更加趋近于黑色），类似于less中的darken函数 */
  color: color(#aabbcc shade(50%));
}
```

编译结果

```css
body {
  /* 使用颜色#aabbcc，不做任何处理，等同于直接书写 #aabbcc */
  color: rgb(170, 187, 204);
  /* 将颜色#aabbcc透明度设置为90% */
  color: rgba(170, 187, 204, 0.9);
  /* 将颜色#aabbcc的红色部分设置为90% */
  color: rgb(230, 187, 204);
  /* 将颜色#aabbcc调亮50%（更加趋近于白色），类似于less中的lighten函数 */
  color: rgb(213, 221, 230);
  /* 将颜色#aabbcc调暗50%（更加趋近于黑色），类似于less中的darken函数 */
  color: rgb(85, 94, 102);
}
```

**postcss-import**

可以在 postcss 文件中导入其他样式代码，通过该插件将他们合并，它做的事 webpack 已经做了，所以这个插件了解即可

**stylelint**

和 ESlint 功能类似，在书写代码时，可能会书写一些不符合规范的代码，它会进行提示。

它本身并没有提供具体的规则验证，可以依靠 `stylelint-config-standard` 来完成

```shell
npm i -D stylelint stylelint-config-standard
```

需要在 `.stylelintrc` 文件中进行配置，该文件时 json 格式的

```json
{
  "extends": "stylelint-config-standard"
}
```

还需要在 postcss 配置文件中配置

```js
module.exports = {
  map: false,
  plugins: {
    stylelint: {},
  },
};
```

如果需要对某个规则进行详细的配置，文档：https://stylelint.io/

```json
{
  "extends": "stylelint-config-standard",
  "rules": {
    "规则名": null // 禁用规则
  }
}
```

如果需要书写代码时就提示错误，可以使用 vscode 插件 `stylelint` 完成

### webpack - 抽离 CSS 文件

css-loader 将转换后的代码交给 style-loader 处理，style-loader 将代码加入到页面的 style 元素中。

如果需要生成 css 文件，需要使用 `mini-css-extract-plugin` 库完成。

该库提供了一个 loader 和一个 plugin

loader：负责记录要生成的 CSS 文件内容，同时导出开启 css module 后的样式对象，这时就不需要 style-loader 了

plugin：负责生成 CSS 文件。

配置文件

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  output: {
    publicPath: "/",
  },

  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, "css-loader?modules"],
      },
    ],
  },

  plugins: [
    new MiniCssExtractPlugin({
      filename: "css/[name].[contenthash:5].css",
    }),
  ],
};
```

如果依赖多个 css 文件，和 js 的处理方法一样，将多个 css 文件的内容合并到一个 css 文件中

```js
import styles from "./assets/index.css";
import "./assets/a.css";
import styles2 from "./assets/b.css";
```

生成文件

```css
.fbe0ATlXu9vq1YraTNHyS {
  width: 300px;
  height: 300px;
  margin: 0 auto;
  background-image: url(/./img/78371.jpg);
  background-size: 100% 100%;
}

._1RCvshM67G9AfgBvv8qF9D {
  color: red;
}

._1s1qLDwqUQOEwhqRtZFwcy {
  color: blue;
}
```

如果时多入口，和生成 js 文件的规则一样，根据 chunk 生成多个 css 文件。

### JS 兼容性

babel：巴别塔，时一个编译器，类似于 webpack、postcss，可以将不同标准（ES 不同版本）的语言，编译成兼容性的语言。

babel 本身只提供了一些基本的分析功能，具体怎么转换需要依靠插件来完成。

##### babel 的安装

babel 可以和构建工具一起联用，也可以独立使用。

如果需要独立使用，需要安装下面两个库

- @babel/core：babel 的核心库，提供了编译过程中所使用到的所有 babelAPI
- @babel/cli：提供了一个命令，可以调用 babelAPI 来完成编译

```shell
npm i -D @babel/core @babel/cli
```

##### babel 的使用

```shell
npx babel 需要编译的文件 -o 最终编译结果放置的文件 # 用于编译文件
npx babel 需要编译的目录 -d 最终编译结果放置的目录 # 用于编译目录
```

```shell
npx babel js/a.js -o js/b.js -w # -w 监测文件的变化
npx babel js -d scripts # 将js目录下的所有文件编译到scripts目录中
```

##### babel 的配置

babel 本身只提供了一些分析的功能，真正的编译工作是由 “babel 的插件” 和 “babel 的预设” 来完成的

babel 的预设和 postcss-preset-env 差不多，都是指多个插件的集合，用于解决一些常见的兼容问题。

babel 的配置文件 `.babelrc`

```json
{
  // 需要使用的预设，如果使用多个预设，和loader一样的执行顺序，从数组的最后一项开始
  "presets": ["@babel/preset-env"],
  "plugins": [] // 需要使用的插件
}
```

##### babel 的预设

`@babel/preset-env`：可以使用最新的 JS 语法书写，无需对每种语法的转换设置具体的插件。

源码文件

```js
const a = 1;
```

编译结果文件

```js
"use strict";
var a = 1;
```

也可以设置浏览器的兼容范围，使用 `.browserslistrc` 文件配置

```json
last 2 version
> 1%
not ie <= 8
```

和 `postcss-preset-env` 一样，该预设也可以进行自身的配置：https://www.babeljs.cn/docs/babel-preset-env

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "配置项": "配置的值"
      }
    ]
  ]
}
```

其中有一个配置项 `useBuiltIns`，默认值 false，（babel 默认表示仅转换新语法）不会对新的 API 进行处理，例如 Promise

```js
const pro = new Promise((resolve, reject) => {
  if (Math.random() < 0.5) {
    resolve(true);
  } else {
    resolve(false);
  }
});
const a = 2 ** 3;
```

转换结果

```js
"use strict";

// API 没有进行转换
var pro = new Promise(function (resolve, reject) {
  if (Math.random() < 0.5) {
    resolve(true);
  } else {
    resolve(false);
  }
});
var a = Math.pow(2, 3); // 新语法转换了
```

可以配置 `"useBuiltIns": "useage"`，则会按需（需要哪个 API 导入哪个）构建新的 API

配置文件

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage" // 按需构建新的API
      }
    ]
  ]
}
```

编译结果文件

```js
"use strict";

// core-js：该库包含了所有新API的实现
require("core-js/modules/es6.promise.js");
require("core-js/modules/es6.object.to-string.js");

var pro = new Promise(function (resolve, reject) {
  if (Math.random() < 0.5) {
    resolve(true);
  } else {
    resolve(false);
  }
});
var a = Math.pow(2, 3);
```

代码还虽然转换了，但是还是不能使用，应为使用 Promise 时导入了一个库 `core-js`，需要安装

```shell
npm i core-js
```

core-js 编写时的版本是 2.xxx.xxx 版本，安装的是 3.xxx.xxx 版本，需要配置一下

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": 3 // 使用 core-js 的主版本是 3
      }
    ]
  ]
}
```

如果需要使用 async 和 await 语法，还需要安装 `regenerator-runtime` 库

```shell
npm i regenerator-runtime
```

##### babel 的插件

插件：和预设一样，也是转换代码用的，在预设之前运行，如果有多个插件，按照数组的先后顺序运行（从左到右）

通常情况下，预设只转换那些已成为正式标准的语法和 API，对于更早阶段的语法或 API，需要使用插件转换。

更多插件参考：https://www.babeljs.cn/docs/plugins

**@babel/plugin-proposal-class-properties**

该插件可以让你在类中书写初始化字段

```shell
npm i -D @babel/plugin-proposal-class-properties
```

配置文件

```json
{
  // "plugins": ["@babel/plugin-proposal-class-properties"] // 简写
  "plugins": [
    [
      "@babel/plugin-proposal-class-properties",
      {
        // 插件的配置
        "loose": true // 编译时宽松一点
      }
    ]
  ]
}
```

源码文件

```js
class Test {
  a = 1; // 初始化字段
  constructor(value = 10) {
    this.value = value;
  }
}
```

编译结果文件

```js
"use strict";

function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

function _defineProperty(obj, key, value) {
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerable: true,
      configurable: true,
      writable: true,
    });
  } else {
    obj[key] = value;
  }
  return obj;
}

var Test = function Test() {
  _classCallCheck(this, Test);

  _defineProperty(this, "a", 1);

  this.value = 10;
};
```

**@babel/plugin-proposal-function-bind**

该插件可以让你为某个方法轻松的绑定 this

```shell
npm i -D @babel/plugin-proposal-function-bind
```

配置文件

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ]
  ],
  "plugins": ["@babel/plugin-proposal-function-bind"]
}
```

源码文件

```js
function print() {
  console.log(this.value);
}

const obj = {
  value: 123,
};

obj::print(); // 相当于 print.call(obj)
```

编译结果文件

```js
"use strict";

function print() {
  console.log(this.value);
}

var obj = {
  value: 123,
};
print.call(obj);
```

**@babel/plugin-proposal-optional-chaining**

该插件用于安全的读取某个对象的属性。如果某个属性不存在，直接返回 undefined，不会报错。

以前的写法，如果读取 null 或 undefined 中的属性会报错

```shell
npm i -D @babel/plugin-proposal-optional-chaining
```

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ]
  ],
  "plugins": ["@babel/plugin-proposal-optional-chaining"]
}
```

```js
const obj = {
  a: {
    b: {
      c: {
        d: 100,
      },
    },
  },
};

const d = obj?.a?.b?.c?.d;
console.log(d); // 100

const d = obj?.e;
console.log(d); // undefined
```

如果上面用普通语法书写会写成下面这种代码，很麻烦

```js
const d = obj && obj.a && obj.a.b && obj.a.b.c && obj.a.b.c.d;
```

**babel-plugin-transform-remove-console**

去除编译结果中所有的输出语句

```shell
npm i -D babel-plugin-transform-remove-console
```

这是一个老的插件，可以不使用前缀书写配置文件

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ]
  ],
  // "plugins": ["babel-plugin-transform-remove-console"],
  "plugins": ["transform-remove-console"] // 简写
}
```

**@babel/plugin-transform-runtime**

该插件用于提供一些公共的 API，这些 API 会帮助代码转换（例如在转换时需要依赖一些函数，这个库中都有），依赖 `@babel/runtime` 库

```shell
npm i -D @babel/plugin-transform-runtime
npm i @babel/runtime
```

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ]
  ],
  "plugins": [
    "@babel/transform-runtime" // 带有命名空间的库可以简写
  ]
}
```

编译结果文件，从 `@babel/runtime` 库中导入的

```js
var _classCallCheck2 = _interopRequireDefault(
  require("@babel/runtime/helpers/classCallCheck")
);
```

没有这个插件的编译结果文件。在文件中书写的函数，文件多了会造成重复代码

```js
function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}
```

##### webpack 中使用 babel

```shell
npm i -D @babel/core babel-loader
```

webpack.config.js 配置文件

```js
module.exports = {
  mode: "development",
  devtool: "source-map",

  module: {
    rules: [{ test: /\.js$/, use: "babel-loader" }],
  },
};
```

`.babelrc` 配置文件

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ]
  ],
  "plugins": ["@babel/plugin-transform-runtime"]
}
```

依赖的包

```json
"devDependencies": {
    "@babel/core": "^7.12.16",
    "@babel/plugin-transform-runtime": "^7.12.15",
    "@babel/preset-env": "^7.12.16",
    "babel-loader": "^8.2.2",
    "webpack": "^5.22.0",
    "webpack-cli": "^4.5.0"
  },
  "dependencies": {
    "@babel/runtime": "^7.12.13",
    "core-js": "^3.8.3",
    "regenerator-runtime": "^0.13.7"
  }
```

> 转换 async 和 await 时，需要将他们转换成生成器，而生成器也是新语法，也需要转换，转换生成器就需要 regenerator-runtime 这个库的支持，这个库是通过 “迭代器+可迭代协议+状态机” 实现的。

### 性能优化

性能包括：

- 构建性能：开发阶段打包的时间
- 传输性能：JS 文件从服务器到客户端的时间（文件的总传输量、文件数量、利用浏览器缓存）
- 运行性能：JS 代码在浏览器端的运行速度

> 不要过早的关注性能，怎么舒服、怎么好维护、怎么好阅读就怎么写，性能优化是以后的事。

##### 构建性能优化

**减少模块的解析**

模块解析包括：AST 抽象语法树分析、分析依赖（记录依赖到 dependencies 中）、替换导入函数这三部分。

如果不解析模块，那么 loader 处理的结果就是最终文件中的代码，如果没有使用 loader，那么源码就是最终文件中的代码，

总结：如果不对模块解析，可以提高构建性能。

不依赖其他模块的模块不需要解析，没有 require 或 import 的第三方库：如 jquery

```js
module.exports = {
  mode: "development",
  module: {
    noParse: /jquery/, // 值是一个正则表达式，表示不解析 jquery 库
  },
};
```

> 最终结果中还是包括了 jquery，只是不要解析，因为里面没有依赖，没有必要浪费性能

**优化 loader 性能**

- Markdown 文件在浏览器打开有侧边目录

文件 => 首选项 => 设置 => `Markdown Preview Enhanced` => `Markdown-preview-enhanced: Enable Script Execution` 选中，表示开启脚本

- Markdown 预览中的图片设置

1. 按 F1，输入：`Markdown Preview Enhanced: Customize Css` 命令，在 less 文件中书写如下代码

```less
/* Please visit the URL below for more information: */
/*   https://shd101wyy.github.io/markdown-preview-enhanced/#/customize-css */

.markdown-preview.markdown-preview {
  // modify your style here
  // eg: background-color: blue;
  font-family: "consolas", "Noto Sans S Chinese";
  font-size: 1em;
}

.markdown-img-description {
  text-align: center;
  margin-top: -1em;
  color: #666;
  margin-bottom: 2em;
}

html body img {
  border: 2px solid #ccc;
}

.markdown-p-center {
  text-align: center;
}
```

2. 然后按 F1，输入 `Markdown Preview Enhanced: Extend Parser` 命令，在 JS 文件中书写如下代码

```js
const scripts = `
<script>
    function setCurrent(){
        const links = document.querySelectorAll(".md-sidebar-toc a")
        for(const link of links){
            link.style.color="";
        }
        const hash = location.hash;
        const a = document.querySelector('a[href="'+hash+'"]');
        if(a){
            a.style.color = "#f40";
        }
    }
    setCurrent();
    window.onhashchange = setCurrent;
</script>
`;
var fs = require("fs");
module.exports = {
  onWillParseMarkdown: function (markdown) {
    return new Promise((resolve, reject) => {
      const reg = /\!\[(.*)\]\((\S+)\)/gm;
      markdown = markdown.replace(reg, function (match, g1, g2) {
        var width = "100%";
        if (g1) {
          var w = g1.split("|");
          if (w.length > 1) {
            width = w[1] + "px";
            g1 = w[0];
          }
        }
        return `
<p class="markdown-p-center">
  <img src="${g2}" alt="${g1}" style="max-width:${width}"/>
</p>
<p class="markdown-img-description">
  ${g1}
</p>
  `;
      });
      resolve(markdown);
    });
  },
  onDidParseMarkdown: function (html) {
    return new Promise((resolve, reject) => {
      return resolve(scripts + html);
    });
  },
};
```

经过上面两步，效果如下

![loader的执行过程](24.性能优化/优化loader性能/2020-02-21-13-32-36.png)

> 言归正传，下面学习优化 loader 的性能

1. 限制 loader 的应用范围（对某些库不使用 loader）

例如：babel-loader 可以将 ES6 及更高版本的语法转换成 ES5 语法，可是有些库本来就使用 ES5 语法编写，这时就不需要转换，转换反而浪费构建的时间。

例如：lodash 是使用 ES3 的语法编写的，这时就不需要转换了。

- 使用 `module.rules.exclude` 排除不需要使用 loader 的场景，使用正则匹配
- 使用 `module.rules.include` 包含需要应用 loader 的场景，使用正则匹配

需要使用 `@babel/preset-env` 和 `core-js`

```js
module.exports = {
  mode: "development",
  devtool: "source-map",

  module: {
    rules: [
      {
        test: /\.js$/,
        // exclude: /lodash/, // 排除掉lodash
        // exclude: /node_modules/, // 排除掉整个 node_modules 目录
        include: /src/, // 只编译 src 目录
        use: "babel-loader",
      },
    ],
  },
};
```

> 这种做法是对 loader 的限制，与 noParse 不冲突，因为 npParse 不经过 loader。

1. 缓存 loader 的结果

假设：如果某个文件的内容不变，那么经过 loader 处理后的结果也不变

如果假设成立，那么我们可以把第一 loader 处理的结果缓存起来，后续再编译就不要编译这个文件了

`cache-loader` 可以实现这个功能，只需要把 `cache-loader` 放到数组的第一项即可

```shell
npm i cache-loader
```

```js
module.exports = {
  mode: "development",
  devtool: "source-map",

  module: {
    rules: [
      {
        test: /\.js$/,
        // use: ["cache-loader", "babel-loader"],
        use: [
          {
            loader: "cache-loader",
            options: {
              cacheDirectory: "./cache", // 指定缓存目录
            },
          },
          "babel-loader",
        ],
      },
    ],
  },
};
```

`cache-loader` 放在第一个，loader 都是从后往前运行的，当运行到 `cache-loader` 的时候，别的 loader 都运行完了，那么其他 loader 怎么知道要用缓存呢？

其实在 loader 的运行过程中，包含一个 pitch 过程

```js
function loader(source) {
  return "new source";
}

loader.pitch = function (filePath) {
  // 可以返回，可以不返回，如果要返回则返回源代码字符串
};

module.exports = loader;
```

![loader的执行过程](24.性能优化/优化loader性能/2020-02-21-13-32-36.png)

3. 为 loader 开启多线程

`thread-loader` 会开启一个线程池，线程池中包含适量的线程（和 cpu 的核数一样），它会把后续的 loader 放入到线程池中运行，以提高构建性能。

在实际的开发中，可以测试后再决定 `thread-loader` 放置到什么位置，它后面书写的 loader 都在线程池中，前面的 loader 不在线程池中。

由于后续的 loader 放入到线程池中，所以后续的 loader

- 无法使用 webpackApi 生成文件
- 无法使用自定义的 pluginApi（如：mini-css-extract-plugin）
- 无法访问 webpack 配置

开启和管理线程都需要耗费时间，在小型项目中使用反而会增加构建时间。

```js
const cpus = require("os").cpus();
console.log(cpus); // 获取 cpu 核数
```

**热替换（热更新）：HMR**

热替换并不能降低构建时间，有时候会增加构建时间，但是可以降低代码改动到效果呈现的时间。

例如：我们有一个很大的表单元素，有 100 个，当填写到 99 个表单的时候，我们发现有个地方有错误，需要更改源代码，经过编译打包，页面重新刷新了，之前的 99 项全部白填了，崩溃了。

这时我们就需要热替换（热更新），页面不会刷新，浏览器仅请求改动后的资源，更新需要改动的地方。

`new webpack.HotModuleReplacementPlugin()` 可选的，webpack4 之后只要开启了 `hot: true`，默认会使用这个插件，热更新也是通过这个插件完成的。

第一步：配置文件

```js
module.exports = {
  mode: "development",

  devServer: {
    hot: true,
  },

  // plugins: [new webpack.HotModuleReplacementPlugin()],
};
```

第二部：任意文件加入下面代码，这段代码会参与运行

```js
// 是否开启了热更新
if (module.hot) {
  module.hot.accept(); // 接受热更新
}
```

当配置文件中开启了热更新，`webpack-dev-server` 会向最终结果中注入 `module.hot` 属性。

在默认情况下，`webpack-dev-server` 不管是否开启了热更新，当重新打包后，都会调用 `location.reload` 刷新页面，如果调用 `module.hot.accept` 方法会改变该行为。

`module.hot.accept` 方法的作用是让 `webpack-dev-server` 通过 `web socket` 管道把服务器更新的内容发送到浏览器，然后将返回结果交给 `webpack.HotModuleReplacementPlugin` 插件进行处理，该插件会根据更新内容覆盖原始代码，让代码重新执行。

**所以：热更新发生在代码运行期间**

样式代码同样可以进行热更新，需要使用 `style-loader`，因为热更新发生时，`HotModuleReplacementPlugin` 插件只会简单的覆盖并运行模块代码，因此代码一运行 `style-loader` 就会重新设置 style 元素中的样式，而 `mini-css-extract-plugin` 生成的文件是在构建期间，运行期间无法改动文件，因此它对热替换无效。

##### 传输性能优化

分包：将整体的代码，分布到不同的打包文件中（将公共的第三方库或代码抽离出去）

优点：减少公共代码的重复，从而降低总体积，并且有利于浏览器的缓存。

什么时候需要分包：有多个 chunk，并且公共模块代码的体积较大或很少变动，分包时尽量不要影响源码。

**手动分包**

原理：

1. 对于公共的模块，需要重新创建配置文件，先对这些公共模块进行打包，公共模块会被打包成动态链接库（dll：用于暴露全局变量），并产生资源清单（类似 source-map）
2. 根据入口模块进行正常打包，打包时，如果发现模块中使用了资源清单中描述的模块，导入模块时就会导入这个全局变量，而不导入整个第三方库。

第一步：公共模块的配置文件：`dll.config.js`

```js
const webpack = require("webpack");
const path = require("path");

module.exports = {
  mode: "production", // 生产环境

  // 入口公共模块
  entry: {
    jquery: "jquery", // 如果报错，这里请使用数组
    lodash: "lodash", // 如果报错，这里请使用数组
  },

  output: {
    filename: "dll/[name].js", // 在dll目录中按照chunk生产和原来名字一样的js文件
    library: "[name]", // 每个bundle暴露的全局变量使用原来的名字
  },

  plugins: [
    new webpack.DllPlugin({
      path: path.resolve(__dirname, "dll", "[name].manifest.json"), // 生产资源清单的路径和文件名
      name: "[name]", // 暴露的变量，通常和 library 保持一致
    }),
  ],
};
```

配置公共模块的启动脚本

```json
"dll": "webpack --config dll.config.js"
```

单独打包公共模块

```shell
npm run dll
```

第二步：手动引入公共模块

```html
<!-- 引入公共模块文件 -->
<script src="./dll/jquery.js"></script>
<script src="./dll/lodash.js"></script>
```

配置 `clean-webpack-plugin`，为了避免清除公共模块

```js
new CleanWebpackPlugin({
  cleanOnceBeforeBuildPatterns: ["**/*", "!dll", "!dll/*"],
  // "**/*"：清空所有文件
  // "!dll"：排除掉dll目录
  // "!dll/*"：排除掉dll目录下的所有文件
});
```

目录和文件的匹配规则使用的是 `globbing patterns` 语法，参考：https://github.com/sindresorhus/globby#globbing-patterns

运行入口模块打包命令

```shell
npm run dev
```

第三步：动态链接库引入插件 `webpack.DllReferencePlugin` 控制打包结果

```js
new webpack.DllReferencePlugin({
  manifest: require("./dll/jquery.mainfest.json"),
}),

new webpack.DllReferencePlugin({
  manifest: require("./dll/lodash.mainfest.json"),
}),
```

总结：

1. 开启 `output.library` 通过全局变量暴露公共模块
2. 使用 `webpack.DllPlugin` 创建资源清单
3. 使用 `webpack.DllReferencePlugin` 使用资源清单

注意事项：

- 资源清单不参与运行，不需要放置到打包目录
- 需要手动的引入公共模块内容，还要配置 `clean-webpack-plugin` 不要清除 dll 目录和里面的文件
- 不要对小型的公共 JS 库使用手动分包

优点：

- 极大提升自身模块的打包速度
- 极大缩小文件体积
- 有利于浏览器缓存公共模块代码

缺点：

- 使用起来比较繁琐
- 如果第三方库中包含复杂的依赖关系，操作起来会极其繁琐

**自动分包**

不同于手动分包，它时从实际角度出发看问题（分包：实际上就是提取公共模块），而不对具体要把哪个包分出去进行控制。

这种分包操作起来很方便，也满足实际的开发需要，对构建性能稍微有点负面影响。

要进行自动分包，关键点是：配置一个合理的分包策略（就像会议精神一样）

有了这个分包策略之后，不需要安装什么插件，webpack 会按照分包策略进行自动分包（webpack 内部使用 `optimization.splitChunks` 插件完成自动分包）

分包流程：

- 分包策略很重要，它将决定 webpack 如何进行分包
- 分包时，webpack 会开启一个新的 chunk，对分离的模块（公共）进行打包
- 打包结果中，会根据新的 chunk 形成一个单独的文件

配置：

webpack 提供了 `optimization` 配置项，用于配置一些优化信息，在其中使用 `splitChunks` 配置分包策略

```js
module.exports = {
  optimization: {
    splitChunks: {
      // 分包策略
      chunks: "all",
      maxSize: 2000, // 慎用
      automaticNameDelimiter: "-", // 控制连接符
      minChunks: 1, // 一个模块被引用了几次之后才能进行分包，默认值 1
      minSize: 30000, // 当分包达到了多少字节后才允许被真正的拆分，默认值 30000
    },
  },
};
```

事实上，分包策略有默认的配置，我们只需要轻微的改动，就可以应付绝大多数的场景。

1. chunks：用于配置需要使用分包策略的 chunk，取值

- async：默认值，仅针对异步 chunk（懒加载）使用分包策略
- all：针对所有的 chunk 使用分包策略
- initial：针对普通 chunk 使用分包策略

2. maxSize：用于配置包的最大字节数，如果超过了这个数字，webpack 会尽可能的将其分离出更多的包（基础单位是模块），这种设置并没有改变总体积，可能会造成更多的 HTTP 请求，慎用

**分包中的缓存组**

之前配置的分包策略都是全局的，实际上分包策略是基于缓存的。

每个缓存组提供一套独有的策略，webpack 会按照缓存组的优先级依次处理每个缓存组，被缓存组处理过的分包不需要再次分包。

默认情况下，webpack 提供了两个缓存组：`vendors` 和 `default`

```js
module.exports = {
  optimization: {
    splitChunks: {
      // 分包策略
      chunks: "all", // 全局配置

      cacheGroups: {
        // key：缓存组名称，会影响分包后的chunk名称
        // value：缓存组配置，会继承所有的全局配置，在这里配置自己的特有配置
        vendors: {
          test: /[\\/]node-modules[\\/]/, // 正则匹配，当匹配成功后，将这些模块进行单独打包
          priority: -10, // 缓存组的优先级，默认值是 0 ，优先级越高，会先进行处理。
        },

        // default：缓存组名称，会影响分包后的chunk名称
        default: {
          minChunks: 10, // 会覆盖全局配置
          priority: -20, // 优先级低于vendors
          reuseExistingChunk: true, // 重用已经被分离出去的chunk
        },
      },
    },
  },
};
```

当我们需要抽离公共样式时，需要使用缓存组

```js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: "all",
      cacheGroups: {
        styles: {
          test: /\.css$/, // 匹配样式模块
          minSize: 0, // 覆盖默认的最小尺寸，这里仅仅是作为测试
          minChunks: 2, // 覆盖默认的最小chunk引用数
        },
      },
    },
  },
  module: {
    rules: [
      { test: /\.css$/, use: [MiniCssExtractPlugin.loader, "css-loader"] },
    ],
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html",
      chunks: ["index"],
    }),
    new MiniCssExtractPlugin({
      filename: "[name].[hash:5].css",
      // chunkFilename是配置来自于分割chunk的文件名
      chunkFilename: "common.[hash:5].css",
    }),
  ],
};
```

自动分包原理：

1. 检查每个 chunk 生成的编译结果（模块 ID 和编译结果）
2. 根据分包策略找到满足条件的模块
3. 根据分包策略生成新的 chunk 将满足条件的模块进行打包
4. 把打包出来的模块从原始编译结果中移除，并对原始编译结果做修正处理

**代码压缩**

用于单模块体积优化，移除模块内的无效代码，为了减少代码体积，降低代码的可读性，提升破解成本，用于生产环境

压缩工具

- UglifyJS：不支持 ES6 语法
- Terser：支持 ES6 以上的语法，webpack 内置了 Terser，在生产环境中自动开启。

关于副作用（side effect）

函数运行过程中，可能会对外部环境造成影响的功能

如果函数中包含以下代码，该函数叫做副作用函数:

- 异步代码
- localStorage
- 对外部数据的修改

如果一个函数没有副作用，同时，函数的返回结果仅依赖参数，则该函数叫做纯函数(pure function)

webpack 自动集成了 Terser，而且进行了相当好的配置，一般情况下我们不会改动。

如果你想更改、添加压缩工具，又或者是想对 Terser 进行配置，使用下面的 webpack 配置即可

```js
const TerserPlugin = require("terser-webpack-plugin");
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");

module.exports = {
  optimization: {
    // minimize: true, // 是否要启用压缩，默认情况下，生产环境会自动开启，如果设置成true，开发环境也会开启
    minimizer: [
      // 压缩时使用的插件，可以有多个
      new TerserPlugin(), // 需要需要别的压缩工具，这里设置别的压缩工具
      new OptimizeCSSAssetsPlugin(), // 该插件可以对 CSS 进行压缩
    ],
  },
};
```

**tree-shaking**

用于单模块体积优化，可以移除模块之间的无效代码（移除掉某个库中没有用到的导出）

webpack 支持 tree-shaking，只要开始了生产环境，tree-shaking 自动开启。

webpack 选择 ES6 导入语句分析依赖是因为

- ES6 的导入语句必须在最顶部
- import 导入的模块名必须是字符串常量
- import 导入的变量是常量

具体分析依赖时

- webpack 坚持的原则：保证代码正常运行的情况下，尽量 tree-shaking
- 如果依赖的是一个导出的对象，为了保证代码的正常运行，webpack 不会移除对象中的任何信息

所以我们应该尽量使用 ES6 的基本导出（`export const xxx = xxx`），ES6 的基本导入（`import { xxx } from 模块路径`）或`import * as obj from 模块路径`（模块对象 obj 中属性不可变）

> 强烈建议：使用 `export const xxx = xxx` 导出，使用 `import { xxx } from 模块路径` 导入

依赖分析完毕后，webpack 会把没有使用到的模块内容标记为 `dead code`，然后交给代码压缩工具处理，压缩工具不会把标记为 `dead code` 的代码打包进最终代码。

某些第三方库使用的 CommonJS 的方式导出（如：lodash），或者某些库没有提供 ES6 的基本导出方式，对于这种库，tree-shaking 无法发挥作用。

很多流行的，但是这些没有使用 ES6 的第三方库，都发布了相应 ES6 的版本，如：lodash-es

tree-shaking 本身没有完善的作用域分析，可能导致一些标记为 `head code` 的代码仍然会视为依赖，可以通过 `webpack-deep-scope-plugin` 插件解决（该库可以进行深度的依赖分析）。

当 webpack 无法确定某段代码是否有副作用（是否对外界有影响）时，它将不会被 tree-shaking，如果我们确定没有副作用可以在 `package.js` 文件中加入 `sideEffects: false`，表示我们工程中没有副作用（这种做法太暴力）

可以精确的控制副作用文件，一般情况下我们不会进行这样的设置

```json
"sideEffects": [
  "!src/xxx.js" // 表示除了这个文件，别的文件都有副作用
]
```

webpack 无法对 css 完成 tree-shaking，因此需要插件 `purgecss-webpack-plugin`，这个插件对 `css module` 不起作用。

```js
new PurgeCSSPlugin({
  paths: [path.resolve(__dirname, "public/index.html"), ....其他绝对路径] // 和什么文件进行对比然后进行tree-shaking
}),
```

但是 `purgecss-webpack-plugin` 如果需要匹配多个文件时，书写太麻烦，可以使用 `glob-all` 这个库解决，这个库可以使用多个规则进行匹配文件

```js
const srcAbs = path.resolve(__dirname, "src");

globAll.sync([
  `${srcAbs}**/*.js`, // src路径下的所有JS文件全部匹配到
]);
```

**懒加载**

就是自动分包时 chunks: async，浏览器会使用 JSONP 的方式远程读取 JS 模块，import 语法会返回一个 Promise，结果类似于 `* as obj`

```js
const xxx = await import("模块"); // 这个模块就是懒加载的模块，什么时候用什么时候加载进来。
```

经过上面的操作之后，tree-shaking 不了了，根本原因是：没有静态依赖，可以使用下面的方式解决

1. 创建一个 js 模块文件，使用静态依赖导入模块内容 `export { chunk } from "lodash-es"`
2. 懒加载的时候导入自己的模块文件 `const { chunk } = await import("./自己的模块文件")`

**gzip**

gzip 是一种压缩文件的算法

优点：传输效率可能大幅度提升。
缺点：服务器压缩需要时间，客户端解压也需要时间。

webpack 会进行预压缩，使用 `compression-webpack-plugin` 插件完成，这样，服务器就可以不用压缩了，直接将压缩好的文件内容发送给客户端。（这么一来服务器就失去了一些灵活度）

```shell
npm i -D compression-webpack-plugin
```

```js
const CompressionPlugin = require("compression-webpack-plugin");

module.exports = {
  plugins: [new CompressionPlugin()],
};
```

- minRatio：压缩后体积要小于源码的多少比例才会压缩，默认值 0.8

##### 其他性能优化

**ESlint**

代码风格检查，用于解决团队协作代码风格的统一，间接影响性能优化

与构建工具一起使用：打包的时候会警告
与编辑器一起使用：书写代码时会警告，通常会配合编辑器一起使用

检查的工作还是交给 ESlint 库完成的，如果当前工程没有，则使用全局的，如果都没有，无法完成检查

检查工作根据 `.eslintrc` 配置文件里面的配置做检查，如果找不到工程中的配置文件，则无法检查。

1. 编辑器安装 `eslint` 插件

2. 安装 `eslint` 库

推荐使用 powershell 安装

```shell
npm i -D [-g] eslint
```

3. 创建 `.eslintrc` 配置文件

可以手动创建，也可以通过命令 `npx eslint --init` 创建。

eslint 会识别工程中的 `.eslintrc` 配置文件中的配置，也会识别 `package.json` 文件中的 `eslintConfig` 字段中的配置。

```shell
npx eslint --init
```

配置文件

- env：该配置项用于设置代码的运行环境
- extends：该配置项用于设置继承自哪里，值可以时字符串或数组
- globals：该配置项用于定义全局变量，eslint 还支持注释形式的定义全局变量：`/* global a, b */` | `/* global c: readonly, d: writable */`，仅对本文件有效
- parser：该配置项用于指定解析器
- parserOptions：该配置项用于设置要对哪些语言支持
- rules：该配置项用于设置 eslint 的规则集，具体规则参考：https://eslint.bootcss.com/docs/rules/
  - 每条规则将影响某个方面的代码风格
  - 每条规则有以下的取值
    - off、0、false：表示不使用规则，也就不检查
    - warn、1、true：警告级别，不会导致程序退出
    - error、2：错误级别警告，当触发时会导致程序退出
  - 还可以使用注释的形式配置规则：`/* eslint eqeqeq: "off", curly: "error" */`，仅对本文件有效

使用 `.eslintignore` 文件，排除掉不需要使用 eslint 检查的文件，和 `.gitignore` 的功能一样，将该文件创建在项目的根目录中

```text
**/dist/**/*.js
node_modules
```

```json
{
  // 该配置项用于设置代码的运行环境
  "env": {
    "browser": true, // 是否在浏览器环境中运行
    "es2021": true // 是否开启 es2021 的 API
  },
  "extends": "eslint:recommended",

  // 该字段用于配置全局变量
  "globals": {
    "a": "readonly", // key:全局变量，value：变量的描述，只读属性
    "b": "writable" // 可写属性
  },

  "parser": "Espree", // 指定解析器

  // 该配置项用于设置 eslint 要对哪些语言支持
  "parserOptions": {
    "ecmaVersion": 12, // 支持 ES 语法的版本
    "sourceType": "module" // 源码类型支持模块化脚本，选填 script：源码类型支持传统脚本
  },
  "rules": {}
}
```

**bundle analyzer**

bundle （最终）结果分析，对打包的结果进行分析，它并不会优化代码，而是提示我们当前需要做什么优化。

需要使用插件 `webpack-bundle-analyzer`，通过 npm 安装

```js
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer")
  .BundleAnalyzerPlugin; // 使用插件

module.exports = {
  plugins: [new BundleAnalyzerPlugin()],
};
```

会打开一个图形化工具的页面

### 不确定的依赖

导入什么模块可能来自用户的输入，这种情况下，webpack 会导入带目录下的所有的模块，防止以防万一，用户输入是已经存在的模块

如果遇到这种情况，需要告诉 webpack 这样的模块应该在什么目录下

```js
const a = require(`./util` + value); // 会导入util下面的所有模块
```

实际上面的操作会被转换成下面的代码，仅在 webapck 运行过程中有效

```js
require.context("./util", true, /\.js$/); // 返回一个函数
// 参数1：表示目录，引入模块的目录
// 参数2：是否递归寻找子目录
// 参数3：正则表达式，如果没有值，则打包全部模块到打包结果
```

返回的函数可以使用该目录下的对应模块

```js
const context = require.context("./util", true, /\.js$/);
const a = context("./a.js"); // 执行 a 模块，返回 a 模块的导出结果
console.log(a);

context.keys(); // 是一个数组，包含导入的所有模块路径
```

一般用于将该目录中的所有模块全部导入到一个模块中，进行模块整合
