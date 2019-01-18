> 记录一下nodejs的学习之路

## NodeJS相关概念

### 阻塞和非阻塞

阻塞I/O：I/O时进程休眠等待I/O完成后进行下一步
非阻塞I/O：I/O时函数立即返回，进程不等待I/O完成

### 事件驱动

I/O等异步操作结束后的通知
观察者模式

### NodeJS是什么？

> * Node.js is a JavaScript runtime built on Chrome's V8
> * Node.js uses an event-driven,non-blocking I/O model

### 为什么偏爱NodeJS?

> CPU密集：压缩、解压、加密、解密
>
> I/O密集：文件操作、网络操作、数据库操作

* 前端职责范围变大，统一开发体验
* 在处理高并发，I/O密集场景（`web场景`）性能优势明显

web常见场景

* 静态资源的获取
* 数据库操作
* 渲染页面

### 高并发应对之道

- 增加机器数、负载均衡
- 增加每台机器的CPU数-多核

>  进程：是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位。
> 线程：进程内一个相对独立的、可调度的执行单元，与同属一个进程的线程共享进程的资源
> 多进程：启动多个进程，多个进程可以一块执行多个任务

### NodeJS工作模型

![32ebbe6b-2658-4d60-9b37-1973d3e5f977](C:\Users\xiongxiaolong5\Downloads\32ebbe6b-2658-4d60-9b37-1973d3e5f977.png)

###  NodeJS的单线程

单线程只是针对主进程，I/O操作系统底层多线程调度
Node单线程并不是单进程（node有一个集群（cluster）模块用来处理多进程，cpu有几个核就启动几个进程）

### 常用场景

* Web Server
* 本地代码构建（现在前端各种框架、ES6、模块化等的出现，前端代码变得异常的复杂，无法直接在浏览器上运行，需要一些转化工作）：虽然是I/O操作比较少，也许就几个文件，主要是CPU运算，但是这些都是在处理前端代码，有很多前端的逻辑
* 实用工具开发：爬虫（虽然并不是最佳）

### 环境

**CommonJS**

- 每个文件是一个模块，有自己的作用域
- 在模块内部module变量代表模块本身
- module.exports属性代表模块对外接口

**global**（Node没有BOM，DOM）
**process** 当前执行的进程， 挂载在global下面

require规则
`/`表示绝对路径，`./`表示相对于当前文件的路径
支持js，json，node拓展名，不写依次尝试
不写路径则认为是build-in模块或者各级node_modules内的第三方模块。
require特性

**module被加载的时候执行，加载后缓存**

```javascript
// a.js
console.log('This is a module!')
const testVar = 100
function test () {
    console.log(testVar)
}

module.exports.testVar = testVar
module.exports.testFn = test

// b.js
var mod = require("./a.js")
console.log(mod.testVar)
mod.testFn()
// This is a module!
// 100
// 100

// c.js
require("./a.js")
require("./a.js")
// This is a module!
```

**一旦出现某个模块被循环加载，就只输出已经执行的部分，还未执行的部分不会输出（要避免循环加载）**

```javascript
// modA.js
module,exports.test = 'A'

const modB = require('./modB')
console.log('modA: ', modB.test)

module.exports.test = 'AA'

// modB.js
module,exports.test = 'B'

const modA = require('./modA')
console.log('modB: ', modA.test)

module.exports.test = 'BB'

// main.js
var modA = require('./modA')
var modB = require('./modB')
console.log(modA.test)
console.log(modB.test)

// modB : A
// modA: BB
// AA
// BB

```

**module.exports和exports的区别**

exports = module.exports
在一般情况下，exports就是module.exports的快捷方式；
可以向exports里添加属性，但不能改变exports的指向（改变了指向后和普通的对象没什么区别）。

```javascript
(
    function(exports, require, module, _filename, _dirname){
          // code
     }
);
比如：
exports.test = 100; // 可以正常引用
exports={               // 改变了exports的指向，require后，mod.test是undefined。
    test: 100,
    a: 200
}
module.exports={               // 正常，mod.test是100
    test: 100,
    a: 200
}
```

## 基本API

### process

