# 作用域插槽
- 简单来说就是子组件通过slot和v-bind将自己的数据传递给父组件，父组件拿到这些数据后选择如何渲染到子组件的插槽里。
# 异步组件
```js
const asyncComp = defineAsyncComponent(() => {
    return new Promise(resolve => {
        resolve( 
            // 获取到的组件
        )
    })
});
// 一般搭配import动态导入使用，如：
const asyncComp1 = defineAsyncComponent(() => import('./components/xxx.vue'));
// 当如还有很多高级选项
const AsyncComp = defineAsyncComponent({
  // 加载函数
  loader: () => import('./Foo.vue'),

  // 加载异步组件时使用的组件
  loadingComponent: LoadingComponent,
  // 展示加载组件前的延迟时间，默认为 200ms
  delay: 200,

  // 加载失败后展示的组件
  errorComponent: ErrorComponent,
  // 如果提供了一个 timeout 时间限制，并超时了
  // 也会显示这里配置的报错组件，默认值是：Infinity
  timeout: 3000
})
// 惰性激活，详见vue官方文档异步组件章节

<template>
    <asyncComp/>
    <asyncComp1/>
</template>
```
# vue常见API
1. 
2. 
# vue内置组件
1. 
2. 