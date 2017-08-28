# webpack查看每个chunk的modeul情况
1. install `stats-webpack-plugin`
2. add profile in webpack.config.js
```javascript
webpackConfig.profile = true
```
3. add plugin config in webpack.config.js
```javascript
webpackConfig.plugins.push(
  new StatsPlugin('stats.json', {
    chunkModules: true
  })
)
```
4. build
5. stats.json file will be generated in ```dist``` folder
6. open [webpack-visualizer](https://chrisbateman.github.io/webpack-visualizer/)

# 不同文件使用不同的publicPath
* js：可以使用webpack自己的publicPath。
* css：因为css是通过extract-text-webpack-plugin生成的，这个插件的extract方法可以接收一个publicPath来覆盖webpack的
* 其他文件，其他文件基本是使用file-loader，或者url-loader，file-loader可以指定publicPath

# commonChunkPlugin
comp1: requires common.js and common2.js
comp2: requires common.js and common2.js

1. 提取所有公共代码
可以不在entry里添加新的entry point，直接在commonChunkPlugin中指定：
``` javascript
    new webpack.optimize.CommonsChunkPlugin({
        filename: 'xxx.js'
    })
```
这样，common.js和common2.js的内容，都被打到了xxx.js，comp1和comp2中没有公共部分代码

2. 抽取runtime部分
web pack会给第一个加载到js添加runtime代码，如果按照例子1的方式，runtime部分会被放倒xxx.js里。我们可以把这段代码提出来
``` javascript
    new webpack.optimize.CommonsChunkPlugin({
      names: ['common', 'runtime']
    })
```
这样，common.js 和 common2.js 会被打包到xxx.js，runtime部分的代码，会打到runtime.js里

3. 只把common2打到公共代码里
先指定entry
``` javascript
entry: {
    “common2”: “common2”,
    “comp1: “comp1”,
    “comp2”: “comp2”
}
```
``` javascript
    new webpack.optimize.CommonsChunkPlugin({
      names: ['common2', 'runtime']，
      minChunks: Infinity
    })
```
这样，xxx.js中就只有common2，common的代码被打入到comp1和comp2中了。
