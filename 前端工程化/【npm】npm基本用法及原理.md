&emsp; &emsp;作为前端开发者，应该每个人都用过 npm，那么 npm 到底是什么东西呢？npm run,npm install 的时候发生了哪些事情呢？下面做详细说明。

### 1.npm 是什么

npm 是 JavaScript 语言的包管理工具，它由三个部分组成:

- npm 网站 [进入](https://www.npmjs.com/)
  npm 官网上可以查找包，查看包信息。
- 注册表
  一个巨大的数据库，存放包的信息
- 命令行工具 npm-cli
  开发者运行 npm 命令的工具

这三者中，与我们打交道最多的就是 npm-cli，其实我们所说的 npm 的使用，就是指这个工具的使用，那它到底是个什么东西呢？我们先来看看它被放在哪里，在系统命令行（window cmd）工具中输入 `where npm`（安装 node 会自带 npm），就能找到它的位置：
![顶顶顶顶](https://img-blog.csdnimg.cn/202105191707089.png)
然后根据路径找到 npm 文件打开：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210519170940571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAwMTM0MDU=,size_16,color_FFFFFF,t_70)
从标红的地方可以看出，这其实就是一个脚本，它最终执行的是: `node npm-cli.js`

&emsp; &emsp;所以到目前为止，我们可以知道当在命令行输入`npm`时，其实是在 node 环境中，执行了一段 npm-cli.js 代码，这是对 npm 的一个直观的认识。
&emsp; &emsp;至于 npm-cli.js 里面的逻辑是什么，就是研究源码层面的事了，这里不涉及。我们主要来看 npm 的用法和功能层面的原理。首先来看 npm 的配置文件 package.json。

### 2.package.json 文件

当我们运行命令`npm init`,根据提示输入一些信息后（`npm init -y`不需输入信息），会在当前目录下生成一个 package.json 文件:

```json
{
  "name": "testNpm",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

这里就是一个 npm 包的基本信息，包括包名 name，版本 version，描述 description，作者 author，主文件 main，脚本 scripts 等等， 这里先主要来看下`main`：

#### 2.1 入口文件 main

&emsp;&emsp; `main`配置项的值是一个 js 文件的路径，它将作为程序的主入口文件。也就是说当别人引用了这个包时`import testNpm from 'testNpm'`，其实引入的就是`testNpm/index.js`文件所 export 出的模块。

#### 2.2 脚本 scripts

npm scripts 脚本应该是我们打交道最多的一个配置项了，它一个 json 的对象，由脚本名称和脚本内容组成：

```json
"scripts":{
	"star":"echo star npm",
	"echo":"echo hello npm"
}
```

一般用`npm run xxx`来运行，但是一些关键命令比如：start,test,stop,restart 等等，可以直接`npm xxx`来执行。那 scripts 是如何执行脚本的呢？又可以执行哪些脚本呢？

**npm 脚本可以执行的命令**
其实当我们`npm run xxx`的时候，就是把 xxx 的内容生成了一个 shell 脚本，然后执行脚本，那么 npm 的 shell 具体是什么呢？我们可以运行`npm config get -l`来查看 npm 的全部配置：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210527162840893.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAwMTM0MDU=,size_16,color_FFFFFF,t_70)
可能个人的系统和配置不同，以我个人电脑配置为例，其实就是`cmd.exe`，其实就是 window 系统的 cmd 命令行工具。所以在 cmd 中可以执行的命令，在 npm 的 scripts 中都可以执行，举例说明：

```json
"scripts":{
	/*系统命令*/
	"echo":"echo hello npm",
	"dir":"dir",
	"ip":"ipconfig"
}
```

像 dir，ipconfig,echo 这些都是可以直接在 cmd 命令行中执行的命令，在 npm 的 scripts 中都可以通过`npm run xxx`来执行。这一类是系统 cmd 的内部命令，不需要安装额外的插件，就可以直接执行。
还有一种就是我们在 cmd 还可以执行外部命令，比如我们如果安装了 node,git 等客户端，可以直接在 cmd 窗口执行（需配置了系统的环境变量）：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210527165212647.png)
这一类的命令 npm 也可以执行：

```json
"scripts":{
	/*系统命令*/
    "echo":"echo hello npm",
    "dir":"dird",
    "ip":"ipconfig",
    /*全局外部命令*/
    "git":"git --version",
    "node":"node -v",
}
```

这是全局引入的外部命令，还有些项目内部才有的命令，比如我们在项目下安装 eslint: `npm install eslint --save-dev`，在 scripts 中配置了脚本的话，我们可以直接运行`npm run eslint`

```json
"scripts":{
	/*系统命令*/
    "echo":"echo hello npm",
    "dir":"dird",
    "ip":"ipconfig",
    /*全局外部命令*/
    "git":"git --version",
    "node":"node -v",
    /*项目内外部命令*/
    "eslint":"eslint -v"
}
```

但是如果我们直接在 cmd 窗口执行`eslint -v`,则会报错，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210528123008721.png)
这是因为系统找不到 eslint 的位置（没有配系统环境变量），但是既然 cmd 室 npm 脚本执行的环境，为什么`npm run eslint`可以执行呢？
这是因为当我们通过`npm run xxx`执行脚本的时候，会把当前目录的'node_modules/.bin'加入到环境变量，也就是说 npm 执行脚本的时候，会自动到`node_modules/.bin`目录下找，如果找到则可以正常执行，我们来看一下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210528124705380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAwMTM0MDU=,size_16,color_FFFFFF,t_70)
在 node_modules/.bin 目录下果然是 eslint.cmd 脚本的，而它作的其实就是`node eslint.js`,用 node 来执行 eslint.js 的代码。

**npm 脚本可以执行的命令总结：**

- cmd 内部命令,例如 dir,ipconfig...
- 外部命令
  - 全局命令，加入了系统环境变量
  - 项目下命令，这部分会放在 node_modules/.bin 目录下，而 npm 会自动链接到此目录。

#### 2.3 npm 脚本其他配置

**路径通配符**
我们在写脚本命令的时候，常常要匹配文件，这就要用到路径的通配符。
总的来说`*`表示任意字符串，在目录中表示 1 级目录，`**`表示 0 级或多级目录，例如：
`src/*`：src 目录下的任意文件，匹配 src/a.js; src/b.json；不匹配 src/aa/a.js
`src/*.js`：src 目录下任何 js 文件，匹配 src/a.js; 不匹配 src/b.json；src/aa/a.js
`src/*/*.js`：src 目录下一级的任意 js 文件，匹配 src/aa/a.js; 不匹配 src/a.js；src/a/aa/a.js
`src/**/*.js`：src 目录下的任意 js 文件，匹配 src/a.js; src/a/a.js; src/a/aa/a.js

**命令参数**
关于 npm 的参数，我们先来看一段代码：
node 代码：

```javascript
//index.js

console.log(process.env.npm_package_name);
console.log(process.env.npm_config_env);
console.log(process.argv);
```

npm 配置：

```json
//package.json

{
  "name": "npm",
  "version": "1.0.0",
  "scripts": {
    "node": "node index.js --name=node age=28"
  }
}
```

然后我们执行命令`npm run node --env=npmEnv`，结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210601183911789.png)

下面来做下说明，其实 npm 的参数都是指 node 环境下的参数，用 node 的全局变量 process 来获取。

- npm 内部变量
  当我们在执行 npm 命令的时候，就会把 package.json 的参数加上 npm*package*前缀，加入到 process.env 的变量中，所以在上面的 node 代码可以通过`process.env.npm_package_name`获取到 package.json 里面配置的 name 属性。
- 命令参数
  当我们在运行 npm 命令时，带上以双横线为后缀的参数：`npm 命令 --xx=xx`，npm 就会把 xx 加上 npm*config*前缀，加入到 process.env 变量中，如果原来有同名的，命令参数的优先级最高，会覆盖掉原来的，所以在上面的 node 代码可以通过`process.env.npm_config_env`获取到`npm run node --env=npmEnv`命令里的参数 env 的值，如果参数没有赋值：`npm run node --env`，则默认值为`true`
- 脚本参数
  这个其实要根据脚本的内容来看，比如我们上面的脚本是`node index.js --env=node`,这其实是纯粹的 node 命令了，可以通过[process.argv](http://nodejs.cn/api/process.html#process_process_argv)来获取 node 的命令参数，这是个数组，第一个为 node 命令路径，第二个为执行文件路径，后面的值为用空格隔开的其他参数，如上面打印的结果所示。

**执行顺序**
npm 脚本的执行顺序分为两部分:

- 命令钩子
  npm 脚本有`pre`,`post`两类钩子，一个是执行前，一个是执行后。比如，当我们执行`npm run start`时，会按照以下顺序执行`npm run prestart` ->`npm run start` ->`npm run poststart`
- 多任务并行
  如果要执行多个脚本，可以用`&`或`&&`来连接 - `npm run aa & npm run bb` 并行执行，没有先后关系 - `npm run aa && npm run bb` 串行执行，先执行完 aa 再执行 bb

### 3.npm 包管理

npm 做完包管理工具，主要的作用还是包的安装及管理。

#### 3.1 安装包 npm install xxx

`npm install xxx` 命令用于安装包。
我们先来运行`npm install vue`和`npm install eslint --save-dev`,会发现项目会有以下变化：

- 添加了目录 node_modules
  安装的包和包的依赖都存放在这里，引入的时候，会自动到此目录下找。
- package.json 文件自动添加了如下配置：
  ```json
    "dependencies": {
      "vue": "^2.6.13"
    },
    "devDependencies": {
      "eslint": "^7.27.0"
    }
  ```
  npm 在安装包的同时，会把包的名称和版本加入到`dependencies`配置中，这表明这是项目必需的包。
  如果带上参数`--save-dev`，则加入到`devDependencies`配置中，这表明这是项目开发时才需要的工具包，不是项目必需的。
- 添加了 package-lock.json 文件
  锁定包的版本和依赖结构。

#### 3.2 从 package.json 配置文件安装包

**包依赖类型**
现在把 node_modules 目录和 package-lock.json 文件都删除，然后运行`npm install`，会发现项目会自动安装 vue 和 eslint 包。
如果我们执行`npm install --production`则表明我们只是想安装项目必须的包，用于生产环境，这是就只会安装`dependencies`对象下的包。
其实 npm 包除了这两种还有其他包的依赖类型：

- `dependencies `
  业务依赖，是项目的必须包，是项目线上代码的一部分。`npm install --production`只会安装此配置下的包。
- `devDependencies`
  开发环境依赖，只在开发环境需要。`npm install --save-dev`安装包并添加到此配置下。
- `peerDependencies`
  同行依赖，当运行`npm install`，会提示安装此配置下的包。注意只是警告提示，不会自动安装。
- `optionalDependencies`
  可选依赖，表明即使安装失败，也不影响项目的安装过程。会覆盖掉`dependencies`中的同名包。
- `bundledDependencies`
  打包依赖，发布当前包的时候，会把此配置下的依赖包也一起打包。必须先在 `dependencies` 和 `devDependencies` 声明过，否则打包会报错。

**包版本说明**
npm 采用[semver](https://segmentfault.com/a/1190000014405355)作为包版本管理规范。此规范规定软件版本由三个部分组成：

- `主版本号`做了不兼容的重大变更
- `次版本号`做了向下兼容的功能添加
- `补丁版本号`做了向下兼容的 bug 修复

除了版本号之外，还有一些版本修饰，后面可以带上数字：

- `alpha`内测版 eg：3.0.0-alpha.1
- `beta`公测版 eg：3.0.0-beta.10
- `rc`正式版本的候选版 eg：3.0.0-rc.3

**版本匹配**

- `*`/`x`:匹配任意值
  `1.1.* ` = `>=1.1.0 <1.2.0`
  `1.x ` = `>=1.0.0 <2.0.0`
- `^xxx`: 最左侧非 0 版本号不变，不小于 xxx
  `^1.2.3` = `>=1.2.3 <2.0.0` 主版本号不变
  `^0.1.2` = `>=0.1.2 <0.2.0` 主、次版本号不变
  `^0.0.2` = `= 0.0.2` 主、次、补丁版本号都不变
- `~xxx`:如果列出了次版本号，则次版本号不变，如果没有列出次版本号，则主版本号不变，均不小于 xxx
  `~1.2.3` = `>=1.2.3 <1.3.0` 主、次版本号不变
  `~1` = `>=1.0.0 <2.0.0` 主版本号不变

#### 3.3 package-lock.json 作用

**固定版本**
当我们安装包的时候，会自动添加`package-lock.json`文件，那么这个文件的作用是什么呢？在这个问题之前，先来看看`npm install`的安装原理：

```json
//package.json
{
  "name": "npm",
  "version": "1.0.0",
  "dependencies": {
    "vue": "^2.5.1"
  },
  "devDependencies": {
    "eslint": "^7.0.0"
  }
}
```

有上面一份 npm 配置文件，当`npm install`时会安装两个包：`vue ^2.5.1`,`eslint ^7.0.0` ，符合所配置版本的包是一个范围多个，npm 会会安装符合版本配置的最新版本。比如：
`vue ^2.5.1` = `>=2.5.1 <3.0.0`, npm 会选择安装`2.6.13`，因为它在匹配版本范围内，且是目前最新的 vue2 的版本，它不会选择`2.5.0`和`3.0.0`。
那么如果只有一份 package.json 文件，就很可能导致项目依赖的版本不一样。比如开发时候 vue2 的最新版本是 2.6.13，过了几个月项目要上线，部署的时候 vue2 的最新版本已经是 2.7.0 了，那么线上就会安装最新的版本。如果 2.7.0 有一些不兼容 2.6.13 的地方，或者有 bug，那就会导致我们开发的一个经典问题：开发环境没问题，一上线就坏。如果项目是多个人协同开发，甚至会导致开发环境都不一样。
那么我们来看看 package-lock.json 文件怎么解决这个问题的：

```json
//package-lock.json
{
  "name": "npm",
  "version": "1.0.0",
  "lockfileVersion": 1,
  "requires": true,
  "dependencies": {
    "vue": {
      "version": "2.6.13",
      "resolved": "https://registry.nlark.com/vue/download/vue-2.6.13.tgz?cache=0&sync_timestamp=1622664849693&other_urls=https%3A%2F%2Fregistry.nlark.com%2Fvue%2Fdownload%2Fvue-2.6.13.tgz",
      "integrity": "sha1-lLLBsx/d8d/MNPKOyEi6jwHqTFs="
    },
	.....
  }
}
```

我们看到 package-lock.json 文件里直接记录了 vue 的固定版本号和下载地址。

npm 在执行 install 的时候，会把每个需要安装的包先在`package-lock.json`里查找，如果找到并且版本符合`package.json`的配置范围（在范围内就行，不需要最新），就会直接按照`package-lock.json`里的地址安装。如果没找到或者不符合范围，则安装原本的逻辑安装（符合版本要求的最新版）。
这样就确保，不管时间过了多久，只要 package-lock.json 文件不变，`npm install`安装的包的版本都是一致的，避免代码运行的依赖环境不同。

**固定依赖结构**
我们的一个项目通常会有很多依赖包，而这些依赖包很可能又会依赖其他的包，那如何来避免重复安装呢？
比如：

```json
//package.json
{
  "name": "npm",
  "version": "1.0.0",
  "dependencies": {
    "esquery": "^1.4.0",
    "esrecurse": "^4.3.0",
    "eslint-scope": "^5.1.1"
  }
}
```

依赖关系如下：

- esquery ： ^1.4.0,
  - `estraverse ： ^5.1.0`
- `esrecurse ： ^4.3.0`
  - `estraverse ： ^5.2.0`
- eslint-scope ：^5.1.1
  - `esrecurse ： ^4.3.0`
    - `estraverse ：^5.2.0`
  - `estraverse ：^4.1.1`

如果按照这个嵌套结构来安装包的话也是可以的，而且 npm 原来的版本就是这么做的，这样可以保证每个包都安装完整，但是问题是会导致一些包重复安装，如果这个依赖很多的话，重复的数量也会很多。那 npm 是怎么处理的呢？
npm 采用的是用扁平结构，包的依赖，不管是直接依赖，还是子依赖的依赖，都会优先放在第一级。
如果第一级有找到符合版本的包，就不重复安装，如果没找到，则在当前目录下安装。
比如上面的包会被安装成如下的结构：

- esquery ：1.4.0,
  - `estraverse ： 5.2.0`
- esrecurse ： 4.3.0
  - `estraverse ： 5.2.0`
- eslint-scope ： 5.1.1
- estraverse ： 4.3.1

包安装的数量从开始的 8 个减少到了 6 个，虽然还是有重复，但是因为这个 json 的结构，又是以包名为键名，所以同一级下只能有一个同名的包，就像 `estraverse ： 5.2.0`不能放在外层，因为外层已经有了以`estraverse `为名的对象：`estraverse ： 4.3.1`。
`package-lock.json`记录的就是上面的依赖结构（上面只是简写，每一项还包含一些其他的信息，比如下载地址），这也是`node_modules`里面包的结构。
所以一个项目只要`package-lock.json`不变，它的依赖结构就不变，而且 npm 不用重新解析包的结构了，直接从`package-lock.json`文件就可以安装完整且正确的包依赖，也提高了重新安装的效率。

#### 3.4 包缓存

npm 安装包不是每一次都从服务器直接下载，而是有缓存机制。当 npm 安装包时，会在本地的缓存一份。执行`npm config get cache`可以查看缓存目录：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607181511406.png)
按照路径打开文件夹，会发现`_cacache`缓存文件夹，打开文件夹会有`index-v5`和`content-v2`两个目录。
其中`index-v5`存放的是包的索引，而`content-v2`则存放的是缓存的压缩包。

**缓存查找**
那么 npm 是如何找到缓存包的呢？以 vue 包为例：

- 1.首先安装 vue 包： `npm install vue `
- 2.查看 package-lock.json 文件,根据包信息获取`resolved`,`integrity`字段，构造字符串：
  `pacote:range-manifest:{resolved}:{integrity}`
- 3.把上面字符串按`SHA256`加密,得到加密字符串：
  `2686ae12fd03809c9e5704cd01db518f1d7d07efe5ab61e6ef386e95b8481360`
- 4.上面加密字符串的前 4 位就是`_cacache/index-v5`目录的下两级，索引文件的位置：
  `_cacache/index-v5/26/86/ae12fd03809c9e5704cd01db518f1d7d07efe5ab61e6ef386e95b8481360`
- 5.打开按照上面路径找到的索引文件，在索引文件中找到`_shasum`字段：
  `94b2c1b31fddf1dfcc34f28ec848ba8f01ea4c5b`
- 6.上面符串就是缓存包的位置，其前 4 位就是`_cacache/content-v2/sha1`目录的下两级，包位置：
  `_cacache/content-v2/sha1/94/b2/c1b31fddf1dfcc34f28ec848ba8f01ea4c5b`
- 7.把按照上面路径找到的文件的拓展名改为`.tgz`，然后解压，会得到`vue.tar`包，再解压，就是我们熟悉的 vue 包了。

#### 3.5 npm install 原理流程图

把 npm install 原理总结为下面的流程图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021060811394321.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAwMTM0MDU=,size_16,color_FFFFFF,t_70)

### 4.npm 常用命令

- `npm init [-y] ` 创建 package.json 文件 [直接创建]
- `npm run xxx [--env]` 运行脚本 [参数]
- `npm config get [-l] ` 查看 npm 配置 [全部配置]
- `npm install xxx [--save-dev] [-g] ` 安装 npm 包 [添加到开发依赖] [全局安装]
- `npm uninstall xxx [-g]` 删除包 [删除全局包]
- `npm info xxx ` 查看包信息
- `npm view xxx version` 查看包最新版本
- `npm update [-g] xxx` 更新包 [全局包]
- `npm root [-g]` npm 包安装的目录 [全局包安装目录]
- `npm ls [-g] ` 查看项目安装的包 [全局安装的包]
- `npm install [--production] ` 安装项目 [只安装项目依赖]
- `npm ci ` 安装项目，不对比 package.json，只从 package-lock.json 安装，并且会先删除 node_modules 目录
- `npm config get cache ` 查看缓存目录
- `npm cache clean --force ` 清除 npm 包缓存

---

**参考**

- [前端工程化 - 剖析 npm 的包管理机制](https://mp.weixin.qq.com/s?__biz=MzI3ODU4MzQ1MA==&mid=2247484959&idx=2&sn=2622440031c578524d8a74de00a0d772&scene=19#wechat_redirect)
- [什么是 npm —— 写给初学者的编程教程](https://mp.weixin.qq.com/s/Scq7iz4oG35d8NTRzDGXTA)
- [npm 缓存浅析](https://www.yuque.com/ericlee/fontend/yhhgp5)
