# 初始化项目
- 通过npm init -y初始化项目，生产package.json文件
- 创建src文件夹，根目录下创建vite.config.ts文件
- 对于ts项目，vite原生支持，只需在安装依赖typescript后创建tsconfig.json文件，vite自动识别
# 安装依赖
- npm i -D vite typescript @types/node
- 如果创建的是vue项目则 npm i vue，npm i -D vue-tsc @vitejs/plugin-vue。vue-tsc用于vue3中ts的类型检查，@vitejs/plugin-vue提供vue单文件的编译和解析功能
- 如果创建的是react项目则 npm i react react-dom，npm i -D @types/react @types/react-dom @vitejs/plugin-react。react-dom用于将react组件渲染到DOM树，@types/react-dom则提供类型支持；@vitejs/plugin-react提供对react组件jsx语法支持以及快速热更新。
# 在package.json文件中配置启动和打包脚本
- react： run--vite   build--tsc -b && vite build
- vue： run--vite   build--vue-tsc -b && vite build
# 引入eslint对js语法检查，对代码风格进行格式化
- 安装核心依赖 npm install --save-dev eslint @eslint/js 会询问一些基础配置信息然后生成eslint.config.js文件
- 配置启动脚本：eslint . --ext .js,.ts,.vue
- 配置修复脚本：eslint . --ext .js,.ts,.vue --fix
- 安装vite插件，npm i -D vite-plugin-eslint并配置：
```js
import eslintPlugin from 'vite-plugin-eslint';
export default defineConfig({
  plugins: [
    eslintPlugin({
      include: ['src/**/*.js', 'src/**/*.vue', 'src/**/*.ts', 'vite.config.ts'],    // 指定需要检查的文件
      exclude: ['node_modules/**', 'dist/**'],    // 指定不需要检查的文件
      fix: true,    // 是否自动修复
      cache: false    // 是否启用缓存
    })
  ]
});
```
# vite原生支持postcss，只需创建postcss.config.js文件即可使用postcss
- 安装依赖 npm i postcss autoprefixer cssnano postcss-preset-env
- 做如下配置
```js
import autoprefixer from "autoprefixer";
import cssnanoPlugin from "cssnano";
import postcssPresetEnv from 'postcss-preset-env';

export default {
    plugins: [
        autoprefixer(), // 自动添加前缀
        cssnanoPlugin({ // css压缩
            preset: 'default'
        }),
        postcssPresetEnv({ // 允许使用css最新特性而不需要考虑浏览器兼容性
            stage: 1, // 选择阶段，1表示启用所有较新的特性
            // 其他新特性
        })
    ]
};
```