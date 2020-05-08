# vue cli
## 脚手架的意义
- 是开发现代web应用的必备
- 充分利用webpack，babel，eslint等工具辅助项目开发
- 开箱即用，零配置，无需手动配置繁琐的工具即可使用，专注在撰写应用上，而不必花好几天去纠结配置的问题。
- 提供了配套的图形管理界面，用于创建，开发和管理你的项目

## 版本记录
  - vue cli 4.0系列 正式版发布时间，2019-10-16
  - vue cli 3.0系列 正式版发布时间，2018-08-10
  - 之前是2.0版本

## 单文件组件（.vue）
- 优点
  - 完整语法高亮
  - CommonJS 模块
  - 组件作用域的 CSS
- template
  - 就是配置组件里的template模板
- script
  - 导出的就是组件对象
- style
  - <style lang="less" scoped>
  - lang，指定语言，less,sass...
  - scoped，使css只作用于当前组件

## 系统组件
CLI
- 一个全局安装的 npm 包，提供了终端里的 vue 命令。它可以通过 vue create 快速创建一个新项目的脚手架，或者直接通过 vue serve 构建新想法的原型。也可以通过 vue ui 通过一套图形化界面管理所有项目。

CLI 服务
- CLI 服务 (@vue/cli-service) 是一个开发环境依赖。它是一个 npm 包，局部安装在每个 @vue/cli 创建的项目中。
- CLI 服务构建于 webpack 和 webpack-dev-server 之上。包含了：
  - 加载其它 CLI 插件的核心服务；
  - 一个针对绝大部分应用优化过的内部的 webpack 配置；
  - 项目内部的 vue-cli-service 命令，提供 serve、build 和 inspect 命令。

CLI 插件
- CLI 插件是向Vue项目提供可选功能的 npm 包，例如 Babel/TypeScript 转译、ESLint 集成、单元测试和 end-to-end 测试等。Vue CLI 插件的名字以 @vue/cli-plugin- (内建插件) 或 vue-cli-plugin- (社区插件) 开头，非常容易使用。当在项目内部运行 vue-cli-service 命令时，它会自动解析并加载 package.json 中列出的所有 CLI 插件。
- 插件可以作为项目创建过程的一部分，或在后期加入到项目中。它们也可以被归成一组可复用的 preset。

## 基础配置
### 安装
- npm i -g @vue/cli
- Mac全局安装需要：sudo...
- 检查是否安装正确
  - vue --version

### 快速原型开发
可以使用 vue serve 和 vue build 命令对单个 *.vue 文件进行快速原型开发，不过这需要先额外安装一个全局的扩展：
```js
npm install -g @vue/cli-service-global
```
vue serve 的缺点就是它需要安装全局依赖，这使得它在不同机器上的一致性不能得到保证。因此这只适用于快速原型开发。

### 创建一个项目
```
vue create 项目名
```
一般在命令行中配置项目preset,可以选择默认的，也可以手动配置，可以用vue ui在可视化窗口中完成babel，router，eslint，vuex等的配置。
- 被保存的 preset 将会存在用户的 home 目录下一个名为 .vuerc 的 JSON 文件里。如果想要修改被保存的 preset / 选项，可以编辑这个文件。

- `vue create --help` 可以查看常用指令

切换到项目根目录 
```js
cd <yourproject>
```

启动项目
```js
yarn serve
# or
npm run serve
```

webpack配置，在vue.config.js
- 调整端口，设置自动打开浏览器

eslint
- 配合vs code插件
- 保存时自动修复

### 插件和preset
#### 插件
Vue CLI 使用了一套基于插件的架构。如果你查阅一个新创建项目的 package.json，就会发现依赖都是以 @vue/cli-plugin- 开头的。插件可以修改 webpack 的内部配置，也可以向 vue-cli-service 注入命令。在项目创建的过程中，绝大部分列出的特性都是通过插件来实现的。

##### 在现有的项目中安装插件
每个 CLI 插件都会包含一个 (用来创建文件的) 生成器和一个 (用来调整 webpack 核心配置和注入命令的) 运行时插件。
如果你想在一个已经被创建好的项目中安装一个插件，可以使用 vue add 命令，比如：
```
vue add eslint
```
这个命令将 @vue/eslint 解析为完整的包名 @vue/cli-plugin-eslint，然后从 npm 安装它，调用它的生成器。等价于：
```
vue add cli-plugin-eslint
```

