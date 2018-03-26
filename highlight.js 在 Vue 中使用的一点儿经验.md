highlight.js 在 Vue 中使用的一点儿经验

使用 markdown 来给程序写文档是非常方便的，自从用顺了 markdown 之后，都很久没打开过 Word 了。

既然是程序的文档，少不了需要插入一些示例代码，而对代码进行语法高亮渲染并配以合适的颜色主题，会让文档显得更炫，也更便于阅读。

要实现文档代码高亮渲染其实并不难。

### 实现方法

首先，把 markdown 文件加载为 vue 组件，这需要一个合适的 loader，自己目前使用 `vue-markdown-loader`。webpack 配置的 `module.rules` 中进行如下配置：

```js
{
    test: /\.md$/,
    loader: 'vue-markdown-loader',
    options: {
        preset: 'default',
        breaks: true,
        preventExtract: true
    }
}
```

然后就可以在项目中直接 import md 文件了。比如：

```html
<template>
    <MyMarkdown/>
</template>

<script>
export default {
    components: {
        'MyMarkdown': () => import('xxx.md')
    }
}
</script>
```

当然，通常情况下，我们会与 vue-router 一起使用，把 md 文件作为一个视图组件加载到 `router-view` 中去。

```js
{
    path: 'path/home',
    component: () => import('../markdown/home.md')
},
```

看到这里可能奇怪，这些与文题中提到的 highlight.js 有毛关系？这是因为，vue-markdown-loader 中已经内置了对代码高这的支持。你只需要在页面中引入相关的样式，例如： 

```js
import 'highlight.js/styles/atom-one-dark.css'
```

然后主可以看到代码高亮的效果，通常是这样的。


![clipboard.png](https://segmentfault.com/img/bV6JMs?w=1093&h=632)


看起来还不错，但这样的高亮有个问题，那就是他的背景色并不随着你所加载了 highlight.js 主题样式而改变，而且不同语言的代码在配色上的一些差异也没有很好的渲染出来。而从 highlight.js 官网示例可以看到，这些问题本不应该出现的。

为了实现与 highlight.js 官网示例中的主题效果，可以在页面中自己完成代码高亮的渲染。

```js
<script>
import hljs from 'highlight.js'
import 'highlight.js/styles/atom-one-dark.css'

const highlightCode = () => {
  const preEl = document.querySelectorAll('pre')

  preEl.forEach((el) => {
    hljs.highlightBlock(el)
  })
}

export default {
    mounted () {
        highlightCode()
    },

    updated () {
        highlightCode()
    }
}
</script>
```

可以看到，代码中使用了 highlight.js 的 `highlightBlock()` 方法而不是官方默认示例里提到的 `initHighlighting()`，因为后者一般用于**静态页面**的渲染。如果使用它，当使用 vue-router 导航到一个新的‘页面’之后，新页面中的代码块可能无法被正确渲染。这也是为什么在 updated 钩子中再次调用 `highlightCode()`的原因。（实际上自己在此坑了很久，查阅不少文档才找到这一原因）

做完这些之后再看渲染效果：

![clipboard.png](https://segmentfault.com/img/bV6JJZ?w=1207&h=557)

果然好多了！

### 后记

既然是自己渲染代码高亮，那么其实 loader 中对代码块块的处理就不必要或者显得有点儿多余了，因为这些处理会增加一些计算量。所以你也可以找一些别的 loader 来替代 vue-markdown-loader,甚至尝试自己写一个 loader。

对于一个软件，官方文档是有必要仔细读的，就像前面提到的 highlight.js 中 `initHighlighting()` 方法的问题，其实在官方文档中也有解释。