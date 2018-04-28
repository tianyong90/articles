# 自己整个 vue markdown loader

最近，当我把 vue-loader 升级到 15.0.0 后发现，自己项目中所使用的 vue-markdown-loader 没法用了，正当我一筹莫展的时候，发现 vuepress 中使用了当时还处于 rc 版本的最新的 vue-loader,仔细研究其源码后发现，vuepress 对于 markdown 的支持相当完善。

## 关于 webpack loader

## 安装需要的包

```bash
yarn add -D markdown-it markdown-it-anchor markdown-it-container markdown-it-emoji markdown-it-table-of-contents
```

| 包名称 | 功能说明 |
| ---- | ---- |
| markdown-it | 渲染 markdown 基本语法 |
| markdown-it-anchor | 为各级标题添加锚点 |
| markdown-it-container | 用于创建自定义的块级容器 |
| markdown-it-emoji | 渲染 emoji |
| markdown-it-table-of-contents | 自动生成目录 |


```js
const fs = require('fs')
const path = require('path')
const hash = require('hash-sum')
const LRU = require('lru-cache')
const hljs = require('highlight.js')

// markdown-it 插件
const emoji = require('markdown-it-emoji')
const anchor = require('markdown-it-anchor')
const toc = require('markdown-it-table-of-contents')

// 自定义块
const containers = require('./containers')

const md = require('markdown-it')({
  html: true,
  highlight: function (str, lang) {
    if (lang && hljs.getLanguage(lang)) {
      try {
        return '<pre class="hljs"><code>' +
          hljs.highlight(lang, str, true).value +
          '</code></pre>'
      } catch (__) {}
    }

    return '<pre v-pre class="hljs"><code>' + md.utils.escapeHtml(str) + '</code></pre>'
  }
})
  // 使用 emoji 插件渲染 emoji
  .use(emoji)
  // 使用 anchor 插件为标题元素添加锚点
  .use(anchor, {
    permalink: true,
    permalinkBefore: true,
    permalinkSymbol: '#'
  })
  // 使用 table-of-contents 插件实现自动生成目录
  .use(toc, {
    includeLevel: [2, 3]
  })
  // 定义自定义的块容器
  .use(containers)

const cache = LRU({ max: 1000 })

module.exports = function (src) {
  const isProd = process.env.NODE_ENV === 'production'

  const file = this.resourcePath
  const key = hash(file + src)
  const cached = cache.get(key)

  // 重新模式下构建时使用缓存以提高性能
  if (cached && (isProd || /\?vue/.test(this.resourceQuery))) {
    return cached
  }

  const html = md.render(src)

  const res = (
    `<template>\n` +
    `<div class="content">${html}</div>\n` +
    `</template>\n`
  )
  cache.set(key, res)
  return res
}
```

// 以下为上面代码中引用到的 containers.js 中代码
```js
const container = require('markdown-it-container')

module.exports = md => {
  md
    .use(...createContainer('tip', 'TIP'))
    .use(...createContainer('warning', 'WARNING'))
    .use(...createContainer('danger', 'WARNING'))
    // explicitly escape Vue syntax
    .use(container, 'v-pre', {
      render: (tokens, idx) => tokens[idx].nesting === 1
        ? `<div v-pre>\n`
        : `</div>\n`
    })
}

function createContainer (klass, defaultTitle) {
  return [container, klass, {
    render (tokens, idx) {
      const token = tokens[idx]
      const info = token.info.trim().slice(klass.length).trim()
      if (token.nesting === 1) {
        return `<div class="${klass} custom-block"><p class="custom-block-title">${info || defaultTitle}</p>\n`
      } else {
        return `</div>\n`
      }
    }
  }]
}
```

## 使用

在 webpack 配置的  `module.rules` 数组中加入如下规则：

```js
{
  test: /\.md$/,
  use: [
    {
      loader: 'vue-loader',
      options: {
        compilerOptions: {
          preserveWhitespace: false
        }
      }
    },
    {
      // 这里用到的就是刚写的那个 loader
      loader: require.resolve('./markdownLoader')
    }
  ]
},
```

然后，就可以在自己的组件中引入 markdown 文件了。

假如你有一个名叫 something-cool.md 的 markdown 文件源文如下:

```md
# hello world

[[toc]]

## 二级标题1

有钱就是可以为所欲为！ :+1:

## 二级标题2

我特么真的没钱 :cry:

## 二级标题3

### 三级标题1

### 三级标题2

:::tip
友情提示
:::

:::danger
卧槽，粗大事了……
:::

```

```html
<template>
  <my-markdown/>
</template>

<script>
export default {
  components: {
    'my-markdown': () => import('./something-cool.ms')
  }
}
</script>
```