- 提示
vue add 的设计意图是为了安装和调用 Vue CLI 插件。这不意味着替换掉普通的 npm 包。对于这些普通的 npm 包，你仍然需要选用包管理器。
- 提示
如果出于一些原因你的插件列在了该项目之外的其它 package.json 文件里，你可以在自己项目的 package.json 里设置 vuePlugins.resolveFrom 选项指向包含其它 package.json 的文件夹。
- 警告
推荐在运行 vue add 之前将项目的最新状态提交，因为该命令可能调用插件的文件生成器并很有可能更改你现有的文件。

##### 项目本地的插件
如果需要在项目里直接访问插件 API 而不需要创建一个完整的插件，可以在 package.json 文件中使用 vuePlugins.service 选项：
```
{
  "vuePlugins": {
    "service": ["my-commands.js"]
  }
}
```
每个文件都需要暴露一个函数，接受插件 API 作为第一个参数。
也可以通过 vuePlugins.ui 选项添加像 UI 插件一样工作的文件：
```
{
  "vuePlugins": {
    "ui": ["my-ui.js"]
  }
}
```
#### Preset
一个 Vue CLI preset 是一个包含创建新项目所需预定义选项和插件的 JSON 对象，让用户无需在命令提示中选择它们。在 vue create 过程中保存的 preset 会被放在你的 home 目录下的一个配置文件中 (~/.vuerc)。你可以通过直接编辑这个文件来调整、添加、删除保存好的 preset。
示例：
```
{
  "useConfigFiles": true,
  "cssPreprocessor": "sass",
  "plugins": {
    "@vue/cli-plugin-babel": {},
    "@vue/cli-plugin-eslint": {
      "config": "airbnb",
      "lintOn": ["save", "commit"]
    },
    "@vue/cli-plugin-router": {},
    "@vue/cli-plugin-vuex": {}
  }
}
```
Preset 的数据会被插件生成器用来生成相应的项目文件。也可以为集成工具添加配置：
```
{
  "useConfigFiles": true,
  "plugins": {...},
  "configs": {
    "vue": {...},
    "postcss": {...},
    "eslintConfig": {...},
    "jest": {...}
  }
}
```
这些额外的配置将会根据 useConfigFiles 的值被合并到 package.json 或相应的配置文件中。例如，当 "useConfigFiles": true 的时候，configs 的值将会被合并到 vue.config.js 中。

