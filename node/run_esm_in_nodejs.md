## 如何在 Node.js CLI 中运行 ESM 模块

在 Node.js 环境中运行 ECMAScript Modules（ESM）模块可以通过几种不同的方法实现。以下是详细的步骤和方法：

1. 使用 Native Node.js 运行 ESM

从 Node.js 版本 13.2.0 开始，Node.js 原生支持 ESM 特性。因此，使用高版本的 Node.js（建议使用 LTS 版本，如 14 或更高）可以直接运行 ESM 模块。你可以通过以下两种方式来实现：

在 package.json 中指定模块类型：在你的项目根目录下的 package.json 文件中添加 "type": "module" 字段。这将告诉 Node.js 当前项目使用 ESM 模块。示例如下：
```json
{
  ...
  "type": "module"
}
```

然后，你可以直接运行你的 JavaScript 文件，例如：
```sh
> node index.js
```

使用 `.mjs` 扩展名：如果不想修改 `package.json`，你也可以将文件扩展名改为 `.mjs`，这会自动将该文件视为 ESM 模块。例如，将文件命名为 `index.mjs`，然后运行：
```sh
> node index.mjs
```

这两种方法的主要区别在于作用域：第一种方法是包级别的作用域，而第二种则是文件级别的作用域。

2. 使用第三方库

对于较低版本的 Node.js（例如 12.x 或更早），你可能需要依赖一些第三方库来帮助你运行 ESM 模块。这些库提供了模块加载器或命令行工具，可以让你在不进行复杂配置的情况下直接执行 ESM 代码。

- Module Loader：例如，可以使用 [`standard-things/esm`](https://github.com/standard-things/esm) 库，它允许你预加载 ESM 包，从而无需大型编译工具。使用方式如下：
  ```sh
  > node -r esm index.js
  ```

- Command Line Tools：另一个选择是使用 babel-node 或 [egoist/esbuild-register](https://github.com/egoist/esbuild-register) [tsx](https://github.com/esbuild-kit/tsx) 等工具，这些工具允许你直接在命令行中执行 ESM 代码。例如，使用 tsx 的命令如下：
  ```sh
  > tsx index.ts
  ```

这些工具适合于开发环境，但不推荐用于生产环境，因为它们可能会导致性能问题。

3. 总结

如今，Node.js 和浏览器都对 ESM 有了良好的原生支持。在 Node.js CLI 中运行 ESM 模块的方法包括利用原生支持或借助第三方库。选择合适的方法取决于你的项目需求和所用的 Node.js 版本。

## 参考
- [在Node 中优雅使用esm](https//juejin.cn/post/7216584694792929337)
- [再谈模块化：Node中ESM与CJS的解析策略](https://chlorinec.top/2024/02/18/Development/how-node-resolve-modules/)

