# Nginx反向代理node，实现让静态文件在同一域

>不管是Vue还是React，还是传统的网站，与node服务端连接都需要实现跨域；这样的做法都很麻烦，所以就想让静态文件和node的服务器在同一域下，这样就不需要跨域了；（当然专业的运维做这个是分分钟的事）

## 搭建node服务器

1、下载安装包

+ koa
+ koa-router

我们这里使用koa2来搭建服务器，来通过koa-router实现后端路由，提供接口；具体初始化文件夹什么的，这里就不阐述了；

2、创建app.js

    const koa = require('koa');
    const Router = require('koa-router');
    
    const app = new koa();
    const router = new Router();
    
    router.get('/api/index',async(ctx,next) => {
        ctx.status = 200;
        ctx.body = {
            data:1
        } 
    });
    
    app.use(router.routes());
    app.use(router.allowedMethods());
    
    app.listen(3000,()=>{
        console.log('连接成功!')
    })

搭建一个端口为3000，使用koa-router实现一个get请求为/api/index，返回一个json数据；

## 创建一个能够请求的index.html

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>
    <body>
        <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
        <script>
            $(function(){
                $.get('/api/index',function(res){
                    console.log(res);
                },'json')
            })
        </script>
    </body>
    </html>
    
这里通过CDN因为JQuery，使用Ajax请求上面node服务器创建的/api/index接口；

## 配置Nginx
    
    server {
        liseten 80;
        server_name localhost;
        
        location / {
            root www;
            index index.html;
        }
        
        location /api/ {
            proxy_pass http://127.0.0.1:3000;
        }
    }
    
上面是Nginx的一些基本配置，网上教程很多，就不做赘述了；这里主要是记录Nginx怎么将两个路径的资源，放在同一域下；
这里配置 
    
    location /api/ 

的意思是当服务器请求地址的首部为/api/时，将代理到下面的路径；
