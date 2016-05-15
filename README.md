# ES6-Learning
ECMAScript 6 简易教程

## 1. 简介
ECMAScript 6（以下简称ES6）是JavaScript语言的下一代标准，已经在2015年6月正式发布了。它的目标，是使得JavaScript语言可以用来编写复杂的大型应用程序，成为企业级开发语言。

## 2. Babel转码器
截至目前，大部分的浏览器都还只支持到ES5语法（有的浏览器已经可以支持部分ES6语法）。[Babel](https://babeljs.io/)是一个广泛使用的ES6转码器，可以将ES6代码转为ES5代码，从而在现有环境执行。
``` JavaScript
// 转码前
input.map((item) => {
    return item + 1;
});
// 或简写成
input.map(item => item + 1);

// 转码后
input.map(function (item) {
  return item + 1;
});
```

### 2.1 配置文件.babelrc
Babel的配置文件是.babelrc，存放在项目的根目录下。使用Babel的第一步，就是配置这个文件。该文件用来设置转码规则和插件，基本格式如下。
``` Json
{
  "presets": [],
  "plugins": []
}
```

`presets`字段设定转码规则，官方提供以下的规则集，你可以根据需要安装。
``` Shell
# ES2015转码规则
$ npm install --save-dev babel-preset-es2015

# ES7不同阶段语法提案的转码规则（共有4个阶段），选装一个
$ npm install --save-dev babel-preset-stage-0
$ npm install --save-dev babel-preset-stage-1
$ npm install --save-dev babel-preset-stage-2
$ npm install --save-dev babel-preset-stage-3
```

然后，将这些规则加入.babelrc。
``` Json
{
  "presets": [
    "es2015",
    "stage-0"
  ],
  "plugins": []
}
```

### 2.2 命令行转码babel-cli
Babel提供`babel-cli`工具，用于命令行转码。它的安装命令如下。
``` Shell
$ npm install -g babel-cli
```

基本用法如下。
``` Shell
# 转码结果输出到标准输出
$ babel example.js

# 转码结果写入一个文件
# --out-file 或 -o 参数指定输出文件
$ babel example.js --out-file compiled.js
# 或者
$ babel example.js -o compiled.js

# 整个目录转码
# --out-dir 或 -d 参数指定输出目录
$ babel src --out-dir lib
# 或者
$ babel src -d lib

# -s 参数生成source map文件
$ babel src -d lib -s
```

然后，改写package.json。
``` Json
{
  // ...
  "scripts": {
    "build": "babel src -d assets"
  },
}
```

转码的时候，就执行下面的命令。
``` Shell
$ npm run build
```

### 2.3 babel-node
babel-cli工具自带一个babel-node命令，提供一个支持ES6的REPL(Read-Eval-Print Loop)环境。它支持Node的REPL环境的所有功能，而且可以直接运行ES6代码。

它不用单独安装，而是随babel-cli一起安装。然后，执行babel-node就进入REPL环境。
``` Shell
$ babel-node
> (x => x * 2)(1)
2
```

babel-node命令可以直接运行ES6脚本。将上面的代码放入脚本文件es6.js，然后直接运行。
``` Shell
$ babel-node es6.js
2
```

然后，改写package.json。
``` Shell
{
  "scripts": {
    "script-name": "babel-node script.js"
  }
}
```

上面代码中，使用babel-node替代node，这样script.js本身就不用做任何转码处理。

### 2.4 babel-core
如果某些代码需要调用Babel的API进行转码，就要使用babel-core模块。

安装命令如下。
``` Shell
$ npm install babel-core --save
```

然后，在项目中就可以调用babel-core。
``` JavaScript
var babel = require('babel-core');

// 字符串转码
babel.transform('code();', options);
// => { code, map, ast }

// 文件转码（异步）
babel.transformFile('filename.js', options, function(err, result) {
  result; // => { code, map, ast }
});

// 文件转码（同步）
babel.transformFileSync('filename.js', options);
// => { code, map, ast }

// Babel AST转码
babel.transformFromAst(ast, code, options);
// => { code, map, ast }
```

配置对象options，可以参看官方文档<http://babeljs.io/docs/usage/options/>。

下面是一个例子。
``` JavaScript
var es6Code = 'let x = n => n + 1';
var es5Code = require('babel-core')
  .transform(es6Code, {
    presets: ['es2015']
  })
  .code;
// '"use strict";\n\nvar x = function x(n) {\n  return n + 1;\n};'
```

上面代码中，transform方法的第一个参数是一个字符串，表示需要被转换的ES6代码，第二个参数是转换的配置对象。

### 2.5 babel-polyfill
Babel默认只转换新的JavaScript句法（syntax），而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码。

举例来说，ES6在Array对象上新增了Array.from方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用babel-polyfill，为当前环境提供一个垫片。

安装命令如下。
``` Shell
$ npm install --save babel-polyfill
```

然后，在脚本头部，加入如下一行代码。
``` JavaScript
import 'babel-polyfill';
// 或者
require('babel-polyfill');
```

Babel默认不转码的API非常多，详细清单可以查看babel-plugin-transform-runtime模块的[definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js)文件。

### 2.6 在线转换
Babel提供一个[REPL在线编译器](https://babeljs.io/repl/)，可以在线将ES6代码转为ES5代码。转换后的代码，可以直接作为ES5代码插入网页运行。