##### Preset 插件的版本管理
可以显式地指定用到的插件的版本：
```
{
  "plugins": {
    "@vue/cli-plugin-eslint": {
      "version": "^3.0.0",
      // ... 该插件的其它选项
    }
  }
}
```
对于官方插件来说这不是必须的——当被忽略时，CLI 会自动使用 registry 中最新的版本。推荐为 preset 列出的所有第三方插件提供显式的版本范围。
##### 允许插件的命令提示
Vue CLI 假设所有的插件选项都已经在 preset 中声明过了，有些情况下你可能希望 preset 只声明需要的插件，同时让用户通过插件注入的命令提示来保留一些灵活性。对于这种场景你可以在插件选项中指定 "prompts": true 来允许注入命令提示：
```
{
  "plugins": {
    "@vue/cli-plugin-eslint": {
      // 让用户选取他们自己的 ESLint config
      "prompts": true
    }
  }
}
```
##### 远程 Preset
可以通过发布 git repo 将一个 preset 分享给其他开发者。发布 repo 后，你就可以在创建项目的时候通过 --preset 选项使用这个远程的 preset 了：
```
# 从 GitHub repo 使用 preset
vue create --preset username/repo my-project
```
如果要从私有 repo 获取，请确保使用 --clone 选项
##### 加载文件系统中的 Preset
当开发一个远程 preset 的时候，你必须向远程 repo 发出 push 进行反复测试。为了简化流程，你也可以直接在本地测试 preset。如果 --preset 选项的值是一个相对或绝对文件路径，或是以 .json 结尾，则 Vue CLI 会加载本地的 preset:
```
# ./my-preset 应当是一个包含 preset.json 的文件夹
vue create --preset ./my-preset my-project

# 或者，直接使用当前工作目录下的 json 文件：
vue create --preset my-preset.json my-project
```
### CLI服务
#### 使用命令
在一个 Vue CLI 项目中，@vue/cli-service 安装了一个名为 vue-cli-service 的命令。
可以通过 npm 或 yarn 调用这些 script：
```
npm run serve
# or
yarn serve
```
也可以直接使用npx
#### vue-cli-service serve
此命令会启动一个开发服务器 (基于 webpack-dev-server) 并附带开箱即用的模块热重载 (Hot-Module-Replacement)。
```
用法：vue-cli-service serve [options] [entry]
选项：
  --open    在服务器启动时打开浏览器
  --copy    在服务器启动时将 URL 复制到剪切版
  --mode    指定环境模式 (默认值：development)
  --host    指定 host (默认值：0.0.0.0)
  --port    指定 port (默认值：8080)
  --https   使用 https (默认值：false)
```
除了通过命令行参数，你也可以使用 vue.config.js 里的 devServer 字段配置开发服务器。命令行参数 [entry] 将被指定为唯一入口，而非额外的追加入口。尝试使用 [entry] 覆盖 config.pages 中的 entry 将可能引发错误。
#### vue-cli-service build
此命令会在 dist/ 目录产生一个可用于生产环境的包，带有 JS/CSS/HTML 的压缩，和为更好的缓存而做的自动的 vendor chunk splitting。它的 chunk manifest 会内联在 HTML 里。
```
用法：vue-cli-service build [options] [entry|pattern]
选项：
  --mode        指定环境模式 (默认值：production)
  --dest        指定输出目录 (默认值：dist)
  --modern      面向现代浏览器带自动回退地构建应用
  --target      app | lib | wc | wc-async (默认值：app)
  --name        库或 Web Components 模式下的名字 (默认值：package.json 中的 "name" 字段或入口文件名)
  --no-clean    在构建项目之前不清除目标目录
  --report      生成 report.html 以帮助分析包内容
  --report-json 生成 report.json 以帮助分析包内容
  --watch       监听文件变化
```
#### vue-cli-service inspect
审查一个 Vue CLI 项目的 webpack config
```
用法：vue-cli-service inspect [options] [...paths]
选项：
  --mode    指定环境模式 (默认值：development)
```
#### 配置时无需 Eject
通过 Vue CLI 创建的项目让你无需 eject 就能够配置工具的几乎每个角落。
#### 此外还有缓存和并行处理，git hook等详见官网

## 开发详细配置
### 浏览器兼容性
#### browserslist
你会发现有 package.json 文件里的 browserslist 字段 (或一个单独的 .browserslistrc 文件)，指定了项目的目标浏览器的范围。这个值会被 @babel/preset-env 和 Autoprefixer 用来确定需要转译的 JavaScript 特性和需要添加的 CSS 浏览器前缀。
#### Polyfill
##### useBuiltIns: 'usage' 
一个默认的 Vue CLI 项目会使用 @vue/babel-preset-app，它通过 @babel/preset-env 和 browserslist 配置来决定项目需要的 polyfill。
默认情况下，它会把 useBuiltIns: 'usage' 传递给 @babel/preset-env，这样它会根据源代码中出现的语言特性自动检测需要的 polyfill。这确保了最终包里 polyfill 数量的最小化。
#### 现代模式
转译后的包通常都比原生的 ES2015+ 代码会更冗长，运行更慢。现如今绝大多数现代浏览器都已经支持了原生的 ES2015,Vue CLI 提供了一个“现代模式”帮你解决这个问题。以如下命令为生产环境构建：

```
vue-cli-service build --modern
```

Vue CLI 会产生两个应用的版本：一个现代版的包，面向支持 ES modules 的现代浏览器，另一个旧版的包，面向不支持的旧浏览器。
在生产环境下，现代版的包通常都会表现出显著的解析速度和运算速度，从而改善应用的加载性能。

- 提示
```
<script type="module"> 需要配合始终开启的 CORS 进行加载。这意味着你的服务器必须返回诸如 Access-Control-Allow-Origin: * 的有效的 CORS 头。如果你想要通过认证来获取脚本，可使将 crossorigin 选项设置为 use-credentials。
```

