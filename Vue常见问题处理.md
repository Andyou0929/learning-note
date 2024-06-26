# Vue常见问题处理

## 表格分页

表格分页时需要对表格的index做相应处理，处理方法:

` index = (this.currentPage-1) * this.pageSize + index`

`this.currentPage`为分页的当前页数，

`this.pageSize`为每页允许呈现几个

## 渲染照片方法

`require('')`

## 关于scope.row

![image-20211209123825859](C:\Users\123\AppData\Roaming\Typora\typora-user-images\image-20211209123825859.png)

注意：循环时使用插值语法之间放item

## element使用计算属性

需要先j将参数传到里面的函数中return一个函数：如计算年龄：

![image-20211214155939852](C:\Users\123\AppData\Roaming\Typora\typora-user-images\image-20211214155939852.png)

![image-20211214160023885](C:\Users\123\AppData\Roaming\Typora\typora-user-images\image-20211214160023885.png)





## **rount-link**

### 1.  to

（必选参数）：类型string/location

表示目标路由的链接，该值可以是一个字符串，也可以是动态绑定的描述目标位置的对象

```vue
<router-link :to="'index'">Home</router-link>
```

### 2.tag

类型: string

默认值: "a"

如果想要 <router-link> 渲染成某种标签，例如 <li>。 于是我们使用 tag prop 类指定何种标签，同样它还是会监听点击，触发导航。

```vue
 <router-link
     :to="'index'"
     tag="li"
     event="mouseover">Home
 </router-link>
```

### 3、active-class

类型: string

默认值: "router-link-active"

设置 链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置。

```vue
<router-link :to="{path:'/about'}"
                   active-class="activeClass"                  
      >about</router-link>
```



## 计算属性，通过生日计算年龄

```vue
// 计算属性
        computed:{
        // 计算年龄
            getage(){
                return function(index){
                    const birthStr = index
                    let birthdays = new Date(birthStr.replace(/-/g, "/"));
                    let d = new Date();
                    let age = d.getFullYear() - 
                    birthdays.getFullYear() - 
                    (d.getMonth() < birthdays.getMonth() || (d.getMonth() == birthdays.getMonth() && d.getDate() < birthdays.getDate()) ? 1 : 0);
                    return age
                }
                
            }, 
        //计算工作年限
            yearsWork(){
                return function(index){
                    const birthStr = index
                    let birthdays = new Date(birthStr.replace(/-/g, "/"));
                    let d = new Date();
                    let age = d.getFullYear() - 
                    birthdays.getFullYear() - 
                    (d.getMonth() < birthdays.getMonth() || (d.getMonth() == birthdays.getMonth() && d.getDate() < birthdays.getDate()) ? 1 : 0);
                    return age
                }
                
            }, 
        }, 
```

## element UI 导出excel表单

1. 第一步、安装第三方插件

​	`cnpm install --save xlsx`

​    `cnpm install --save file-saver`

2. 第二步、导入

   `import FileSaver from "file-saver"`

   `import XLSX from "xlsx"`

3. 在按钮上添加事件

   `<button @click="exportTable">导出</button>`

4. 在`<el-table>`标签中添加`id="table-list"`，

5. 在mouths写导出代码，如下：

   ```vue
   methods:{
        	 exportTable() {
   	        /* 从表生成工作簿对象 */
   	        var wb = XLSX.utils.table_to_book(document.querySelector("#table-list"));
   	        /* 获取二进制字符串作为输出 */
   	        var wbout = XLSX.write(wb, {
   	            bookType: "xlsx",
   	            bookSST: true,
   	            type: "array"
   	        });
   	        try {
   	            FileSaver.saveAs(
   	            //Blob 对象表示一个不可变、原始数据的类文件对象。
   	            //Blob 表示的不一定是JavaScript原生格式的数据。
   	            //File 接口基于Blob，继承了 blob 的功能并将其扩展使其支持用户系统上的文件。
   	            //返回一个新创建的 Blob 对象，其内容由参数中给定的数组串联组成。
   	            new Blob([wbout], { type: "application/octet-stream" }),
   	            //设置导出文件名称
   	            "sheetjs.xlsx"
   	            );
   	        } catch (e) {
   	            if (typeof console !== "undefined") console.log(e, wbout);
   	        }
   	        return wbout;
   	        }
        	 }
   ```

   ## 加密本地/会话存储方法

### 		第一步：在项目中安装CryptoJS

```
cnpm install crypto-js
```

### 		第二步：在需要加密或解密的文件中引入crypto-js

```v
import CryptoJS from "crypto-js";
```

### 		第三步：加密代码

```js
sessionStorage.setItem('ruleForm',CryptoJS.AES.encrypt(JSON.stringify(response.data),'secretkey'))
    			   //储存的key值                        //JSON格式化对象   //储存的值     //加密的密钥
```

### 		第四步:解密代码

```js
JSON.parse(CryptoJS.AES.decrypt(sessionStorage.getItem('ruleForm'),"secretkey").toString(CryptoJS.enc.Utf8))
//解析JSON格式化的对象                				// 储存的key值    //加密的密钥      //解密操作
```

