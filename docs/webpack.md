# webpack

webpack 是一个为现代 JavaScript 应用提供**静态模块（引入的资源）打包**功能的工具。webpack 并不局限于处理 js，它把所有引入的资源都当做是模块。

当 webpack 处理模块时，它会首先在内部递归构建一张依赖图，描述各个模块之间的引用关系，然后将所有的模块打包成一个或多个 `bundle`。

## 优势

1. 模块化编程，利于后期维护。

2. 支持引入不同的前端模块化方案（cjs、esm 等），有统一的输出，兼容性高。

3. 万物皆可是模块，所有资源文件的加载都可以通过代码控制。

## bundle，chunk，module

- `bundle`：最终输出打包的文件。

- `chunk`：代码块，有多个模块组合而成，用于代码分割和合并。

- `module`：模块，指引入的资源文件，不局限于 js。在 webpack 中，一切资源皆为模块，一个模块对应一个文件。 

## 和 rollup 的比较

webpack 适用于大型复杂的前端站点构建，拥有丰富的周边生态：大量的 `loader` 和插件。经过 webpack 打包后的 js 实际包裹在立即执行函数中。这些立即执行函数都只接受一个参数，参数是一个模块对象，键为各个模块的路径，值为模块内容。函数内部则处理各个模块之间的引用，并执行模块。

rollup 适用于基础库的打包，支持多种模块化输出方案。rollup 就是将各个模块打包进⼀个⽂件中，并且通过 Tree-shaking 来删除⽆⽤的代码，可以最⼤程度上降低代码体积。相较于 webpack，rollup 更具聚焦于 js 的打包，也更轻量，配置更简单。因此 rollup 的功能不如 webpack 全面，它天然不支持代码分割、按需加载等高级功能。

## loader 和 plugin

`loader` 是文件加载器，用于加载并处理文件资源，比如转译、压缩等。原生 webpack 只能解析 js 和 json 文件，如果想要打包其他文件，就需要配置 `loader`。更直观地说，`loader` 让 webpack 有了加载和处理非 js 文件的能力。 

`loader` 在模块的解析规则 `module.rules` 中配置，每一项规则都是对象，描述了对于什么类型的文件使用什么 `loader`。

`loader` 遵循单一原则，一个 `loader` 只做一种处理工作。

`plugin` 是插件，用于扩展 webpack 的功能，解决 `loader` 无法处理的问题，比如打包优化、资源管理、环境变量注入等。在 webpack 整个运行过程中，会广播出很多事件，`plugin` 可以在生命周期钩子中监听这些事件，在合适的时机通过 webpack 提供的 API 改变输出结果。

`plugin` 在插件选项 `plugins` 中配置。选项 `plugins` 的类型为数组，每⼀项都是 `webpack plugin` 的实例，插件的设置项通过构造函数传⼊。

## 常见的 loader

- `style-loader`：将 css 注入 js 中，浏览器将通过创建 style 标签引入 css。

- `css-loader`：处理 css 文件，分析 css 模块之间的关系，然后合并。

- `less-loader`：处理 less 文件，将 less 转译成 css。

- `sass-loader`：处理 sass 文件，将 sass 转译成 css。

- `postcss-loader`：调用 postcss 处理 CSS 文件，支持自动添加属性前缀，提高兼容性。

- `babel-loader`：调用 babel 将 es6 代码转译成 es5。

- `image-loader`：加载并压缩图片。

- `file-loader`：将⽂件复制到输出⽂件夹中，使得在代码中可以通过相对 URL 去引⽤输出⽂件。

- `url-loader`：功能与 `file-loader` 类似，但是如果文件很小，`url-loader` 能将文件内容转译成 base64 字符串注入到 js 中。

- `source-map-loader`：加载额外的 Source Map ⽂件，以⽅便断点调试 

## 常⻅的 plugin

- `html-webpack-plugin`：打包结束后，通过一个模板自动生成一个 html 文件，并将打包后的 js 引入到 html 中。

- `clean-webpack-plugin`：清理输出文件夹。

- `mini-css-extract-plugin`：将 css 代码提取到一个单独的文件中，支持按需加载。

- `define-plugin`：webpack 的内置插件，允许在编译时创建配置的全局对象。