```javascript
// a.js
const {argv, argv0, execArgv, execPath} = process;

argv.forEach(item => {  // 查看参数，比较常用
    console.log(item)
});
console.log(argv0)
console.log(execArgv)
console.log(execPath)  // 执行脚本的路径

命令行输入：node --inspect a.js --test a=1 b=2
/* /usr/local/bin/node
   /Users/nodejs/demos/a.js
   --test
   a=1
   b=2
   node
   [ '--inspect' ]
      /usr/local/bin/node
*/

// process 执行环境相关
const {env} = process;
console.log(env);

console.log(process.cwd()); // 打印当前命令执行的路径，和Linux的pwd一样
```

### timer

```javascript
setImmediate(() => {             // 下一个队列的队首
    console.log('setImmediate');
});

setTimeout(() => {               // 两者之间
    console.log('setTimeout')
}, 0);

process.nextTick(() => {         // 当前队列的最后一个，会影响后面正常异步的执行
    console.log('nextTick');
        process.nextTick(() => {
            console.log('nextTick');
        })
})

/* nextTick             idle
    nextTick             
   setTimeout          io
   setImmediate      check
*/
```

> **调试**
> node --inspect-brk a.js （加brk在入口文件停住），在谷歌浏览器里打开chrome://inspect/#devices
> 直接在IDE中调试

### path

```javascript
const {normalize} = require('path');
// const normalize = require('path').normalize;
console.log(normalize('/usr/local/bin'));
console.log(normalize('/usr//local/../bin'));
// \usr\local\bin
// \usr\bin

// join，拼接自动处理斜线
const {join} = require('path');
console.log(join('/usr','local','bin/'));
// \usr\local\bin\

// resolve，把相对路径转换成绝对路径
const {resolve} = require('path');
console.log(resolve('./'));
// C:\Users\jrxiongxiaolong\Desktop\myWork\nodeJS

const {basename, dirname, extname} = require('path');
const filePath = '/usr/loacl/bin/no.txt';
console.log(basename(filePath));
console.log(dirname(filePath));
console.log(extname(filePath));
// no.txt
// /usr/loacl/bin
// .txt

// 当要修改路径中的某一个值的话，经常用转化
const {parse, format} = require('path');
const ret = parse('/usr/local/bin/no.txt');
console.log(ret);
console.log(format(ret));
// { root: '/',
// dir: '/usr/local/bin',
// base: 'no.txt',
// ext: '.txt',
// name: 'no' }
// /usr/local/bin\no.txt

const {
sep,
delimiter,
win32,
posix
} = require('path');

console.log('sep: ', sep);
console.log('PATH: ', process.env.PATH);
console.log('delimiter: ', delimiter);
// sep: \
// PATH: C:\Users\jrxiongxiaolong\bin;C:\Program Files\Git\mingw64\bin;C:\Program
// Files\Git\usr\local\bin;C:\Program Files\Git\usr\bin;C:\Program Files\Git\usr\bi
// n;C:\Program Files\Git\mingw64\bin;C:\Program Files\Git\usr\bin
// delimiter: ;
console.log('win sep: ', win32.sep);
console.log('win delimiter: ', win32.delimiter);
// win sep: \
// win delimiter: ;
console.log('posix sep: ', posix.sep);
console.log('posix delimiter: ', posix.delimiter);
// posix sep: /
// posix delimiter: :
```

> \_\_dirname,\_\_filename 总是返回文件的绝对路径
>
> process.cwd()             返回执行node命令的文件夹
>
> ./                                 在require方法中总是相对当前文件所在文件夹，在其他地方和process.pwd()一样，相对node启动文件夹

### Buffer

挂载在global对象上

- Buffer用于处理二进制数据流
- 实例类似整数数组，大小固定
- 使用C++代码在V8堆外分配物理内存

