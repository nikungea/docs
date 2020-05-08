# Commander.js
commander灵感来自 Ruby，它提供了用户命令行输入和参数解析的强大功能，可以帮助我们简化命令行开发。

## 安装

```
npm install commander
```
## 示例
#### 首先看一个简单的示例，了解基本的使用方法：
创建app.js
```
var program = require('commander');

program
    .version('0.0.1')
    .option('-p, --peppers', 'Add peppers')
    .option('-P, --pineapple', 'Add pineapple')
    .option('-b, --bbq', 'Add bbq sauce')
    .option('-c, --cheese [type]', 'Add the specified type of cheese [marble]', 'marble')
    .parse(process.argv);

console.log('you ordered a pizza with:');
if (program.peppers) console.log('  - peppers');
if (program.pineapple) console.log('  - pineapple');
if (program.bbq) console.log('  - bbq');
console.log('  - %s cheese', program.cheese);
```
命令行输入测试：

无参数 node app.js

输出：
```
you ordered a pizza with:
  - marble cheese
```
一个参数 node app.js -c

输出：
```
you ordered a pizza with:
  - marble cheese
```
多个参数 node app.js -cbp

输出：
```
you ordered a pizza with:
  - peppers
  - bbq
  - marble cheese
```
查看帮助 node app.js --help

输出：
```
Usage: app.js [options]
  Options:
    -h, --help           output usage information
    -V, --version        output the version number
    -p, --peppers        Add peppers
    -P, --pineapple      Add pineapple
    -b, --bbq            Add bbq sauce
    -c, --cheese [type]  Add the specified type of cheese [marble]
```
查看版本 node app.js -V

输出：
```
0.0.1
```
## 声明program变量

Commander为了方便快速编程导出了一个全局对象。

```js
const { program } = require('commander');
program.version('0.0.1');
```

对于可能以多种方式使用commander的大型程序，包括单元测试，最好创建一个本地Command对象来使用。

```js
const { Command } = require('commander');
const program = new Command();
program
.version('0.0.1')
.parse(process.argv);
# 执行
node index.js -V
# 结果
0.0.1
```

## 选项

`.option()` 方法用来定义带选项的 commander，同时也用于这些选项的文档。每个选项可以有一个短标识(单个字符)和一个长名字，它们之间用逗号或空格或'|'分开。

 选项会被放到 Commander 对象的属性上，多词选项如"--template-engine"会被转为驼峰法`program.templateEngine`。
 多个短标识可以组合为一个破折号开头的参数：布尔标识和值，并且最后一个标识可以附带一个值。
 例如，`-a -b -p 80` 也可以写作 `-ab -p80` 甚至 `-abp80`。

你可以使用`--`来指示选项的结束，任何剩余的参数会正常使用，而不会被命令解释
这点对于通过另一个命令来传递选项值的情况尤其适用，如:`do -- git --version`

命令行中的选项位置不是固定的，可以在别的命令参数之前或之后指定

### 常用选项类型，boolean和值

最常用的两个选项类型是boolean(选项后面不跟值)和选项跟一个值（使用尖括号声明）。除非在命令行中指定，否则两者都是`undefined`。

`program.parse(arguments)`会处理参数，没有被使用的`program`选项会被存放在`program.args`数组中。

### 默认选项值

可以为选项设置一个默认值。

```js
const program = require('commander');

program
  .option('-c, --cheese <type>', 'add the specified type of cheese', 'blue');

program.parse(process.argv);

console.log(`cheese: ${program.cheese}`);
```

```bash
$ pizza-options
cheese: blue
$ pizza-options --cheese stilton
cheese: stilton
```

### 其他选项类型，可忽略的布尔值和标志值

选项的值为boolean类型时，可以在其长名字前加`no-`规定这个选项值为false。
单独定义同样使选项默认为真。

### 自定义选项处理

可以指定一个函数来处理选项的值，接收两个参数：用户传入的值、上一个值(previous value)，它会返回新的选项值。

可以将选项值强制转换为所需类型，或累积值，或完全自定义处理。

可以在函数后面指定选项的默认或初始值。

### 必需的选项

可以使用`.requiredOption`指定一个必需的(强制性的)选项，这样的选项在被解析后必须有一个值，通常情况下这个值在命令行中被指定，或者也许它拥有一个默认的值(可以说，来自环境)。另外，这个方法在格式上和使用`.option`一样采用标志和说明，以及可选的默认值或自定义处理。

```js
const program = require('commander');

program
  .requiredOption('-c, --cheese <type>', 'pizza must have cheese');

program.parse(process.argv);
```

```bash
$ pizza
error: required option '-c, --cheese <type>' not specified
```

### 版本选项

`version`方法会处理显示版本命令，默认选项标识为`-V`和`--version`，当存在时会打印版本号并退出。

```js
program.version('0.0.1');
```

```bash
$ ./examples/pizza -V
0.0.1
```

你可以自定义标识，通过给`version`方法再传递一个参数，语法与`option`方法一致。版本标识名字可以是任意的，但是必须要有长名字。

```bash
program.version('0.0.1', '-v, --vers', 'output the current version');
```

## Commands

可以使用 `.command` 或者 `.addCommand()` 为你的最高层命令指定子命令。
这里我们有两种方法可以实现：为命令绑定一个操作处理程序(action handler)，或者将命令单独写成一个可执行文件(稍后我们会详细讨论)。这样的子命令是可以嵌套的。

在 `.command` 的第一个参数你可以指定命令的名字以及任何参数。参数可以是 `<required>`(必须的) or `[optional]`(可选的) , 并且最后一个参数也可以是 `variadic...`(可变的).

