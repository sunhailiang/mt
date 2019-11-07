# koa
- 安装
   - cnpm i -g koa-generator
- 创建项目
   - koa2  -e  projectName
      - -e：声明是用ejs模板引擎
- 安装依赖
   - npm start projectName
- 运行项目
   - npm run dev

# global全局对象

# asycn&await
- 异步请求的黄金组合
   - 有async可以没有await，但是没有async肯定不能有await 

# koa中间件的原理

> 单独的文件夹

- 所谓的中间件，就是独立处理一个功能的函数

```js

function middle (ctx) {
  global.console.log('koa-pv', ctx.path);
}
module.exports = function () { // 在一个独立的js文件中到处一个函数
  return async function (ctx, next) {
    middle(ctx) // 中间件的功能
    await next() // 执行下一个中间件
  }
}

```

# koa的路由
- 引入koa的路由
   - const router=require('koa-router')
- 路由的方法
   - get
   - post
   - put
   - delete
   - del
   - all  代表匹配所有类型，一般放在中间件末尾
- 重定向
   - router.redirect('/',fn)
- 动态路由
   - router.get('/users/:id',(ctx,next)>{})
   - 访问形式：/users/001 
   - router.get('/users/:gender/:id',(ctx,next)>{})
   - 参数并列，动过ctx.params获取
- 命名路由
   - router.get('user', '/users/:id', (ctx, next) => {});
- 多个中间件路由
```js
   router.get(
     '/users/:id',
      (ctx, next) => {
         return User.findOne(ctx.params.id).then(function(user) {
                ctx.user = user;
                next();
          });
      },
      ctx => {
         console.log(ctx.user);
    }
);
```
- 嵌套路由
```js
var forums = new Router();
var posts = new Router();

posts.get('/', (ctx, next) => {...});
posts.get('/:pid', (ctx, next) => {...});
forums.use('/forums/:fid/posts', posts.routes(), posts.allowedMethods());

// responds to "/forums/123/posts" and "/forums/123/posts/123"
app.use(forums.routes());

```

- 前缀路由

```js
var router = new Router({
  prefix: '/users'
});

router.get('/', ...); // responds to "/users"
router.get('/:id', ...); // responds to "/users/:id"
```

# cookie

- 写入cookie ctx.cookies.set('cookieName',cookieValue)
- 读取cookie ctx.cookies.get('cookieName')

# mongoose

- 安装mongoose
   - npm i mongoose

- 启动
   - mongod
   - 默认访问端口 27017

- 可视化管理工具(常用)
   - adminMongo
   - git下将adminMongo源码从github上clone下来： 
   - git clone https://github.com/mrvautin/adminMongo.git
   - 进入adminMongo路径：
   - cd adminMongo
   - 安装依赖： 
   - npm install
   - 启动应用： 
      - npm start或者node app
      - Connection name：随便写一个，必填项
      - Connection string:
      - 连接 MongoDB 的规则：mongodb://<user>:<password>@127.0.0.1:<port>/<db> 
      - 其中<user>:<password> <db>可以不填写
      - 输入启动 mongo 时显示的url ： mongodb://127.0.0.1:27017


# mongoose基本使用
- 首先在项目中新建一个文件夹用于存放数据模型，如：db
- 在文件夹中新建文件config.js
```js
module.exports= { dbs: 'mongodb://localhost:27017/dbs' }
```
- 新建文件夹models用于存放数据模型
   - 文件夹内存放数据模型 如:Person.js

``` js
const mongoose = require('mongoose')
//  创建数据模型
let personSchema = new mongoose.Schema({
  name: String,
  age: Number
})

//  映射数据模型并导出
module.exports = mongoose.model('Persons', personSchema)
```

- 引入mongoose

> app.js

```js

var db = mongoose.connect(dbConfig.dbs, { useNewUrlParser: true })
var db = mongoose.connection;
db.on('error', function (error) {
  console.log('错了', error);

});
db.once('open', function () {
  console.log('链接成功');
});

```

- 业务处理
> router>index.js

