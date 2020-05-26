?> 作者：DongChen_Hu ，供稿日期：2020/05/26

[rollup.js](https://www.rollupjs.org/guide/en/): JavaScript 模块打包工具，常用于打包库（如 vue-next, ...）

## 特点

- 使用 es6 标准模块化方案
- 静态分析，剔除无用代码
- 配置简单，打包速度快

## 本地安装

```sh
yarn add rollup --dev
```

## 配置文件

```js
// rollup.config.js
export default {
  input: 'main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
}
// 等价于 rollup --input main.js --file bundle.js --format cjs
// 简写   rollup -i main.js -o bundle.js -f cjs
```

## 常见用法

### 1. 打包压缩，自动剔除无用代码

```js
// ./main.js
import { message } from './utils'

export default () => console.log(message)

// ./utils.js
export const isArray = Array.isArray
export const message = 'Hello world'

// ./rollup.config.js
import { terser } from 'rollup-plugin-terser'

export default {
  input: 'main.js',
  output: [
    {
      file: 'bundle.cjs',
      format: 'cjs'
    },
    {
      file: 'bundle.mjs',
      format: 'es'
    },
    {
      file: 'bundle.min.js',
      format: 'umd',
      name: 'MyBundle',
      plugins: [terser()]
    }
  ]
}
```

运行 `npx rollup -c` 编译源码，生成 `bundle.cjs`, `bundle.mjs`, `bundle.min.js`

- bundle.cjs

```js
'use strict'

const message = 'Hello world'

var main = () => console.log(message)

module.exports = main
```

- bundle.mjs

```js
const message = 'Hello world'

var main = () => console.log(message)

export default main
```

- bundle.min.js

```js
!(function (e, o) {
  'object' == typeof exports && 'undefined' != typeof module
    ? (module.exports = o())
    : 'function' == typeof define && define.amd
    ? define(o)
    : ((e = e || self).MyBundle = o())
})(this, function () {
  'use strict'
  return () => console.log('Hello world')
})
```

运行 `node -e "require('./bundle.cjs')()"` 检查执行结果

### 2. 动态导入，代码拆分

```js
// ./main2.js
export default () =>
  import('./utils').then(({ message }) => console.log(message))

// ./utils.js
export const isArray = Array.isArray
export const message = 'Hello world'

// ./rollup.split.config.js
import { terser } from 'rollup-plugin-terser'

export default {
  input: 'main2.js',
  output: {
    dir: 'dist',
    entryFileNames: 'bundle-[format].js',
    format: 'cjs'
  },
  plugins: [terser()]
}
```

运行 `npx rollup -c rollup.split.config.js` 编译源码，生成 `./dist/bundle-es.js`, `./dist/utils-65ee99c3.js`

- bundle-es.js

```js
'use strict'

module.exports = () =>
  Promise.resolve()
    .then(function () {
      return require('./utils-65ee99c3.js')
    })
    .then(({ message: e }) => console.log(e))
```

- utils-65ee99c3.js

```js
'use strict'
const r = Array.isArray
;(exports.isArray = r), (exports.message = 'Hello world')
```

运行 `node -e "require('./dist/bundle-cjs.js')()"` 检查结果

### 3. 自定义命令行参数

```js
// main3.js
import { version } from '../package.json'

export default () => console.log('version:', version)

// rollup.args.config.js
import json from '@rollup/plugin-json'

export default args => {
  const { prefix = 'bundle' } = args
  delete args.prefix
  return {
    input: 'main3.js',
    output: {
      file: `${prefix}.js`,
      format: 'cjs'
    },
    plugins: [json()]
  }
}
```

运行 `npx rollup -c rollup.args.config.js --prefix custom` 编译源码，生成 `custom.js`

```js
'use strict'

var version = '1.0.0'

var main3 = () => console.log('version:', version)

module.exports = main3
```

运行 `node -e "require('./custom.js')()"` 检查结果

### 4. 从捆绑包中剔除外部依赖

```js
// main4.js
import _ from 'lodash'
import qs from 'qs'

console.log(qs.stringify({ a: 1 }))
console.log(_.isString(''))

// rollup.external.config.js
import resolve from '@rollup/plugin-node-resolve'
import commonjs from '@rollup/plugin-commonjs'

export default {
  input: 'main4.js',
  output: {
    file: 'bundle-e.js',
    format: 'cjs'
  },
  plugins: [resolve(), commonjs()],
  external: ['lodash', 'qs']
}
```

运行 `npx rollup -c rollup.external.config.js` 编译源码，生成 `bundle-e.js`

```js
'use strict'
function _interopDefault(ex) {
  return ex && typeof ex === 'object' && 'default' in ex ? ex['default'] : ex
}

var _ = _interopDefault(require('lodash'))
var qs = _interopDefault(require('qs'))

console.log(qs.stringify({ a: 1 }))
console.log(_.isString(''))
```

运行 `node -e "require('./bundle-e.js')"` 检查结果

### 5. 使用 typescript

安装 typescript

```sh
yarn add rollup-plugin-typescript2 typescript --dev
```

```js
// main.ts
import { version } from '../package.json'
import qs from 'qs'

export default () => {
  console.log('version:', version)
  console.log('stringify:', qs.stringify({ a: 123, b: ['xyz', { c: false }] }))
}

// rollup.ts.config.js
import resolve from '@rollup/plugin-node-resolve'
import commonjs from '@rollup/plugin-commonjs'
import json from '@rollup/plugin-json'
import ts from 'rollup-plugin-typescript2'

export default {
  input: 'main.ts',
  output: {
    file: 'lib/bundle-ts.js',
    format: 'cjs'
  },
  plugins: [
    resolve(),
    commonjs(),
    json(),
    ts({ useTsconfigDeclarationDir: true })
  ],
  external: 'qs'
}
```

- tsconfig.json

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "outDir": "lib",
    "declarationDir": "lib",
    "declaration": true,
    "esModuleInterop": true,
    "resolveJsonModule": true
  },
  "files": ["main.ts"]
}
```

运行 `npx rollup -c rollup.ts.config.js` 编译源码，生成 `lib/bundle-ts.js`, `main.d.ts`

- bundle-ts.js

```js
'use strict'