你可以使用 `.addCommand()` 来向`program`增加一个已经配置好的子命令

例如:

```js
// 通过绑定操作处理程序实现命令 (这里description的定义和 `.command` 是分开的)
// 返回新生成的命令(即该子命令)以供继续配置
program
  .command('clone <source> [destination]')
  .description('clone a repository into a newly created directory')
  .action((source, destination) => {
    console.log('clone command called');
  });

// 通过独立的的可执行文件实现命令 (注意这里description是作为 `.command` 的第二个参数)
// 返回最顶层的命令以供继续添加子命令
program
  .command('start <service>', 'start named service')
  .command('stop [service]', 'stop named service, or all if no name supplied');

// 分别装配命令
// 返回最顶层的命令以供继续添加子命令
program
  .addCommand(build.makeBuildCommand());  
```

可以通过调用 `.command()` 来传递配置的选项。为`opts.noHelp`指定`true` 则该命令不会出现生成的帮助输出里。为`opts.isDefault`指定`true`会在没有别的指定子命令的时候执行这个命令。

### 指定参数语法

你可以通过使用 `.arguments` 来为最顶级命令指定参数，对于子命令来说参数都包括在 `.command` 调用之中了。尖括号(e.g. `<required>`)意味着必须的输入，而方括号(e.g. `[optional]`)则是代表了可选的输入

```js
const program = require('commander');

program
  .version('0.1.0')
  .arguments('<cmd> [env]')
  .action(function (cmd, env) {
    cmdValue = cmd;
    envValue = env;
  });

program.parse(process.argv);

if (typeof cmdValue === 'undefined') {
  console.error('no command given!');
  process.exit(1);
}
console.log('command:', cmdValue);
console.log('environment:', envValue || "no environment given");
```

一个命令有且仅有最后一个参数是可变的，你需要在参数名后加上 `...` 来使它可变，例如

```js
const program = require('commander');

program
  .version('0.1.0')
  .command('rmdir <dir> [otherDirs...]')
  .action(function (dir, otherDirs) {
    console.log('rmdir %s', dir);
    if (otherDirs) {
      otherDirs.forEach(function (oDir) {
        console.log('rmdir %s', oDir);
      });
    }
  });

program.parse(process.argv);
```

可变参数的值以 `数组` 的形式传递给操作处理程序。

### 操作处理程序(Action handler) (子)命令

你可以使用操作处理程序为一个命令增加选项options。
操作处理程序会接收每一个你声明的参数的变量，和一个额外的参数——这个命令对象自己。这个命令的参数包括添加的命令特定选项的值。

```js
const program = require('commander');

program
  .command('rm <dir>')
  .option('-r, --recursive', 'Remove recursively')
  .action(function (dir, cmdObj) {
    console.log('remove ' + dir + (cmdObj.recursive ? ' recursively' : ''))
  })

program.parse(process.argv)
```

你可以自行实现一个`async`操作处理程序，同时调用`.parseAsync`代替`.parse`。

```js
async function run() { /* 在这里编写代码 */ }

async function main() {
  program
    .command('run')
    .action(run);
  await program.parseAsync(process.argv);
}
```


当一个命令在命令行上被使用时，它的选项必须是合法的。使用任何未知的选项会报错。

### 独立的可执行(子)命令

当 `.command()` 带有描述参数时，不能采用 `.action(callback)` 来处理子命令，否则会出错。这告诉 Commander，你将采用独立的可执行文件作为子命令。
Commander 将会尝试在入口脚本（例如 `./examples/pm`）的目录中搜索 `program-command` 形式的可执行文件，例如 `pm-install`, `pm-search`。
你可以使用配置选项 `executableFile` 来指定一个自定义的名字

你可以在可执行文件里处理可执行(子)命令的选项，而不必在顶层声明它们

```js
// file: ./examples/pm
const program = require('commander');

program
  .version('0.1.0')
  .command('install [name]', 'install one or more packages')
  .command('search [query]', 'search with optional query')
  .command('update', 'update installed packages', {executableFile: 'myUpdateSubCommand'})
  .command('list', 'list packages installed', {isDefault: true})
  .parse(process.argv);
```

如果你打算全局安装该命令，请确保可执行文件有对应的权限，例如 `755`。

## 自动化帮助信息 help

帮助信息是 commander 基于你的程序自动生成的，默认的帮助选项是`-h,--help` ([样例](./examples/pizza))。

```bash
$ node ./examples/pizza --help
Usage: pizza [options]

一个用于下单订购披萨的应用

Options:
  -V, --version        输出版本号
  -p, --peppers        增加peppers
  -c, --cheese <type>  指定增加特殊种类的cheese (默认: "marble")
  -C, --no-cheese      你不想要任何cheese
  -h, --help           展示命令的帮助信息
```

如果你的命令中包含了子命令，`help`命令默认会被添加，它可以单独使用或者通过一个子命令名字来展示子命令的进一步帮助信息。实际上这和有隐式帮助信息的`shell`程序一样:

```bash
shell help
shell --help

shell help spawn
shell spawn --help
```

## 自定义事件监听

你可以通过监听命令和选项来执行自定义函数。

```js
program.on('option:verbose', function () {
  process.env.VERBOSE = this.verbose;
});

program.on('command:*', function (operands) {
  console.error(`error: unknown command '${operands[0]}'`);
  const availableCommands = program.commands.map(cmd => cmd.name());
  mySuggestBestMatch(operands[0], availableCommands);
  process.exitCode = 1;
});
```