```js
const Person = require('../db/models/person')

// 增加数据
router.post('/addPerson', async function (ctx) {
  const person = new Person({
    name: ctx.request.body.name,
    age: ctx.request.body.age
  })
  var code
  try {
    await person.save()
    code = 0
  } catch (e) {
    code = -1
  }
  ctx.body = {
    code
  }
})

// 查询数据

router.post('/getPerson', async function (ctx) {
  // 根据条件查询一条数据
  const person = await Person.findOne({ name: ctx.request.body.name })
  // 根据条件查询一堆数据
  const persons = await Person.find({ name: ctx.request.body.name })
  ctx.body = {
    code: 0,
    person,
    persons
  }
})
// 更新数据
router.post('/updatePerson', async function (ctx) {
  // 更新一条数据
  const person = await Person.where({ name: ctx.request.body.name }).update({
    age: ctx.request.body.age
  })
  // 更新一堆数据
  const person = await Person.where({ name: ctx.request.body.name }).updateMany({
    age: ctx.request.body.age
  })
  ctx.body = {
    code: 0
  }
})

// 删除数据


```

# redis基本使用
> 为了防止用户量过大，服务器内存压力大，选择redis高性能数据库进行session存储
- 安装redis
- 启动redis服务
   - redis安装目录->redis-server.exe
- 想要用于reidis存储session需要两个中间件
      - koa-generic-session
      - koa-redis
- 引入中间件App.js

>App.js

```js

var session = require('koa-generic-session')
var redis = require('koa-redis')
// 使用中间件
app.keys = ['keys'] // 用于session加密
app.use(session({
  store: new redis()
}))

```

> koa-pv.js（测试中间件）

```js
function pv (ctx) {
  // 写session
  ctx.session.count++
  global.console.log("pv", ctx.path)
}
``` 

> 浏览器>控制台>Application>cookies
- 观察session

## 改变session的key名称
> App.js

```js
// 修改session的key值

app.use(session({
  key: 'test', // 指定key
  prefix: "mt", // 设置存储字段的前缀   
  store: new redis()
}))

```
## 如何读写session
- koa-generic-session 这个中间件会让koa上下文对象（ctx）多一个session属性,我们可以对这个对象进行读写

## redis查看存储的值
- redis目录>redis-cli  
   - 查看所有key: keys * 

## 直接读写操作redis
> 路由页面

```js
// 引入中间件
const Redis =require('koa-redis')

// 新建redis客户端
const Store=new Redis().client

// 存储数据
router.get('/saveKey', async function (ctx, next) {
  const st = await Store.hset('fix', 'name', Math.random())
  ctx.body = {
    code: 0
  }
})

// 此时访问端口就存储了数据
// redis客户端命令： hget key名称 name 即可找到对应key的值

```

# Nuxt.js
- 解决什么问题？
   - 服务端渲染数据。直接返回渲染好的页面，提高交互体验
   - 解决spa的seo问题

- vue 做SSR的最佳实践库
   - vue2.0 支持
   - 支持vue router 
   - 支持vuex
   - 支持vue server render
   - vue-mate

- 安装
   - npx create-nuxt-app <项目名> 
   - 接下来一路选ui模板，node框架等等
   - 后端框架KOA或者其他
   - cd 项目名  进入项目
   - cnpm i

# 项目实践
## 创建项目
   - npx create-nuxt-app <项目名>  

# 项目配置

## node中不支持import语法
- 原因
   - 可以看package.json中的scripts,dev命令可以发现是采用node执行服务
   - 而node中不支持es6语法
- 解决
   - 使用babel转码
- 解决方法
> package.json

- 安装插件

```js
yarn add babel-cli babel-core babel-preset-es2015  babel-preset-stage-0
```

- 将--exec babel-node追加到dev和start后面

```js
  "scripts": {
    "dev": "cross-env NODE_ENV=development nodemon server/index.js --watch server --exec babel-node",
    "build": "nuxt build",
    "start": "cross-env NODE_ENV=production node server/index.js --exec babel-node",
    "generate": "nuxt generate",
    "test": "jest"
  },
```
- 重启重新编译

## 使用sass

- 安装sass
   - yarn add sass-loader node-sass

## nuxt.config.js

- css
   - element-ui.css
   - element-reset.css
   - 自定义打包后的main.css
- build
   - 开启缓存
   - cache:true


# 默认布局页面

- 使用elementUI布局容器

> layout>default
```html
<el-container>
  <el-header>Header</el-header>
    <el-main>   
      <nuxt />
    </el-main>
  <el-footer>Footer</el-footer>
</el-container>
```

- 创建公共组件
> components 
- mkdir public   
   - mkdir header
      - touch index.vue  
   - mkdir footer
      - touch index.vue



  
   





   
