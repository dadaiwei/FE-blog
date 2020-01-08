# 前言

webpack 作为前端领域的模块化打包工具，相信大家都不陌生。现在很火的 react 和 vue 的一些脚手架都是基于 webpack 开发定制的，因此，了解并会配置 webpack 还是很有必要的（文章基于 webpack4.x 版本来讲解）。

PS：文章内容可能有点长，大家提前做好心理准备。
# 1.webpack 是什么

**官方定义**：
<br>
webpack 是一个现代 JavaScript 应用程序的静态模块打包器。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

**个人理解**：
<br>
webpack 作为一个模块化打包工具，根据入口文件（任何类型文件，不一定是 js 文件）递归处理模块中引入的 js/css/scss/image 等文件，将其转换打包为浏览器可以识别的基础文件（js/css/image 文件等）。

![webpack](https://user-gold-cdn.xitu.io/2019/8/26/16ccc0f4b4ffd0df?w=1499&h=689&f=png&s=72823)

**与 grunt/gulp 等区别：**
<br>
1.grunt 与 gulp 属于自动化流程工具，通过配置文件指明对哪些文件执行编译、组合、压缩等具体任务，由工具自动完成这些任务。
<br>
2.webpack 作为模块化打包工具，把项目作为一个整体，通过入口文件，递归找到所有依赖文件，通过 loader 和 plugin 针对文件进行处理，最后打包生成不同的 bundle 文件。

# 2.webpack 基本配置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当你想使用 webpack 打包项目时，需要在项目目录下新建 webpack.config.js，webpack 默认会读取 webpack.config.js 作为配置文件，进而执行打包构建流程。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先来看一下 webpack 的基本配置项，留个印象先。

**webpack.config.js**

```
const path = require('path');

module.exports = {
  mode: 'production/development/none', // 打包模式，使用对应模式的内置优化
  entry: './src/index.js', // 入口，支持单入口、多入口
  output: { // 输出相关配置
    filename: 'xx.js', // 输出文件的文件名
    path: path.resolve(__dirname, 'dist') // 输出文件的绝对路径，默认为dist
  },
  module: { // 针对不同类型文件的转换
    rules: [
      {
        test: /\.xx$/, // 针对某类型文件处理，使用正则匹配文件
        use: [
          {
            loader: 'xx-loader', // 使用xx-loader进行转换
            options: {} // xx-loader的配置
          }
        ]
      }
    ]
  },
  plugins: [ // 插件，完成特定任务，如压缩/拆分
    new xxPlugin({ options });
  ]
};

```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;webpack 有五个核心概念：入口(entry)、输出(output)、模式（mode）、loader、插件(plugins)。

## 2.1.入口（entry）

入口指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。默认值为`./src`。

### 2.1.1.单入口

单入口是指 webpack 打包只有一个入口，单入口支持单文件和多文件打包。

> 通常像 vue/react spa 应用都属于单入口形式，以`src/index.js`作为入口文件。

（1）单文件打包

> 不指定入口文件的 entryChunkName 时，默认为 main。

```
// webpack.config.js

module.exports = {
  entry: "./src/index.js"
};
```

上面的单入口语法，是下面的简写：

```
module.exports = {
  entry: {
    main: "./src/index.js"
  }
};
```

main 表示 entryChunkName 为 main，打包后生成的文件 filename 为 main。

webpack 打包后，dist 文件夹生成 main.js 文件。

![webpack打包单文件](https://user-gold-cdn.xitu.io/2019/8/26/16ccc0fb6d9eeb17?w=161&h=139&f=png&s=5293)

也可以将 entryChunkName 修改为其他值，打包出的 filename 也会对应改变。

（2）多文件打包

多文件打包入口以数组形式表示，表示将多个文件一起注入到 bundle 文件中。

```
module.exports = {
  entry: ["./src/index.js", "./src/main.js"]
};

```

### 2.1.2.多入口

多入口是指 webpack 打包有多个入口模块，多入口 entry 一般采用对象语法表示，应用场景：

（1）分离应用程序 app 和第三方入口(vendor)

```
module.exports = {
  entry: {
    app: "./src/index.js",
    vendor: "./src/vendor.js"
  }
};
```

webpack 打包后，生成应用程序 app.js 和 vendor.js。

![分离应用程序和第三方](https://user-gold-cdn.xitu.io/2019/8/26/16ccc10c8bdb078f?w=160&h=182&f=png&s=8095)

（2）多页面打包，一般指多个 html 文档形式，每个文档只使用一个入口。

```
module.exports = {
  entry: {
    app: "./src/app.js",
    home: "./src/home.js",
    main: "./src/main.js"
  }
};
```

webpack 打包后，dist 文件夹下生成 app.js、home.js、main.js 三个文件。

![多入口打包](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1119c097e06?w=176&h=229&f=png&s=7635)

## 2.2.输出（output）

output 选项可以控制 webpack 如何输出打包文件，output 属性包含 2 个属性：

- filename：输出文件的文件名
- path：输出目录的绝对路径（注意是绝对路径）

> 即使存在多个入口起点，webpack 只有一个输出配置，不对 output 进行配置时，默认输出到./dist 文件夹。

### 2.2.1.单入口输出

单入口打包常用配置如下：

```
const path = require("path");

module.exports = {
  entry: {
    app: "./src/app.js"
  },
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist") // __dirname表示js文件执行的绝对路径，使用path.resolve生成dist文件夹的绝对路径
  }
};
```

webpack 打包后，dist 文件夹下生成 bundle.js 文件

![单入口输出](https://user-gold-cdn.xitu.io/2019/8/26/16ccc11af2dc85f5?w=158&h=141&f=png&s=5314)

### 2.2.2.多入口输出

当存在多入口时，应该使用占位符来确保每个文件具有唯一的名称，否则 webpack 打包会报错。

![webpack打包报错](https://user-gold-cdn.xitu.io/2019/8/26/16ccc12778c7f000?w=811&h=153&f=png&s=10706)

> **占位符 name 与 entry 对象中的 key 一一对应**。

正确的写法如下：

```
const path = require("path");

module.exports = {
  entry: {
    app: "./src/app.js",
    main: "./src/main.js",
    home: "./src/home.js"
  },
  output: {
    filename: "[name].js", // 使用占位符来表示
    path: path.resolve(__dirname, "dist")
  }
};
```

webpack 打包后，在 dist 文件夹下生成了 app.js、home.js、main.js 文件。

![多入口输出](https://user-gold-cdn.xitu.io/2019/8/26/16ccc12e25e608a9?w=158&h=227&f=png&s=8079)

### 2.2.3.hash、chunkhash、contenthash 揭秘

在揭秘 hash、chunkhash、contenthash 之前，我们先看下 webpack 打包输出信息。

![webpack打包信息](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1382a605ba9?w=754&h=297&f=png&s=38529)

Hash：与整个项目构建相关，当项目中不存在文件内容变更时，hash 值不变，当存在文件修改时，会生成新的一个 hash 值。
<br>
Version：webpack 版本
<br>
Time：构建时间
<br>
Build at：开始构建时间
<br>
Asset：输出文件
<br>
Size：输出文件大小
<br>
Chunks：chunk id
<br>
ChunkNames：对应 entryChunkName
<br>
Entrypoint：入口与输出文件的对应关系

如果使用占位符来表示文件，当文件内容变更时，仍然生成同样的文件，无法解决浏览器缓存文件问题。借助于 hash、chunkhash、contenthash 可以有效解决问题。

#### （1）hash

整个项目构建生成的一个 md5 值，项目文件内容不变，hash 值不变。

使用 hash 关联输出文件名称

```
const path = require("path");

module.exports = {
  entry: {
    app: "./src/app.js",
    main: "./src/main.js",
    home: "./src/home.js"
  },
  output: {
    filename: "[name].[hash].js",
    path: path.resolve(__dirname, "dist")
  }
};

```

> filename: "[name].[hash:7].js"表示去 hash 值的前 7 位

webpack 打包，看到新生成文件带上了 hash 值

![hash](https://user-gold-cdn.xitu.io/2019/8/26/16ccc13f24414105?w=268&h=223&f=png&s=11813)

当我们修改 app.js 文件内容后，重新打包，发现可以生成了新的 hash 值，所有文件的名称都发生了变更。

![新hash](https://user-gold-cdn.xitu.io/2019/8/26/16ccc143165091a4?w=317&h=229&f=png&s=14103)

**问题**：当我修改了项目中的任何一个文件时，导致未修改文件缓存都将失效。

#### （2）chunkhash

webpack 构建时，根据不同的入口文件，构建对应的 chunk，生成对应的 hash 值，每个 chunk 的 hash 值都是不同的。

使用 chunkhash 关联文件名

```
const path = require("path");

module.exports = {
  entry: {
    app: "./src/app.js",
    main: "./src/main.js",
    home: "./src/home.js"
  },
  output: {
    filename: "[name].[chunkhash].js",
    path: path.resolve(__dirname, "dist")
  }
};

```

使用 webpack 打包后，dist 目录下，每个 bundle 文件都带有不同的 chunkash 值。

![chunkhash](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1486d462bb6?w=294&h=225&f=png&s=13154)

修改 app.js 内容，重新打包，只有 app 文件名称发生了变更。

![新chunkash](https://user-gold-cdn.xitu.io/2019/8/26/16ccc15396c54f7a?w=285&h=224&f=png&s=14406)

使用 chunkhash 可以有效解决 hash 缓存失效问题，但是当在 js 文件里面引入 css 文件时，将 js、css 分别打包，若 js 件内容变化时，css 文件名称也会变更。

app.js 中引入了 css 文件

```
import "./css/style.css";

console.log("app");
```

webpack 配置

```
const path = require("path");
const miniCssExtractPlugin = require("mini-css-extract-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
  entry: {
    app: "./src/app.js",
    main: "./src/main.js",
    home: "./src/home.js"
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [miniCssExtractPlugin.loader, "css-loader"]
      }
    ]
  },
  output: {
    filename: "[name].[chunkhash].js",
    path: path.resolve(__dirname, "dist")
  },
  plugins: [
    new CleanWebpackPlugin(), // 清空dist目录
    new miniCssExtractPlugin({
      // 抽离css文件
      filename: "css/style.[chunkhash].css"
    })
  ]
};
```

打包，dist 文件夹下生成了 css 与 js 文件，chunkhash 一致。

![chunkhash-css-js](https://user-gold-cdn.xitu.io/2019/8/26/16ccc15a9cab3d5d?w=313&h=317&f=png&s=17173)

当我们修改 app.js 文件内容后，重新打包，发现 css 文件名也变更了，css 文件缓存将失效，这显然不是我们想要的结果。

![chunk-css-js1](https://user-gold-cdn.xitu.io/2019/8/26/16ccc15dbb4a82e4?w=306&h=323&f=png&s=18120)

**问题**：js 引入 css 等其他文件时，js 文件变更，css 等文件名也会变更，缓存失效。

#### （3）contenthash

contenthash 表示由文件内容产生的 hash 值，内容不同产生的 contenthash 值也不一样。借助于 contenthash 可以解决上述问题，只要 css 文件不变，缓存一直有效。

修改 webpack 配置，css filename 使用 contenthash 表示。

```
const path = require("path");
const miniCssExtractPlugin = require("mini-css-extract-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
  entry: {
    app: "./src/app.js",
    main: "./src/main.js",
    home: "./src/home.js"
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [miniCssExtractPlugin.loader, "css-loader"]
      }
    ]
  },
  output: {
    filename: "[name].[contenthash].js",
    path: path.resolve(__dirname, "dist")
  },
  plugins: [
    new CleanWebpackPlugin(), // 清空dist目录
    new miniCssExtractPlugin({
      // 抽离css文件
      filename: "css/style.[contenthash].css"
    })
  ]
};

```

打包后，dist 目录下，生成了 css、js 文件，app 文件包含 chunkhash 值，css 文件包含 contenthash 值。

![contenthash](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1635c0ba463?w=303&h=295&f=png&s=17122);

修改 app.js 文件内容，重新打包，app 文件重命名了，css 文件没变，缓存有效。

![contenthash1](https://user-gold-cdn.xitu.io/2019/8/26/16ccc167095f80b4?w=281&h=295&f=png&s=16414)

> **项目中 css 等非 js 文件抽离最好使用 contenthash。**

## 2.3.模式（mode）

webpack 提供了 mode 配置选项，用来选择使用响应的内置优化，不配置 mode 选项时，默认使用 production 模式。

mode 选项有 3 个可选值：production（生产模式、默认）、development（开发模式）、none。

### 2.3.1.production 模式

production 模式下，会自动开启 Tree Shaking（去除无用代码）和文件压缩（UglifyJs）。

在 fun.js 中定义了 2 个函数

```
export function f1() {
  console.log("f1");
}

export function f2() {
  console.log("f2");
}
```

在 app.js 中只引入了 f1

```
import { f1 } from "./fun";

f1();
```

production 模式打包，查看打包后的文件，只引入了 f1，并且代码进行了压缩。

![production模式打包](https://user-gold-cdn.xitu.io/2019/8/26/16ccc17476a2ecb6?w=734&h=54&f=png&s=10314)

### 2.3.2.development 模式

development 模式下，webpack 会启用 NamedChunksPlugin 和 NamedModulesPlugin 插件。

同样的代码，development 模式下打包，将 f1 和 f2 都一起打包了，而且代码并没有进行压缩。

![development模式下打包](https://user-gold-cdn.xitu.io/2019/8/26/16ccc17894f9396a?w=1361&h=71&f=png&s=13313)

### 2.3.3.none 模式

none 模式下，webpack 不会使用任何内置优化，这种模式很少使用。

## 2.4.loader

loader 用于对模块的源代码进行转换。loader 可以实现文件内容的转换，比如将 es6 语法转换为 es5 语法，将 scss 转换为 css 文件，将 image 转换为 base 64 位编码。一般 loader 配置在 module 的 rules 选项中。

常用的 loader 有：

- 处理 js/jsx/ts
  <br>
  babel-loader：将代码转换为 ES5 代码
  <br>
  ts-loader：将 ts 代码转换为 js 代码
- 处理样式
  <br>
  style-loader：将模块的导出作为样式添加到 DOM style 中
  <br>
  css-loader：解析 css 文件后，使用 import 加载，并且返回 CSS 代码
  <br>
  less-loader：加载和转译 less 文件
  <br>
  sass-loader：加载和转译 sass/scss 文件
- 文件转换
  <br>
  file-loader：将文件发送到输出文件夹，返回相对 url，一般用于处理图片、字体
  <br>
  url-loader：和 file-loader 功能一样，但如果文件小于限制，返回 data URL，常用于图片 base 64 转换

下面就以 scss 转换的例子，描述如何使用 loader

app.js 中引入了 main.scss 文件

```
// app.js

import "./css/main.scss";
```

webpack 配置如下

```
const path = require("path");
const miniCssExtractPlugin = require("mini-css-extract-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
  mode: "development",
  entry: {
    app: "./src/app.js"
  },
  module: { // 针对项目中不同类型模块的处理
    rules: [ // 匹配请求的规则数组
      {
        test: /\.scss$/, // 检测scss文件结尾的文件
        exclude: /node_modules/, // 排除查找范围
        include: [path.resolve(__dirname, "src/css")], // 限定查找范围
        use: [miniCssExtractPlugin.loader, "css-loader", "sass-loader"] // loader链式调用，从最右边向左处理
      }
    ]
  },
  output: {
    filename: "[name].[chunkhash].js",
    path: path.resolve(__dirname, "dist")
  },
  plugins: [
    new CleanWebpackPlugin(), // 清空dist目录
    new miniCssExtractPlugin({
      // 抽离css文件
      filename: "css/style.[contenthash].css"
    })
  ]
};

```

其中，sass-loader 用于将 scss 文件编译成 css 文件，css-loader 用于解释 import()，miniCssExtractPlugin 用于将 css 抽离到单独的文件中。

关于 loader 有几点说明：

1.loader 支持链式调用，一组链式 loader 按照相反的顺序执行，loader 链中的前一个 loader 返回值给下一个 loader，最后一个 loader 输出文件。
<br>
上面例子中，loader 执行顺序：sass-loader => css-loader => miniCssExtractPlugin.loader。

2.loader 可以使用 options 对象进行配置，像下面这样：

```
module: {
    //
    rules: [
      {
        test: /\.scss$/, // 检测scss文件结尾的文件
        exclude: /node_modules/, // 排除查找范围
        include: [path.resolve(__dirname, "src/css")], // 限定查找范围
        use: [
          miniCssExtractPlugin.loader,
          {
            loader: "css-loader",
            options: {
              modules: true
            }
          },
          "sass-loader"
        ] // loader链式调用，从最右边向左处理
      }
    ]
  },
```

## 2.5.plugins（插件）

插件是 webpack 的支柱功能，旨在解决 loader 无法实现的其他事。插件可以携带参数，配置插件需要向 plugins 数组中传入 new 实例。

常用的插件有：

clean-webpack-plugin：清空 dist 文件夹
<br>
html-webpack-plugin：生成 html 文件
<br>
mini-css-extract-plugin：抽离 css 文件
<br>
optimize-css-assets-webpack-plugin：优化和压缩 css
<br>
css-split-webpack-plugin：针对 css 大文件进行拆分
<br>
webpack-bundle-analyzer：webpack 打包结果分析
<br>
webpack.DllPlugin：创建 dll 文件和 manifest 文件
<br>
webpack.DllReferencePlugin：把只有 dll 的 bundle 引用到需要的预编译的依赖。
<br>
SplitChunksPlugin：拆分代码块，在 optimization.splitChunks 中配置。

下面以 html-webpack-plugin 为例说明 plugins 的用法，这里只列出 plugins 部分的配置

```
plugins: [
    new CleanWebpackPlugin(), // 清空dist目录
    new miniCssExtractPlugin({
      // 抽离css文件
      filename: "css/style.[contenthash].css"
    }),
    new HtmlWebpackPlugin({ // 生成插件实例
      filename: "index.html", // 生成模板的名称
      minify: {
        collapseWhitespace: true, // 去除空格
        minifyCSS: true, // 压缩css
        minifyJS: true, // 压缩js
        removeComments: true, // 移除注释
        removeEmptyElements: true, // 移除空元素
        removeScriptTypeAttributes: true, // 移除script type属性
        removeStyleLinkTypeAttributes: true // 移除link type属性
      }
    })
  ]
```

打包后，生成了 index.html 文件

![html-webpack-plugin1](https://user-gold-cdn.xitu.io/2019/8/26/16ccc17fcfe482b9?w=238&h=251&f=png&s=13791)

打开 index.html，看到 css 和 js 文件被引入了

![html-webpack-plugin2](https://user-gold-cdn.xitu.io/2019/8/26/16ccc182dfbfc9e0?w=694&h=225&f=png&s=18498)

接下来描述 webpack 其他常用的一些配置 resolve、devServer、devtool。

## 2.6.resolve（解析）

resolve 选项设置模块如何被解析。

### 2.6.1.alias

创建 import 或 require 的别名，确保模块引入变得简单。

下面的例子针对 css、util 文件夹 设置了 alias 别名，引入文件夹下面的文件可以直接使用相对地址。

```
resolve: {
    alias: {
      css: path.resolve(__dirname, "src/css"),
      util: path.resolve(__dirname, "src/util")
    }
  },
```

app.js 文件中引入 util 文件夹下的 common.js 文件，就会引入 src/util/common.js 文件。

```
import fun1 from "util/common.js";
```

### 2.6.2.extensions

自动解析引入模块的扩展，按照从左到右的顺序解析。

```
resolve: {
  extensions: [".js", ".json"]
}
```

在 app.js 中引入 common.js 可以不携带后缀，由 webpack 自动解析。

```
import fun1 from "util/common";
```

### 2.6.3.mainFiles

解析目录时要使用的文件名，默认

```
mainFiles: ["index"]
```

也可以指定多个 mainFiles，会依次从左到右解析

```
mainFiles: ["index", "main"]
```

比如需要从 util 文件夹下引入 index.js 文件，import 只需要导入到 util 文件夹，webpack 会自动从 util 文件夹下引入 index.js 文件。

```
import index from "util";
```

> **extentsion 和 mainFiles 属性虽然会方便开发者简写，但是会增加 webpack 额外的解析时间。**

## 2.7.devServer

devServer 主要用于 development 模式配置本地开发服务器，需要安装 webpack-dev-server。

```
npm i webpack-dev-server -g
```

devServer 常用的配置项如下：

```
devServer: {
    contentBase: path.resolve(__dirname, "dist"), // 告诉服务器从哪里提供内容
    host: "localhost", // 制定一个host，默认localhost
    port: 3000, // 请求端口号，默认8080
    compress: true, // 启用gzip压缩
    https: true, // 开启http服务
    hot: true, // 启用模块热替换
    open: true, // 自动打开默认浏览器
    index: "index.html", // 页面入口html文件，默认index.html
    headers: {
      // 所有响应中添加首部内容
      "X-Custom-Foo": "bar"
    },
    proxy: {
      "/api": "http://localhost:3000"
    }
  }
```

读取配置文件，启动开发服务器。

```
webpack-dev-server --config webpack.dev.js
```

## 2.8.devtool

source map 一个存储源代码与编译代码对应位置的映射信息文件，它是专门给调试器准备的，它主要用于 debug。

webpack 通过配置 devtool 属性来选择一种 source map 来增强调试过程。

以下是官方对于 devtool 的各种 source map 的比较：

![devtool](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1873fe58077?w=988&h=877&f=png&s=132226)

> **development 模式下 devtool 设置为 cheap-module-eval-source-map，production 模式下 devtool 设置为 souce-map。**

# 3.webpack 实践

接下来将通过一个完整的例子实现 react 项目的完整 webpack 配置。
先全局安装 webpack 和 webpack-dev-server。

```
npm i webpack webpack-dev-server -g
```

## 3.1.配置执行文件

新建一个目录，结构如下：

![目录完整结构](https://user-gold-cdn.xitu.io/2019/8/26/16ccc19854f8f5ea?w=302&h=460&f=png&s=21706)

其中 public 文件夹下包含 index.html 入口 html 文件，src 文件夹下包含 index.js 入口 js 文件，css 文件夹、font 文件夹、image 文件夹。

webpack 配置如下：

```
// webpack.config.js

const path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "[name].[chunkhash:7].js",
    path: path.resolve(__dirname, "dist")
  }
};
```

在目录下使用`npm init`新建 package.json 文件，设置 dev 和 build 的 script，
分别用于开发模式和生产模式。

![package.json](https://user-gold-cdn.xitu.io/2019/8/26/16ccc18d4647b163?w=472&h=343&f=png&s=31979)

## 3.1.处理 jsx、es6

在 react 项目中，我们使用 jsx 和 es6 语法，为了兼容低版本浏览器，需要通过 babel 转换。

先安装 babel 相关依赖包

```
npm i babel-loader @babel/core @babel/preset-env  @babel/plugin-transform-runtime  @babel/preset-react @babel/polyfill @babel/runtime -D
```

babel-loader：处理 ES6 语法，将其编译为浏览器可以执行的 js 语法
<br>
@babel/core-babel：babel 核心模块
<br>
@babel/preset-env：转换 es6 语法，支持最新的 javaScript 语法
<br>
@babel/preset-react：转换 jsx 语法
<br>
@babel/plugin-transform-runtime: 避免 polyfill 污染全局变量，减小打包体积
<br>
@babel/polyfill: ES6 内置方法和函数转化垫片

将 index.js 作为入口文件，引入 App.jsx 组件

```
//index.js
import React from "react";
import ReactDOM from "react-dom";
import App from "./views/App";
console.log(App);

ReactDOM.render(<App />, document.getElementById("root"));

//App.jsx
import React, { Component } from "react";

class App extends Component {
  render() {
    return <h2>This is a react app.</h2>;
  }
}

export default App;
```

webpack 配置如下:

```
const path = require("path");
const { CleanWebpackPlugin } = require("clean-webpack-plugin"); // clean-webpack-plugin用来清空dist文件夹

module.exports = {
  mode: "production",
  entry: {
    app: "./src/index.js"
  },
  module: {
    rules: [
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        use: [
          {
            loader: "babel-loader"
          }
        ]
      }
    ]
  },
  resolve: {
    extensions: [".jsx", ".js"]
  },
  output: {
    filename: "[name].[chunkhash:7].js",
    path: path.resolve(__dirname, "dist")
  },
  plugins: [new CleanWebpackPlugin()]
};
```

> **clean-webpack-plugin：清空 dist 文件夹**

新建.babelrc 文件

```
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": ["@babel/plugin-transform-runtime"]
}
```

执行`npm run build`，打包成功，在 dist 文件夹下生成了 app.js 文件

![jsx转换](https://user-gold-cdn.xitu.io/2019/8/26/16ccc192cf6bbf42?w=345&h=288&f=png&s=13020)

## 3.2.配置 html 模板

配置 html 模板表示配置 index.html 文件相关配置，将打包后的文件引入到 index.html 文件，通过 html-webpack-plugin 插件实现。

先安装 html-webpack-plugin 插件

```
npm i html-webpack-plugin -D
```

webpack 配置如下：

```
const path = require("path");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin"); // 引入html-webpack-plugin插件

module.exports = {
  mode: "production",
  entry: {
    app: "./src/index.js"
  },
  module: {
    rules: [
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        use: [
          {
            loader: "babel-loader"
          }
        ]
      }
    ]
  },
  resolve: {
    extensions: [".jsx", ".js"]
  },
  output: {
    filename: "[name].[chunkhash:7].js",
    path: path.resolve(__dirname, "dist")
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      filename: "index.html", // 模板文件名
      template: path.resolve(__dirname, "public/index.html"), // 模板文件源
      minify: {
        collapseWhitespace: true, // 压缩空格
        minifyCSS: true, // 压缩css
        minifyJS: true, // 压缩js
        removeComments: true, // 移除注释
        caseSensitive: true, // 去除大小写
        removeScriptTypeAttributes: true, // 移除script的type属性
        removeStyleLinkTypeAttributes: true // 移除link的type属性
      }
    })
  ]
};
```

执行 npm run build，打包成功，在 dist 文件夹下生成了 index.html 和 app.js 打包文件

![模板配置](https://user-gold-cdn.xitu.io/2019/8/26/16ccc19e52196671?w=321&h=239&f=png&s=11513)

打开 index.html，引入了 app.js 打包文件

![](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1a2e41d722f?w=532&h=338&f=png&s=29023)

## 3.3.编译 css、scss

在 index.js 中引入 main.scss 文件

```
import React from "react";
import ReactDOM from "react-dom";
import App from "./views/App";
import "./css/main.scss";

ReactDOM.render(<App />, document.getElementById("root"));
```

当在 js 文件中引入 css/scss 文件时，需要经过 loader 转换，才能引入到 index.html 文件中。

安装相关依赖包

```
npm i css-loader sass-loader node-sass mini-css-extract-plugin optimize-css-assets-webpack-plugin css-split-webpack-plugin -D
```

sass-loader：将 scss/sass 文件编译为 css
<br>
css-loader：解析 import/require 导入的 css 文件
<br>
mini-css-extract-plugin：将 js 中引入的 css 文件抽离成单独的 css 文件
<br>
optimize-css-assets-webpack-plugin：优化和压缩 css 文件
<br>
css-split-webpack-plugin：css 文件拆分

webpack 配置如下：

```
const path = require("path");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");
const CSSSplitWebpackPlugin = require("css-split-webpack-plugin").default;

module.exports = {
  mode: "production",
  entry: {
    app: "./src/index.js"
  },
  module: {
    rules: [
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        use: ["babel-loader"]
      },
      {
        test: /\.(sa|sc|c)ss$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "sass-loader"]
      }
    ]
  },
  resolve: {
    extensions: [".jsx", ".js"]
  },
  output: {
    filename: "[name].[chunkhash:7].js",
    path: path.resolve(__dirname, "dist")
  },
  plugins: [
    new CleanWebpackPlugin(),
    new MiniCssExtractPlugin({
      filename: "css/[name].[hash:7].css",
      chunkFilename: "[id].css"
    }),
    new OptimizeCSSAssetsPlugin({
      assetNameRegExp: /\.css$/g,
      cssProcessor: require("cssnano"), // //引入cssnano配置压缩选项
      cssProcessorPluginOptions: {
        preset: [
          "default",
          {
            discardComments: {
              // 移除注释
              removeAll: true
            },
            normalizeUnicode: false
          }
        ]
      },
      canPrint: true
    }),
    new CSSSplitWebpackPlugin({
      size: 4000, // 超过4kb的css文件进行拆分
      filename: "[name]-[part].[ext]"
    }),
    new HtmlWebpackPlugin({
      filename: "index.html", // 模板文件名
      template: path.resolve(__dirname, "public/index.html"), // 模板文件源
      minify: {
        collapseWhitespace: true, // 压缩空格
        minifyCSS: true, // 压缩css
        minifyJS: true, // 压缩js
        removeComments: true, // 移除注释
        caseSensitive: true, // 去除大小写
        removeScriptTypeAttributes: true, // 移除script的type属性
        removeStyleLinkTypeAttributes: true // 移除link的type属性
      }
    })
  ]
};
```

执行`npm run build`，在 dist 文件夹下生成了 css 文件夹和编译的 css 文件

![编译css、scss文件](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1a70104002b?w=283&h=313&f=png&s=12791)

打开 index.html，css 文件被引入到 index.html 中。

![编译css、scss到index.html](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1aa3e504aa9?w=554&h=270&f=png&s=27201)

## 3.4.处理图片、字体文件

在 index.js 中引入图片，在 main.scss 文件中引入字体库

```
// index.js
import React from "react";
import ReactDOM from "react-dom";
import App from "./views/App";
import "./css/main.scss";
import image from "./image/image1.png";

const newImage = new Image();
newImage.src = image;
newImage.style.cssText = "width: 100px; height: 100px;";
document.body.append(newImage);

ReactDOM.render(<App />, document.getElementById("root"));
```

在 main.scss 文件中引入字体库

```
// main.scss
@font-face {
  font-family: 'MyFont';
  src: url('../font/icomoon.eot') format('eot'),
    + url('../font/icomoon.woff') format('woff');
  font-weight: 600;
  font-style: normal;
}

body {
  background-color: blue;
}
```

当在 js 文件中引入图片或字体文件时，需要通过 url-loader 和 file-loader 来处理。

file-loader：解析 import/require 导入的文件，将其输出到生产目录，并产生一个 url 地址。
<br>
url-loader：不超过限定 limit 时，转换为 base64 url。

安装 file-loader 和 url-loader

```
npm i url-loader file-loader -D
```

webpack module 部分配置如下：

```
...
module: {
  ...
  rules: [
    {
        test: /\.(png|jpg|jpeg|gif|svg)/,
        use: [
          {
            loader: "url-loader",
            options: {
              name: "[name]_[hash].[ext]",
              outputPath: "images/",
              limit: 204800 // 小于200kb，进行base64转码
            }
          }
        ]
      },
      {
        test: /\.(eot|woff2?|ttf)/,
        use: [
          {
            loader: "url-loader",
            options: {
              name: "[name]-[hash:5].min.[ext]",
              limit: 5000,
              outputPath: "fonts/"
            }
          }
        ]
      }
  ]
  ...
}
...
```

执行`npm run build`打包，在 dist 目录下生成了 image 文件夹和 font 文件夹

![图片字体转换](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1ad91a24b77?w=336&h=313&f=png&s=18243)

打开 index.html，图片成功引入

![图片字体转换](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1b1e1f61dc7?w=519&h=212&f=png&s=19530)

## 3.5.配置 devServer

webpack-dev-server 就是在本地为搭建了一个小型的静态文件服务器，有实时重加载的功能，为将打包生成的资源提供了 web 服务，适用于本地开发模式。

```
 devServer: {
    contentBase: path.join(__dirname, "../dist"), // 资源目录
    host: 'localhost', // 默认localhost
    port: 3000, // 默认8080
    hot: true, // 支持热更新
    inline: true
  }
```

执行`npm run dev`，web 服务起在 localhost:3000

![devServer配置](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1b6ecb93f09?w=770&h=396&f=png&s=58919)

## 3.6.提取公共代码

当我们在代码里引入了第三方库和公共代码时，可以使用 splitChunks 提取公共代码，避免加载的包太大。

webpack 配置如下：

```
...
optimization: {
    splitChunks: {
      // 提取公共代码
      chunks: "all", //  async(动态加载模块)，initital（入口模块），all（全部模块入口和动态的）
      minSize: 3000, // 抽取出来的文件压缩前最小大小
      maxSize: 0, // 抽取出来的文件压缩前的最大大小
      minChunks: 1, // 被引用次数,默认为1
      maxAsyncRequests: 5, // 最大的按需(异步)加载次数，默认为 5；
      maxInitialRequests: 3, // 最大的初始化加载次数，默认为 3；
      automaticNameDelimiter: "~", // 抽取出来的文件的自动生成名字的分割符，默认为 ~；
      name: "vendor/vendor", // 抽取出的文件名，默认为true，表示自动生成文件名
      cacheGroups: {
        // 缓存组
        common: {
          // 将node_modules模块被不同的chunk引入超过1次的抽取为common
          test: /[\\/]node_modules[\\/]/,
          name: "common",
          chunks: "initial",
          priority: 2,
          minChunks: 2
        },
        default: {
          reuseExistingChunk: true, // 避免被重复打包分割
          filename: "common.js", // 其他公共函数打包成common.js
          priority: -20
        }
      }
    }
  },
  ...
```

执行`npm run build`，在 dist 文件夹下生成了 vendor.js 包

![提取公共代码](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1bde2ed3f35?w=350&h=307&f=png&s=14732)

打开 index.html，vendor.js 成功引入

![提取公共代码index.html](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1c0d8b87533?w=466&h=230&f=png&s=21383)

## 3.7.分离 webpack 配置文件

由于开发环境和生产环境下的 webpack 配置存在公共配置，因此最好将公共配置抽离成 webpack.common.js，然后针对开发环境和生产环境分别配置，通过 webpack-merge merge 配置，即可满足开发环境和生产环境不同的配置。

先安装 webpack-merge，用来 merge webpack 配置项

```
npm i webpack-merge -D
```

在目录下新建 tools 文件夹，存放 webpack 相关配置

![建立tools文件夹](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1c44c014a84?w=319&h=416&f=png&s=22981)

新建 pathConfig.js 文件，返回 entry js、output 目录及 index.html 模板目录的绝对地址

```
// pathConfig.js
const path = require("path");
const fs = require("fs");

const appDirectory = fs.realpathSync(process.cwd()); // 获取当前根目录
const resolvePath = (relativePath) => path.resolve(appDirectory, relativePath);

module.exports = {
  appHtml: resolvePath("public/index.html"), // 模板html
  appBuild: resolvePath("dist"), // 打包目录
  appIndexJs: resolvePath("src/index.js") // 入口js文件
};
```

webpack 公共配置，引入 pathConfig.js 文件

```
// webpack.common.js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");
const CSSSplitWebpackPlugin = require("css-split-webpack-plugin").default;
const { appIndexJs, appBuild, appHtml } = require("./pathConfig");

module.exports = {
  entry: {
    app: appIndexJs
  },
  output: {
    filename: "[name].[hash:7].js",
    path: appBuild
  },
  module: {
    rules: [
      {
        test: /\.js[x]?$/, // jsx、js处理
        exclude: /node_modules/,
        use: ["babel-loader"]
      },
      {
        test: /\.(sa|sc|c)ss$/, // scss、css处理
        use: [MiniCssExtractPlugin.loader, "css-loader", "sass-loader"]
      },
      {
        test: /\.(png|jpg|jpeg|gif|svg)/, // 图片处理
        use: [
          {
            loader: "url-loader",
            options: {
              name: "[name]_[hash].[ext]",
              outputPath: "images/",
              limit: 204800 // 小于200kb采用base64转码
            }
          }
        ]
      },
      {
        test: /\.(eot|woff2?|ttf)/, // 字体处理
        use: [
          {
            loader: "url-loader",
            options: {
              name: "[name]-[hash:5].min.[ext]",
              limit: 5000, // 5kb限制
              outputPath: "fonts/"
            }
          }
        ]
      }
    ]
  },
  resolve: {
    extensions: [".jsx", ".js"]
  },
  optimization: {
    splitChunks: {
      // 提取公共代码
      chunks: "all", //  async(动态加载模块)，initital（入口模块），all（全部模块入口和动态的）
      minSize: 3000, // 抽取出来的文件压缩前最小大小
      maxSize: 0, // 抽取出来的文件压缩前的最大大小
      minChunks: 1, // 被引用次数,默认为1
      maxAsyncRequests: 5, // 最大的按需(异步)加载次数，默认为 5；
      maxInitialRequests: 3, // 最大的初始化加载次数，默认为 3；
      automaticNameDelimiter: "~", // 抽取出来的文件的自动生成名字的分割符，默认为 ~；
      name: "vendor/vendor", // 抽取出的文件名，默认为true，表示自动生成文件名
      cacheGroups: {
        // 缓存组
        common: {
          // 将node_modules模块被不同的chunk引入超过1次的抽取为common
          test: /[\\/]node_modules[\\/]/,
          name: "common",
          chunks: "initial",
          priority: 2,
          minChunks: 2
        },
        default: {
          reuseExistingChunk: true, // 避免被重复打包分割
          filename: "common.js", // 其他公共函数打包成common.js
          priority: -20
        }
      }
    }
  },
  plugins: [
    new CleanWebpackPlugin(),
    new MiniCssExtractPlugin({
      filename: "css/[name].[hash:7].css",
      chunkFilename: "[id].css"
    }),
    new OptimizeCSSAssetsPlugin({
      assetNameRegExp: /\.css$/g,
      cssProcessor: require("cssnano"), // //引入cssnano配置压缩选项
      cssProcessorPluginOptions: {
        preset: [
          "default",
          {
            discardComments: {
              // 移除注释
              removeAll: true
            },
            normalizeUnicode: false
          }
        ]
      },
      canPrint: true
    }),
    new CSSSplitWebpackPlugin({
      size: 4000, // 超过4kb进行拆分
      filename: "[name]-[part].[ext]"
    }),
    new HtmlWebpackPlugin({
      filename: "index.html", // 模板文件名
      template: appHtml, // 模板文件源
      minify: {
        collapseWhitespace: true, // 压缩空格
        minifyCSS: true, // 压缩css
        minifyJS: true, // 压缩js
        removeComments: true, // 移除注释
        caseSensitive: true, // 去除大小写
        removeScriptTypeAttributes: true, // 移除script的type属性
        removeStyleLinkTypeAttributes: true // 移除link的type属性
      }
    })
  ]
};
```

开发环境 webpack 配置

```
// webpack.dev.config.js
const path = require("path");
const merge = require("webpack-merge");
const baseConfig = require("./webpack.config");

module.exports = merge(baseConfig, {
  mode: "development",
  devtool: "cheap-module-eval-source-map",
  devServer: {
    contentBase: path.join(__dirname, "../dist"),
    port: 3000,
    historyApiFallback: true,
    hot: true,
    inline: true
  }
});
```

生产环境配置，启用 webpack-bundle-analyzer 进行打包分析，启用 compression-webpack-plugin 生成 gzip 压缩。

```
// webpack.prod.config.js
const merge = require("webpack-merge");
const baseConfig = require("./webpack.com.config");
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer").BundleAnalyzerPlugin;
const CompressionWebpackPlugin = require("compression-webpack-plugin");

module.exports = merge(baseConfig, {
  mode: "production",
  devtool: "source-map",
  plugins: [
    new CompressionWebpackPlugin({
      filename: "[path].gz[query]", // path-原资源路径，query-原查询字符串
      algorithm: "gzip", // 压缩算法
      threshold: 0, // 文件压缩阈值
      minRatio: 0.8 // 最小压缩比例
    }),
    new BundleAnalyzerPlugin()
  ]
});
```

修改 package.json 中 script 中 dev 和 build，--config 表示读取后面的文件作为配置文件。

```
 "scripts": {
    "dev": "webpack-dev-server --config ./tools/webpack.dev.config.js",
    "build": "webpack --config ./tools/webpack.prod.config.js"
  },
```

执行`npm run build`，项目打包成功

![打包成功](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1c8df71d553?w=319&h=329&f=png&s=16190)

执行`npm run dev`，启动开发者模式，运行在 localhost:3000。

![开发成功](https://user-gold-cdn.xitu.io/2019/8/26/16ccc1cd1ff23e14?w=1156&h=220&f=png&s=57567)

到此为止，我们的案例就完成了。

代码地址：[案例链接](https://github.com/dadaiwei/webpack-learning/tree/master/webpack-compltete-practice)

# 结语

看完这篇文章，相信大家对于 webpack 已经有了一个初步的了解，学习 webpack 最好的方式还是多动手实践，觉得不错的小伙伴可以点个赞（码字不易，灰常感谢）。

# 相关链接

webpack 官方链接：https://www.webpackjs.com/
