# webpack从零搭建vue3+ts项目
1. 安装webpack核心依赖：npm i -D webpack webpack-cli（命令行工具）
2. 在src平级目录下创建webpack.config.js，构建时会自动加载该配置文件，也可以--config xxx来指定自定义文件。注意：项目初始化是package.json的选项type设为commonjs。
3. 指定构建打包入口和出口：
```js
module.exports = {
    entry: './src/main.ts',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist'),
    }
}
// 再运行npx webpack，此时输出dist目录，基础webpack初始化成功。
```
4. HTML模板文件处理。webpack只能处理js文件，对html处理需要通过插件plugin或者loader。
- 安装插件 npm i html-webpack-plugin -D
- 创建与src平级index.html文件作为模板
- webpack配置html资源处理plugin
```js
// webpack.config.js
module.exports = {
  // ... 其他配置
  plugins: [
    new HtmlWebpackPlugin({
      template: "./index.html",
      title: 'xxx'
    }),
  ],
};

// index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
    <div id="app"></div>
</body>
</html>
```
5. css样式文件处理。
- 安装依赖 npm i css-loader style-loader -D
```
css-loader：负责将 css 文件编译成 Webpack 能识别的模块
style-loader：会动态创建一个 Style 标签，里面放置 Webpack 中 css 模块内容
构建之后，css样式以style标签的形式在页面生效
```
- 处理css代码兼容性，安装 npm i -D postcss-loader postcss postcss-preset-env
```
postcss-loader: 使用postcss处理css代码
postcss： 用js工具和插件生态系统转换css代码的工具，允许使用css最新语法，并且还可以通过插件扩展
postcss-preset-env： 提供了未来css规范中的特性，使得你可以在现代浏览器中使用它们，同时保持向后兼容性。
```
- webpack.config.js添加rules模块控制css资源处理
```js
module.exports = {
  // ... 其他配置
  module: {
    rules: [
      /** 处理 CSS 文件 */
        {
            test: /\.css$/,
            use: ['style-loader', 'css-loader', {
                loader: 'postcss-loader', // postcss兼容性
                options: {
                    postcssOptions: {
                        plugins: [
                            'postcss-preset-env'
                        ]
                    }
                }
            }]
        },
    ]
  }
};
```
6. vue文件处理
- 安装vue框架 npm i vue@latest
- 安装编译.vue文件依赖 npm i vue-loader @vue/compiler-sfc -D
```
vue-loader：允许你在 webpack 模块打包器中使用 Vue.js 单文件组件（.vue 文件）
@vue/compiler-sfc：这个包是 Vue.js 的一个核心库，用于将 Vue 的单文件组件转换成 JavaScript 模块
```
- webpack.config.js配置
```js
const { VueLoaderPlugin, default: loader } = require('vue-loader');
module.exports = {
  /** ...其他配置 */
  module: {
    rules: [
      /** 处理 Vue 文件， vue-loader 不支持 oneOf */
      {
        test: /\.vue$/,
        use: ["vue-loader"],
      }
    ]
  },
  plugins: [
    /** vue文件处理 */
    new VueLoaderPlugin(),
  ]
};
```
7. 对ts代码的处理和支持
- 安装 npm i -D ts-loader
- webpack.config.js配置
```js
module.exports = {
  /** ...其他配置 */
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: "ts-loader",
        exclude: /node_modules/
      }
    ]
  }
}
```
- 创建与src目录平级的文件tsconfig.json，配置ts规则
- 引入ts之后，.vue文件的引入会报错，在项目中创建文件custom.d.ts文件，使ts推断资源类型
```js
declare module '*.vue' {
    import type { DefineComponent } from 'vue'
    const component: DefineComponent<{}, {}, any>
    export default component
}

declare module '*.css';
// 定义任何ts无法推断的自定义类型
```
8. 网页媒体资源拷贝处理。如果直接在index.html文件中通过link标签引入icon资源是无法做到的，因为构建之后引入的资源不在dist目录下。
- 安装 npm i copy-webpack-plugin -D
- 将图片，字体等资源放在public目录下
- index.html文件中 <link rel="icon" type="image/svg+xml" href="./favicon.svg"> 引入网页icon
- webpack.config.js配置
```js
const path = require("path");
const CopyPlugin = require("copy-webpack-plugin");

/** 配置项 */
module.exports = {
  /** ...其他配置 */
  plugins: [
    //复制public资源到index里面
    new CopyPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, "./public"), //将根文件夹下 public文件夹复制到dist目录下
          to: path.resolve(__dirname, "./dist")
        }
      ]
    })
  ]
};
```
9. 图片资源的引入处理。webpack5内置对图片，字体等资源的处理，无需安装依赖
- webpack.config.js配置
```js
module.exports = {
  mudule: {
    output: {
      /** 图片、字体等资源输出命名方式（注意用hash） */
      assetModuleFilename: "static/media/[name].[hash:6][ext]",
    },
    rules: [
      /** 处理图片文件 */
      {
        test: /\.(jpe?g|png|gif|webp|svg)$/,
        type: 'asset/resource', // 资源类型为 asset，Webpack 会根据文件类型选择合适的加载器进行处理, 比如 url-loader
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
          }
        }
      }
    ]
  }
}
```
10. 开发环境下热更新
- 安装依赖 npm i webpack-dev-server -D
- webpack.config.js配置
```js
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  // ... 其他配置
  /* 配置开发服务器 */
  devServer: {
    host: "localhost", // 启动服务器域名
    port: "3000", // 启动服务器端口号
    open: true, // 是否自动打开浏览器
    hot: true, // 热更新
    compress: true, // 压缩编译后文件
  },
  mode: "development",
}
```
- 在package.json配置启动指令
```json
  "scripts": {
	  "dev": "npx webpack serve",
  }
```
11. 引入ESLint语法检查
- 安装依赖 npm i eslint-webpack-plugin eslint -D
```
eslint-webpack-plugin：是一个 webpack 插件，将 ESLint 集成到 webpack 构建流程中，使得在构建时可以执行代码检查。
eslint：是一个流行的 JavaScript 代码质量和代码风格检查工具，它可以根据自定义的规则来检查 JavaScript 代码中的问题， 用于静态代码分析。
```
- 在src平级目录下创建.eslintrc.js并配置
```js
// .eslintrc.js
module.exports = {
    parser: "@babel/eslint-parser", // 支持最新的最终 ECMAScript 标准
    parserOptions: { // 解析选项
      ecmaVersion: 6, // ES 语法版本
      sourceType: "module", // ES 模块化
      ecmaFeatures: {
        
      },
    },
    env: { // 环境配置
      node: true, // 启用node中全局变量
      browser: true, // 启用浏览器中全局变量
      es6: true, // 启用ES6全局变量
    },
    // 继承其他规则--继承 Eslint 规则
    extends: ["eslint:recommended"],
    // ...
    // 其他规则详见：https://eslint.bootcss.com/docs/user-guide/configuring
    // 具体检查规则
    rules: {
      "no-var": "error", // 不能使用 var 定义变量
    },
  }
```
- webpack.config.js配置
```js
new ESLintWebpackPlugin({
  // 指定检查文件的根目录
  context: path.resolve(__dirname, "../src"),
  exclude: "node_modules", // 默认值
  cache: true, // 开启缓存，提升构建速度
  // 缓存目录
  cacheLocation: path.resolve(
    __dirname,
    "../node_modules/.cache/.eslintcache"
  ),
}),

```

