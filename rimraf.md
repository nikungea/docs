# rimraf
## 地址
[npm](https://www.npmjs.com/package/rimraf) | [Github](https://github.com/isaacs/rimraf)
## 安装
```
npm i rimraf
```
## 作用
以包的形式包装rm -rf命令，用来删除文件和文件夹的，不管文件夹是否为空，都可以删除。
## API
rimraf(f, [opts], callback)
- f 第一个参数将被解释为文件的通配模式。如果您想禁用通配符，可以使用opts.disableGlob（默认为 false）进行。
- callback,如果抛出错误会调用回调函数进行处理。
- 有一些错误已经默认处理好：
  - Windows：EBUSY和ENOTEMPTY，rimraf在终止之前将尝试opts.maxBusyTries设置的次数，每次尝试之后多增加100ms，预设maxBusyTries值为3。
  - ENOENT -如果文件不存在，rimraf将成功返回，因为文件已经不存在了。
  - EMFILE-由于readdir需要打开文件描述符，如果使用过多的文件描述符，可能会触发EMFILE。同步时，无需做任何事情。但异步情况下，rimraf会逐渐退出，超时时间最多为opts.emfileWaitms，默认为1000。

## 使用
### 删除node_modules
npm中可以通过rimraf来删除文件夹，无论该文件夹是否为空，需要全局安装rimraf
```
rimraf node_modules
```

### webpack打包中的应用（每次打包前先删除上次打包的文件）
配置 package.json中的script中写入以下内容
```
"clean": "rimraf build/static"
```

### 也可以引入使用
在demo.js中：
```
const rimraf = require('rimraf');
rimraf('./test.js', function (err) { // 删除当前目录下的 test.js
  console.log(err);
});
```
然后终端中执行`node demo.js`, 会删除当前目录下的test.js