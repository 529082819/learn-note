# koa2实现简易的webpack-dev-server热更新

>闲来无事，用koa2撸了一个简易的webpack-dev-server;其实网上很多express搭建的热更新，但是koa2很少；欢迎大佬拍砖和赐教；
[源码](https://github.com/zhuangZhou/koa-webpack-dev-server)

## 配置基本的webpack

1、下载安装包

+ webpack
+ webpack-cli
+ html-webpack-plugin
+ css-loader
+ style-loader

由于这里用的是最新的webpack版本，所以需要安装webpack-cli。

2、创建并配置webpack.config.js

这里就不具体配置了，如需要详细配置教程，[请上这里](https://github.com/zhuangZhou/webpack_demo);

基础配置完成后，在plugins里配置HotModuleReplacementPlugin插件用于热更新；由于HotModuleReplacementPlugin是webpack内置的插件，所以不需要安装，直接引用就行
    
    const webpack = require('webpack');
    plugins:[
        new webpack.HotModuleReplacementPlugin()
    ]
    
## 实现webpack-dev-middleware的koa2中间件

webpack-dev-middleware是实现webpack-dev-server的核心中间件；

1、下载安装包

     npm run webpack-dev-middleware -D

2、创建devMiddleware.js

    const webpackDev  = require('webpack-dev-middleware')

    const devMiddleware = (compiler, opts) => {
        const middleware = webpackDev(compiler, opts)
        return async (ctx, next) => {
            await middleware(ctx.req, {
                end: (content) => {
                    ctx.body = content
                },
                setHeader: (name, value) => {
                    ctx.set(name, value)
                }
            }, next)
        }
    }
    
    module.exports=devMiddleware;
    
## 实现webpack-hot-middleware的koa2中间件

1、下载安装包

    npm run webpack-hot-middleware -D
    
2、创建hotMiddleware.js文件

    const webpackHot = require('webpack-hot-middleware')
    const PassThrough = require('stream').PassThrough;
    
    const hotMiddleware = (compiler, opts) => {
        const middleware = webpackHot(compiler, opts);
        return async (ctx, next) => {
            let stream = new PassThrough()
            ctx.body = stream
            await middleware(ctx.req, {
                write: stream.write.bind(stream),
                writeHead: (status, headers) => {
                    ctx.status = status
                    ctx.set(headers)
                }
            }, next)
        }
        
    }
    
    module.exports = hotMiddleware;
    
## 配置koa入口文件和引入中间件

1、下载安装包

    npm run koa -D
    
2、创建app.js

    const koa = require('koa');
    const webpack = require('webpack');
    
    const app = new koa();
    const config = require('../webpack.config');
    const compiler = webpack(config);
    
    const devMiddleware = require('./devMiddleware');
    const hotMiddleware = require('./hotMiddleware');
    
    app.use(devMiddleware(compiler, {
        noInfo: true,
        publicPath: config.output.publicPath
    }));
    
    app.use(hotMiddleware(compiler,{
        reload:true
    }));
    
    // app.use(serve(__dirname + '/src/', {
    //     extensions: ['html']
    // }));
    
    app.listen(3000, () => {
        console.log('Example app listening om port 3000!\n');
    });

## 运行

1、在package.json里配置运行命令

    "scripts": {
        "watch": "webpack --watch",
        "server": "node server/app.js",
        "build": "webpack"
    }
    
2、启动刚刚配置好的server
    
    npm run server
    
3、解决问题

当运行后，在浏览器里输入localhost:3000,运行成功；但是当修改index.js后不会自动刷新浏览器；

在webpack.config.js里的entry修改入口文件

    entry: {
        index: ['webpack-hot-middleware/client?noInfo=true&reload=true','./src/index.js']
    }
    
这样就可以自动刷新了；

## 关于热更新HTML

这里还是有一个问题，就是关于热更新html文件;由于HTML-webpack-plugin插件运行时是不经过entry入口的，所以就不能自动编译HTML文件；

在网上查了很多种方法，其中就是有一种就是:使用raw-loader，将index.html文件引入index.js中，这样就可以编译了

    module:{
        rules:[
            {
                test:/\.(htm|html)$/,  use: 'raw-loader'
            }
        ]
    }
    
但是仔细想想，这样是有问题的，就是不能将index.html文件分离出来，所以不推荐；目前还正在研究怎么热更新，后续会更上；

## 总结

[参考](https://www.cnblogs.com/liuyt/p/7217024.html)