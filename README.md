## 目录
* [webpack核心概念](#webpack核心概念)  
  * [entry](#entry)
  * [output](#output)
  * [loaders](#loaders)
  * [plugins](#plugins)
  * [mode](#mode)
  * [其他](#其他)
* [webpack打包思想简括](#webpack打包思想简括)
* [webpack优化策略](#webpack优化策略)
  * [热更新技术](#热更新技术)
  * [sourcemap](#sourcemap)
  * [分包](#分包)
  * [tree-shaking](#tree-shaking)
  * [构建速度分析](#构建速度分析)
  * [构建体积分析](#构建体积分析)
* [webpack打包原理](#webpack打包原理)

***
### webpack核心概念

#### entry
- entry用来指定webpack的入口;
- 依赖图的入口是entry，对于非代码，比如图片、字体依赖也会不断加入到依赖图;
- entry用法：

  1. 单入口entry是一个字符串：
  ```javascript
  module.exports = {
    entry: './path/to/my/entry/file.js'
  };
  ```
  
  2. 多入口entry是一个对象：
  ```javascript
  module.exports = {
    entry: {
      app: './src/app.js',
      adminApp: './src/adminApp.js'
    }
  };
  ```
#### output
- output用来告诉webpack如何将编译后的文件输出到磁盘;
- output的用法：

  1. 单入口配置
  ```
  module.exports = {
    entry: './path/to/my/entry/file.js',
    output: {
      filename: 'bundle.js',
      path: _dirname + '/dist'
    }
  };
  ```

  2. 多入口配置：
  ```
  module.exports = {
    entry: {
      app: './src/app.js',
      search: './src/search.js'
    },
    output: {
      filename: '[name].js',
      path: __dirname + '/dist'
    }
  };
  ```
#### loaders
- webpack开箱即用只支持js和json两种文件类型，通过loaders去支持其他文件类型，并把它们转化成有效的模块，并且可以添加到依赖图中;
- loaders本身是一个函数，接受源文件作为参数，返回转换的结果;
- 常见的loaders有哪些：
  
  名称 | 描述
  ------ | ------
  Transpiling
  babel-loader | 装换ES6、ES7等JS新语法特性
  ts-loader | 将TS装换为JS
  Styling
  css-loader | 支持.css文件的加载和解析
  less-loader | 将less文件转换为css
  sass-loader | 将sass/scss文件转换为css
  postcss-loader | 使用PostCSS加载和转换css文件
  stylus-loader | 将stylus文件转换为css
  Files
  file-loader | 运行图片、字体等的打包
  url-loader | 类似于file-loader，但是可以返回一个[Data URL](https://tools.ietf.org/html/rfc2397)
  raw-loader | 将文件以字符串形式导入
  thread-loader | 多进程打包JS和CSS
  
- loaders的用法：
  1. test - 指定匹配规则;
  2. use - 指定使用的loader名称;
  
  ```javascript
  module.exports = {
    module: {
      rules: [
        {
          test: /\.css$/,
          use: [
            { loader: 'style-loader' },
            {
              loader: 'css-loader',
              options: {
                modules: true
              }
            },
            { loader: 'sass-loader' }
          ]
        }
      ]
    }
  };
  ```
#### plugins
- 插件用于bundle文件的优化，资源管理和环境变量注入;
- 作用于整个构建过程;
- 常见的plugins有哪些：

  名称 | 描述
  ------ | ------
  CommonsChunkPlugin | 将chunks相同的模块代码提取为公共js
  CleanWebpackPlugin | 清理构建目录
  CopyWebpackPlugin | 将文件或者文件夹拷贝到构建的输出目录
  DefinePlugin | 定义环境变量
  HotModuleReplacementPlugin | 热替换（开发环境使用）
  HtmlWebpackPlugin | 创建Html去承载输出的bundle
  MiniCssExtractPlugin | 将JS包含的CSS提取成独立的CSS文件（生产环境使用）
  OptimizeCSSAssetsWebpack Plugin | 压缩CSS
  TerserWebpackPlugin | 压缩js
  ZipWebpackPlugin | 将打包的资源生成一个zip包
  
- plugins的用法：

  ```javascript
  module.exports = {
    entry: './path/to/my/entry/file.js',
    output: {
      filename: 'bundle.js'
    },
    plugins: [
      new HtmlWebpackPlugin({template: './src/index.html'})
    ]
  };
  ```
#### mode
- mode用来指定当前的构建环境是：production、development还是none;
- 设置mode可以使用webpack内置的函数，默认值为production;
- mode内置函数概念：

  选项 | 功能
  ------ | ------
  development | 会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 development。启用 NamedChunksPlugin 和 NamedModulesPlugin。
  production | 会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 production。启用 FlagDependencyUsagePlugin, FlagIncludedChunksPlugin, ModuleConcatenationPlugin, NoEmitOnErrorsPlugin, OccurrenceOrderPlugin, SideEffectsFlagPlugin 和 TerserPlugin。
  none | 退出任何默认优化选项
#### 其他
__webpack常见术语__

- module：指在模块化编程中我们把应用程序分割成的独立功能的代码模块。
- chunk：指模块间按照引用关系组合成的代码块，一个 chunk 中可以包含多个 module。
- chunk group：指通过配置入口点（entry point）区分的块组，一个 chunk group 中可包含一到多个 chunk 。
- asset/bundle：打包产物。  

__文件指纹__  

什么是文件指纹？
- 打包后输出文件名的后缀;
- 通常用于版本管理;
- hash一般是结合CDN缓存来使用，通过webpack构建之后，生成对应文件名自动带上对应的MD5值。如果文件内容改变的话，那么对应文件哈希值也会改变，对应的HTML引用的URL地址也会改变，触发CDN服务器从源服务器上拉取对应数据，进而更新本地缓存;

文件指纹如何生成？

- Hash：和整个项目的构建相关，每次项目构建对应的 hash 值就会更改；
- Chunkhash：根据不同的入口文件(Entry)进行依赖文件解析、构建对应的chunk，生成对应的哈希值；
- Contenthash：根据文件内容来定义 hash ，文件内容不变，则 contenthash 不变；

JS的文件指纹设置
- 设置 output 的 filename，使用 [chunkhash]，或[contenthash]；
 ```javascript
 module.exports = {
     entry: {
         app: './src/app.js',
         search: './src/search.js'
     },
     output: {
         filename: '[name][chunkhash:8].js',
         path: __dirname + '/dist'
     }
 };
 ```
CSS 的文件指纹设置
- 设置 MiniCssExtractPlugin 的 filename，使用 [contenthash]；
 ```javascript
 module.exports = {
     entry: {
         app: './src/app.js',
         search: './src/search.js'
     },
     output: {
         filename: '[name][chunkhash:8].js',
         path: __dirname + '/dist'
     },
     plugins: [
         new MiniCssExtractPlugin({
             filename: `[name][contenthash:8].css`
         }),
     ]
 };
 ```
 图片，字体的文件指纹设置
 - 设置 file-loader（或url-loader） 的 name，使用 [hash]
 ```javascript
 module.exports = {
   entry: './src/index.js',
   output: {
       filename: 'bundle.js',
       path: path.resolve(__dirname, 'dist')
   },
   module: {
       rules: [
           {
               test: /\.(png|svg|jpg|gif)$/,
               use: [{
                   loader: 'file-loader',
                   options: {
                       name: 'img/[name][hash:8].[ext] '
                   }
               }]
           }
       ]
   }
 };
 ```
***
### webpack打包思想简括
> 1. 一切源代码文件均可通过各种 Loader 转换为 JS 模块 （module），模块之间可以互相引用。
> 2. webpack 通过入口点（entry point）递归处理各模块引用关系，最后输出为一个或多个产物包 js(bundle) 文件。
> 3. 每一个入口点都是一个块组（chunk group），在不考虑分包的情况下，一个 chunk group 中只有一个 chunk，该 chunk 包含递归分析后的所有模块。每一个 chunk 都有对应的一个打包后的输出文件（asset/bundle）。
***
### webpack优化策略
#### 热更新技术
#### sourcemap
#### 分包
#### tree-shaking
#### 资源文件压缩
#### 构建速度分析
#### 构建体积分析
***
### webpack打包原理
