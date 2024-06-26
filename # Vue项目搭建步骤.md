# Vue项目搭建步骤

###  1.vue-cil脚手架初始化项目

`vue create vuetest`

### 2.配置packe.json文件

使用open命令，项目运行起来的时候，浏览器自动打开

```json
"scripts": {
    "serve": "vue-cli-service serve --open",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint"
  },
```

### 3.关闭eslint校验功能

   在根目录下创建一个vue.config.js文件

![image-20220131110340965](C:\Users\123\AppData\Roaming\Typora\typora-user-images\image-20220131110340965.png)

```js
module.exports = {
  // 关闭eslint校验工具
  lintOnSave: false,
}
```

### 4.配置jsconfig.json

  配置别名@【代表src文件夹】

```json
{
    "compilerOptions": {
        "baseUrl":"./",
        "paths": {
                    "@/*":["src/*"]
                 }
            },
        "exclude": ["node_modules","dist"]
}
```

### 5.配置路由

第一步、在src文件夹里创建router文件夹，在创建index.js和routers.js文件

> index.js       配置路由的文件
>
> routers.js    配置路由地址，路由名称，引入路由组件的文件

第二步、在index.js中配置router

```js
// 配置路由的地方·
import Vue from "vue";
import VueRouter from "vue-router";
import store from "@/store";
// 使用插件
Vue.use(VueRouter);
// 引入路由
import routes from "./routes";
//配置路由

const router = new VueRouter({
 // 配置路由
  routes,
});

//全部暴露出
export default router;

```

routers文件夹写法：

```js
// 引入路由组件
import Home from "../pages/Home";
//将文件暴露出
export default [
  // 重定向，在项目跑起来的时候，访问/。立马让他定向到首页
  {
    path: "/",
    redirect: "/home",
  },
  {
    path: "/home",
    name: "Home",
    component: Home,
    meta: { showFooter: true },
  },
]
```

在入口文件引入router

第三步、解决**编程式路由跳转到当前路由(参数不变)，多次执行会抛出NavigiationDuplicated错误**

解决方法：在index.js文件重写push和replace

```js
// 把VueRouter原型对象的push，先保存一份
let originPush = VueRouter.prototype.push;
let originReplace = VueRouter.prototype.replace;
// 重写push|replace
// 第一个参数：告诉原来push方法，往那里跳转（传递那些参数）
// 第二个参数：成功的回调
// 第三个参数：失败的回调
// call||apply区别
// 相同点，都可以调用函数一次，都可以篡改函数的上下文一次
// 不同点，call和apply传递参数：call传递参数用逗号隔开，apply方法执行，传递数组
VueRouter.prototype.push = function (location, resolve, reject) {
  if (resolve && reject) {
    originPush.call(this, location, resolve, reject);
  } else {
    originPush.call(
      this,
      location,
      () => {},
      () => {}
    );
  }
};
VueRouter.prototype.replace = function (location, resolve, reject) {
  if (resolve && reject) {
    originReplace.call(this, location, resolve, reject);
  } else {
    originReplace.call(
      this,
      location,
      () => {},
      () => {}
    );
  }
};
```

#### Vue路由的常见面试题

1. 路由传递参数（对象写法）path是否可以结合params参数一起使用？

   答：路由跳转传参的时候，对象的写法是name、path形式。但是需要注意的是，path这种写法不可以和params参数一起使用

2. 如何指定params参数可传可不穿？

   答：在配置路由的时候，在占位符后边加上一个问好【params就可以可传可不传】

   ```js
   {
       path: "/detail/:skuId?",  //在占位符后边加上一个问好
       name: "Detail",
       component: Detail,
       meta: { showFooter: true },
     },
   ```

   

3. params参数可以传递也可以不传递，但是如果传递时空串，如何解决？

   答：使用undefined解决，params参数可以传递、不传递【空的字符串】

   ```js
    this.$router.push({
             name: "Search",
             params: {
               keyword: this.keyWord || undefined,  //使用undefined解决,params传递空串
             },
             query: this.$route.query,
           });
   ```

   路由组件能不能传递props参数？

   答：可以，有三种方法，在router文件夹下的index.js里需要传参的路由里写

      1.第一种写法：布尔值写法

   ​			`props:true`				

     2. 第二种写法：对象写法

           `props:{a:1,b:2}`

     3. 第三种写法：函数写法 （常用）

        `props($route) => ({keyWord:$route.params.keyWord,k:$route.query.keyWord})`

### 4.axios二次封装

​	第一步、安装axios插件

​	`npm i axios`

​	第二步、在vue.config.js配置跨域，接口统一管理

```js
module.exports = {
  // 关闭eslint校验工具
  lintOnSave: false,
  // 代理跨域
  devServer: {
    proxy: {
      "/api": {
        target: "http://39.98.123.211",   请求地址
        // pathRewrite: { "^/api": "" },
      },
    },
  },
};

```



第三步、在src文件夹下创建api文件夹，在api文件夹下创建index.js和request.js文件

在request.js文件夹中二次封装axios

```js
	// 对于axios进行二次封装
import axios from "axios";
// 1:利用axios对象的方法create，去创建一个axios实例
// 2：request就是axios，只不过稍微配置了一下
let requests = axios.create({
  // 配置对象
  // 基础路径，发请求的时候，路径当中会出现api
  baseURL: "",
  // 代表请求超时的时间5s
  timeout: 5000,
});
// 请求拦截器：在发请求之前，请求拦截器可以检测到，可以在请求发出去之前做一些事情
requests.interceptors.request.use((config) => {
  
  return config;
});
// 响应拦截器
requests.interceptors.response.use(
  // 成功的回调函数；服务器相应数据回来以后，响应拦截器可以检测到，可以做一些事情
  (res) => {
    return res;
  }
),
  (err) => {
    // 响应失败的回调函数
    alert("服务器响应数据失败");
  };

// 对外暴露
export default requests;
```

在入口文件main.js引入

在index.js文件配置接口

```js
// 当前这个模块：API进行统一管理
import requests from "./request";
// 三级联动接口
export function reqCategoryList() {
  return requests({
    url: "/api/product/getBaseCategoryList",
    method: "get",
  });
}
```

### 5.配置vuex

第一步、安装vuex插件

`npm i vuex`

第二步、配置store

​	在src文件夹下创建store文件夹，在store文件夹下创建index.js文件

```js
import Vue from "vue";
import Vuex from "vuex";

// 需要使用插件一次
Vue.use(Vuex);
// state:仓库储存数据的地方
const state = {};
// mutations:修改state的唯一手段
const mutations = {};
// actions:处理actions，可以书写自己的业务逻辑，也可以异步处理
const actions = {};
// getters:理解为计算属性，用于简化仓库数据，让组件获取仓库的数据更加方便
const getters = {};

export default new Vuex.Store({
  state,
  mutations,
  actions,
  getters
});
```

在入口文件main.js文件中引入

> 当需要配置vuex的文件较多时，可以使用模块化

在store文件夹下创建modules文件夹，里面放置对应的仓库

>组件化后的文件需要打开命名空间

` namespaced: true,`

### 打包项目

`npm run build`



### 宝塔更换后端地址

![image-20220204223423884](C:\Users\123\AppData\Roaming\Typora\typora-user-images\image-20220204223423884.png)
