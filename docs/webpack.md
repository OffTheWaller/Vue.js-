## webpack工具
### nrm和cnpm
- `npm i nrm -g`全局安装nrm,使用`nrm ls`查看所有的镜像地址,使用`nrm use 地址`切换不同的镜像地址
- 注意：nrm只是切换下载包的地址，安装包的时候依然使用`npm`命令
- 要使用`cnpm`指令来安装包，必须全局安装cnpm,`npm i cnpm -g`,这样在装包的时候可以使用`cnpm i 包名`
### 基本使用
- 第一步：在项目根目录下`npm init -y`初始化一个项目。如果项目根目录是中文，则不能使用`-y`，对包名项需要单独手动设置，且不要设置成中文
- 第二步：创建两个文件夹`src`--用来存放源码，`dist`--打包输出的文件夹
    - 在`src`下面创建`index.html`--项目首页
    - 在`src`下面创建`main.js`--js入口文件
- 第三步：安装webpack
    - 运行`npm i webpack -g`全局安装webpack，这样就能在全局使用webpack的命令
    - 在项目根目录中运行`npm i webpack --save-dev`安装到项目依赖中
        - -dev是开发依赖，在开发环境中需要用到，上线部署的项目不依赖这些包
- 第四步：运行`webpack 入口文件路径 输出文件路径`对`main.js`进行处理，输出为`bundle.js`，这时候可以在首页中引入`bundle.js`
### webpack的版本问题
- 如果是webpack 3.0+版本，则可以按以上步骤(这里使用的是`3.6.0`版本)
- 在webpack4.0+中，有了新的变化，需要安装`webpack-cli`和`mode`设置
- 以往的项目使用 webpack3 脚手架生成项目初始模板都会有两个甚至三个配置文件，比如 webpack.base.conf.js 、 webpack.prod.conf.js 、 webpack.dev.conf.js 而现在可以做到一个配置文件都不需要，直接在启动命令中传入参数 `--mode development | production` 达到区分不同模式的效果
- 修改package.json设置不同的模式
```
"scripts": {
     "dev": "webpack --mode development",
     "build": "webpack --mode production"
 },
```

### 使用webpack的配置文件简化打包时候的命令
1. 在`项目根目录`中创建`webpack.config.js`
2. 由于运行webpack命令的时候，webpack需要指定入口文件和输出文件的路径，所以，我们需要在`webpack.config.js`中配置这两个路径：
```
    // 导入处理路径的模块
    var path = require('path');

    // 导出一个配置对象，将来webpack在启动的时候，会默认来查找webpack.config.js，并读取这个文件中导出的配置对象，来进行打包处理
    module.exports = {
        entry: path.join(__dirname, 'src/js/main.js'), // 项目入口文件
        output: { // 配置输出选项
            path: path.join(__dirname, 'dist'), // 配置输出的路径
            filename: 'bundle.js' // 配置输出的文件名
        }
    }
```
3. 在配置完入口和出口时，要打包时只需要运行`webpack`就可以实现打包

### 实现webpack的实时打包构建
1. 由于每次重新修改代码之后，都需要手动运行webpack打包的命令，比较麻烦，所以使用`webpack-dev-server`来实现代码实时打包编译，当修改代码之后，会自动进行打包构建。
2. 运行`cnpm i webpack-dev-server --save-dev`或者`cnpm i webpack-dev-server -D`安装到开发依赖
3. 安装时提示缺少本地`webpack`，这是因为前面的webpack是全局安装的，可以用来执行webpack指令的。这时候需要的是在项目中安装一个webpack。在项目根目录下运行`cnpm i webpack -D`安装到开发依赖
4. 安装完成之后，在命令行直接运行`webpack-dev-server`来进行打包，发现报错，此时需要借助于`package.json`文件中的指令，来进行运行`webpack-dev-server`命令，在`package.json`文件中`scripts`节点下新增`"dev": "webpack-dev-server"`指令，发现可以进行实时打包，但是dist目录下并没有生成`bundle.js`文件，这是因为`webpack-dev-server`将打包好的文件放在了内存中
 ```
 "dev": "webpack-dev-server --open --port 3000 --contentBase src --hot"
 ```
- --open 自动打开浏览器
- --port 3000 修改端口号为3000，默认为8080
- --contentBase src 默认要打开的目录(如果用了html-webpack-plugin)可以不用设置
- --hot 启动热更新(`热更新`的作用是不用每次编译重新生成一个完整的bundle.js，只是把修改的代码作为补丁生成。而且在改样式的时候可以不用刷新浏览器就能更新样式,减少不必要的请求)
- webpack-dev-server托管的路径为'/'，所以这时就要修改index页面中script的src属性为`<scriptsrc="/bundle.js"></script>`，或者把script标签的引用注释掉，换为使用`html-webpack-plugin`插件
1. 运行`npm run dev`来实现`webpack-dev-server`的启动

