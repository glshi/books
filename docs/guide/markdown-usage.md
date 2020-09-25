## 段落背景格式

> A magical documentation site generator.

docsify generates your documentation website on the fly. Unlike GitBook, it does not generate static html files. Instead, it smartly loads and parses your Markdown files and displays them as a website. To start using it, all you need to do is create an `index.html` and [deploy it on GitHub Pages](deploy.md).

?> docsify generates your documentation website on the fly. Unlike GitBook, it does not generate static html files. Instead, it smartly loads and parses your Markdown files and displays them as a website. To start using it, all you need to do is create an `index.html` and [deploy it on GitHub Pages](deploy.md).

!> Docsify only looks for `_sidebar.md` in the current folder, and uses that, otherwise it falls back to the one configured using `window.$docsify.loadSidebar` config.

## 链接方式

See the [Quick start](quickstart.md) guide for more details.

Support server-side rendering ([example](https://github.com/docsifyjs/docsify-ssr-demo))

<a href="https://vercel.com/?utm_source=docsifyjsdocs" target="_blank"><img src="_media/vercel_logo.svg" width="100px"></a>

Activate the cover feature by setting `coverpage` to **true**, compare [coverpage configuration](quickstart.md#Manual-initialization).

## 列表使用格式

- No statically built html files
- Simple and lightweight
- Smart full-text search plugin
- Multiple themes
- Useful plugin API
- Emoji support
- Compatible with IE11
- Support server-side rendering ([example](https://github.com/docsifyjs/docsify-ssr-demo))


## 使用Emoji

Please consider donating if you think docsify is helpful to you or that my work is valuable. I am happy if you can help me [buy a cup of coffee](https://github.com/QingWei-Li/donate). :heart:

## 编程代码块

```html
<!-- load css -->
<link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify/lib/themes/vue.css">

<!-- load script -->
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

```js
window.$docsify = {
  logo: '/_media/icon.svg',
};
```

```markdown
## This is test

<script>
  console.log(2333)
</script>
```

