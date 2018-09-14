# tree table

> 基于vue和element-ui中table实现的tree table，传入树形接口数据即可。可无限层级。

## demo

> [tree table](https://no-simple.github.io/vue-tree-table/)

## 方式说明

>   * 主要依靠el-table中的`:row-style="toggleDisplayTr"` 方法实现，具体可看代码注释
>   * `formatConversion`该方法将树形数据扁平化，并添加折叠相应字段以及标识
``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
