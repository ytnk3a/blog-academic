+++
title = "因子分析に関して備忘録"
author = ["YusukeTANAKA"]
description = "データサイエンスを学ぶ上で、必要な言語をまとめてブログで記載するために"
date = 2021-01-25
draft = false
toc = true
+++

## はじめに {#はじめに}

因子分析に関しては、多くの文献がネット上にも存在しているが、整理のためにメモとして書いておく。


### 因子分析とは {#因子分析とは}

多変量解析の一種であり、多数の変数間の相関関係をもとにその観測結果がどのような潜在要因から生成されているかを
説明するための統計モデルである。心理学でよく用いられているイメージであるが、マーケティングにおいてもアンケート結果に
対して用いられ、インサイトや潜在的なニーズの発見などに用いられる。
また、次元圧縮・削減の一種であることから、セグメンテーションを行う際に因子分析した結果を用いてクラスタリングを
行うこともある。

因子分析には探索的因子分析 / 確認的因子分析の2種類がある。Rでは、各々の分析を行うためのパッケージが異なったりするので、
今回は探索的因子分析のみの備忘録とする。


### Rで行う場合のパッケージとデータ {#rで行う場合のパッケージとデータ}

Rでは、 `stats::factonal()` 関数や `psych:fa()` 関数を用いて分析を行うことが可能である。
使用できる回転方法を増やすために `GPArotation` , 観測データのタイプによって、相関係数を使い分けるために、
`polycor` などのパッケージも併用する。

{{< highlight R >}}
options(crayon.enabled = FALSE) # org-modeでANSIコードが対応していないため、追記
library(tidyverse) # データ加工、ggplot2など
library(psych)
library(GPArotation)
library(polycor)
{{< /highlight >}}


### サマリーコード / 分析の手順 {#サマリーコード-分析の手順}

因子分析は基本的に

1.  因子抽出の適切性の確認
2.  相関係数行列の作成
3.  因子数の決定
4.  回転方法の決定
5.  分析・解釈

のように進める。（当然、仮説の構築も必要なので、作業レベルでの手順）

Rでは分析だけであれば、10行未満のコードで行うことができるが、データの加工や結果の考察などを
行うための可視化を行なったりするので、その辺りも備忘録として残す。
(サンプルデータは `psych` に内包されている `bfi` を用いる)

{{< highlight R >}}
# --- data import
d <- bfi[1:25]
head(d)

# --- convert data frame to correlation matrix
d_cor <- cor(d, use = "complete.obs")

# --- assumption check
KMO(d_cor)

# --- the num of factor
fa.parallel(d_cor, n.obs = nrow(d)) # インポートデータで相関行列の場合は
VSS(d_cor, n.obs = nrow(d))         # 観測数 n.obsの指定が必要。

# --- factor analysis
res_fa <- fa(d, nfactors = 5, n.obs = nrow(d), fm = "ml")

# --- check result
summary(res_fa)
{{< /highlight >}}


## 分析フロー {#分析フロー}


### 相関行列への変換に関して {#相関行列への変換に関して}

因子分析は変数間の相関を元に因子を抽出する分析手法であるため、変数の尺度によって算出する相関係数を変えなければならない。
共に量的な尺度である場合は一般的なピアソンの相関係数を用いるが、アンケートなどの場合は5段階尺度や01の回答結果なども
あるため、相関係数の選定も重要になってくる。心理学では7段階尺度以上の場合は量的な尺度とみなすため、
それ以下の尺度の場合の相関係数に関しても下記に列挙しておく。[^fn:1]

-   質的な変数同士であり、片方の変数のカテゴリの数が3種類以上である場合 → ポリコリック相関係数 `polycor::polychor()`
-   質的な変数同士であり、両方の変数のカテゴリ数が2種類である場合 → テトラコリック相関係数 `psych::tetrachoric()`
-   質的な変数と量的な変数の組み合わせで、質的な変数が2つのとき → バイシリアル相関係数 `psych::biserial()`
-   質的な変数と量的な変数の組み合わせで、質的な変数が3つ以上のとき → ポリシリアル相関係数 `polycor::polyserial`

<!--listend-->

