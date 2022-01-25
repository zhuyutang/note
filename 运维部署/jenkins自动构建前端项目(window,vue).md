我们把一个多人协作的 vue 前端项目发布服务器，一般要经过以下步骤：

- git 更新最新的代码
- 构建项目
- 把构建后的代码上传到服务器

如果用 jenkins 来构建的话，只需要点击一次构建按钮，就可以自动完成以上的步骤了，而且根据需求,可以实现更多的功能。

### 1.下载安装 jenkin

#### 1.1 java 环境

jenkins 需要运行在 Java 的环境中，所以前提是需要先安装 jdk，测试 jdk 是否安装好，在命令行输入：`java -version`
![在这里插入图片描述](https://img-blog.csdnimg.cn/845d91815c0145f9960a60481fce88ff.png)

#### 1.2 下载 jenkins

下载地址 [jenkins 下载](https://www.jenkins.io/download/)
![请添加图片描述](https://img-blog.csdnimg.cn/7ad2650ca4d248d7bb0c21ab4f942eea.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

下载后点击安装，默认端口是 8080，然后打开 `http://localhost:8080/`，会出现如下页面：
![请添加图片描述](https://img-blog.csdnimg.cn/6e9111ae77f344d4a7ca954f68b89c09.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

根据提示，输入管理员密码，，然后安装推荐插件，等待安装完毕后，然后设置登录的用户名和密码，就可以进入 jenkins 的页面了：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c6466a6635c140c7b8c1658970e59f21.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 2 配置

#### 2.1 配置远程服务器环境

首先下载插件`publish over ssh:`

- 进入系统管理，然后点击插件管理
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/aee1c2f05e3144a5912411ab3128233f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
-     搜索插件，并安装
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/be44350833bf40bbb6a2de07d7ecc5a0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
- 进入系统配置，配置远程服务器
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/4dbfab8717e5407d898d2577229599fe.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
- 配置远程服务器
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/1576e9497d7f46d5ad5d8078d089c3cd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
- 点击右下角`test configuration`按钮，测试配置

#### 2.3 配置 node 环境

- 安装插件`nodejs plugin`
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/0fd5f70bcc3042f784b2b9d593a3e290.png)
- 配置添加 node 版本，点击`global tool configuration`
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/d50e8593b7254e15897ebb32373ca12c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
- 添加 node 版本
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/c29fc1e4deb443478b2842ac617b8c99.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 3 构建

一些必要的插件安装配置过后，就可以开始构建项目了

#### 3.1 新建项目

- 点击新建项目
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/b47ebb7f7e52493e80515b3daa79fc4a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
- 选择项目模板
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/81112e92453649f9b9fb00e0351c6f9d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 3.2 源码管理

- 滚动页面到`源码管理`配置项，选择`git`,此配置目的是自动从远程拉取代码，
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/35de940cecb049f38542469b908f61b4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
  其中 URL 就是项目的 git 的地址，credentials 是 git 登录凭据，还可以指定项目分支，
  其中 credentials 如果是第一次的话，需要点击`添加`完成配置后，才能选择：![在这里插入图片描述](https://img-blog.csdnimg.cn/3c22897016e5468689ee412fc3908925.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 3.3 构建环境配置

我们这里是 vue 项目，所以选择 node 环境，并选择`2.3配置node环境`所配置的 node 版本:
![在这里插入图片描述](https://img-blog.csdnimg.cn/9d5c3ed13c7b4f219294e4cd3311506b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 3.4 构建

这一步 jenkins 会依次执行我们所配置的脚本，

- 用脚本构建项目
  选择`execute windows batch command`,这里面就可以添加脚本，像在命令行一样
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/5599b90bb7c9455b92deec45c6b84e85.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
- 上传至服务器
  点击`增加构建步骤`，然后选择`send files or execute commands over ssh`
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/414550175c9c4a1585ed224cd920d201.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
  其中当前项目目录，如果没有更改过的话，默认在`C:\Windows\System32\config\systemprofile\AppData\Local\Jenkins\.jenkins\workspace\项目名称`

#### 3.5 验证自动构建

- 开始构建
  经过以上的配置，就已经配置好了一个 jenkins 项目了，返回到首页，就会有一个项目，点击构建按钮，就可以自动构建了
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/a28236c284754ca49a4a72ef54f4f58a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
- 构建历史
  点击项目名称，就可以进入到项目详情页，然后左下角会有构建历史，点击构建历史，就可以查看每次构建的控制台输出情况：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/b7ddc441807d41cba7e2a9d016befa0c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)
  控制台输出的信息：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/4bc97d33b24c4558b60dd358092f0d07.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

这样一个最基本的 vue 项目的持续构建就完成啦，不需要再手动构建项目，手动上传到服务器。
