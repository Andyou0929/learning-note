## 一、搭建脚手架并快速创建Vue项目

​        现在的Vue脚手架已经升级到3.x版本，即vue-cli3。脚手架升级之后，安装的命令发生了变化，所以这篇文章会跟大家演示新旧版本的脚手架安装过程，以及使用新旧版本脚手架创建项目的过程。
下面的安装过程均是在window平台下安装。

### 1.准备工作

#### 1.1安装node.js和npm

​       Vue的脚手架是依赖于node.js的，所以无论是安装新版本还是旧版本，我们都要安装node.js。

我们可以直接到[node.js官网](https://nodejs.org/zh-cn/)下载，然后像安装普通软件一样安装node.js。

**npm**(node package manager)是node的包管理工具，我们在后面主要是使用npm来搭建脚手架和安装一些常用的组件。

​    node.js成功安装之后，npm一并安装成功，这个时候我们可以打开cmd窗口，输入 **`node -v`** 和 **`npm -v`** 来查看node.js和npm的版本，如果能够显示出版本，说明已经安装成功。

##### 2.安装淘宝镜像

​        为什么要安装淘宝镜像呢？因为我们使用npm来搭建脚手架的时候，是从国外的npm服务器上下载需要的文件，这就导致下载过程会很漫长。我们安装了淘宝镜像之后，就可以从国内的镜像服务器下载搭建脚手架所需的文件，可以很快的完成下载任务。

我们在cmd窗口中输入以下命令来安装淘宝镜像。安装完成之后，我们可以使用命令cnpm -v来查看其版本，如果能够显示版本说明安装成功。

​      npm install -g cnpm --registry=https://registry.npm.taobao.org

完成之后，我们就可以用`cnpm`命令代替`npm`命令来安装依赖包了。

## 2、安装新版本脚手架（即vue-cli3.x）

​       在安装新版本的脚手架之前，如果我们安装过旧版本的脚手架，那么我们需要使用`npm uninstall vue-cli -g`命令删除旧版本的脚手架。

​       安装新版本的Vue脚手架，最好保证node.js的版本在8.1.1.0以上，如果你是最近从node官网下载的node，那么无需关心这个问题，node版本会在8.1.1.0以上。

 准备妥当之后，我们可以使用可以使用下列任一命令安装这个新的包：

```bash
npm install -g @vue/cli
cnpm install -g @vue/cli
# OR
yarn global add @vue/cli
```

安装完成之后，我们可以vue --version或vue -V命令来查看我们安装的版本。 现在我们已经成功安装了新版本的Vue脚手架，现在我们就可以使用脚手架来快速创建我们的项目。

## 3、使用脚手架快速创建项目

​       首先我们在电脑的合适位置新建一个文件夹，然后使用`cd`命令来到这个文件夹。比如我在自己的桌面下新建一个vue文件夹。

  以管理员身份运行：Windows PowerShell 输入   

```
set-ExecutionPolicy RemoteSigned
```

  在创建的文件夹右键，点击 在Windows终端打开；

按照下边方法进行：

(1)  新建vue项目

```kotlin
vue create vuetest
```

进入新建项目

打开vscode，打开创建的文件夹，

右击创建的vue项目

选择在集成终端打开

输入：

```bash
npm run serve
```

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zODc5NjAzLTgwZTJlMjRiNjVkMTM5ZmQucG5n?x-oss-process=image/format,png)

四、备注

​          在上面的教程中，我们首先安装的是Vue脚手架（即vue-cli）。在我看来，Vue脚手架就是一个工具，是用来帮助我们快速搭建Vue项目的工具。r