12. 引入Babel处理js代码兼容性。Babel是js代码的编译器，主要将es6的语法转换为向旧版本兼容的js语法，以便能运行在当前或者旧版浏览器中。
- 安装依赖 npm i babel-loader @babel/core @babel/preset-env @babel/plugin-transform-runtime -D
```
babel-loader：用于在 webpack 构建过程中将 JavaScript 文件从 ES6+转换为 ES5，以便在旧版浏览器中运行。
@babel/core：这是 Babel 编译器的核心包，提供转译 JavaScript 代码的功能。
@babel/preset-env：这是 Babel 的一个预设，用于将 ES6+代码转换为向后兼容的 JavaScript 版本，以便在当前和旧版浏览器中运行。
@babel/plugin-transform-runtime: Babel 对一些公共方法使用了非常小的辅助代码，比如 _extend。默认情况下会被添加到每一个需要它的文件中。这个依赖可以将这些辅助代码作为一个独立
```
- 在与src平级的目录下创建babel.config.js
```js
// babel.config.js
module.exports = {
  // 预设
  presets: ["@babel/preset-env"],
  // 插件
  plugins: ["@babel/plugin-transform-runtime"], // 统一引入babel辅助代码-减少代码体积
};
```
- webpack.config.js配置
```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/, // 排除node_modules代码不编译
        loader: "babel-loader"
      }
    ]
  }
}
```
- babel配置缓存，可使构建速度大幅度增加
```js
/** JS 文件的 babel 处理代码语法兼容性 */
{
  test: /\.js$/,
  exclude: /node_modules/, // 排除node_modules代码不编译
  use: [
    {
      loader: "babel-loader",
      options: {
        cacheDirectory: true, // 开启babel编译缓存
        cacheCompression: false, // 缓存文件不要压缩
      },
    },
  ],
}
```
13. 引入CoreJS处理高版本语法。如果是 async 函数、promise 对象、数组的一些方法（includes）等，babel是没办法处理。更低版本浏览器会报错。
- 安装依赖 npm i @babel/eslint-parser -D
```
core-js：这是要安装的包的名称，core-js 是一个常用的 JavaScript 库，提供了对旧浏览器的兼容性支持，实现了一些新的 ECMAScript 特性。
```
- 配置 babel.config.js 按需引入对应的语法补丁
```js
module.exports = {
  // 智能预设：能够编译ES6语法
  presets: [
    [
      "@babel/preset-env",
      // 按需加载core-js的polyfill
      { useBuiltIns: "usage", corejs: { version: "3", proposals: true } }
    ]
  ]
}
```
- 配置 .eslintrc.js 解释器
```js
module.exports = {
  parser: "@babel/eslint-parser", // 支持最新的最终 ECMAScript 标准
  // 环境配置
  env: {
    node: true, // 启用node中全局变量
    browser: true, // 启用浏览器中全局变量
    es6: true, // 启用除模块之外的所有 ECMAScript 6 功能
  }
}
```
自此，一个由webpack5搭建的vue3+ts前端项目框架已经完成了。真是配置一小时，实现vite一分钟配置的功能，不过webpack强大出色的定制化能力，复杂构建流程的配置，精细的性能优化（代码分割，缓存策略，懒加载），丰富的插件生态以及灵活性在开发大型项目的时候不可或缺。