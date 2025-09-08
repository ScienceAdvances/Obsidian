## Bonferroni Correction

the p-values are multiplied by the number of comparisons.

这种校正方法比较粗暴，p 值直接乘以 p 值数量。

比如这3个 p 值需要Bonferroni Correction，那么

```
p <- runif(3)*0.1
# 0.076693110 0.008295902 0.092487229
p * length(p)
# 0.23007933 0.02488771 0.27746169
```

## Benjamini and Hochberg Correction

最常用的校正方法，BH Correction或者也叫FDR。从 R 的源代码可以看出来两者是相同的

```
method <- match.arg(method)
if (method == "fdr") method <- "BH"
```

FDR control the false discovery rate, the expected proportion of false discoveries amongst the rejected hypotheses. The false discovery rate is a less stringent condition than the family-wise error rate, so these methods are more powerful than the others.

先看下 FDR方法校正 P 值的结果

```
library(tidyverse)
set.seed(1314)
p <- runif(3)*0.1
Bonferroni <- p.adjust(p,method="bonferroni")
BH <- p.adjust(p,method="BH")
FDR <- p.adjust(p,method="fdr")
p_tibble <- tibble(RawPvalue=p,Bonferroni=Bonferroni,BH=BH,FDR=FDR) %>% 
    dplyr::mutate(Rank=rank(RawPvalue),.before=RawPvalue)
#    Rank RawPvalue Bonferroni     BH    FDR
#   <dbl>     <dbl>      <dbl>  <dbl>  <dbl>
# 1     2   0.0767      0.230  0.0925 0.0925
# 2     1   0.00830     0.0249 0.0249 0.0249
# 3     3   0.0925      0.277  0.0925 0.0925
```

## 先了解一下 R 内置的几个函数的用法

- order

order 对向量排序，默认是从小到大，order 要和原来的数据一起使用比如 p[order(p)]，这样的效果才等于sort(p)

- cum 系列

最简单的是 cumsum，它的作用是累加；而 cummin 的作用是从第一个元素开始依次比较最小的数，如果前大于后，那么保留后，如果后大于前，那么保留前，总之是保留小的。

```
cumsum(1:5)==c(1,1+2,1+2+3,1+2+3+4,1+2+3+4+5)
# 1  3  6 10 15
cummin(c(9,2,8,3)) 
# 9 2 2 2 # 因为 2 后边的数都比 2 大
```

- pmin

保证所有计算出来的 p 值不会大于 1，如果 p 值＜1，那么保留对应的 p 值，如果 p 值大于 1，保留 1.

```
pmin(1,c(1,0.00001,0.999,1.23))
# 1.00000 0.00001 0.99900 1.00000
```

## 从源码一步步计算 P 值校正结果

p[o]即对 p 值从大到小排序，从最大 P 值开始比较

```
n <- length(p)
i <- n:1L
o <- order(p, decreasing = TRUE)
ro <- order(o)
```

i就相当与 p 值对应的 rank(p)，即他们的排序，因为p[o]已经把 p 值从大到小排序，所以 p 值的 rank 就是 n:1；

对于每个 p 值需要乘以 p 值的数目 n 再除以它的 rank；

```
n/i * p[o]
# 0.09248723 0.11503967 0.0248877
```

再接着对得到的 结果cummin依次找到最小的值

```
cummin(c(0.09248723, 0.11503967, 0.0248877))
# 0.09248723 0.09248723 0.02488770
```

pmin保证 p 值不会大于 1

```
pmin(1，c(0.09248723, 0.09248723, 0.02488770))
```

恢复 p 值到原来的顺序，得到最终结果

```
c(0.09248723, 0.09248723, 0.02488770)[ro]
# 0.09248723 0.02488770 0.09248723
```

## Reference

```
https://blog.csdn.net/qazplm12_3/article/details/127002716
https://evolution.gs.washington.edu/gs560/2011/lecture9.pdf
https://www.researchgate.net/post/What-is-the-False-Discovery-Rate-adjusted-p-value-Is-there-any-function-in-Matlab-What-is-the-effect-on-FDR-results-compared-with-p-value#:~:text=Regarding%20the%20FDR-adjusted%20p-value%20here%27s%20a%20formula%3A%20i%3DRanked,p%20-value%20%28can%20also%20be%20called%20q%20value%29.
https://mp.weixin.qq.com/s/Kj0iOWUiR88mTddlx1-CnQ
https://mp.weixin.qq.com/s/SdolEx8sCQDUdmpFLff9BA
```