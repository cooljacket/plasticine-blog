---
title: "定制hugo主题01——极简版"
date: 2023-01-29T02:02:01+08:00
draft: false
mermaid: true
---

用别人写的theme很依赖对方的文档写清楚，并且自己想做点定制化的东西又有点困难。如有此类需求的话，我觉得自己定制主题起来会更简单清晰。

另外希望自己在diy的过程能把hugo的架构、接口设计理清楚其中逻辑，最好还能沉淀一些common的主题定制能力/方法，这样也能方便其他想定制主题的人。

# 一行命令建主题
hugo对自定义主题已经给了支持，很方便拥有自己的主题。但new完之后发现只有`themes/diy/layouts/_default/baseof.html`有内容，其他文件都需要自己写。

参考文档：https://gohugo.io/commands/hugo_new_theme/
```shell
➜  my-blog git:(master) ✗ hugo new theme diy
➜  my-blog git:(master) ✗ cd themes 
➜  themes git:(master) ✗ tree
.
└── diy
    ├── LICENSE
    ├── archetypes
    │   └── default.md
    ├── layouts
    │   ├── 404.html
    │   ├── _default
    │   │   ├── baseof.html
    │   │   ├── list.html
    │   │   └── single.html
    │   ├── index.html
    │   └── partials
    │       ├── footer.html
    │       ├── head.html
    │       └── header.html
    ├── static
    │   ├── css
    │   └── js
    └── theme.toml
```

该文件的内容实际上相当于没内容……只是把header、content、footer拼起来变成一个页面而已，而这三者目前都是空白的。

```html
<!DOCTYPE html>
<html>
    {{- partial "head.html" . -}}
    <body>
        {{- partial "header.html" . -}}
        <div id="content">
        {{- block "main" . }}{{- end }}
        </div>
        {{- partial "footer.html" . -}}
    </body>
</html>
```

## 分析theme的组成结构
各个文件的含义如下。

```
── diy
    ├── LICENSE （主题源码的开源协议）
    ├── archetypes （创建文章的“原型”，通俗理解就是文章元信息的模板）
    │   └── default.md
    ├── layouts
    │   ├── 404.html （顾名思义……）
    │   ├── _default
    │   │   ├── baseof.html （所有页面的模板）
    │   │   ├── list.html （列表页内容模板）
    │   │   └── single.html （详情页模板）
    │   ├── index.html （网站首页的模板）
    │   └── partials
    │       ├── footer.html （页面底部的模板）
    │       ├── head.html （页面html的<head>标签模板）
    │       └── header.html （页面头部的模板）
    ├── static （专门用来放模板用到的静态资源）
    │   ├── css
    │   └── js
    └── theme.toml
```

{{< mermaid >}}
mindmap
  root((网站页面))
    页面分类
      首页
      详情页（展示特定某篇文章）
      列表页（泛指多个同类内容的导航页面）
      404页面
      自定义页面（如About Me、友情链接等等）
    页面组成结构（其实就是HTML页面结构）
      页面元信息（head）
      页眉（header）
      正文内容
      页脚（footer）
{{< /mermaid >}}


## 极简版需实现的页面
极简版我打算只实现这两个个页面，因为有了它们，博客就能完备浏览了。
- 详情页
- 首页

# 先让单篇文章能展示出来
参考文档：https://gohugo.io/templates/single-page-templates/

用命令`hugo new posts/demo/demo_post1.md`可以创建文章，文章路径为content/posts/demo/demo_post1.md，在里边写点markdown内容。然后默认地hugo可用`localhost:1313/demo/demo_post1`来访问该文章，但发现浏览器显示`Page Not Found`……

