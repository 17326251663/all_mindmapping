## 实现 vue ssr 的具体过程

#### 创建vue-cli3工程

````vue
vue create ****
````



### 安装依赖

渲染器 vue-server-render

nodejs服务器 express



 ````
npm install vue-server-renderer express -D
 ````

在根目录下创建server目录

创建index.js

````javascript
//nodejs 的服务器
const express = require('express')
const Vue = require('vue')

//创建vue实例和express实例
const app = express();
//创建渲染器
const renderer = require('vue-server-renderer').createRenderer()

//用渲染器来渲染我们的page可以得到HTML内容
const page = new Vue({
    template:"<div>HELLO VUE SSR!!!</div>"
})
app.get('/',async (req,res)=>{
    try{
        const html = await renderer.renderToString(page)
        console.log(html)
        res.send(html)
    }catch(error){
        res.status(500).send('服务器内部错误')
    }

});
app.listen(3000,()=>{
    console.log('渲染服务器启动成功')
})
````



### 编写服务端启动脚本

引入router路由

````
npm i vue-router -D
````



### 使用webpack构建项目

### 代码结构

![1581403201252](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1581403201252.png)

### 通用入口

用于创建vue实例,创建app.js

````js
//创建vue实例
import Vue from 'vue'
import App from './App.vue'
import router from './router'

export default function createApp() {
     const router = router
     const app = new Vue({
         router,
         render:h => h(App)
     })
     return {app,router}
}


````





### 服务端入口

src下创建 entry-server.js文件

````js
//渲染首屏
import createApp from './app'

//context哪来的?
export default context => {
    return new Promise((resolve, reject) => {
        const { app, router } = createApp();
        //进入首屏
        router.push(context.url)
        router.onReady(() => {
            resolve(app);
        }, reject)
    });
}
````



### 客户端入口

````js
//挂载,激活App
import createApp from './app.js'

const {app,router} = createApp();
router.onReady(()=>{
    app.$mount()
})
````



### webpack打包

编写vue.config.js

````js
// webpack插件
const VueSSRServerPlugin = require("vue-server-renderer/server-plugin");
const VueSSRClientPlugin = require("vue-server-renderer/client-plugin");
const nodeExternals = require("webpack-node-externals");
const merge = require("lodash.merge");

// 环境变量：决定入口是客户端还是服务端
const TARGET_NODE = process.env.WEBPACK_TARGET === "node";
const target = TARGET_NODE ? "server" : "client";

module.exports = {
  css: {
    extract: false
  },
  outputDir: './dist/'+target,
  configureWebpack: () => ({
    // 将 entry 指向应用程序的 server / client 文件
    entry: `./src/entry-${target}.js`,
    // 对 bundle renderer 提供 source map 支持
    devtool: 'source-map',
    // 这允许 webpack 以 Node 适用方式处理动态导入(dynamic import)，
    // 并且还会在编译 Vue 组件时告知 `vue-loader` 输送面向服务器代码(server-oriented code)。
    target: TARGET_NODE ? "node" : "web",
    node: TARGET_NODE ? undefined : false,
    output: {
      // 此处告知 server bundle 使用 Node 风格导出模块
      libraryTarget: TARGET_NODE ? "commonjs2" : undefined
    },
    // 外置化应用程序依赖模块。可以使服务器构建速度更快，并生成较小的 bundle 文件。
    externals: TARGET_NODE
      ? nodeExternals({
          // 不要外置化 webpack 需要处理的依赖模块。
          // 可以在这里添加更多的文件类型。例如，未处理 *.vue 原始文件，
          // 你还应该将修改 `global`（例如 polyfill）的依赖模块列入白名单
          whitelist: [/\.css$/]
        })
      : undefined,
    optimization: {
      splitChunks: undefined
    },
    // 这是将服务器的整个输出构建为单个 JSON 文件的插件。
    // 服务端默认文件名为 `vue-ssr-server-bundle.json`
    plugins: [TARGET_NODE ? new VueSSRServerPlugin() : new VueSSRClientPlugin()]
  }),
  chainWebpack: config => {
    config.module
      .rule("vue")
      .use("vue-loader")
      .tap(options => {
        merge(options, {
          optimizeSSR: false
        });
      });
  }
};
````



````vue
npm i webpack-node-externals lodash
````



### 脚本配置

安装依赖

````
npm i cross-env  -D
````

定义创建脚本,package.json

````js
{
  "name": "ssr",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "build:client": "vue-cli-service build",
    "build:server": "cross-env WEBPACK_TARGET=node vue-cli-service build --mode server",
    "build": "npm run build:server && npm run build:client"
  },
  "dependencies": {
    "core-js": "^2.6.5",
    "vue": "^2.6.10",
    "vue-router": "^3.1.3"
  },
  "devDependencies": {
  //  "@vue/cli-plugin-babel": "^3.11.0",
    "@vue/cli-plugin-eslint": "^3.11.0",
    "@vue/cli-service": "^3.11.0",
    "babel-eslint": "^10.0.1",
    "cross-env": "^5.2.1",
    "eslint": "^5.16.0",
    "eslint-plugin-vue": "^5.0.0",
    "express": "^4.17.1",
    "lodash.merge": "^4.6.2",
    "vue-server-renderer": "^2.6.10",
    "vue-template-compiler": "^2.6.10",
    "webpack-node-externals": "^1.7.2"
  },
  "eslintConfig": {
    "root": true,
    "env": {
      "node": true
    },
    "extends": [
      "plugin:vue/essential",
      "eslint:recommended"
    ],
    "rules": {},
    "parserOptions": {
      "parser": "babel-eslint"
    }
  },
  "postcss": {
    "plugins": {
      "autoprefixer": {}
    }
  },
  "browserslist": [
    "> 1%",
    "last 2 versions"
  ]
}

````

执行打包



### 宿主文件

最后需要定义宿主文件,创建./src/index.temp.html

````html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>vue ssr</title>
</head>
<body>
    <!--vue-ssr-outlet-->
</body>
</html>
````



### 服务器启动文件

修改服务器启动文件,所有路由都由vue接管,使用bundle渲染器生成内容, ./server/index.js

````js
//nodejs 的服务器
const express = require('express')
const Vue = require('vue')
const fs = require('fs')

//创建vue实例和express实例
const app = express();
//创建渲染器
const {createBundleRenderer} = require('vue-server-renderer')
const serverBundle = require('../dist/server/vue-ssr-server-bundle.json')
//创建客户端清单
const clientManifest = require('../dist/client/vue-ssr-client-manifest.json')
const renderer = createBundleRenderer(serverBundle,{
    runInNewContext: false,
    //必备选项
    template: fs.readFileSync('../public/index.temp.html','utf-8'),//宿主模版文件
    clientManifest
})


//用渲染器来渲染我们的page可以得到HTML内容


//中间件处理静态文件请求
app.use(express.static('../dist/client',{index: false}))

//将路由处理交给vue
app.get('*',async (req,res)=>{
    try{
        const context = {
            url: req.url
        }
        const html = await renderer.renderToString(context)
        console.log(html)
        res.send(html)
    }catch(error){
        res.status(500).send('服务器内部错误')
    }

});
app.listen(3000,()=>{
    console.log('渲染服务器启动成功')
})
````



首屏渲染在服务端完成,无法使用localStorage.鉴权方式需要进行改变.



