---
title: "Markdown 语法"
description: "Markdown 基础语法，快速入门。"
tags: ["markdown"]
date: 2024-09-22T20:11:03+08:00
image: "markdownsyntax-corver.png"
math: true
hidden: false
draft: true
---

## 标题

`# 内容`为标题，几个#代表几级标题。

# 一级标题

## 二级标题

### 三级标题

```markdown
# 一级标题

## 二级标题

### 三级标题
```

## 引用块

`> 内容`为引用块，用于引用内容。

> 引用块
>
> > 🪆 套娃引用块

```markdown
> 引用块
> >> 🪆 套娃引用块
```

## 代码块与代码段

`代码块`。

```markdown
代码段
```

````markdown
`代码块`。

# 其中“java”为代码块语言，可以替换。

```java
System.out.println("Hello, World!");
```
````

## 字体样式

### 斜体

`*斜体*`
_斜体_

### 粗体

`**粗体**`

**粗体**

### 删除线

`~~删除线内容~~`

~~删除线内容~~

### 下划线

`<u>下划线内容</u>`

<u>下划线内容</u>

### 文本高亮

`==内容==`

==内容==

### 文本上下标

我是文本^我是上标^

我是文本~我是下标~

```markdown
我是文本^我是上标^

我是文本~我是下标~
```

## 分隔符

```markdown
---
```

---

## 表格

最上一排就是表头，需要在左右使用|表示这是一个表格，同时，下方需要添加分割线，分割线使用-减号表示。

| 一   | 二   | 三   | 四   | 五   |
| ---- | ---- | ---- | ---- | ---- |
| 1    | 2    | 3    | 4    | 5    |

```markdown
| 一  | 二  | 三  | 四  | 五  |
| --- | --- | --- | --- | --- |
| 1   | 2   | 3   | 4   | 5   |
```

其中：

- `:---` 为左对齐
- `:---:` 为剧中对齐
- `---:` 为右对齐

|   一 |   二 |  三  | 四   | 五   |
| ---: | ---: | :--: | :--- | :--- |
|    1 |    2 |  3   | 4    | 5    |

```markdown
|  一 |  二 | 三  | 四  | 五  |
| --: | --: | :-: | :-- | :-- |
|   1 |   2 |  3  | 4   | 5   |
```

## 有序列表

输入当前序号后按退格键（Tab）可以变为子序号。

1. 一
   1. 二
      1. 三
         1. 四

```markdown
1. 一
   1. 二
      1. 三
         1. 四
```

## 无序列表

用法与有序列表类似。

- 一
  - 二
    - 三
      - 四

```markdown
- 一
  - 二
    - 三
      - 四
```

## 勾选框

- [ ] 我是未完成的任务
- [x] 我是已完成的任务

```markdown
- [ ] 我是未完成的任务
- [x] 我是已完成的任务
```

## 图片插入

html 写法
`<img src="图片路径" width=200 heigth=200>`

<img src="/Users/limincai/code/hugo-blog/content/post/markdown-syntax/markdownsyntax-corver.png" width=200 heigth=200>

markdown 写法，这种写法无法主动修改图片尺寸。

`![图片描述](图片地址 "图片名称，可选")`

![StarWars](/Users/limincai/code/hugo-blog/content/post/markdown-syntax/markdownsyntax-corver.png "123")

## 链接

`[链接文本](链接地址)`

