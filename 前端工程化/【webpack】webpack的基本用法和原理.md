## 1 webpack 是什么

所有工具的出现，都是为了解决特定的问题，那么前端熟悉的 webpack 是为了解决什么问题呢？

#### 1.1 为什么会出现 webpack

**js 模块化：**
浏览器认识的语言是 HTML，CSS，Javascript，而其中 css 和 javascript 是通过 html 的标签`link`,`script`引入进来。

随着前端项目的越来越复杂，css 和 js 文件会越来越庞大，那么在开发阶段，就必须要把 css 和 js 按功能拆分成几个小文件，方便开发。

那么拆分的小文件如何引入到 html 中呢？css 可以通过`link`标签或者`@import`css 语法，但是 js 因为没有模块导入的语法（ES6 有了 import，但还不是所有浏览器兼容），就只能通过`script`标签引入。但是这样的话会导致很多问题：

1. http 请求大量增多，影响页面呈现速度。
2. 全局变量混乱，难以维护。

针对 js 模块化出现了很多的解决方案，总结来说有几种规范：

1. CommonJs，语法为：require(), module.exports（同步加载，适用于 node 服务器环境）
2. ES6 Mode,语法为：import,export（异步加载，适用于浏览器环境)
3. AMD,语法为：require(),define()（异步加载，适用于浏览器环境）

**工程化：**
除了 js 模块化的问题之外，前端还有很多其他的问题，比如代码混淆，代码压缩，scss，less 等 css 预编译语言的编译，typescript 的编译，eslint 检验代码规范，如果这些任务都需要手工去执行的话，太繁琐，也容易出错。

#### 1.2 webpack 能做什么

1.  模块化
    其实 webpack 的核心就是解决 js 模块化问题的工具，运行在 node 环境中，同时可以支持 commonjs，es6，amd 的模块语法（可以使用：reuire/mocule.exports,import/export,require/define 的方式来导入导出模块）。

        可以将开发时候拆分为不同文件的js代码，打包成一个js文件。也可以通过配置灵活的拆分js代码，通过 tree shaking 删减没有使用到的代码。

        模块化打包时webpack的核心功能，但是它还有两个非常重要的机制loader和plugin。

2.  loader
    webpack 本身只支持 js,json 文件的模块化打包，但是有开放出 loader 接口，通过不同的 loader 可以将其他格式的文件转化为可识别的模块，比如：
    css-loader 可以识别 css 文件，raw-loader 可以直接将文件当作模块，less-loader，sass-loader 可以直接识别 less，sass 文件。
3.  plugin
    插件机制是 webpack 的另一个重要的拓展，webpack 在打包的过程中，会暴露出不同的生命周期事件，而插件会监听这些事件，然后做出对应的操作，比如：
    UglifyJsPlugin 可以混淆压缩代码，EslintWebpackPlugin 可以执行 eslint 的代码格式检测和自动修复。

总结：
webpack 是一个运行在 node 环境下，对 js 文件进行模块化打包的工具。通过 loader 机制可以实现除 js 格式外的其他格式文件，通过 plugin 机制可以实现自动执行一些工程化需要的任务。

## 2 webpack 怎么用

那么 webpack 要怎么使用呢？

#### 2.1 安装运行

