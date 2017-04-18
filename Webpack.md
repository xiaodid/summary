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