{{< highlight R >}}
d <- bfi[1:25]
d_cor <- cor(d, use = "complete.obs")
d_cor[1:5, 1:5]
{{< /highlight >}}

```text
           A1         A2         A3         A4         A5
A1  1.0000000 -0.3509055 -0.2736361 -0.1567536 -0.1926976
A2 -0.3509055  1.0000000  0.5030411  0.3508562  0.3974002
A3 -0.2736361  0.5030411  1.0000000  0.3849176  0.5156787
A4 -0.1567536  0.3508562  0.3849176  1.0000000  0.3256442
A5 -0.1926976  0.3974002  0.5156787  0.3256442  1.0000000
```

`cor()` 関数の引数 `use` に関して&nbsp;[^fn:2]

`"everything"`
: （デフォルト）全ての組み合わせを使うため、NAがあると計算結果でNAを返す

`"all.obs"`
: 組み合わせが完全でない場合エラーを返す

`"complete.obs"`
: NAがある組み合わせは除外して完全な組み合わせだけで計算する

`"na.or.complete"`
: 完全な組み合わせがない場合はNAを返す

`"pairwise.complete.obs"`
: 3変数以上の場合に使用可。1つの変数にNAがあっても、残りの変数で相関を計算する。


### 因子抽出の適切性 {#因子抽出の適切性}

因子分析の実行においてサンプル数は300以上が推奨されるが、最低でも分析対象変数の10倍程度は確保することが望ましい。[^fn:3]
サンプル数がある程度確保された後は、分析対象データセットから因子を抽出してもいいかの適切性の確認を分析を行う前に行なった方がいい。
因子分析における適切性を判断する方法としてバートレットの球面性検定・カイザー・マイヤー・オルキンのサンプリングの適切性指標・
反イメージ相関行列などがあるみたいだが、ここではカイザー・マイヤー・オルキンのサンプリングの適切性指標に関してメモしておく。
Rでは下記のように `KMO()` 関数で求めることができる。

{{< highlight R >}}
KMO(d_cor)
{{< /highlight >}}

```text
Kaiser-Meyer-Olkin factor adequacy
Call: KMO(r = d_cor)
Overall MSA =  0.85
MSA for each item =
  A1   A2   A3   A4   A5   C1   C2   C3   C4   C5   E1   E2   E3   E4   E5   N1
0.75 0.84 0.87 0.88 0.90 0.84 0.80 0.85 0.83 0.86 0.84 0.88 0.90 0.88 0.89 0.78
  N2   N3   N4   N5   O1   O2   O3   O4   O5
0.78 0.86 0.89 0.86 0.86 0.78 0.84 0.77 0.76
```

結果の見方は

Overall MSA
: 変数全体のサンプリング適切性指標

MSA for each item
: 個々の変数間のサンプリング適切性指標

のように全体と個々の変数における適切性を一度に見ることができる。

規準値は

| KMO       | 判定 　　　  | 使用有無 |
|-----------|---------|------|
| 0.9 ~ 1.0 | marvelous    | 使用可 |
| 0.8 ~ 0.9 | meritorious  | 使用可 |
| 0.7 ~ 0.8 | midding      | 使用可 |
| 0.6 ~ 0.7 | mediocre     | 使用可 |
| 0.5 ~ 0.6 | miserable    | 使用不可 |
| ~ 0.5     | unacceptable | 使用不可 |

のように示されており、KMOが0.6未満の場合は使わない方が良いとされる。[^fn:4] <sup>, </sup>[^fn:5]
KMOはそれぞれの変数間における相関係数と偏相関係数の比で求められる。
KMOが小さいということは、2変数間の相関関係が他の変数によって、説明できないということを表すので、
因子を抽出するには適切でないということを表す.

計算式はのように計算される。[^fn:4] 変数$i$と変数$j$の間の相関係数を \\(r\_{ij}\\) 、偏相関係数を \\(a\_{ij}\\) とする。

-   Overall MSA

\\[  KMO = \frac{\sum \sum\_{i \neq j} r\_{ij}^2 }{ \sum \sum\_{i \neq j} r^2\_{ij}+ \sum \sum\_{i \neq j} a^2\_{ij}} \\]

-   MSA for each item