首先要安装 webapck，使用 npm（[npm 基本用法及原理](https://blog.csdn.net/u010013405/article/details/116588718)），安装 webpack（核心），webpack-cli（命令行工具）:

```cmd
npm install webpack webpack-cli
```

然后创建以下两个文件：name.js，index.js，
我们以 es6 的语法导入导出模块，es6 模式的模块变量的导出时按引用导出，就是在模块的变量如果在外部被修改，也会作用到模块内部，而 commonjs 的模式是按值导出，即模块外部的修改，不会影响到模块内部。

```javascript
//name.js
let name = '小明';
function say() {
  console.log('my name is ', name);
}
export { name, say };

//index.js
import { name, say } from './name.js';

name = '小红';
say();
console.log('he name is ', name);
```

然后运行打包命令：

```javascript
//用npx直接运行webpack命令
npx webpack

//或者用npm的脚本运行打包
//package.json
{
	script:{
		pack:'webpack'
	}
}
npm run pack
```

webpack 默认从 index.js 文件开始打包，所以如果开始文件的名称为 index，就可以不需要写配置文件，就可以直接打包。
默认打包的模式是‘production'即生产模式，打包成功后，会自动创建 dist 文件夹，并生成 main.js 文件：

```javascript
//main.js
(() => {
  'use strict';
  let e = '小明';
  (e = '小红'),
    console.log('my name is ', '小红'),
    console.log('he name is ', '小红');
})();
```

我们看到打包后的文件把 index.js 和 name.js 两个文件合成了一个文件，并对代码进行了混淆压缩（生产模式），这是最基本的 webpack 最核心的功能 6— 打包。
但是显然，在实际工作中我们不会这么简单的使用，那就需要用的配置文件了，下面是一个比较接近实际工作中的例子。

#### 2.2 配置文件 webpack-config.js

以下的项目会有几个文件：index.js , utils.js, style.scss, index.html, webpack-config.js。
基本功能就是在 index.js 文件中引入 utils.js 文件里的方法并调用。然后用 scss 语法编写样式，最后把打包的文件加入到已有的 index.html 文件中。
通过命令：`npx webpack serve(可以放入npm脚本配置中，然后运行 npm run xxx)`，实现的效果是：

- scss 自动编译
- index.js utils.js style.scss 文件打包成一个文件
- 把打包的文件自动添加到 index.html 中
- 打包完成后，自动打开默认浏览器，查看页面
- 文件有变更的话，会自动重新打包，刷新页面

```javascript
//utils.js
export function sayHello() {
  console.log('hello world');
}
```

```javascript
//index.js
import './styles.scss';
import { sayHello } from './utils';
sayHello();
```

```css
/*styles.scss*/
$bg: black;
$fontC: rgb(218, 17, 117);
body {
  background: $bg;
  h3 {
    color: $fontC;
  }
}
```

```html
<!--index.html-->

<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>webpack</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>
  <body>
    <h3>hello webpack</h3>
  </body>
</html>
```

```javascript
//webpack-config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  // 打包模式
  mode: 'development', //development,production,none
  devtool: 'cheap-source-map', // eval-source-map,source-map,cheap-source-map
  // 入口配置
  entry: {
    app: './src/index.js',
  },
  // 出口配置
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist'),
    clean: true, //每次打包，清除dist文件夹
  },

  // 本地服务器
  devServer: {
    port: 8888, //端口
    open: true, //自动打开浏览器
    hot: true, //启动热更新
  },

  // loader
  module: {
    // 处理scss文件
    rules: [
      {
        test: /\.s[ac]ss$/i,
        use: [
          'style-loader', //将js模块生成style标签节点
          'css-loader', //将css转化成js模块
          'sass-loader', //将scss文件编译成css文件
        ],
      },
    ],
  },

  // 插件
  plugins: [
    // 自动把打包后的文件加入到html文件
    new HtmlWebpackPlugin({
      // 生成html文件的模板
      template: './index.html',
    }),
  ],
};
```

webpack 的打包是在 node 环境下执行的，所以 node 的语法这里都可以用，最终输出的是一个 js 对象。
配置可以分成几个部分[（参考 webpack 配置文档）](https://webpack.docschina.org/configuration/)：

- 打包模式

  - **mode** 预置了开发环境和生产环境的一些优化。
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/0419dc2333bf471a9f3780e95b55c9b8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeXV0YW5nbWVuZw==,size_20,color_FFFFFF,t_70,g_se,x_16)

  - **devtool** 控制是否生成，以及如何生成 source map。有了 source map 文件的话，如果代码有报错可以映射到打包之前的代码（源代码），可以方便定位错误。
    可以有很多的选择，一般来说，在开发环境下选择：eval-cheap-module-source-map，cheap-source-map。生产环境选择：不配置，source-map。

- 入口、出口
  - **entry** 入口文件，webpack 会从这个文件开始查找依赖的包，可以配置多个
  - **output** 出口文件，webpack 会根据这里的配置，输出打包后的文件。
- 本地服务器
  此功能需要安装 webpack-dev-server 插件`npm install --save-dev webpack-dev-server`,启动时需要用 serve 命令`npx webpack serve`.
  启动成功后，会在本地开启一个 web 服务器，并且有实时更新，热模块替换等功能。
  配置项是在`devServer`中。
- loader
  webpack 本是只支持对 js 文件的打包，但是因为有 loader 机制，可以通过配置`rules`实现对其他文件的打包。
  示例代码中实现的是对 scss/sass 文件的打包，同一个 relues 中的 loader 的执行顺序是从右到左（逆序），所以顺序不能乱。第一个执行的 loader 会将其结果（被转换后的资源）传递给下一个 要执行的 loader。
- 插件 plugins
  webpack 在打包的时候，会暴露其生命周期，插件就是在特定的生命周期执行的操作，通过插件的机制，可以实现很多强大的功能。
  示例代码中使用了`HtmlWebpackPlugin`插件，功能是在打包完成后，自动把打包后的代码加入到 html 文件中。如果没有任何配置，则会自动生成一个 html 文件，并通过\<script\>标签把 js 文件引入进来。

## 3 实践中的优化

#### 3.1 配置文件拆分与合并--merge

在实际项目中，开发环境和生产环境的配置往往会有很大的区别，所以会有两个配置文件，而这两个配置文件又会有一些公共的配置，所以就会有如下三个配置文件：

- webpack.dev.js
- webpack.prod.js
- webpack.common.js
  那么这些配置文件是如何结合的呢?这就要用到 webpack-merge 插件了。

```javascript
// webpack.common.js

const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  // 入口配置
  entry: {
    app: './src/index.js',
  },
  // 出口配置
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist'),
    clean: true, //每次打包，清除dist文件夹
  },
  // 插件
  plugins: [
    // 自动把打包后的文件加入到html文件
    new HtmlWebpackPlugin({
      // 生成html文件的模板
      template: './index.html',
    }),
  ],
};
```

```javascript
//webpack.dev.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'eval-source-map',
  // 本地服务器
  devServer: {
    port: 8888, //端口
    open: true, //自动打开浏览器
    hot: true, //启动热更新
  },
});
```

```javascript
//webpack.prod.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'production',
  devtool: 'source-map',
});
```

```javascript
// package.json
{
	...
  "scripts": {
    "dev": "webpack serve --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js",
  },
	...
}
```

然后分别执行`npm run dev`,`npm run build`就可以了。

#### 3.2 代码分离

webpack 会把所有代码打包成一个文件（包括业务代码，npm 包），这样最后的包就会很大，打包效率也很慢，所以可以有时候需要做代码分离。

- **第三方库分离**
  有一些第三方库可能会需要独立引入，而不是放在业务代码里面，因为不会改动或者需要 cdn 服务，比如 jquery 有免费的 cdn 服务：https://upcdn.b0.upaiyun.com/libs/jquery/jquery-2.0.2.min.js 。
  那这些独立引入的 js 文件就不需要加入到 webpack 打包，只需要在`externals`添加配置就行。

      ```javascript
      // webpack.config.js
      const path = require('path');
      const HtmlWebpackPlugin = require('html-webpack-plugin');

      module.exports = {
        // 打包模式
        mode:'development', //development,production,none
        devtool:'cheap-source-map', // eval-source-map,source-map,cheap-source-map
        // 入口配置
        entry: {
          app: './src/index.js',
        },
        // 出口配置
        output: {
          filename: '[name].[contenthash].js',//contenthash 是文件内容的hash值
          path: path.resolve(__dirname, 'dist'),
          clean: true,//每次打包，清除dist文件夹
        },

        // 插件
        plugins: [
          // 自动把打包后的文件加入到html文件
          new HtmlWebpackPlugin({
            // 生成html文件的模板
            template: './index.html'
          }),
        ],
      };
      ```
      这样的话，虽然index.js里有引入jquery，webpack也不会把jquery打包进来，打包时间会减少，包的体积也会变小。

- **npm 包分离**
  第三方库除了一些可以用 script 标签引入的，大多数是通过 npm 引入的，这一类的 js 包也会也会合并到最后的 app.js 总包之中，使得 app.js 文件会过大，而且如果业务代码有一点改动的话，app.js 的包就会全部都变动，导致浏览器就会重新下载 app.js 文件，使用不了浏览器内置的缓存机制。
  我们可以通过配置，让 npm 里的包与业务代码分开。
  `javascript // index.js import _ from 'lodash' import $ from 'jquery' import { sayHello } from "./utils" $('#title').text('hello jquery') `
  ```javascript
  // webpack.config.js
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  module.exports = {
  // 打包模式
  mode:'development', //development,production,none
  devtool:'cheap-source-map', // eval-source-map,source-map,cheap-source-map
  // 入口配置
  entry: {
  app: './src/index.js',
  },
  // 出口配置
  output: {
  filename: '[name].[contenthash].js', //contenthash 是文件内容的 hash 值
  path: path.resolve(\_\_dirname, 'dist'),
  clean: true,//每次打包，清除 dist 文件夹
  },
  // 插件
  plugins: [
  // 自动把打包后的文件加入到 html 文件
  new HtmlWebpackPlugin({
  // 生成 html 文件的模板
  template: './index.html'
  }),
  ],
  optimization: {
  runtimeChunk: 'single',// 把 webpack 引导文件独立出来
  splitChunks: {
  cacheGroups: {
  vendor: {
  //所有 node_modules 下的包合并成一个，并独立出来
  test: /[\\/]node_modules[\\/]/,
  //控制哪种导入方式的 js 包才分离出来， 'all'-全部的 js 包,'async'-异步导入的 js 包,'initial'-初始导入的 js 包
  chunks: 'all',
  name:'vendor' //独立后的包的名称
  }
  }
  }
  },
  };

      ```
      这样打包目录下就有三个文件：
      - app.js： 业务代码
      - runtime.js：webpack的引导代码
      - vendor.js: npm引入的js包代码

      一般有变动的就只有app.js文件了。npm引入的js包是否可以再分成几个文件呢？可以的，[参看 SplitChunksPlugin文档](https://webpack.docschina.org/plugins/split-chunks-plugin/)

- **业务代码分离**
  有时候不仅第三方库需要分离，我们自己写的业务代码可能也会很大，也需要分离。要实现业务代码的分离只要添加多个入口就可以了。

      ```javascript
      // index.js
      import { sayHello } from "./utils"

      sayHello()
      console.log('hello index')
      ```
      ```javascript
      // utils.js
      export function sayHello(){
          console.log('hello utils')
      }
      ```
      ```javascript
      // webpack.config.js
      const path = require('path');
      const HtmlWebpackPlugin = require('html-webpack-plugin');

      module.exports = {
       mode: 'development',

  devtool: 'cheap-source-map',
  // 入口配置
  entry: {
  app: {
  import:'./src/index.js',
  dependOn:'utils'
  },
  utils:'./src/utils.js'
  },
  // 出口配置
  output: {
  filename: '[name].[contenthash].js',
  path: path.resolve(\_\_dirname, 'dist'),
  clean: true,//每次打包，清楚 dist 文件夹
  },
  // 插件
  plugins: [
  // 自动把打包后的文件加入到 html 文件
  new HtmlWebpackPlugin({
  // 生成 html 文件的模板
  template: './index.html'
  }),
  ],
  };
  ``` 这样utils.js文件也从主包app.js中分离了出来，要注意的是app入口加了`dependOn:'utils'`,为了让 app.js 里面不要重复打包 utils.js。

- **动态加载**
  有时候不是需要页面一开始的时候，就加载全部的 js 包，而是等到特定的时机再去加载某些 js 包，这就需要动态加载了，只需要用到`import()`就可以了，注意这是 import 的函数使用方式。
  `javascript // index.js const element = document.createElement('div'); element.id = 'title' element.innerHTML ='Hello webpack'; const button = document.createElement('button'); button.innerHTML = 'Click me'; button.onclick = importJquery; document.body.appendChild( element); document.body.appendChild( button); async function importJquery(){ const { default: $ } = await import('jquery'); $('#title').text('hello jquery') } `
  `javascript // webpack.config.js const path = require('path'); const HtmlWebpackPlugin = require('html-webpack-plugin'); module.exports = { mode: 'development', devtool: 'cheap-source-map', // 入口配置 entry: { app: { import:'./src/index.js', }, }, // 出口配置 output: { filename: '[name].[contenthash].js', path: path.resolve(__dirname, 'dist'), clean: true,//每次打包，清楚dist文件夹 }, // 插件 plugins: [ // 自动把打包后的文件加入到html文件 new HtmlWebpackPlugin({ // 生成html文件的模板 template: './index.html' }), ], }; `
  如果运行代码的话，会发现页面一开始进入的时候，并没有引入 jquery 的包，但是点击按钮的时候，就开始导入了，这就动态加载。
  并且发现 webpack.config.js 并没有做什么特殊的配置，这是因为动态导入的 js 包 webpack 会自动给独立为一个 js 文件。
  import()返回的是一个 promise 对象。

#### 3.3 动态链接库 dll

webpack 每次打包的时候，都会把涉及到的 js 包都处理一遍。但是实际上有些 js 包是不会有改动到的，所以打包过后的文件每次都是一样的，每次都重新打包的话，会加增打包时间。
有一种解决方案是：把不会变动的 js 包先打包一次，以后每次打包的时候，直接引用就可以了。
先添加一个独立的配置文件 `webpack.dll.config.js`

```javascript
//webpack.dll.config.js

const path = require('path');
const webpack = require('webpack');

module.exports = {
  mode: 'production',
  // 入口文件
  entry: {
    // 项目中用到该两个依赖库文件
    jquery_lodash: ['jquery', 'lodash'],
  },
  // 输出文件
  output: {
    // 文件名称
    filename: '[name].dll.js',
    // 将输出的文件放到dll目录下
    path: path.resolve(__dirname, 'dll'),

    // 文件输出的全局变量
    library: '_dll_[name]',
  },
  plugins: [
    // 使用插件 DllPlugin
    new webpack.DllPlugin({
      // 动态链接库的全局变量名称，需要和 output.library 中保持一致
      // 该字段的值也就是输出的 manifest.json 文件 中 name 字段的值
      name: '_dll_[name]',
      // 描述动态链接库的 manifest.json 文件输出时的文件名称
      path: path.join(__dirname, 'dll', '[name].manifest.json'),
    }),
  ],
};
```

执行打包脚本 `npx webpack --config webpack.dll.config.js`,就会再 dll 目录下输出文件：`jquery_lodash.dll.js`,`jquery_lodash.manifest.json`。
主要用到的插件是：

- `DllPlugin` 的作用就是生成 manifest.json 文件。

然后配置项目打包用的 webpack.config.js 文件：

```javascript
//webpack.config.js

const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack');
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');

module.exports = {
  mode: 'development',
  devtool: 'cheap-source-map',
  // 入口配置
  entry: {
    app: {
      import: './src/index.js',
    },
  },
  // 出口配置
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist'),
    clean: true, //每次打包，清楚dist文件夹
  },

  // 插件
  plugins: [
    // 自动把打包后的文件加入到html文件
    new HtmlWebpackPlugin({
      // 生成html文件的模板
      template: './index.html',
    }),
    // 引用dll中的文件,
    new webpack.DllReferencePlugin({
      context: __dirname,
      manifest: require('./dll/jquery_lodash_dll.manifest.json'),
    }),
    //把dll文件加入到index.html中
    new AddAssetHtmlWebpackPlugin({
      filepath: path.resolve(__dirname, './dll/jquery_lodash_dll.dll.js'),
      publicPath: './',
    }),
  ],
};
```

主要用到的插件是:

- `DllReferencePlugin` 检索引用文件的时候，如果发现 manifest.json 里面有，就告诉 webpack 不要打包该文件，因为已经打包好了。
- `AddAssetHtmlWebpackPlugin` 因为 webpack 没有打包 dll 里的文件，所以需要手动把它加入到 index.html 中。

然后就可以正常的打包项目了：

```javascript
// index.js
import _ from 'lodash';
import $ from 'jquery';

$('#title').text('hello jquery');
```

执行打包脚本 `npx webpack`，会自动使用 webpack.config.js 配置文件打包。会发现打包速度提高了很多。