- `copy-webpack-plugin`：将文件或者某个目录下的文件复制到输出文件夹中。 

## 构建流程

webpack 的运⾏流程是⼀个串⾏的过程，从启动到结束会依次执⾏以下流程： 

1. 初始化参数：从配置⽂件和 Shell 语句中读取并合并参数，得出最终的配置。
2. 开始编译：通过配置初始化 `Compiler` 对象，加载插件，然后调用  `Compiler` 对象的 `run` ⽅法开始编译。
3. 确定⼊⼝：根据配置中的 `entry` 找出所有的⼊⼝⽂件。
4. 编译模块：从⼊⼝⽂件开始，调⽤ `loader` 对模块进⾏处理，然后找出该模块的依赖模块，递归处理，直到所有模块都完成处理。
5. 完成模块编译：所有模块处理完成后，得到了每个模块被处理后的最终内容以及它们之间的依赖关系。
6. 输出资源：根据⼊⼝和模块之间的依赖关系，将模块组装成⼀个个 [chunk](#bundle，chunk，module)，然后把每个 [chunk](#bundle，chunk，module) 转换成单独的⽂件，最后加⼊到输出列表。这步是可以修改输出内容的最后机会。
7. 输出完成：在确定输出内容后，根据配置中的输出路径和⽂件名，将输出内容写⼊⽂件系统。

在以上过程中，webpack 会在特定的时间点⼴播出特定的事件，插件在监听到感兴趣的事件后会执⾏特定的逻辑，并且插件可以调⽤ webpack 提供的 API 改变 webpack 的运⾏结果。

## 前端性能优化

### 代码压缩

删除多余的代码、注释、简化代码的写法等等。

1. js 压缩

`terser-webpack-plugin` 是 webpack 的内置插件，作用是调用 `terser` 压缩、丑化 js 代码。在生产环境下，webpack 默认启用了该插件。

2. css 压缩

因为很难去修改选择器、属性名等值，所以 css 压缩通常是去除空格和注释。

css 压缩需要使用插件：`css-minimizer-webpack-plugin`。

3. html 压缩

和 css 压缩类似，html 压缩通常也是去除空格和注释。

可以在 html-webpack-plugin 插件生成 HTML 模板的时候，通过配置属性 `minify` 进行压缩。实际上，设置了属性 `minify`，使用的是另一个插件 html-minifier-terser。

4. 图片压缩

一般来说，相较于 js 或是 css 文件，一些图片文件更大。

图片处理有两种方式：

- 使用 image-webpack-loader 对较大图片进行压缩。

- 使用 url-loader 将较小的图片转为 base64 字符串注入 js 中，这能降低发送网络请求的频率。

图片压缩速度较慢，因此不建议在开发环境下进行压缩，最好只在生产环境下进行，或者将图片压缩操作与 webpack 配置分离，必要时再进行手动压缩。

5. tree Shaking

tree shaking 是一个术语，在计算机中表示消除死代码。

在 webpack 中，实现 tree shaking 有两种不同的方案。

- `usedExports`：通过标记某些函数、变量是否被使用，之后通过 terser 删除未使用的代码。

- `sideEffects`：通过给 package.json 加入 sideEffects: false 声明该包模块是否包含 sideEffects(副作用)，从而可以为 tree shaking 提供更大的优化空间。原理是 webpack 能将标记为 side-effects-free 的包由 import {a} from xx 转换为 import {a} from 'xx/a'，从而自动修剪掉不必要的导入，跳过解析模块中没有引入的文件。

6. 文件大小压缩

对文件大小进行压缩，可以减少 http 传输过程中的损耗。

```shell
npm install compression-webpack-plugin -D
```

```JavaScript
new ComepressionPlugin({
    test:/\.(css|js)$/,  // 哪些文件需要压缩
    threshold:500, // 设置文件多大开始压缩
    minRatio:0.7, // 至少压缩的比例
    algorithm:"gzip", // 采用的压缩算法
})
```

7. css tree shaking

css 可以借助 purgecss-plugin-webpack 插件进行 tree shaking。

```shell
npm install purgecss-plugin-webpack -D
```

```JavaScript
const PurgeCssPlugin = require('purgecss-webpack-plugin')
module.exports = {
    ...
    plugins:[
        new PurgeCssPlugin({
            path:glob.sync(`${path.resolve('./src')}/**/*`), {nodir:true}// src里面的所有文件
            satelist:function(){
                return {
                    standard:["html"]
                }
            }
        })
    ]
}
```

- `paths`：表示要检测哪些目录下的内容需要被分析，配合使用 glob

默认情况下，Purgecss 会移除掉 html 标签中的样式，如果希望保留，可以添加一个 safelist 的属性。

8. 代码分割

有些时候，一次性加载并运行所有的 js 代码，会影响首页的加载速度，造成白屏现象。我们可以通过将 js 代码分割成更小的 bundle，以及控制加载优先级，提高首页的加载速度。

webpack 可以借助 splitChunksPlugin 插件实现代码分割，该插件是 webpack 的内置插件，默认集成和安装。

splitChunks 主要配置如下：

- `chunks`：对同步代码还是异步代码进行分割。

- `minSize`：生成 `chunk` 的最小体积。

- `maxSize`：将体积大于 `maxSize` 的包进行分割，每个部分不小于 `minSize`。

- `minChunks`：被引入的次数，默认为 1。

默认情况下，仅针对异步引入的模块进行代码分割。

9. 内联 chunk

通过 `InlineChunkHtmlPlugin` 插件将一些 chunk 模块内联到 html。适用于一些代码量不大，但是必须加载的模块。

## 构建速度优化

1. 优化 loader 配置。

在使用 loader 时，可以通过配置 `test` 选项匹配文件，通过配置 `include` 和 `exclude` 选项来规定哪些目录应用 loader，哪些目录不应用 loader。

2. 合理使用 `resolve.extensions`。

`resolve.extensions` 选项（接受一个字符串数组）用于在解析文件时自动添加扩展名。

当项目引入文件没有扩展名时，webpack 会尝试根据 `resolve.extensions` 数组中的扩展名依次添加查找，一旦解析到文件，则跳过其余的扩展名。因此，`resolve.extensions` 接受的扩展名不宜过多，否则会引发多次文件的查找，降低打包速度。

3. 优化 `resolve.modules`。

`resolve.modules` 用于配置 webpack 去哪些目录下寻找第三方模块,默认值是 `[node_modules]`。因此，可以指明存放第三方模块的绝对路径，减少查找过程。

4. 优化 `resolve.alias`。

`resolve.alias` 用于给常用路径起一个别名。当项目目录层级较深时，可以配置 `resolve.alias` 减少查找过程。

5. 使用 `cache-loader`。

`cache-loader` 用于将前一 loader 的处理结果缓存至磁盘。在一些性能开销较大的 loader 前添加 `cache-loader`，可以显著提高二次构建速度。

因为保存和读取缓存文件也需要时间开销，所以对于性能开销较小的 loader，不建议使用 `cache-loader`。

6. terser 启动多线程。

启用 terserPlugin 的多线程，并行运行，可以提高构建速度。

7. 合理选择 sourceMap。

打包生成 sourceMap 时，如果信息越详细，打包速度越慢。因此在开发环境下，需合理配置 sourceMap。

总结：优化 webpack 构建的方式有很多，主要可以从优化搜索时间、缩小文件搜索范围、减少不必要的编译等方面入手。

## 热更新（hot module replacement）

HMR 全称 Hot Module Replacement，也就是模块热替换，指在应用程序运行过程中，无需重新刷新页面，就能替换、添加和删除模块的技术。

在 webpack 中配置开启热模块更新也十分简单，将 `devServer.hot` 设置为 `true` 即可。

但实际上，修改和保存 css 文件确实能够在不刷新的形式下更新到页面中。但是，修改并保存 js 文件时，页面依然自动刷新了，没有触发热更新。因此，HMR 不像 Webpack 的其他特性一样可以开箱即用，它需要指定模块。

webpack-dev-server 创建了两个服务器，一个用于提供静态资源服务（express），一个是基于 webSocket 的长连接。当 socket server 监听到模块发生变化，会生成两个文件，并将这两个文件发送给浏览器。浏览器拿到后，通过 HMR runtime 机制，加载这两个文件，并进行更新。