### 使用`html-webpack-plugin`插件配置启动页面
由于使用`--contentBase`指令的过程比较繁琐，需要指定启动的目录，同时还需要修改index.html中script标签的src属性，所以推荐大家使用`html-webpack-plugin`插件配置启动页面.
1. 运行`cnpm i html-webpack-plugin -D`安装到开发依赖
2. 修改`webpack.config.js`配置文件如下：
```
    // 导入处理路径的模块
    var path = require('path');
    // 导入自动生成HTMl文件的插件
    var htmlWebpackPlugin = require('html-webpack-plugin');

    module.exports = {
        entry: path.join(__dirname, 'src/js/main.js'), // 项目入口文件
        output: { // 配置输出选项
            path: path.join(__dirname, 'dist'), // 配置输出的路径
            filename: 'bundle.js' // 配置输出的文件名
        },
        plugins:[ // 添加plugins节点配置插件
            new htmlWebpackPlugin({
                template:path.join(__dirname, 'src/index.html'),//模板路径
                filename:'index.html'//自动生成的HTML文件的名称
            })
        ]
    }
```
3. 将index.html中script标签注释掉，因为`html-webpack-plugin`插件会自动把bundle.js注入到index.html页面中！

### 使用webpack打包css文件
1. 运行`cnpm i style-loader css-loader --save-dev`
2. 修改`webpack.config.js`这个配置文件：
```
module: { // 用来配置第三方loader模块的
        rules: [ // 文件的匹配规则
            { test: /\.css$/, use: ['style-loader', 'css-loader'] }//处理css文件的规则
        ]
    }
```
3. 注意：`use`表示使用哪些模块来处理`test`所匹配到的文件；`use`中相关loader模块的调用顺序是从后向前调用的；

### 使用webpack打包less文件
1. 运行`cnpm i less-loader less -D`
2. 修改`webpack.config.js`这个配置文件：
```
{ test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader'] },
```

### 使用webpack打包sass文件
1. 运行`cnpm i sass-loader node-sass -D`
2. 在`webpack.config.js`中添加处理sass文件的loader模块：
```
{ test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader'] }
```
### 使用webpack处理css中的路径
1. 运行`cnpm i url-loader file-loader --save-dev`
2. 在`webpack.config.js`中添加处理url路径的loader模块：
```
{ test: /\.(png|jpg|gif)$/, use: 'url-loader' }
```
3. 可以通过`limit`指定进行base64编码的图片大小；只有小于指定字节（byte）的图片才会进行base64编码：
```
{ test: /\.(png|jpg|gif)$/, use: 'url-loader?limit=43960' },
```

### 使用babel处理高级JS语法
1. 运行`cnpm i babel-core babel-loader babel-plugin-transform-runtime --save-dev`安装babel的相关loader包
2. 运行`cnpm i babel-preset-es2015 babel-preset-stage-0 --save-dev`安装babel转换的语法
3. 在`webpack.config.js`中添加相关loader模块，其中需要注意的是，一定要把`node_modules`文件夹添加到排除项：
```
{ test: /\.js$/, use: 'babel-loader', exclude: /node_modules/ }
```
4. 在项目根目录中添加`.babelrc`文件，并修改这个配置文件如下：
```
{
    "presets":["es2015", "stage-0"],
    "plugins":["transform-runtime"]
}
```
5. **注意：语法插件`babel-preset-es2015`可以更新为`babel-preset-env`，它包含了所有的ES相关的语法；**
```
{
    "presets":["env", "stage-0"],
    "plugins":["transform-runtime"]
}
```
6. 在json格式的文件中，不能添加注释，不然会报错，因为json格式规定不允许有注释
7. 在提交的git仓库的时候，由于`node_modules`文件太大太多，需要配置一个`.gitignore`来忽略上传的文件，在该文件中写入`node_modules/`代表忽略node_modules下的文件不提交到仓库

### 普通网页中使用vue和使用webpack构建的项目中使用vue
#### 普通网页中
- 普通网页中使用vue直接在`script`标签中，引入vue.js就可以了，然后就可以使用插值表达式之类的东西
#### 使用webpack构建的项目中使用vue
- 如果只是`import Vue from 'vue'`，这时候在html中使用插值表达式是不行的，会报错，运行时环境，因为此时使用import导入的不是最全的vue包
- 回顾：包的查找规则
    - 在项目根目录中查找 node_modules 文件夹
    - 在 node_modules 中，根据包名，找对应的vue文件夹
    - 在对应的vue文件夹中，找一个叫做 package.json 的包配置文件
    - 在 package.json 文件中， 找一个 main 属性[ 该属性指定了这个包在加载的时候的入口文件]
- 所以vue功能不全的解决方法有
    - 第一种：直接修改vue中package.json中的main属性，使其指向完整的vue.js文件
    - 第二种：import Vue from '../node_modules/vue/dist/vue.js'
    - 第三种：在main.js中还是直接使用`import Vue from 'vue'`，在`webpack.config.js`中加一个节点
        ```javascript
        resolve: {
            alias: {
                'vue$': 'vue/dist/vue.js'
            }
        }
        ```

### .vue单文件

- .vue的单文件包括<templete><style><script>，其中<style>标签上要加scoped属性，加上scoped表示当前css只在该组件内有效，使用less处理要加<style lang="less">
- 使用.vue单文件时除了安装vue的loader，还要安装ES6的loader

### ES6补充

- 箭头函数：箭头函数中的this指向与普通函数是不一样的，箭头函数体内的this对象就是定义时所在的对象，而不是使用时所在的对象。例如：

```javascript
function Timer () {
    this.id = 1;

    var _this = this;
    setTimeout(function () {
        console.log(this.id); //undefined
        console.log(_this.id); //1
    }, 1000);

    setTimeout(() => {
        console.log(this.id); //1
    }, 5000);
}
Timer();
```