### HTML 和静态资源
#### HTML
##### Index 文件
public/index.html 文件是一个会被 html-webpack-plugin 处理的模板。在构建过程中，资源链接会被自动注入。另外，Vue CLI 也会自动注入 resource hint (preload/prefetch、manifest 和图标链接 (当用到 PWA 插件时) 以及构建过程中处理的 JavaScript 和 CSS 文件的资源链接。
##### Preload
<link rel="preload"> 是一种 resource hint，用来指定页面加载后很快会被用到的资源，所以在页面加载的过程中，我们希望在浏览器开始主体渲染之前尽早 preload。
默认情况下，一个 Vue CLI 应用会为所有初始化渲染需要的文件自动生成 preload 提示。

##### Prefetch
<link rel="prefetch"> 是一种 resource hint，用来告诉浏览器在页面加载完成后，利用空闲时间提前获取用户未来可能会访问的内容。
默认情况下，一个 Vue CLI 应用会为所有作为 async chunk 生成的 JavaScript 文件 (通过动态 import() 按需 code splitting 的产物) 自动生成 prefetch 提示。

##### 不生成 index
当基于已有的后端使用 Vue CLI 时，你可能不需要生成 index.html，这样生成的资源可以用于一个服务端渲染的页面。

##### 构建一个多页应用
不是每个应用都需要是一个单页应用。Vue CLI 支持使用 vue.config.js 中的 pages 选项构建一个多页面的应用。

##### 处理静态资源
静态资源可以通过两种方式进行处理：
- 在 JavaScript 被导入或在 template/CSS 中通过相对路径被引用。这类引用会被 webpack 处理。
  - 当你在 JavaScript、CSS 或 *.vue 文件中使用相对路径 (必须以 . 开头) 引用一个静态资源时，该资源将会被包含进入 webpack 的依赖图中。在其编译过程中，所有诸如 <img src="...">、background: url(...) 和 CSS @import 的资源 URL 都会被解析为一个模块依赖。
  - 在其内部，我们通过 file-loader 用版本哈希值和正确的公共基础路径来决定最终的文件路径，再用 url-loader 将小于 4kb 的资源内联，以减少 HTTP 请求的数量。
  - 可以通过 chainWebpack 调整内联文件的大小限制。例如，下列代码会将其限制设置为 10kb：
  ```
  // vue.config.js
  module.exports = {
  chainWebpack: config => {
    config.module
      .rule('images')
        .use('url-loader')
          .loader('url-loader')
          .tap(options => Object.assign(options, { limit: 10240 }))
    }
  }
  ```
- 放置在 public 目录下或通过绝对路径被引用。这类资源将会直接被拷贝，而不会经过 webpack 的处理。

推荐将资源作为你的模块依赖图的一部分导入，这样它们会通过 webpack 的处理并获得如下好处：
- 脚本和样式表会被压缩且打包在一起，从而避免额外的网络请求。
- 文件丢失会直接在编译时报错，而不是到了用户端才产生 404 错误。
- 最终生成的文件名包含了内容哈希，因此你不必担心浏览器会缓存它们的老版本。

何时使用 public 文件夹
- 你需要在构建输出中指定一个文件的名字。
- 你有上千个图片，需要动态引用它们的路径。
- 有些库可能和 webpack 不兼容，这时你除了将其用一个独立的 <script> 标签引入没有别的选择。

### CSS 相关
所有编译后的 CSS 都会通过 css-loader 来解析其中的 url() 引用，并将这些引用作为模块请求来处理。这意味着你可以根据本地的文件结构用相对路径来引用静态资源。另外要注意的是如果你想要引用一个 npm 依赖中的文件，或是想要用 webpack alias，则需要在路径前加上 ~ 的前缀来避免歧义。
#### 预处理器
可以在创建项目的时候选择预处理器 (Sass/Less/Stylus)。如果当时没有选好，内置的 webpack 仍然会被预配置为可以完成所有的处理。你也可以手动安装相应的 webpack loader：
```
# Sass
npm install -D sass-loader node-sass
# Less
npm install -D less-loader less
# Stylus
npm install -D stylus-loader stylus
```
##### 自动化导入
如果你想自动化导入文件 (用于颜色、变量、mixin……)，你可以使用 style-resources-loader

#### PostCSS
Vue CLI 内部使用了 PostCSS。
可以通过 .postcssrc 或任何 postcss-load-config 支持的配置源来配置 PostCSS。也可以通过 vue.config.js 中的 css.loaderOptions.postcss 配置 postcss-loader。
默认开启了 autoprefixer。如果要配置目标浏览器，可使用 package.json 的 browserslist 字段。
- 在生产环境构建中，Vue CLI 会优化 CSS 并基于目标浏览器抛弃不必要的浏览器前缀规则。因为默认开启了 autoprefixer，你只使用无前缀的 CSS 规则即可。

#### CSS Modules
可以通过 <style module> 以开箱即用的方式在 *.vue 文件中使用 CSS Modules。
如果想在 JavaScript 中作为 CSS Modules 导入 CSS 或其它预处理文件，该文件应该以 .module.(css|less|sass|scss|styl) 结尾：
```
import styles from './foo.module.css'
// 所有支持的预处理器都一样工作
import sassStyles from './foo.module.scss'
```
如果你想去掉文件名中的 .module，可以设置 vue.config.js 中的 css.requireModuleExtension 为 false：
```
// vue.config.js
module.exports = {
  css: {
    requireModuleExtension: false
  }
}
```
#### 向预处理器 Loader 传递选项
有的时候你想要向 webpack 的预处理器 loader 传递选项。你可以使用 vue.config.js 中的 css.loaderOptions 选项。
### webpack 相关
#### 简单的配置方式
调整 webpack 配置最简单的方式就是在 vue.config.js 中的 configureWebpack 选项提供一个对象：
```
// vue.config.js
module.exports = {
  configureWebpack: {
    plugins: [
      new MyAwesomeWebpackPlugin()
    ]
  }
}
```
该对象将会被 webpack-merge 合并入最终的 webpack 配置。
如果你需要基于环境有条件地配置行为，或者想要直接修改配置，那就换成一个函数 (该函数会在环境变量被设置之后懒执行)。该方法的第一个参数会收到已经解析好的配置。在函数内，你可以直接修改配置，或者返回一个将会被合并的对象：
```
// vue.config.js
module.exports = {
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      // 为生产环境修改配置...
    } else {
      // 为开发环境修改配置...
    }
  }
}
```
#### 链式操作 (高级)
Vue CLI 内部的 webpack 配置是通过 webpack-chain 维护的。这个库提供了一个 webpack 原始配置的上层抽象，使其可以定义具名的 loader 规则和具名插件，并有机会在后期进入这些规则并对它们的选项进行修改。

修改 Loader 选项
```
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule('vue')
      .use('vue-loader')
        .loader('vue-loader')
        .tap(options => {
          // 修改它的选项...
          return options
        })
  }
}
```
添加一个新的 Loader
```
// vue.config.js
module.exports = {
  chainWebpack: config => {
    // GraphQL Loader
    config.module
      .rule('graphql')
      .test(/\.graphql$/)
      .use('graphql-tag/loader')
        .loader('graphql-tag/loader')
        .end()
      // 你还可以再添加一个 loader
      .use('other-loader')
        .loader('other-loader')
        .end()
  }
}
```
替换一个规则里的 Loader
```
// vue.config.js
module.exports = {
  chainWebpack: config => {
    const svgRule = config.module.rule('svg')

    // 清除已有的所有 loader。
    // 如果你不这样做，接下来的 loader 会附加在该规则现有的 loader 之后。
    svgRule.uses.clear()

    // 添加要替换的 loader
    svgRule
      .use('vue-svg-loader')
        .loader('vue-svg-loader')
  }
}
```
修改插件选项
- 你需要熟悉 webpack-chain 的 API 并阅读一些源码以便了解如何最大程度利用好这个选项，但是比起直接修改 webpack 配置，它的表达能力更强，也更为安全。
#### 审查项目的 webpack 配置
vue-cli-service 暴露了 inspect 命令用于审查解析好的 webpack 配置。那个全局的 vue 可执行程序同样提供了 inspect 命令，这个命令只是简单的把 vue-cli-service inspect 代理到了你的项目中。
该命令会将解析出来的 webpack 配置、包括链式访问规则和插件的提示打印到 stdout。
#### 以一个文件的方式使用解析好的配置
有些外部工具可能需要通过一个文件访问解析好的 webpack 配置，比如那些需要提供 webpack 配置路径的 IDE 或 CLI。在这种情况下你可以使用如下路径：
```
<projectRoot>/node_modules/@vue/cli-service/webpack.config.js
```
该文件会动态解析并输出 vue-cli-service 命令中使用的相同的 webpack 配置，包括那些来自插件甚至是你自定义的配置。

### 环境变量和模式
可以替换你的项目根目录中的下列文件来指定环境变量：
```
.env                # 在所有的环境中被载入
.env.local          # 在所有的环境中被载入，但会被 git 忽略
.env.[mode]         # 只在指定的模式中被载入
.env.[mode].local   # 只在指定的模式中被载入，但会被 git 忽略
```
一个环境文件只包含环境变量的“键=值”对，被载入的变量将会对 vue-cli-service 的所有命令、插件和依赖可用。
- 为一个特定模式准备的环境文件 (例如 .env.production) 将会比一般的环境文件 (例如 .env) 拥有更高的优先级。
#### 模式
模式是 Vue CLI 项目中一个重要的概念。默认情况下，一个 Vue CLI 项目有三个模式：
- development 模式用于 vue-cli-service serve
- production 模式用于 vue-cli-service build 和 vue-cli-service test:e2e
- test 模式用于 vue-cli-service test:unit

注意模式不同于 NODE_ENV，一个模式可以包含多个环境变量。也就是说，每个模式都会将 NODE_ENV 的值设置为模式的名称——比如在 development 模式下 NODE_ENV 的值会被设置为 "development"。

你可以通过为 .env 文件增加后缀来设置某个模式下特有的环境变量。比如，如果你在项目根目录创建一个名为 .env.development 的文件，那么在这个文件里声明过的变量就只会在 development 模式下被载入。
你可以通过传递 --mode 选项参数为命令行覆写默认的模式。例如，如果你想要在构建命令中使用开发环境变量，请在你的 package.json 脚本中加入：
```
"dev-build": "vue-cli-service build --mode development",
```

### 构建目标
当你运行 vue-cli-service build 时，你可以通过 --target 选项指定不同的构建目标。它允许你将相同的源代码根据不同的用例生成不同的构建。

#### 应用
应用模式是默认的模式。在这个模式中：
- index.html 会带有注入的资源和 resource hint
- 第三方库会被分到一个独立包以便更好的缓存
- 小于 4kb 的静态资源会被内联在 JavaScript 中
- public 中的静态资源会被复制到输出目录中

#### 库
- IE 兼容性

可以通过下面的命令将一个单独的入口构建为一个库：
```
vue-cli-service build --target lib --name myLib [entry]
```
##### Vue vs. JS/TS 入口文件
当使用一个 .vue 文件作为入口时，你的库会直接暴露这个 Vue 组件本身，因为组件始终是默认导出的内容。
然而，当你使用一个 .js 或 .ts 文件作为入口时，它可能会包含具名导出，所以库会暴露为一个模块。

#### Web Components 组件
- IE 兼容性

可以通过下面的命令将一个单独的入口构建为一个 Web Components 组件
```
vue-cli-service build --target wc --name my-element [entry]
```


### 部署
#### 通用指南
如果你用 Vue CLI 处理静态资源并和后端框架一起作为部署的一部分，那么你需要的仅仅是确保 Vue CLI 生成的构建文件在正确的位置，并遵循后端框架的发布方式即可。

如果你独立于后端部署前端应用——也就是说后端暴露一个前端可访问的 API，然后前端实际上是纯静态应用。那么你可以将 dist 目录里构建的内容部署到任何静态文件服务器中，但要确保正确的 publicPath。

#### 本地预览
dist 目录需要启动一个 HTTP 服务器来访问。
在本地预览生产环境构建最简单的方式就是使用一个 Node.js 静态文件服务器，例如 serve:
```
npm install -g serve
# -s 参数的意思是将其架设在 Single-Page Application 模式下
# 这个模式会处理即将提到的路由问题
serve -s dist
```
#### 使用 history.pushState 的路由
需要配置生产环境服务器，将任何没有匹配到静态文件的请求回退到 index.html。参考Vue Router 的文档，提供了常用服务器配置指引。
#### CORS
如果前端静态内容是部署在与后端 API 不同的域名上，你需要适当地配置 CORS
#### PWA
如果你使用了 PWA 插件，那么应用必须架设在 HTTPS 上，这样 Service Worker 才能被正确注册。

#### 平台指南
##### 以GitHub Pages为例
需要在 vue.config.js 中设置正确的 publicPath。
如果打算将项目部署到 https://<USERNAME>.github.io/ 上, publicPath 将默认被设为 "/"，你可以忽略这个参数。
如果打算将项目部署到 https://<USERNAME>.github.io/<REPO>/ 上 (即仓库地址为 https://github.com/<USERNAME>/<REPO>)，可将 publicPath 设为 "/<REPO>/"。举个例子，如果仓库名字为“my-project”，那么 vue.config.js 的内容应如下所示：
```
module.exports = {
  publicPath: process.env.NODE_ENV === 'production'
    ? '/my-project/'
    : '/'
}
```

