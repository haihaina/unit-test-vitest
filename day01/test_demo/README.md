开始
```
 npm init vite test_demo
```

![image-20230808105552907](/Users/shenchunlv/Library/Application Support/typora-user-images/image-20230808105552907.png)

选择react+ts

下载依赖

```
npm install --save-dev jest @types/jest @jest/types

```
依赖项配置
```

✔ Would you like to use Jest when running "test" script in "package.json"? … yes
✔ Would you like to use Typescript for the configuration file? … yes
✔ Choose the test environment that will be used for testing › jsdom (browser-like)
✔ Do you want Jest to add coverage reports? … no
✔ Which provider should be used to instrument code for coverage? › babel
✔ Automatically clear mock calls, instances, contexts and results before every test? … yes
```
前两个配置选项是字面意思，不赘述了。
单测环境（jsdom)：因为我们会涉及到 dom 的单测，不仅仅是纯逻辑，如果是纯逻辑的选 node。
是否需要覆盖率报告（no)：暂时用不上，后面覆盖率章节会着重介绍。
编译代码（babel)： 可以转 ES5，避免一些兼容性问题。
每次测试完是否清理 mock、实例等结果（yes): 每次测试完成后会清理 mock 等上次测试的结果，可以避免用例之间的互相影响
### 因为我们选用了 babel 作为单测的编译，所以这边还需要增加一下对应的配置
```
npm install --save-dev babel-jest @babel/core @babel/preset-env @babel/preset-react @babel/preset-typescript

```
这边除 babel 基础的配置集（presets)，我们还安装了 React 和 TypeScript 的配置集，来帮助我们的单测可以支持使用 ts 来书写，安装完成后，我们在根目录创建一个 babel.config.js 文件用于 babel 的配置，其中@babel/preset-react ，我们为它加上 runtime: "automatic"的配置，这是为了帮助我们可以自动导入 React，不然后续单测的开发会要求对 React 进行

### 新增 babel.config.cjs
```
// ./babel.config.js
module.exports = {
  presets: [
    ["@babel/preset-env", { targets: { node: "current" } }],
    ["@babel/preset-react",{ runtime: "automatic" }], // 自动导入react
    "@babel/preset-typescript",
  ],
};

```
### 安装ts-node
```
npm install ts-node --save-dev

```
src/App.test.ts 下创建
```
// App.test.ts
import React from "react";

test("test", () => {
  expect(1 + 1).toBe(2);
});

```
执行  npm run test   可能会得到
```
● Validation Error:

  Test environment jest-environment-jsdom cannot be found. Make sure the testEnvironment configuration option points to an existing node module.

  Configuration Documentation:
  https://jestjs.io/docs/configuration


As of Jest 28 "jest-environment-jsdom" is no longer shipped by default, make sure to install it separately.
```
### 安装 jest-environment-jsdom
```
npm install jest-environment-jsdom --save-dev

```
### 执行 npm run test就能得到
```
 PASS  src/App.test.ts
  ✓ test (3 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.054 s
Ran all test suites.
```
额外的扩展名识别：因为Jest 不使用 Webpack 等打包工具，因此它不知道如何加载除 js/jsx 之外的其他文件扩展名，所以我们需要为它加一个转换器。
### jest.config.ts
```
// jest.config.ts
export default {
  // ... other config
  transform: {
    // ...
    "^.+\.(js|ts|tsx)$": "<rootDir>/node_modules/babel-jest",
  },
};

```
### svg-transform.js
```
// svg-transform.js
export default {
  process() {
    return { code: "module.exports = {};" };
  },
  getCacheKey() {
    return "svgTransform"; // SVG固定返回这个字符串
  },
};
```
CSS 代理：Jest 本身不知道如何处理不同扩展的文件，我们可以通过配置代理的方式，告诉 Jest 将此对象模拟为导入的 CSS 模块。
```
npm install --save-dev identity-obj-proxy
```
### jest.config.ts
```
// jest.config.ts
export default {
  // ... other config
  moduleNameMapper: {
    "\.(css|less)$": "identity-obj-proxy" // 有使用 sass 需求的同学可以把正则换成 ^\.(css|less|sass|scss)$
  }
};
```
到这里基本配置就完成。
