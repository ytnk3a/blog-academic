+++
title = "Hugoをorg-modeから作る時のサンプルフォーマット"
author = ["YusukeTANAKA"]
description = "データサイエンスを学ぶ上で、必要な言語をまとめてブログで記載するために"
date = 2020-12-30
draft = false
toc = true
+++

## イントロダクション H1 {#イントロダクション-h1}


### 日本語のサンプル文章 H2 {#日本語のサンプル文章-h2}

表示の仕方の確認1 <- インデント


### 日本語のサンプル {#日本語のサンプル}

サンプルテキストとして、書いてみます。初めてのブログや技術メモを書くに
あたり、org-modeとHUGOを採用してみました。
慣れない点もありながら、色々と書いていきたいと思います。


## サンプルフォーマット {#サンプルフォーマット}


### 文字装飾 {#文字装飾}


#### 太字 {#太字}

**bold**


#### イタリック {#イタリック}

_italic_


#### 下線 {#下線}

<span class="underline">underline</span>


#### 取り消し線 {#取り消し線}

~~strikethrogh~~


#### コード {#コード}

`code`


#### 逐語 {#逐語}

`verbtim`


#### 上付き {#上付き}

hoge<sup>super</sup>


#### 下付き {#下付き}

hoge<sub>sub</sub>


#### ギリシャ文字 {#ギリシャ文字}

&alpha;


### リストやチェックボックスに関して {#リストやチェックボックスに関して}

-   [ ] TODO
-   DONE


### 数式 {#数式}


#### インライン {#インライン}

数式 \\(e = mc^{2} \\) を含んだ文章


#### ブロックでの表示 {#ブロックでの表示}

`\[ y = f(x) \]`
\\[ y = f(x) \\]

\begin{equation}
\label{eq:1}
C = W\log\_{2} (1+\mathrm{SNR})
\end{equation}

LaTeX formatted equation: \\(E = -J \sum\_{i=1}^N s\_i s\_{i+1}\\) written by `$E = -J \sum_{i=1}^N s_i s_{i+1}$`

LaTeX formatted equation: \\(E = -J \sum\_{i=1}^N s\_i s\_{i+1}\\) written
by `\(E = -J \sum_{i=1}^N s_i s_{i+1}\)`

{{% callout note %}}
A Markdown callout is useful for displaying notices, hints, or definitions to your readers.
{{% /callout %}}


## プログラミングのアウトプット {#プログラミングのアウトプット}


### R {#r}


#### 出力結果 {#出力結果}

{{< highlight R >}}
x <- 1:10
mean(sqrt(x))
{{< /highlight >}}


#### テーブル {#テーブル}

{{< highlight R >}}
head(iris)
{{< /highlight >}}

| 5.1 | 3.5 | 1.4 | 0.2 | setosa |
|-----|-----|-----|-----|--------|
| 4.9 | 3   | 1.4 | 0.2 | setosa |
| 4.7 | 3.2 | 1.3 | 0.2 | setosa |
| 4.6 | 3.1 | 1.5 | 0.2 | setosa |
| 5   | 3.6 | 1.4 | 0.2 | setosa |
| 5.4 | 3.9 | 1.7 | 0.4 | setosa |


#### 図の出力 {#図の出力}

{{< highlight R >}}
p <- plot(matrix(rnorm(100), ncol=2), type="b")
p
{{< /highlight >}}

{{< highlight R >}}
n <- 50
x <- seq(1, n)
a.true <- 3
print(a.true)
print(x)
{{< /highlight >}}

```text
[1] 3
 [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25
[26] 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50
```

{{< highlight R >}}
n <- 50
x <- seq(1, n)
a.true <- 3
b.true <- 1.5
y.true <- a.true + b.true * x
s.true <- 17.3
y <- y.true + s.true * rnorm(n)
out1 <- lm(y ~ x)
{{< /highlight >}}

{{< highlight R >}}
getwd()
{{< /highlight >}}

```text
[1] "/Users/yuu/Documents/blog_academic/content/org/factor-analysis-01"
```

```text

Call:
lm(formula = y ~ x)

Residuals:
   Min     1Q Median     3Q    Max
-36.88 -13.17  -0.41  15.92  31.92

Coefficients:
            Estimate Std. Error t value Pr(>|t|)
(Intercept)   5.4452     5.1461   1.058    0.295
x             1.4497     0.1756   8.254 9.12e-11 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 17.92 on 48 degrees of freedom
Multiple R-squared:  0.5867,	Adjusted R-squared:  0.5781
F-statistic: 68.13 on 1 and 48 DF,  p-value: 9.124e-11
```

{{< highlight R >}}
boxplot(islands)
{{< /highlight >}}

{{< figure src="/ox-hugo/test1.png" >}}

{{< highlight R "linenos=table, linenostart=1" >}}
boxplot(islands)
{{< /highlight >}}

{{< figure src="/ox-hugo/test1.png" >}}

{{< highlight R >}}

library("ggplot2")
ggplot(iris, aes(x = Sepal.Width, y = Sepal.Length, color = Species)) +
  geom_point()
{{< /highlight >}}

![](/ox-hugo/test2.png) <- inline


### Python {#python}

{{< highlight python >}}
def foo(x):
  if x>0:
    return x+1

  else:
    return x-1

return foo(5)
{{< /highlight >}}

{{< highlight python >}}

import matplotlib
import matplotlib.pyplot as plt
fig=plt.figure(figsize=(3,2))
plt.plot([1,3,2])
fig.tight_layout()

fname = 'myfig.png'
plt.savefig(fname)
# fname # return this to org-mode
{{< /highlight >}}

{{< figure src="/ox-hugo/myfig.png" >}}


### julia {#julia}

{{< highlight ess-julia >}}
println("Hello world!")
{{< /highlight >}}

{{< highlight ess-julia >}}
print(2+3)
{{< /highlight >}}

{{< highlight ess-julia >}}
using Plots
# plot some data
#plot([cumsum(rand(500) .- 0.5), cumsum(rand(500) .- 0.5)])
scatter(rand(100), markersize = 6, c = :red)
{{< /highlight >}}

{{< figure src="/ox-hugo/org-julia-random.png" >}}

{{< highlight ess-julia >}}
using Plots
plot([cumsum(rand(500) .- 0.5), cumsum(rand(500) .- 0.5)])
{{< /highlight >}}

{{< figure src="/ox-hugo/org-julia-random-02.png" >}}
