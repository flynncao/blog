---
title: 为什么CSS的元素难以对齐？
date: 2023-11-06T00:00:00.000+00:00
tags:
  - CSS
  - font
category: code
lang: zh
tocAlwaysOn: true
---

---

在很多年的模糊记忆后，我终于静下心来整理关于CSS Layout的思路。起因是今天在排布一些容易的盒子时浪费了我不少时间，表明我对CSS的概念仍然存在漏洞。在position系统已经不常用的今天，我们以`TailwindCSS`为基础来疏通一下一些概念和应用。

## Flexbox引发的思考

> 参考：<https://css-tricks.com/snippets/css/a-guide-to-flexbox/>

> The main idea behind the flex layout is to give the container the  ability to alter its items’ width/height (and order) to best fill the  available space (mostly to accommodate to all kind of display devices  and screen sizes)

flexbox是相对之前未出现的固定大小的容器（例如固定200px宽100px高的容器，那么很显然只能依据定位来排布元素，而现在Responsive Design已经成为行业标准的今天这一点早就不适用了）

TailwindCSS的另外一个好处是根据rem来确定了元素大小，这在调整，也让网页设计存在更好的发挥空间。

OK，那么rem又是什么？通过google我们可以轻易得出屏幕的默认字体大小是16px。

![image-20231106164134442](https://raw.githubusercontent.com/flynncao/blog-images/main/img/image-20231106164134442.png)

而TailwindCSS的基准1=0.25rem, 我们设这个单位符号为T，那么`1T=0.25*16px=4px`而 `4T=1rem`。也就是4个tailwindcss的单位才能得到一个标准的字体大小。我们修改每个盒子的宽度为4*3=12T，高度为`4*2=8T`应该每个盒子都能放下两行abc。注意，此处为了展示标准大小，我们需要加上`break-words`属性强行让用户看到不连续的单词，即字符根据盒子边界插入换行。

但结果和我们想象的不一样，盒子大小的确是`16*3=48`px，但这里不止三个英文字符了，换中文看看？

![image-20231106165106333](https://raw.githubusercontent.com/flynncao/blog-images/main/img/image-20231106165106333.png)

> 参考:
>
> <https://iamvdo.me/en/blog/css-font-metrics-line-height-and-vertical-align>
>
> <https://juejin.cn/post/6971673576017494053?from=search-suggest#heading-5>

### 行高和字体大小的CSS表现？

我们搬出字体审阅工具**fontforge**来查看具体单个字符的设计大小

TIMES NEW ROMAN字体如图：(em units=2048)

![image-20231106182446978](https://raw.githubusercontent.com/flynncao/blog-images/main/img/image-20231106182446978.png)

Catamaran的参数如图（em units=1000)

![image-20231107001516474](https://raw.githubusercontent.com/flynncao/blog-images/main/img/image-20231107001516474.png)

如果是Windows系统，请使用`win ascent`, `win descent`, `typo line gap`三个参数，如果是Mac系统请使用`HHead Ascent`, `HHead Descent` 和`HHead Line Gap`三个参数来表示下文所说的`ascent`, `descent`, `line gap`三个数值。

这里有一个重要误解（稍后笔者会对其他软件进行测试）：CSS中的line-height并不等于两个字体间baseline的距离。

![image-20231106203734020](https://raw.githubusercontent.com/flynncao/blog-images/main/img/image-20231106203734020.png)

而根据大多数的字体测试，content-area(即纯粹字体会被染色的背景区域，非字体本身上下框线)是多于字体标准大小（如这里的16px）的。

因此line-height:1会导致；而`line-height:normal`在fontforge软件中`line gap`为0的情况下，使得`line-height`=`content-area`。这时候如果没有其他阻碍，外层盒子的高度就是`line-height`。

> 如果是line-gap不为0的字体（非常少见），line-gap的高度会被平均地添加到两行染色区域之间的间隙，导致line-height和content-area不等。

到今天为止，通过生成和字体同大小的图标，我们也无需费心解决图标和文字的对齐问题。在[UnoCSS](https://github.com/unocss/unocss)加持下，只需要一两行代码：

```html
<div flex text-100px>
    <div i-carbon-logo-github />
    <p>Majimay</p>
</div>
```

![image-20231106232522276](https://raw.githubusercontent.com/flynncao/blog-images/main/img/image-20231106232522276.png)

（按baseline对齐)