\\[  MSA\_i = \frac{\sum\_{i \neq j} r\_{ij}^2 }{ \sum\_{i \neq j} r^2\_{ij} + \sum\_{i \neq j} a^2\_{ij} } \\]


### 因子数の決定 {#因子数の決定}

因子分析を行う上で、因子数を決めることが、必須となってくる。
因子数を選ぶ基準としては、カイザーガットマン基準・スクリー基準などが一般的に用いられているが、
今後は平行分析、MAP基準、情報量基準によって、因子数を決めることが推奨されている[^fn:6]ので、
平行分析とMAP基準に関して残す。

平行分析
:

データと同じサイズの乱数データを作成し、その乱数データの相関行列の固有値と比較し、
対象データの因子数を決める方法。乱数データの固有値より大きい固有値を因子数とする。

{{< highlight R >}}
fa.parallel(d_cor, n.obs = nrow(d))
{{< /highlight >}}

```text
Parallel analysis suggests that the number of factors =  6  and the number of components =  5
```

{{< figure src="/ox-hugo/fa_para.png" >}}

点線が乱数データによる固有値をプロットであるため、実線との交点より左にあるなかで最大の因子数を採用する。

MAP（Minimum Average Partial）基準
:

最も効率的に相関行列を説明できる因子数を提案するため、最小の因子数を提案する。
主成分分析で生成される主成分を利用して因子数を決定する方法。
主成分を観測変数に共通して影響を与える変数とし、この変数を因子分析における因子とする。
この変数の影響を取り除いた上で、観測変数の相関（＝偏相関）を求める。この偏相関を全ての係数の組み合わせ分2乗して、平均を求める。
この偏相関が小さいということは、共通部分が多いということなので、この2乗平均の値を最も小さくなるように、
主成分の数を求める手法.

Rでは `VSS()` 関数を用いることで、MAP基準による因子数の推奨を得られることができる。

{{< highlight R >}}
VSS(d_cor, n.obs = nrow(d), rotate = "promax")
{{< /highlight >}}

```text

Very Simple Structure
Call: vss(x = x, n = n, rotate = rotate, diagonal = diagonal, fm = fm,
    n.obs = n.obs, plot = plot, title = title, use = use, cor = cor)
VSS complexity 1 achieves a maximimum of 0.6  with  4  factors
VSS complexity 2 achieves a maximimum of 0.7  with  5  factors

The Velicer MAP achieves a minimum of 0.01  with  5  factors
BIC achieves a minimum of  -500.45  with  8  factors
Sample Size adjusted BIC achieves a minimum of  -93.75  with  8  factors

Statistics by number of factors
  vss1 vss2   map dof chisq     prob sqresid  fit RMSEA   BIC SABIC complex
1 0.50 0.00 0.025 275 12256  0.0e+00      26 0.50 0.125 10073 10947     1.0
2 0.55 0.58 0.019 251  7670  0.0e+00      22 0.58 0.103  5678  6475     1.2
3 0.57 0.65 0.018 228  5273  0.0e+00      18 0.65 0.089  3463  4188     1.2
4 0.60 0.69 0.016 206  3488  0.0e+00      16 0.69 0.075  1853  2508     1.3
5 0.54 0.70 0.015 185  1770 1.9e-256      15 0.71 0.055   301   889     1.4
6 0.53 0.61 0.016 165  1054 1.7e-129      16 0.69 0.044  -255   269     1.7
7 0.51 0.59 0.019 146   726  1.1e-77      17 0.68 0.038  -433    31     1.7
8 0.47 0.61 0.022 128   516  6.1e-48      17 0.68 0.033  -500   -94     1.7
  eChisq  SRMR eCRMS  eBIC
1  24346 0.120 0.126 22163
2  12661 0.087 0.095 10669
3   7204 0.065 0.075  5395
4   3597 0.046 0.056  1962
5   1301 0.028 0.035  -167
6    635 0.019 0.026  -674
7    427 0.016 0.023  -732
8    274 0.013 0.020  -742
```

{{< figure src="/ox-hugo/fa_vss.png" >}}

出力中段 `The Velicer MAP achieves a minimum of 0.01  with  5  factors` より、MAP基準を用いた場合は、
因子数は5を推奨されている。