```javascript
console.log(Buffer.alloc(10));
console.log(Buffer.alloc(10, 1));
console.log(Buffer.allocUnsafe(10)); // 创建但不会初始化
console.log(Buffer.from([1,2,3]));
console.log(Buffer.from('string')); // 默认UTF-8
console.log(Buffer.from('string', 'base64'));
{/* <Buffer 00 00 00 00 00 00 00 00 00 00>
<Buffer 01 01 01 01 01 01 01 01 01 01>
<Buffer e0 0c dd b0 41 01 00 00 00 d4>
<Buffer 01 02 03>
<Buffer 73 74 72 69 6e 67>
<Buffer b2 da e2 9e> */}
/*
Buffer.byteLength()
Buffer.isBuffer()
Buffer.concat()
*/
console.log(Buffer.byteLength('test'));
console.log(Buffer.byteLength('测试')); // 所占字节
console.log(Buffer.isBuffer([1, 2, 3])) // 判断是否为Buffer对象
console.log(Buffer.isBuffer(Buffer.from([1, 2, ,3])));
const buf1 = Buffer.from('This ');
const buf2 = Buffer.from('is ');
const buf3 = Buffer.from('a ');
const buf4 = Buffer.from('test');
const buf = Buffer.concat([buf1, buf2, buf3, buf4]);
console.log(buf.toString());4
// 4
// 6
// false
// true
// This is a test

/*
buf.length
buf.toString()
buf.fill()
buf.equals()
buf.indexOf()
buf.copy()
*/
const buf5 = Buffer.from('This is a test!');
console.log(buf5.length); // Buffer所占的字节数（申请的空间）
console.log(buf5.toString('base64'));

const buf6 = Buffer.allocUnsafe(10);
console.log(buf6);
console.log(buf6.fill(3, 2, 6)); // (val, start, end)
// 15
// VGhpcyBpcyBhIHRlc3Qh
// <Buffer 09 35 1e e8 50 e9 e8 40 26 ea>
// <Buffer 09 35 03 03 03 03 e8 40 26 ea>
console.log(buf5.equals(buf6)); // false
// Buffer是一种类数组的结构
console.log(buf5.indexOf('es'));
console.log(buf5.indexOf('es1'));
// 11
// -1

// 中文编码问题
const StringDecoder = require('string_decoder');
const decoder = new StringDecoder('utf8');
const buf7 = Buffer.from('中文字符串！');
for(let i=0; i<buf7.length; i += 5){
const b = Buffer.allocUnsafe(5);
buf7.copy(b, 0, i);
console.log(b.toString());
}
for(let i=0; i<buf7.length; i += 5){
const b = Buffer.allocUnsafe(5);
buf7.copy(b, 0, i);
console.log(decoder.write(b));
}
```

### events （事件）

```javascript
// events(事件)
// 所有能触发事件的对象都是EventEmitter类的实例
const EventEmitter = require('events');
class CustomEvent extends EventEmitter {

}

var ce1 = new CustomEvent();
ce1.on('test', () => {
console.log('test');
})
setInterval(() => {
ce1.emit('test');
}, 1000)
// test
// test
// test
// ...

var ce2 = new CustomEvent();
ce2.once('test', () => {
console.log('test');
})
setInterval(() => {
ce2.emit('test');
}, 1000)
// test

var ce3 = new CustomEvent();
ce3.on('error', (err, date) => {
console.log(err);
console.log(date);
})
ce3.emit('error', new Error('oops!'), Date.now());

function fn1() {
console.log('fn1');
}
function fn2() {
console.log('fn2');
}
var ce4 = new CustomEvent();
ce4.on('test', fn1);
ce4.on('test', fn2);
setInterval(() => {
ce4.emit('test');
},500)
setTimeout(() => {
// ce4.removeListener('test', fn1);
ce4.removeAllListeners('test'); // 注意有's'
}, 1500)
```

### fs（文件系统）

