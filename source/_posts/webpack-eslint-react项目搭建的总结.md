title: webpack-eslint-react项目搭建的总结
date: 2018-04-25 11:05:36
tags:
- webpack
- react
- redux
- eslint
---
## 前言
离我上一次搭建项目已经快一年了，接下来会折腾下服务端，所以得总结下之前用到的东西。重新构建一个项目，发现很多框架工具又更新了，有时候，搭建好的基础架构，其实是可以通用的，所以弄了个仓库记录([戳我看代码](https://github.com/SelinaYu/webpack-eslint-react))，不定时更新，这篇文章不会教你如何搭起一个项目，因为官方文档比我说的还要详细，主要是总结现在搭起的项目架构遇到的一些问题，采用的技术栈主要是 webpack, react, redux。当前使用的是webpack4 版本。
<!--more-->

##  关于Webpack

  先说下webpack4的新特性：
  
**1.添加 mode 参数，development 和 production， 默认下是 production**
   
development 模式下的特点：会输出开发阶段的详细错误日志和提示

production 模式下的特点：会自动优化代码，去掉在开发阶段运行的代码，使用 uglifyjs 压缩代码，还进行 Scope hoisting和Tree-shaking,生产出更小体积的代码。

**2.使用 optimization 配置**

  (1) webpack4 移除了`commonchunk`插件，使用 `optimization` 进行配置。在 `optimization` 中配置 `runtimeChunk` 和 `splitChunk`, 分别对应 `commonchunk` 插件的 `mainifest` 和 `vendor` 文件。具体配置看官方文档。
  
  (2) `webpack.optimize.UglifyJsPlugin` 也移除了，只需要使用 `optimization.minimize` 为true就行，production mode下面自动为true，如果想使用第三方的压缩插件也可以在 `optimization.minimizer` 的数组列表中进行配置,可以参考上面提到仓库里的配置。

**3.使用 mini-css-extract-plugin**

`extract-text-webpack-plugin` 建议替换成 `mini-css-extract-plugin`，原来的插件还是可以用的，替换后的插件打包出来的css体积更小。
具体的配置[点我点我](https://npm.taobao.org/package/mini-css-extract-plugin)。


## 关于eslint

三种方式配置
1. 使用 `.eslintrc.* `文件
2. 在 `package.json` 中设置 `eslintConfig` 属性
3. 在代码里注释


(1)使用[`babel-eslint`](https://github.com/babel/babel-eslint)检测 ES6 语法

```
parser: "babel-eslint",
```
(2) 使用[ eslint-plugin-react](https://github.com/yannickcr/eslint-plugin-react) 检测 React 的代码规则,或者想修改预定义的规则，

```
  "plugins": [
    "react"
  ],
```
(3)另外，我们要告诉 webpack 构建时使用了eslint, 需要用到 [`eslint-loader`](https://npm.taobao.org/package/eslint-loader),并希望能把代码自动按照eslint的配置进行格式化，则进行如下配置
```
      {
        enforce: 'pre',
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'eslint-loader',
            options: {
              fix: true  // 自动修复
            }
          }
        ]
      },
```
(4)有时，想偷懒，并不想自己定义这么多规则，我们可以直接使用已经符合最佳实践的规则。比如 [Airbnb Style Guide](https://github.com/airbnb/javascript)

```
"extends": "airbnb",
```

更多的配置可以看[ 官网 ](https://eslint.org/docs/user-guide/getting-started)


## 关于 react-router && react-router-redux

以前用`react-router-redux` 主要是把路由的状态保存到`redux` 的`store`中，当使用最新的 `react-router` 时，发现 `router` 的状态可以直接通过 `props` 的`history`属性获取到路由信息。在我看来，如果使用`redux`的话，使用两个都可以的。具体使用参考相关官网。

扩展阅读： [react-router-redux在react-router成为4.0后是不是不需要了](https://segmentfault.com/q/1010000010489394)

**在这里要特意提一个点，我们在页面点击导航跳转到对应的路由往往是可以的，可是跳转路由后，刷新页面或者直接访问该URL时，会发现无法正确响应。看请求可以发现资源请求的路径不对。
开发环境可以通过修改webpack的配置,(或者你可以通过服务器端的配置来解决)：**

**修改webpack配置**

```
devServer: {
  historyApiFallback: true
}
```

如果有修改 publicPath 的话，也需进行相应的修改

```
// output.publicPath: '/assets/'
historyApiFallback: {
  index: '/assets/'
}
```

上面只是适用开发环境，生产环境的打包之后，还是会存在相应的问题，这时还可以通过以下几种方式进行解决，[点这里看文档](https://react-guide.github.io/react-router-cn/docs/guides/basics/Histories.html)

**Node 环境**
```
const express = require('express')
const path = require('path')
const port = process.env.PORT || 8080
const app = express()

// 通常用于加载静态资源
app.use(express.static(__dirname + '/public'))

// 在你应用 JavaScript 文件中包含了一个 script 标签
// 的 index.html 中处理任何一个 route
app.get('*', function (request, response){
  response.sendFile(path.resolve(__dirname, 'public', 'index.html'))
})

app.listen(port)
console.log("server started on port " + port)
```

**nginx环境**
```
	server {
        listen       9091;
        server_name  localhost;
        location / {
            root E:\webpack-eslint-react\dist;
            index  index.html index.htm;
            try_files $uri /index.html;
        }
        location /api {
            proxy_pass http://127.0.0.1:9090;  //这个是对/api开头的请求，重定向成 http://127.0.0.1:9090
        }
    }
```
**Apache**

如果使用`Apache`服务器，则需要在项目根目录创建`.htaccess`文件，文件包含如下内容：

```
RewriteBase /
RewriteRule ^index\.html$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.html [L]
```

## 关于项目架构

```
│  .babelrc
│  .editorconfig
│  .eslintignore
│  .eslintrc.js
│  .gitignore
│  package.json
│  README.md
│  
├─build   // webpack 通用配置
│      webpack.base.js
│      webpack.dev.js
│      webpack.prod.js
│      
└─src
    │  index.js   // 入口文件
    │  routes.js  // 路由配置
    │  
    ├─actions
    ├─assets                  // 工具类文件
    │  │  404.html           // 404页面
    │  │  index.html        // 入口模板文件 
    │  │  
    │  └─images             // 图片资源
    │          404.jpg
    │          favicon.ico
    │          logo.jpg
    │          
    ├─components            // 公共组件
    │  └─NotFoundPage       // 404组件
    │          index.js
    │          index.scss
    │          
    ├─mock                  // 模拟数据
    │      mockData.js
    │      
    ├─pages            // 对应路由，connect 之后的组件
    │  ├─App
    │  │      app.css
    │  │      index.js
    │  │      
    │  ├─Home
    │  │      index.js
    │  │      
    │  └─List
    │          index.js
    │          
    └─reducers
            
```
## 写在最后

说这么多，不如看下成品： [webpack-eslint-react](https://github.com/SelinaYu/webpack-eslint-react)

## 扩展阅读：

[在React+Babel+Webpack环境中使用ESLint](https://www.cnblogs.com/le0zh/p/5619350.html)
[webpack 官网](https://webpack.js.org/guides/)
[elsint 官网](https://eslint.org/docs/user-guide/getting-started)
[react-router](https://reacttraining.com/react-router/)
[react-router-redux](https://github.com/ReactTraining/react-router/tree/master/packages/react-router-redux)
[windows下nginx的安装及使用方法入门](https://www.cnblogs.com/saysmy/p/6609796.html)
[react-router browserHistory刷新页面404问题解决](https://my.oschina.net/u/3451529/blog/1596319)