`VSS()` では、MAP基準だけでなく、複数の基準の結果を算出してくれるため、各基準の説明をヘルプより下記に列挙する。

-   map: Velicer's MAP values (lower values are better)
-   dof: degrees of freedom (if using FA)
-   chisq: chi square (from the factor analysis output (if using FA)
-   prob: probability of residual matrix > 0 (if using FA)
-   sqresid: squared residual correlations
-   RMSEA: the RMSEA for each number of factors
-   BIC: the BIC for each number of factors
-   eChiSq: the empirically found chi square
-   eRMS: Empirically found mean residual
-   eCRMS: Empirically found mean residual corrected for df
-   eBIC: The empirically found BIC based upon the eChiSq
-   fit: factor fit of the complete model

平行分析はやや多めの因子数を提案することが多いため、これ以上の因子数は採用すべきではない。
また、MAP基準は少なめの因子数を提案することが多いため、これ以下の因子数は採用すべきではない。
平行分析の因子数=MAP基準の因子数の場合は、その因子数を採用すべきであるが、一致しない場合は、
平行分析の因子数 ~ MAP基準の因子数の間を、仮説と照らし合わせながら、因子数を決定すればよい。


### 回転方法 {#回転方法}

因子分析を行う上で、もう一つ決める必要があるのが、回転方法である。 `GPArotaion` を用いることで、
多くの回転を試すことができるが、大きく分けて、直行回転と斜行回転の2種類がある。
Rの `fa()` 関数では、引数 `rotate` で指定することができる。

{{< highlight R >}}
res_fa <- fa(d_cor, nfactors = 5, n.obs = nrow(d), fm = "ml", rotate = "promax")
{{< /highlight >}}

S
回転方法に関しては、因子負荷をなるべく単純構造となるように選ぶことが望ましい。
回転の大枠に関しては、分析によって求められる因子が独立であるときは直行回転を使い、
因子間相関を考慮するときは斜行回転を使用する。因子が完全に独立であるということは、
あまり考えられないため、心理学やアンケートなどの分析においては斜行回転を用いることが多い。
クラスター分析における次元圧縮の場合は、軸が独立である場合の方がMECEであると考えられるので、
直行回転を用いる方が良いのではと、個人的には思うが、斜行回転を使った方が単純構造になりやすい。
つまり、データが合っていない場合は直行回転の場合、複雑な構造になってしまう。

`fa()` 関数のデフォルトもオブリミン回転が採用されている。

斜行回転においての選び方であるが、基本的にはプロマックス回転で問題がない。[^fn:7]
また、プロマックス回転は、因子負荷のバランスが悪く、最後の因子には少数の負荷しない構造になることもあるみたいなので、
ただ、仮説・解釈上違和感がある場合などは、他の回転を試せばよい。

構造をより単純にしたい場合は独立クラスター回転、複雑にしたい場合はジェオミン回転あたりを試してみる。


### 結果（解釈・精度） {#結果-解釈-精度}

因子分析の結果をイメージとして図化するときは `fa.diagram()` 関数を使ってグラフ化するとわかりやすい。
`cut` 引数で指定された値以下の矢印を消失させることができる。因子負荷量を数字として記載される。

{{< highlight R >}}
fa.diagram(res_fa, cut = 0.3)
{{< /highlight >}}

{{< figure src="/ox-hugo/fa-diagram.png" >}}

{{< highlight R >}}
res_fa
{{< /highlight >}}

```text
Factor Analysis using method =  ml
Call: fa(r = d_cor, nfactors = 5, n.obs = nrow(d), rotate = "promax",
    fm = "ml")
Standardized loadings (pattern matrix) based upon correlation matrix
     ML2   ML1   ML3   ML5   ML4   h2   u2 com
A1  0.14  0.12  0.04 -0.43 -0.03 0.17 0.83 1.4
A2  0.05  0.11  0.07  0.58 -0.01 0.42 0.58 1.1
A3  0.04  0.21  0.01  0.63 -0.01 0.53 0.47 1.2
A4 -0.02  0.10  0.19  0.43 -0.18 0.31 0.69 1.9
A5 -0.10  0.29 -0.04  0.54  0.02 0.49 0.51 1.6
C1  0.06 -0.05  0.55  0.00  0.17 0.34 0.66 1.2
C2  0.14 -0.11  0.67  0.07  0.07 0.43 0.57 1.2
C3  0.03 -0.10  0.59  0.08 -0.06 0.32 0.68 1.1
C4  0.14  0.04 -0.67  0.05 -0.04 0.49 0.51 1.1
C5  0.18 -0.09 -0.57  0.02  0.10 0.44 0.56 1.3
E1 -0.06 -0.65  0.15 -0.01 -0.03 0.37 0.63 1.1
E2  0.12 -0.70  0.04 -0.02 -0.01 0.55 0.45 1.1
E3  0.08  0.49 -0.06  0.21  0.26 0.44 0.56 2.0
E4 -0.03  0.62 -0.05  0.26 -0.11 0.53 0.47 1.4
E5  0.16  0.49  0.23 -0.01  0.17 0.41 0.59 2.0
N1  0.88  0.23  0.03 -0.26 -0.10 0.73 0.27 1.4
N2  0.84  0.16  0.05 -0.24 -0.04 0.66 0.34 1.3
N3  0.72  0.00 -0.01 -0.01 -0.02 0.52 0.48 1.0
N4  0.49 -0.33 -0.10  0.07  0.09 0.49 0.51 2.0
N5  0.50 -0.16  0.03  0.15 -0.16 0.34 0.66 1.6
O1  0.01  0.16  0.02  0.01  0.51 0.33 0.67 1.2
O2  0.17  0.03 -0.07  0.14 -0.47 0.26 0.74 1.5
O3  0.04  0.26 -0.05  0.06  0.60 0.48 0.52 1.4
O4  0.14 -0.26 -0.02  0.17  0.37 0.25 0.75 2.6
O5  0.09  0.03 -0.03  0.05 -0.52 0.27 0.73 1.1

                       ML2  ML1  ML3  ML5  ML4
SS loadings           2.67 2.47 2.01 1.90 1.53
Proportion Var        0.11 0.10 0.08 0.08 0.06
Cumulative Var        0.11 0.21 0.29 0.36 0.42
Proportion Explained  0.25 0.23 0.19 0.18 0.14
Cumulative Proportion 0.25 0.49 0.68 0.86 1.00

 With factor correlations of
      ML2   ML1   ML3  ML5  ML4
ML2  1.00 -0.28 -0.23 0.01 0.04
ML1 -0.28  1.00  0.39 0.34 0.15
ML3 -0.23  0.39  1.00 0.24 0.21
ML5  0.01  0.34  0.24 1.00 0.18
ML4  0.04  0.15  0.21 0.18 1.00

Mean item complexity =  1.4
Test of the hypothesis that 5 factors are sufficient.

The degrees of freedom for the null model are  300  and the objective function was  7.48 with Chi Square of  20868.91
The degrees of freedom for the model are 185  and the objective function was  0.62

The root mean square of the residuals (RMSR) is  0.03
The df corrected root mean square of the residuals is  0.04

The harmonic number of observations is  2800 with the empirical chi square  1375.84  with prob <  6.7e-181
The total number of observations was  2800  with Likelihood Chi Square =  1714.56  with prob <  1e-245

Tucker Lewis Index of factoring reliability =  0.879
RMSEA index =  0.054  and the 90 % confidence intervals are  0.052 0.057
BIC =  246.14
Fit based upon off diagonal values = 0.98
Measures of factor score adequacy
                                                   ML2  ML1  ML3  ML5  ML4
Correlation of (regression) scores with factors   0.93 0.91 0.89 0.87 0.84
Multiple R square of scores with factors          0.87 0.82 0.79 0.76 0.70
Minimum correlation of possible factor scores     0.74 0.65 0.57 0.52 0.41
```

出力結果の見方に関して&nbsp;[^fn:8]

`Standardized loadings (pattern matrix) based upon correlation matrix`
: -   ML(Maximum Likelihood):
    -   h2: 共通性と呼ばれる因子によって説明された変数の分散の量
    -   u2: 独自性と呼ばれる値で、共通因子によって説明できない部分$1 - h2$で計算される。
    -   com: 因子負荷量の複雑性を表す。1に近いとき、この項目は1つの因子への負荷があり、2近い時は2因子への負荷しているということを示す。
        `Mean item complexity =  1.4` のように複雑性の全体平均も結果下部に出力されている。


`SS loadings`
:

<!--listend-->

```text
                       ML2  ML1  ML3  ML5  ML4
SS loadings           2.67 2.47 2.01 1.90 1.53
Proportion Var        0.11 0.10 0.08 0.08 0.06
Cumulative Var        0.11 0.21 0.29 0.36 0.42
Proportion Explained  0.25 0.23 0.19 0.18 0.14
Cumulative Proportion 0.25 0.49 0.68 0.86 1.00
```

-   SS loadings: 因子負荷量平方和
-   Proportion Var: 寄与率（全変数のうち、この因子で説明できている分散の割合）
-   Cumulative Var: 累積寄与率
-   Proportion Explained: 説明率（寄与率/寄与率の合計で計算される相対的な量）
-   Cumulative Proportion: 累積説明率

<!--listend-->

`With factor correlations of`
:

<!--listend-->

```text
 With factor correlations of
      ML2   ML1   ML3  ML5  ML4
ML2  1.00 -0.28 -0.23 0.01 0.04
ML1 -0.28  1.00  0.39 0.34 0.15
ML3 -0.23  0.39  1.00 0.24 0.21
ML5  0.01  0.34  0.24 1.00 0.18
ML4  0.04  0.15  0.21 0.18 1.00
```

因子間相関行列を表す。今回は斜行回転を使っているので、各因子間においても相関がみられる。

Model Results & Fit Index
:

<!--listend-->

```text
The degrees of freedom for the null model are  300  and the objective function was  7.48 with Chi Square of  20868.91
The degrees of freedom for the model are 185  and the objective function was  0.62

The root mean square of the residuals (RMSR) is  0.03
The df corrected root mean square of the residuals is  0.04

The harmonic number of observations is  2800 with the empirical chi square  1375.84  with prob <  6.7e-181
The total number of observations was  2800  with Likelihood Chi Square =  1714.56  with prob <  1e-245

Tucker Lewis Index of factoring reliability =  0.879
RMSEA index =  0.054  and the 90 % confidence intervals are  0.052 0.057
BIC =  246.14
Fit based upon off diagonal values = 0.98
```

-   `The degrees of freedom for`: 自由度
-   `objective function`: 分析に用いた推定方法によって得られる最小の値。モデルの比較用に用いる
-   `Chi Square`:（調べ中）
-   `The root mean square of the residuals (RMSR)`: 残差の二乗平均平方根
-   `The df corrected root mean square of the residuals`:RMSRの自由度で修正したもの
-   `The harmonic number of observations`:（調べ中）
-   `Tucker Lewis Index of factoring reliability`:TLI指数。1に近いほどよく、0.9以上が望ましい
-   `RMSEA index`:Root Mean Square Error of Approximation 近似平均平方誤差平方根
-   `BIC`:BIC基準。モデル比較に用いられ、相対的に小さい方がよい。
-   `Fit based upon off diagonal values`: (調べ中)

<!--listend-->

`Measures of factor score adequacy`
:

<!--listend-->

```text
Measures of factor score adequacy
                                                   ML2  ML1  ML3  ML5  ML4
Correlation of (regression) scores with factors   0.93 0.91 0.89 0.87 0.84
Multiple R square of scores with factors          0.87 0.82 0.79 0.76 0.70
Minimum correlation of possible factor scores     0.74 0.65 0.57 0.52 0.41
```

-   `Correlation of (regression) scores with factors` : 因子とデータの相関であり、因子スコアの推定値の上限として
-   `Multiple R square of scores with factors` : 多重相関の二乗値
-   `Minimum correlation of possible factor scores` :（調べ中）

また、因子負荷量を棒グラフなどで比較したい時は `ggplot2` を使って、
相関行列と共に、並べるなどを行うとわかりやすいと思う。[^fn:9]

{{< highlight R >}}
res_fa_load <- unclass(res_fa$loadings) %>%
  as_tibble() %>%
  mutate(item = row.names(unclass(res_fa$loadings)))

item_lv <- res_fa_load %>%
  pivot_longer(cols = -item, names_to = "factor", values_to = "load") %>%
  group_by(item) %>%
  arrange(desc(abs(load))) %>%
  mutate(
    row_id = row_number()
  ) %>%
  filter(row_id == 1) %>%
  arrange(factor) %>%
  select(item)

fac_lv <- item_lv %>%
  pull(item)

d_load <- res_fa_load %>%
  mutate(
    item = fct_rev(factor(item, levels = fac_lv))
    ) %>%
  pivot_longer(cols = -item, names_to = "factor", values_to = "loadings")

p_load <- ggplot(d_load, aes(item, abs(loadings), fill = loadings)) +
    facet_wrap(~ factor, nrow = 1) + #place the factors in separate facets
    geom_bar(stat="identity") + #make the bars
    coord_flip() + #flip the axes so the test names can be horizontal
    scale_fill_gradient2(name = "Loading",
			 high = "blue", mid = "white", low = "red",
			 midpoint = 0, guide=F) +
    ylab("Loading Strength") + #improve y-axis label
    xlab("") +
    theme_light(base_size = 10)
{{< /highlight >}}

{{< figure src="/ox-hugo/fa-loading-bar.png" >}}

{{< highlight R >}}
d_cor02 <- d_cor %>%
  as_tibble() %>%
  mutate(item = row.names(d_cor)) %>%
  pivot_longer(cols = -item, names_to = "item2", values_to = "corr") %>%
  mutate(
    item = fct_rev(factor(item, levels = fac_lv)),
    item2 = fct_rev(factor(item2, levels = fac_lv))
  )

p_cor <- ggplot(d_cor02, aes(item2, item, fill = abs(corr))) +
  geom_tile() +
  geom_text(aes(label = round(corr, 2)), size=2.5) +
  theme_minimal(base_size=10) +
  theme(axis.text.x = element_text(angle = 90),
	axis.title.x=element_blank(),
	axis.title.y=element_blank(),
	plot.margin = unit(c(3, 1, 0, 0), "mm")) +
  scale_fill_gradient2(low="blue", mid = "white", high="red", midpoint = 0.4,) +
  guides(fill=F)

p_cor
{{< /highlight >}}

{{< figure src="/ox-hugo/fa-correlation.png" >}}


### 因子得点の推定に関して {#因子得点の推定に関して}

`fa()` 関数のデフォルトはregression(Thurstone)であるが、この手法は直行回転の場合でも得点が相関することがあるらしい。[^fn:10]
そのため、直行回転においては、Anderson-Rubin Methodのように推定する因子の直行性を確保するよる手法を用いた方がいい。
斜行回転の場合は回帰式のregressionも選択肢としてあるが、
tenBergeは、因子間の相関を保存してスコアを計算する[^fn:11]ため、斜行回転においては、この方法を用いるのが良いと思う。

この記事([Exploratory Factor Analysis](http://www.alexanderdemos.org/Class15.html#ten%5Fberge%E2%80%99s))によると、バイアスを補正すると書いてあるが、どのようなバイアスかは未確認。

{{< highlight R >}}
fa_score <- factor.scores(d, res_fa, method = "tenBerge")
print("fa score")
head(fa_score$scores)
print("fa correlation")
fa_score$r.scores
{{< /highlight >}}

```text
[1] "fa score"
              ML2        ML1         ML3         ML5        ML4
61617 -0.39536359 -0.1344741 -1.45558909 -1.14186339 -1.9938560
61618  0.06724783  0.3588507 -0.62172235 -0.12122831 -0.1987675
61620  0.58584696  0.1894743 -0.03449101 -0.83702986  0.2351075
61621 -0.08068938 -0.1506722 -1.19310112  0.02796202 -1.1592495
61622 -0.50033008  0.3536558 -0.07296858 -0.86566860 -0.8029520
61623  0.07415292  1.3699613  1.65022598  0.14722401  0.5689176
[1] "fa correlation"
            ML2        ML1        ML3        ML5        ML4
ML2  1.00000000 -0.2790905 -0.2339293 0.01118896 0.04456613
ML1 -0.27909046  1.0000000  0.3888799 0.33670761 0.14818602
ML3 -0.23392931  0.3888799  1.0000000 0.23576061 0.20694024
ML5  0.01118896  0.3367076  0.2357606 1.00000000 0.18372528
ML4  0.04456613  0.1481860  0.2069402 0.18372528 1.00000000
```


## Appendix {#appendix}


### 用語集 {#用語集}

因子負荷量
:

潜在因子が観測変数に与える影響の強さを表す値で、観測変数と因子得点の相関係数に相当する。
潜在因子と観測変数の間に強い相関があるとき、負荷量の値も大きくなる。[^fn:12]


### 参考文献 {#参考文献}

この記事を書くにあたり、参考にした書籍、サイトを下記に記すが、漏れがあった場合は都度追記する。


#### 書籍 {#書籍}

-   [因子分析入門―Rで学ぶ最新データ解析― | 豊田 秀樹, 豊田 秀樹 |本 | 通販 | Amazon](https://www.amazon.co.jp/%E5%9B%A0%E5%AD%90%E5%88%86%E6%9E%90%E5%85%A5%E9%96%80%E2%80%95R%E3%81%A7%E5%AD%A6%E3%81%B6%E6%9C%80%E6%96%B0%E3%83%87%E3%83%BC%E3%82%BF%E8%A7%A3%E6%9E%90-%E8%B1%8A%E7%94%B0-%E7%A7%80%E6%A8%B9/dp/4489021267)


#### Web {#web}

-   [因子分析の教科書における因子数の選択についての記述 - Takahiro ONOSHIMA's website](https://onoshima.github.io/stat/factornumber/)
-   [因子分析におけるMAP基準について - Takahiro ONOSHIMA's website](https://onoshima.github.io/stat/vss/)
-   [いきなり因子分析（その1） - 猫も杓子も構造化](http://nekomosyakushimo.hatenablog.com/entry/2018/07/04/094944)
-   [いきなり因子分析（その2）：MAPやBICや平行分析による因子数の決定 - 猫も杓子も構造化](http://nekomosyakushimo.hatenablog.com/entry/2018/07/05/082620)
-   [探索的因子分析 - tomokoba website](https://tobayash.github.io/tomokobablog/2020/01/01/%E6%8E%A2%E7%B4%A2%E7%9A%84%E5%9B%A0%E5%AD%90%E5%88%86%E6%9E%90/)
-   [Rと因子分析](https://www1.doshisha.ac.jp/~mjin/R/Chap%5F25/25.html)
-   [Exploratory Factor Analysis](http://www.alexanderdemos.org/Class15.html#efa)
-   JMPオンラインマニュアル >> [回転方法](https://www.jmp.com/support/help/ja/14-2/mm-factor-analysis-7.shtml)

[^fn:1]: [質的変数の相関・因子分析](https://www.slideshare.net/mitsuoshimohata/ss-24419059) 13rd Slide
[^fn:2]: [【R】データにNAがある時の相関係数 - データ分析メモと北欧生活](https://keita43a.hatenablog.com/entry/2018/04/10/034230)
[^fn:3]: <http://www.u.tsukuba.ac.jp/~hirai.akiyo.ft/forstudents/eigokyouikuhyoukaron2012%5F10%5F29%5F1.pdf>
[^fn:4]: [相関係数行列の吟味](http://aoki2.si.gunma-u.ac.jp/lecture/PFA/pfa6.html)
[^fn:5]: <http://www.alexanderdemos.org/Class15.html#extract%5Ffactor%5Fscores>
[^fn:6]: <https://www.slideshare.net/simizu706/r-42283141>
[^fn:7]: [因子分析における因子軸の回転法について | Sunny side up!](https://norimune.net/706)
[^fn:8]: [Michael Clark: Factor Analysis with the psych package](https://m-clark.github.io/posts/2020-04-10-psych-explained/)
[^fn:9]: <https://rpubs.com/danmirman/plotting%5Ffactor%5Fanalysis>
[^fn:10]: <http://mybook-pub-site.sakura.ne.jp/multi%5Fvariate%5Fanalysis/factor%5Fanalysis.pdf>
[^fn:11]: [R: Various ways to estimate factor scores for the factor...](http://www.personality-project.org/r/html/factor.scores.html)
[^fn:12]: [因子負荷量 | 統計用語集 | 統計WEB](https://bellcurve.jp/statistics/glossary/660.html)