```javascript
// 异步方法的最后一个参数都是一个回调函数，传给回调函数的参数取决于具体方法，但回调函数的第一个参数都会保留给异常。如果操作成功，则第一个参数会是null或undefined。
const fs = require('fs');
// 读文件，异步
fs.readFile('./a.js', 'utf8', (err,data) => {
if(err) throw err;
console.log(data); // dara.toString()
})
// 同步
const data = fs.readFileSync('./a.js', 'utf8');
console.log(data);

// 写文件
fs.writeFile('./text', 'This is test1.', { // 可以直接写'utf8'
encoding: 'utf8'
}, (err) => {
if(err) throw err;
console.log('Done!')
})
const content = Buffer.from('This is test2.');
fs.writeFile('./text', content, (err) => { // 用Buffer不用写编码格式了
if(err) throw err;
console.log('Done!')
})

// 详细信息
fs.stat('./a.js', (err, stats) => {
if(err) throw err;
console.log(stats.isFile()); // 是否是文件
console.log(stats.isDirectory()); // 是否是文件夹
console.log(stats);
})
// true
// false
// Stats {
// dev: 1150485948,
// mode: 33206,
// nlink: 1,
// uid: 0,
// gid: 0,
// rdev: 0,
// blksize: undefined,
// ino: 4503599627404705,
// size: 2127,
// blocks: undefined,
// atimeMs: 1532799705478.6035,
// mtimeMs: 1532630314117.9111,
// ctimeMs: 1532630314117.9111,
// birthtimeMs: 1531923690001.36,
// atime: 2018-07-28T17:41:45.479Z,
// mtime: 2018-07-26T18:38:34.118Z,
// ctime: 2018-07-26T18:38:34.118Z,
// birthtime: 2018-07-18T14:21:30.001Z }

// 重命名
fs.rename('./text', 'test.txt', err => {
if(err) throw err;
console.log('Done!');
})

// 删除
fs.unlink('./test.txt', err => {})

// 读文件夹，process.cwd()
fs.readdir('./', (err, files) => {
if(err) throw err;
console.log(files);
})

// 创建文件夹
fs.mkdir('./test', err => {})
// 删除文件夹
fs.rmdir('./test', err => {})

// 监视文件或文件夹的变化，可用于本地代码构建时一些文件的变化引起的一些配置代码的自动改变
fs.watch('./', {
recursive: true // 是否递归监视
}, (eventType, filename) => { // 变化的事件类型
console.log(eventType, filename);
})

// stream，流，有方向的数据（两个要点，方向和数据）
const fs = require('fs');
const rs = fs.createReadStream('./demo_2.js');
rs.pipe(process.stdout); // 流向控制台，相当于fs.readFile() 但更优雅。

// const fs = require('fs');
const ws = fs.createWriteStream('./demo_2.js');  // 理论上只接受Buffer，但是String和Buffer间可以互相转化，Buffer.from() 和 buf.toString()，所以也接受字符串，但是不能是数字。
const tid = setInterval(() => {
var num = Math.floor(Math.random() * 10);
if(num < 7) {
console.log(num);
ws.write(num + '');
} else {
console.log(num);
clearInterval(tid);
ws.end();
}
}, 200)
ws.on('finish', () => {
console.log('done!');
})
```

## 其它

### 解决回调地狱问题

```javascript
onst fs = require('fs');
const promisify = require('util').promisify;

const read = promisify(fs.readFile);
read('./d.js').then(data => {
console.log(data.toString());
}).catch((ex) => {
console.log(ex);
})

async function test() {
try {  // 如果await出现 reject 状态，后面的await都不会执行。所以要用try catch处理
const content = await read('./d.js');
console.log(content.toString());
} catch (ex) {
console.log(ex);
}
}
// 若第一个await和第二个await没有关系，那么滥用asyc awiat是不好的，因为第二个await没必要等待第一个执行完毕再去执行。之所以以同步的形式写异步代码，解决回调地狱，而且前一个await执行成功，才能执行下一个await。是因为回调嵌套时，只有在前一个回调成功后才能执行下一个回调，即上一个回调的执行结果和下一个回调有关系。
```

### node项目相关

`.gitignore`

- 匹配模式前 / 代表项目根目录
- 匹配模式最后加 / 代表是目录  build/
- 匹配模式前加 ! 表示取反（可能一个文件夹中的某个文件不需要被忽略）

`*`代表任意个字符，`？`匹配任意一个字符，`**` 匹配多级目录（正则里的 \* 是量词）\*.swp

`.npmignore` 规则和`./gitignore`一样，有一些是不可忽略的，比如 package.json，README，CHANGELOG，LICENSE / LICENCE
**EditorConfig**
统一的项目的配置，不同的编辑器会有不一样的风格，比如tab是真的tab还是空格；最后一行是否需要一个回车；编码方式等。
**ESlint **
开源的JavaScript代码检查工具，JavaScript是一个动态的弱类型语言，在开发中比较容易出错。因为没有编译程序，为了寻找JavaScript代码错误通常需要在执行过程中不断的调试。*像ESlint这样的可以让程序员在编码的过程中发现问题而不是在执行的过程中*。
**ESlint的初衷是为了让程序员可以创建自己的检测规则**。
自定义规则，比如不允许使用 console.log() ,alert() 等
某一块不开启eslint的某个规则

```javascript
/* eslint-disable no-console, no-alert */  
console.log(123)
/* eslint-disable */

alert('foo') // eslint-disable-line no-alert

// eslint-disable-next-line
alert('foo')

```

