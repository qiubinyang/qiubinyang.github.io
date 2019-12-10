# Hugo主题Bug修复


{{% admonition note %}}
目前使用的LoveIt主题是我好不容易在一大堆主题中找到了最满意的主题，不仅干净简洁优雅，而且能满足我程序员的一切博客要求，简直不要太好！但是夸归夸，由于这个主题非常新，截止目前，Github上也就刚刚发布一两个月不到，问题也非常的多，其中数学公式的问题尤为明显。不忍放弃这么优秀的框架，于是我决定修改源码，提交个pull requests给原作者。
{{% /admonition %}}

在LoveIt中，原作者使用Katex来支持博客的数学公式显示。Katex也是非常的优秀，曾经我在做《Latex实时公式编辑器时》选择了Katex作为项目核心。因为Katex相较于行业中的佼佼者MathJax来说，轻便了不止一点点，能使引用资源达到非常小，这对于前端开发者来说是非常重要的！

但是他也有致命缺点。
>Katex不**直接**支持行内公式显示。

## 具体方案
有两个修改方式：

1. 重新自定义Katex渲染
2. 换成MathJax

为了保持原作者的设计初衷，我决定通过自定义Katex渲染来解决问题。

## 重新自定义Katex渲染
### 在layouts/partials中增加math.html
```html
<script>
    document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
            delimiters: [
              {left: "$$", right: "$$", display: true},
              {left: "$", right: "$", display: false}
            ]
        });
    });
</script>
```

### 在layouts/partials/head.html最后引入math.html
>在head.html的最后加入以下代码
```html
{{ if or .Params.math .Site.Params.math }}
    {{ partial "math.html" . }}
{{ end }}
```
可通过全局math或文章头部math变量控制是否需要文章中公式的渲染。

## 换成MathJax
>当然，一开始为了方便我是换成了MathJax的，这里展示替换方法

### 在layouts/partials/scripts.html中把Katex替换为MathJax
>1. 在Scripts.html中第26行有一段代码为引入katex，直接去掉
```html
<!-- KaTeX https://github.com/KaTeX/KaTeX -->
{{ $katex_css := "" }}
{{ if eq (getenv "HUGO_ENV") "production" | and .Site.Params.cdn.katex_css }}
    {{ $katex_css = .Site.Params.cdn.katex_css }}
{{ else }}
    {{ $res := resources.Get "css/lib/katex/katex.min.css" | resources.Minify }}
    {{ $katex_css = printf "<link rel=\"stylesheet\" href=\"%s\">" $res.RelPermalink }}
{{ end }}
{{ $katex_js := "" }}
{{ if eq (getenv "HUGO_ENV") "production" | and .Site.Params.cdn.katex_js }}
    {{ $katex_js = .Site.Params.cdn.katex_js }}
{{ else }}
    {{ $res := resources.Get "js/lib/katex/katex.min.js" | resources.Minify }}
    {{ $katex_js = printf "<script src=\"%s\"></script>" $res.RelPermalink }}
{{ end }}
{{ $katex_auto_render_js := "" }}
{{ if eq (getenv "HUGO_ENV") "production" | and .Site.Params.cdn.katex_auto_render_js }}
    {{ $katex_auto_render_js = .Site.Params.cdn.katex_auto_render_js }}
{{ else }}
    {{ $res := resources.Get "js/lib/katex/auto-render.min.js" | resources.Minify }}
    {{ $katex_auto_render_js = printf "<script defer src=\"%s\" onload=\"renderMathInElement(document.body);\"></script>" $res.RelPermalink }}
{{ end }}
{{ $katex := delimit (slice $katex_css $katex_js $katex_auto_render_js) "" }}
```

>2. 将以上整体换为以下代码后，需要去config.toml中修改katex的cdn为mathjax的cdn，并且在js文件夹中加入MathJax的js：
```html
{{ $mathjax_js := "" }}
{{ if eq (getenv "HUGO_ENV") "production" | and .Site.Params.cdn.mathjax_js }}
    {{ $mathjax_js = .Site.Params.cdn.mathjax_js }}
{{ else }}
    {{ $res := resources.Get "js/lib/mathjax/mathjax.min.js" | resources.Minify }}
    {{ $mathjax_js = printf "<script src=\"%s\"></script>" $res.RelPermalink }}
{{ end }}
```

>2. 不过太麻烦了。。。以上是为了规范的将MathJax替代，如果仅仅为了方便，可以直接在math.html中写以下代码
```html
<script type="text/javascript" async src="https://cdn.bootcss.com/mathjax/2.7.3/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$','$'], ['\\(','\\)']],
    displayMath: [['$$','$$'], ['\[','\]']],
    processEscapes: true,
    processEnvironments: true,
    skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
    TeX: { equationNumbers: { autoNumber: "AMS" },
         extensions: ["AMSmath.js", "AMSsymbols.js"] }
  }
});
MathJax.Hub.Queue(function() {
    var all = MathJax.Hub.getAllJax(), i;
    for(i = 0; i < all.length; i += 1) {
        all[i].SourceElement().parentNode.className += ' has-jax';
    }
});
</script>
<style>
code.has-jax {
    font: inherit;
    font-size: 100%;
    background: inherit;
    border: inherit;
    color: #515151;
}
</style>
```
最后还需要在CSS中对这种特殊的MathJax进行样式处理，否则行内公式的显示会很奇怪。

### 在layouts/partials中增加math.html
```html
<script>
    document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
            delimiters: [
              {left: "$$", right: "$$", display: true},
              {left: "$", right: "$", display: false}
            ]
        });
    });
</script>
```

### 在layouts/partials/head.html最后引入math.html
>在head.html的最后加入以下代码
```html
{{ if or .Params.math .Site.Params.math }}
    {{ partial "math.html" . }}
{{ end }}
```
可通过全局math或文章头部math变量控制是否需要文章中公式的渲染。