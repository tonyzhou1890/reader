# reader

> 电子书阅读器--只支持txt格式

## 功能支持

1. 目前支持txt格式

2. 目前支持两种方式：url和本地读取

3. url方式：指定查询字段address的值

    比如：reader.tony93.top/?address=http:store.tony93.top/舞舞舞.txt

    编码格式为 utf-8

4. 本地读取：选择编码为 gb2312 的 txt 文本

## Build Setup

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

## 更新日志
### v1.0.0--2019.04.08
1. 支持url指定文本和本地文本

### v1.0.1--2019.04.22
1. 修复进度大于等于100时页面显示不正确的问题（实际上是修复了 TuBook 组件的问题）。
2. 修复右键出现设置面板的问题。

### v1.0.2--2019.04.24
1. 修复设置改变后跳回到首页的问题。
