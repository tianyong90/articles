# 使用 Eslint 保证代码干净高质

## 什么是 Eslint ？为什么用它？

JavaScript 是一种十分灵活的脚本语言，有些地方甚至灵活到没什么规矩。比如，跟不少脚本语言一样它是弱类型的，变量可以不“声明”而直接使用/赋值，行尾的分号可有可无，有些大括号可有可无……

这些错误有的可能只是规范性或者说习惯上的错误，并不影响代码的运行，而有的错误则可能程序运行出错。

## 如何使用

### 安装

- 全局

```shell
$ npm install eslint -g
```

- 项目内

```shell
$ npm install eslint --save-dev
```

> 除了安装 eslint 之处，还需要根据需要安装它的一些插件包，例如：eslint-config-standard, eslint-plugin-promise 等等

### 规则配置，.eslintrc.js 或 .eslintrc.json

### .eslintignore

### 注释语法 eslint-disable

### Editorconfig

### 命令行

### IDE 或编辑器的相关插件

1. WebStorm 和 PhpStorm 较新版本的已经内内置支持 Eslint 同时可以根据 editorconfig 配置。。

2. Visual Studio Code 也有插件，安装后可支持错误标记提示，一键修正等功能。

3. Sublime Text, Atom 等其它编辑器也可以找到合适的插件，可根据自身情况选用。

## 规则配置的若干建议

1. 使用比较流行的规则

    目前有许多成套的成熟规则，如：airbnb,standard…… 这些规则是经过大量实践检验的

2. 选择适合自身项目和习惯的



3. 并不是越严格越好

    遵循适合的规范能保证质量，但并不是越严格越好。就我个人而言，不太喜欢 airbnb 的规则，它太严格太琐碎了，让我无所适从，会拉低编程效率。
    当然，对于大型或超大型项目，严格的规范可能是比较合适的。

4. 结合 prettier 一起使用

    使用 prettier 自动修正代码，能省事许多

如果没有规范的约束，大家一起放飞自我……

## 总结

先乱写，再整理？