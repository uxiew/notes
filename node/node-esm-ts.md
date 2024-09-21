# node-esm-ts

早在 2017 年，Node v8.9.0 引入了 ES 模块的概念。那时，您可以运行带有附加标志 (`--experimental-modules`) 的 Node.js 来使用它。
从 Node v13.2.0 开始，不再需要实验标志。现在只需几个步骤即可使用 ECMAScript 模块。

如果我们想用 TypeScript 进行开发 Node.js 应用，并且希望最终使用 ES6 模块语法。那么我们就会遇到这些问题：
（推荐 ES 模块文件后缀使用 `.mjs`，commonjs 模式推荐 `.cjs`）
1. `package.json` 中必须要指定 `"type": "module"` 来启用 ES6 模块语法。
2. TS 编译之后无法生成 `.mjs` 文件扩展名。
3. 启用 ES6 模块语法后，就必须指定文件的扩展名，否则会报错，比如 `.mjs`。

## ESM 与 CJS

ESM 会有一些与 CJS 不同：

- 只有 ESM 才能使用 top-level await

- ESM 中无法使用 `__dirname/__filename` 这类 全局变量。[alternative-for-dirname-in-node-js-when-using-es6-modules](https://stackoverflow.com/questions/46745014/alternative-for-dirname-in-node-js-when-using-es6-modules)

- ESM 是可以向下兼容 CJS 的。但是要 CJS 向上兼容，导入 ESM 模块，就会很麻烦，必须要使用 `dynamic import()`。

  ```ts
  // main.cjs
  import("./fileA.mjs").then((ctx) => {
    console.log(ctx.name); // "mjs"
  });
  ```

 在 NodeJS 中想要同步和动态导入 ES6 模块，可以考虑：[import-sync](https://github.com/nktnet1/import-sync)

### `.cts` 和 `.mts` 文件
随着 Node.js 12 的引入和 ES 模块支持的增加，TypeScript 引入了新的文件扩展名，以更清楚地区分 CommonJS 和 ES 模块：

- `.cts`：这是一个被视为 CommonJS 模块的 TypeScript 文件。它相当于使用 `--module commonjs` 编译选项。
- `.mts`：这是一个被视为 ES 模块的 TypeScript 文件。它相当于使用 `--module esnext` 编译选项。

### `require` 与 `import`

1. `require` 调用同步 CJS 模块加载器
2. `require` 在 CJS 模块中使用，在 ES 模块中可以使用 [`module.createRequire()`](https://nodejs.org/api/module.html#modulecreaterequirefilename) 构造 `require` 函数。
3. `require` 只能引用 CJS 模块，引用 ES 模块将抛出 `ERR_REQUIRE_ESM`，因为不可能从同步模块调用异步模块加载器。[node.js 22](https://nodejs.org/en/blog/announcements/v22-release-announce) adds `require()` support for synchronous ESM graphs under the flag `--experimental-require-module`，该 [pr](https://github.com/nodejs/node/pull/51977))
4. `import` 调用异步 ES 模块加载器
5. `import` 只能在 ES 模块调用
6. `import` 可以引用 ES 和 CJS 模块。

## JSON 模块

```js
import packageConfig from "./package.json" with { type: "json" };
```

必须使用 `with` 语法，会导致 node 不能识别。所以不要使用这种语法。

## `require('xxx').default`

有时，node 环境下 require 一个包时，会发现需要这么引入：`const axios = require('axios').default`。

其实很简单，axios 这个模块是一个 ES6 模块，是用 ESModule 的标准写的。其内部是这样导出的：`export default xxx`，相当于 `export {xxx as default}`。对外把变量命名为了 `default`，ES6 `default` 语法糖让我们在 import 时，不需要写 `default` 了。

而 require 在 commonJS 中对应 `module.exports` 的部分。当 require 去处理 ES6 的导出时，对它而言，ES6 的代码就会被转化成这样：

```js
module.exports = {
  default: xxx,
};
```

于是，在 ES6 语法糖 `default` 的影响下，require 导出的不是 `xxx` 这个对象，`xxx` 是作为 `default` 的值，被包在一个更大的对象里。

## 更多
> 轮子哥 [sindresorhus](https://github.com/sindresorhus) 的 [Pure ESM package](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c)

## 库的开发

有依赖 commonjs 包的库，考虑输出多包；
同时面对多种环境，优先考虑输出多包。

1. 导出 `cjs\mjs` 两种文件，方便用户使用。
2. 通过 `package.json` 的 `exports` 同时，指定 `require、import、types` 文件导出
3. 使用 ts 编译，生成 `.d.ts` 类型定义文件

### 相关构建工具
- [unjs/mlly](https://github.com/unjs/mlly) - 填补在 Node 中实用 ES 模块进行开发的实用库
- [unjs/unbuild](https://github.com/unjs/unbuild) - node 库构建工具
- [isaacs/tshy](https://github.com/isaacs/tshy)
- [privatenumber/pkgroll](https://github.com/privatenumber/pkgroll)
- [huozhi/bunchee](https://github.com/huozhi/bunchee)

### 最佳实践
1. 使用 Typescript 进行开发，或者编写良好的 JSDoc，还需要提高良好的测试（vitest、jest 工具等）
3. 内部所有 import（包括动态，除了第三方库），都需要指定好后缀名（typescript 编译，需要通过相应工具添加后缀）
4. 推荐使用 named export（命名导出）导出模块成员。default export（默认导出）在灵活性和可维护性、重构难度都不友好，在某些打包工具中，也容易发生意想不到的问题。
5. 通过 `sideEffects` 来声明哪些文件是具有副作用的，进行 Tree-shaking 时应该保留。参考：[深入理解sideEffects配置](https://libin1991.github.io/2019/05/01/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3sideEffects%E9%85%8D%E7%BD%AE/) [Webpack#Mark the file as side-effect-free](https://webpack.js.org/guides/tree-shaking/#mark-the-file-as-side-effect-free)
  有许多代码模式会被判定为具有 `sideEffects`，包括：
    * 顶层函数调用，如：`console.log('a')`；
    * 修改全局状态或对象，如：`document.title = 'new Title'`；
    * IIFE 函数；
    * 动态导入语句，如：`import('./mod')`；
    * 原型链污染，如：`Array.prototype.xxx = function (){xxx}`；
    * 非 JS 资源：Tree-shaking 能力仅对 ESM 代码生效，一旦引用非 JS 资源则无法树摇；
    * 等等；
6. STOP using **Barrel files**（也称为索引文件或桶文件）在TypeScript（TS）项目中经常被使用，它们是一种特殊的文件，仅用于导出其他文件的内容，而本身不包含任何业务逻辑代码。参考 [Stop using barrel files, now!](https://mp.weixin.qq.com/s/I1i-dhgFgmBsfNrpPW-akQ) [Barrel files and why you should STOP using them now](https://dev.to/tassiofront/barrel-files-and-why-you-should-stop-using-them-now-bc4) [Speeding up the JavaScript ecosystem - The barrel file debacle](https://marvinh.dev/blog/speeding-up-javascript-ecosystem-part-7/)

## 参考

- [nonara/ts-patch](https://github.com/nonara/ts-patch)

- [Modules: ECMAScript modules](https://nodejs.org/api/esm.html#modules-ecmascript-modules)

- [ts-bridge/ts-bridge](https://github.com/ts-bridge/ts-bridge)

- [GervinFung/ts-add-js-extension](https://github.com/GervinFung/ts-add-js-extension)

- [tsc-esm-fix](https://www.npmjs.com/package/tsc-esm-fix)

- [Node.js + Typescript with ESM in 2023](https://medium.com/codememo/node-js-typescript-with-esm-in-2023-6b87e6f8e737)