[有问题请百度](https://baidu.com)

## 脚注

java[^1]是世界上最好的语言。

[^1]: 世界上最好的语言。

```markdwon
java[^1]是世界上最好的语言。
[^1]:世界上最好的语言。
```

## 数学公式

### 公式块

需要编写数学公式，我们同样需要在特定的块中编写，公式块使用`$`美元符表示。多行公式使用连续的两个美元符：
$$
我是公式
$$

```markdown
$$
我是公式
$$
```

如果只想在行内编写，一行内容只需要使用一个美元符囊括即可：

$ x = 17 + y $

```markdown
$ x = 17 + y $ 
```

### 特殊数学符号

|         代码         |      符号       |       描述       |
| :------------------: | :-------------: | :--------------: |
|        \not=         |     **\\**=     |      不等于      |
|       \approx        |        ≈        |      约等于      |
|        \times        |        ×        |       乘号       |
|         \div         |        ÷        |       除号       |
|         \leq         |        ≤        |     小于等于     |
|         \geq         |        ≥        |     大于等于     |
|         \pm          |        ±        |      正负号      |
|         \sum         |        ∑        | 求和符号（累加） |
|        \prod         |        ∏        |       累乘       |
|       \coprod        |        ∐        |       累除       |
| \overline{a + b + c} | *a*+*b*+*c* / 3 |      平均值      |

数学中常见特殊字符：

|   代码   | 符号 |  代码  | 符号 |
| :------: | :--: | :----: | :--: |
|  \alpha  | *α*  | \beta  | *β*  |
|  \gamma  | *γ*  | \delta | *δ*  |
| \epsilon | *ϵ*  |  \eta  |  η*  |
|  \theta  | *θ*  |  \pi   | *π*  |
|  \omega  | *ω*  |  \rho  | *ρ*  |
|  \sigma  | *σ*  |  \mu   | *μ*  |

常见的三角函数：

| 代码  | 符号 | 描述 |
| :---: | :--: | :--: |
| \sin  | sin  | 正弦 |
| \cos  | cos⁡  | 余弦 |
| \tan  | tan⁡  | 正切 |
| \cot  | cot⁡  | 余切 |
| \sec  | sec⁡  | 正割 |
| \csc  | csc⁡  | 余割 |
| \circ |  ∘   |  度  |

积分和求导相关：

|  代码   | 符号 |   描述   |
| :-----: | :--: | :------: |
| \infty  |  ∞   |   无穷   |
|  \int   |  ∫   |  定积分  |
|  \iint  |  ∬   | 双重积分 |
| \iiint  |  ∭   | 三重积分 |
|  \oint  |  ∮   | 曲线积分 |
| x\prime |  x′  |   求导   |
|  \lim   | lim⁡  |   极限   |

集合相关：

|   代码    | 符号 |  描述  |
| :-------: | :--: | :----: |
| \emptyset |  ∅   |  空集  |
|    \in    |  ∈   |  属于  |
|  \notin   |  ∉   | 不属于 |
|  \supset  |  ⊃   | 真包含 |
| \supseteq |  ⊇   |  包含  |
|  \bigcap  |  ⋂   |  交集  |
|  \bigcup  |  ⋃   |  并集  |

对数函数相关：

| 代码 | 符号 |        描述        |
| :--: | :--: | :----------------: |
| \log | log⁡  |      对数函数      |
| \ln  |  ln⁡  | 以e为底的对数函数  |
| \lg  |  lg⁡  | 以10为底的对数函数 |

### 分数

`$ \frac{分子}{分母} $`

$ \frac{分子}{分母} $

### 开方

`$\sqrt{4}$`

$\sqrt{4}$

如果需要修改根号上方数值，可以添加中括号。

`$\sqrt[3]{8}$`

$\sqrt[3]{8}$

### 上下标

`^` 表示上标，`_`表示下标。

$ x*_下标 $

 $ x^上标 $

 $ x^上标_*下标 $

~~~markdown
$ x_下标 $
$ x^上标 $
$ x^上标_下标 $
~~~

如果上标或下标内容多于一个字符，需要使用 {} 括起来，包括后续的其他代码如果出现只有一个字符生效的情况下，考虑使用花括号囊括全部内容：

$ x*_{下标} $ 

$ x^{上标} $ 

$ x^{上标}_*{下标} $

~~~markdown
$ x_{下标} $
$ x^{上标} $
$ x^{上标}_{下标} $
~~~

### 积分

`$\int_积分下限^积分上限xdx $*`

$\int_1^2xdx $*

### 极限

`$ \lim_{n\rightarrow+\infty}\frac{1}{n + 1} $*`

$ \lim_{n\rightarrow+\infty}\frac{1}{n + 1} $*

### 其他符号

|       代码       |     符号      |      描述      |
| :--------------: | :-----------: | :------------: |
|  `$ \vec{a} $`   |  $ \vec{a} $  |    向量符号    |
|   `$ \cdots $`   |  $ \cdots $   |   居中省略号   |
|   `$ \ldots $`   |  $ \ldots $   | 靠底部的省略号 |
|   `$ \cdot $`    |   $ \cdot $   |     点乘号     |
| `$ \sum*_1^n $*` | $ \sum_1^n $* |      累加      |

## html 标签

markdown 支持 html 标签，可以更加个性化的自定义内容。

例如：

````markdwon
<iframe src="https://limincai.github.io/" height="1080"
width="1080" sandbox="allow-scripts" scrolling="yes"></iframe
````

<iframe src="https://limincai.github.io/" height="1080" 
width="1080" sandbox="allow-scripts" scrolling="yes"></iframe```



ai.github.io/" allow-top-navigation="false" allow-forms="false" allowfullscreen="true" allow-popups="false" sandbox="allow-scripts allow-same-origin allow-popups" style="box-sizing: border-box; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / 0.5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; margin: 0px auto; max-width: 100%; width: 569px; border: medium;"></iframe>