function _interopDefault(ex) {
  return ex && typeof ex === 'object' && 'default' in ex ? ex['default'] : ex
}

var qs = _interopDefault(require('qs'))

var version = '1.0.0'

var main = () => {
  console.log('version:', version)
  console.log('stringify:', qs.stringify({ a: 123, b: ['xyz', { c: false }] }))
}

module.exports = main
```

- main.d.ts

```ts
declare const _default: () => void
export default _default
```

运行 `node -e "require('./lib/bundle-ts.js')()"` 检查结果

### 6. 以编程方式生成捆绑包

- build.ts

```ts
// 这里是使用 ts 语法书写的，所以需要先编译成 commonjs 格式
// 正常直接使用 commonjs 格式书写，然后在 script 中调用执行
import resolve from '@rollup/plugin-node-resolve'
import commonjs from '@rollup/plugin-commonjs'
import json from '@rollup/plugin-json'
import ts from 'rollup-plugin-typescript2'
import { rollup } from 'rollup'
import type { InputOptions, OutputOptions } from 'rollup'

const inputOptions: InputOptions = {
  input: 'main.ts',
  external: 'qs',
  plugins: [
    resolve(),
    commonjs(),
    json(),
    ts({ useTsconfigDeclarationDir: true })
  ]
}

const outputOptions: OutputOptions = {
  file: 'lib/bundle-build.js',
  format: 'cjs'
}

;(async () => {
  const bundle = await rollup(inputOptions)
  bundle.write(outputOptions)
})()
```

1. 运行 `npx rollup build.ts -p rollup-plugin-typescript2 -o build.js -f cjs` 生成 `build.js`
2. 执行 `node build.js` 生成 `bundle-build.js`, `main.d.ts`
3. 运行 `node -e "require('./lib/bundle-build.js')()"` 检查结果

### 7. 以编程方式加载配置文件

- load.cjs

```js
const loadConfigFile = require('rollup/dist/loadConfigFile')
const { resolve } = require('path')
const { rollup } = require('rollup')

loadConfigFile(resolve(__dirname, 'rollup.ts.config.js'), {
  file: 'lib/bundle-load.js',
  format: 'cjs'
}).then(({ options }) => {
  options.forEach(async opts => {
    const bundle = await rollup(opts)
    await Promise.all(opts.output.map(bundle.write))
  })
})
```

1. 执行 `node load.cjs` 生成 `bundle-load.js`, `main.d.ts`
2. 运行 `node -e "require('./lib/bundle-load.js')()"` 检查结果

## 发布 Npm 包

可以编译出 cjs 和 es 两种格式，在 package.json 中的配置如下

```json
{
  "main": "lib/foo.cjs" /* Node */,
  "module": "lib/foo.mjs" /* 浏览器 */
}
```

## 插件

[插件资源](https://github.com/rollup/awesome)

可能会用到的插件，如下：

- @rollup/plugin-node-resolve
  - 如果引入了安装在 node_modules 中的第三方模块，需要使用此插件分析模块的节点路径
- @rollup/plugin-commonjs
  - 将 CommonJS 模块转换成 ES 模块
- @rollup/plugin-json
  - 将 JSON 文件转换成 ES 模块
- rollup-plugin-typescript2
  - 转译 typescript 文件
- rollup-plugin-terser
  - 压缩代码