[![pS0OnGq.md.png](https://s1.ax1x.com/2023/02/01/pS0OnGq.md.png)](https://imgse.com/i/pS0OnGq)

## 定义single页面显示逻辑
在`diy/layouts/_default/single.html`里填入以下代码：

```html
{{ define "main" }}
<main>
    <article>
    <header>
        <h1>{{ .Title }}</h1>
    </header>
    <aside>
        {{ .TableOfContents }}
    </aside>
        {{ .Content }}
    </article>
</main>
{{ end }}
```

这段代码是Go Template的语法，定义了一个叫做main的block（被上述的baseof.html里引用，可以回头去看下）。页面从上到下依次是标题、文章目录、文章内容。

重跑`hugo server -D`，再访问文章链接，发现还是`Page Not Found`……

这其实是因为没告诉hugo要使用diy主题，在博客根目录的`config.toml`里指定一下。这次，刷新一下页面就能看到文章内容了。
```toml
baseURL = 'http://example.org/'
languageCode = 'en-us'
title = 'My New Hugo Site'
theme = 'diy'
```

## 太丑了，换个样式
目前都是浏览器默认的样式，丑得无法接受。我试了一圈，发现[typora里的vue主题](https://theme.typora.io/theme/Vue/)深得本人喜爱。但如何引入这个样式呢？

首先下载vue主题的样式文件（此处为vue.css和vue目录，后者是一些字体样式），放到静态文件css目录下（`diy/static/css/`），然后需要在页面里引用这些css。

有过H5开发经验的会知道，css一般是放在`<header>`里头引用的，回头看baseof.html，发现它的`<header>`是引用了header.html（完整路径为`diy/layouts/partials/header.html`），所以我们在header.html里引用css即可：
```html
<link rel="stylesheet" type="text/css" href="/css/vue.css">
```

至于这里为啥href路径写的是`"/css/vue.css"`而非完整的路径，涉及到hugo的静态文件寻址规则，现阶段先接受即可，没必要深究。

## 定制文章目录（Table Of Contents）
仔细观察的话，会发现文章目录是从H2开始的，H1怎么就不见了呢？（注意，需要你用来测试的markdown文章里有H1、H2、H3才能看出效果）

阅读hugo文档，找到了配置TOC的方法，在博客根目录的`config.toml`里加上这一段即可。

```toml
[markup]
  [markup.tableOfContents]
    endLevel = 3
    ordered = false
    startLevel = 1
```
参考文档：https://gohugo.io/getting-started/configuration-markup/#table-of-contents

每个字段的含义都由它的名称解释得很清楚了，我设置了`startLevel = 1`，所以H1也就能显示出来了！

至于为啥没配置之前H1无法显示出来，目测是startLevel默认值为2。

可以看出这里是全局配置，那么有个遗留问题：如果部分文章不想从H1开始，可以做到吗？（后边再看看）

# 显示博客首页
参考文档：https://gohugo.io/templates/homepage/

在`diy/layouts/index.html`里填入以下代码：
```html
{{ define "main" }}
    <main aria-role="main">
      <header class="homepage-header">
        <h1>{{.Title}}</h1>
        {{ with .Params.subtitle }}
        <span class="subtitle">{{.}}</span>
        {{ end }}
      </header>
      <div class="homepage-content">
        <!-- Note that the content for index.html, as a sort of list page, will pull from content/_index.md -->
        {{.Content}}
      </div>
      <div>
        {{ range first 10 .Site.RegularPages }}
            {{ .Render "summary"}}
        {{ end }}
      </div>
    </main>
{{ end }}
```

但我发现渲染不出具体内容，查看`.Render`函数的文档：https://gohugo.io/functions/render/ ，发现它是这么说的：
> This example could render a piece of content using the content view located at /layouts/_default/summary.html

也就是说，需要在_default下建多一个summary.html才能渲染出来……咱们极简版就别折腾了，另寻他路。


## 初识“页面变量”
对于极简版首页，这里应该是一个博客列表，每篇显示title、摘要，然后点击title可以跳到博客详情页。

这几个信息都可以通过“Page Variable”来获取到，顾名思义很容易理解啦，就是hugo内置了一些变量来引用页面的各种信息，直接看实例吧：
- 文章title：.Title
- 文章摘要：.Summary
- 文章（相对）链接：.RelPermalink

参考文档：https://gohugo.io/variables/page/

## 自定义摘要
不过实际渲染效果，`.Summary`太多内容了（对于较长文章来说），再看看Summary的文档：https://gohugo.io/content-management/summaries/

有三种摘要方式（以下按优先级排序）：
1. 半自动模式，在文章里摘要、剩余部分之间的位置插入符号 &lt;!--more-->
2. 全手工模式，在文章的元信息部分（即Front Matter）里写summary内容，适合摘要内容跟文章开头不一样的情况。
3. 全自动模式，hugo按照配置项`summaryLength`来截取文章开头相应数量的单词作为摘要

其中1和2都可以人工按需指定，没啥问题。

剩下“全自动模式”，默认值是70个词，亲测分割出来有的会多很多字……无语了。
所以我加了一个函数来截断多余的部分，即下边代码里的`{{ .Summary | truncate 70 }}`，其中truncate是hugo提供的内置函数，然后`|`符号是hugo pipe功能，其实就跟Linux命令行的管道功能一样。这样truncate之后发现就正常了。

综上，首页的渲染模板代码可以这样子写：
```html
{{ define "main" }}
    <main aria-role="main">
      <header class="homepage-header">
        <h1>{{.Title}}</h1>
        {{ with .Params.subtitle }}
        <span class="subtitle">{{.}}</span>
        {{ end }}
      </header>
      <div class="homepage-content">
        <!-- Note that the content for index.html, as a sort of list page, will pull from content/_index.md -->
        {{.Content}}
      </div>
      <div>
        {{ range .Site.RegularPages }}
          <article>
            <!-- this <div> includes the title summary -->
            <div>
              <h2><a href="{{ .RelPermalink }}">{{ .Title }}</a></h2>
              {{ .Summary | truncate 70 }}
            </div>
            <div>
              <a href="{{ .RelPermalink }}">Read More…</a>
            </div>
          </article>
        {{ end }}
      </div>
    </main>
{{ end }}
```

效果如下：

[![pS0XojK.md.png](https://s1.ax1x.com/2023/02/01/pS0XojK.md.png)](https://imgse.com/i/pS0XojK)

# 总结
至此，极简版博客就搭建完成了，至少能用起来了，但其实存在很多不完善的地方。

后续会逐步完善、美化，敬请期待后续文章～