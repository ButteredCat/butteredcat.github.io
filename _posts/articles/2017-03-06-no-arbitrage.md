---
layout: post
title: 无套利定价理论
excerpt: "Financial Engineering and Risk Management I 课堂笔记"
modified: 2017-07-06
categories: articles
tags: [financial-engineering]
comments: true
share: true
---
买入一定的资产并随即卖出（或者相反），利用其中差价获利的行为，称之为**套利（arbitrage）**。无套利（no-arbitrage）假设，是金融产品以及衍生品定价理论的基本假设。其内涵就是，由于市场流动性和信息透明性，市场上不存在套利机会，世上没有免费的午餐。

利用无套利定价理论为金融产品定价时，常常构造两个相反的投资组合（portfolio），模拟同一个交易的双方，金融产品的定价使得双方均不可能在交易中套利。

举例来说，一份合约约定，一年后支付给你 \\(A = 100\\) 元，你当前愿意花多少钱购买这份合约？

是 \\(100\\) 元吗？假设你花 \\(100\\) 元购买的这份合约，对方把收到的钱用于投资（比如存入银行或者购买国债），利率为 \\(r=3\%\\)，一年后他收入 \\(A(1+r)=103\\) 元，却只需要支付 \\(100\\) 元给你，套利 \\(3\\) 元——你会这么傻？即使你傻，金融市场上其他卖方见到套利机会也会蜂拥而至，傻子很快不够用了，卖方为了套利不惜降价，直至套利机会不存在。

那么这份合约定价多少才能让卖方无套利机会呢？设定价为 \\(p\\)，卖方收到钱后用于投资，一年后得到 \\(p(1+r)\\) 元，卖方无法套利的条件是：
\\[
p(1+r) - A \le 0 ,\tag{1}
\\]
作为买方，你可以以利率 \\(r\\) 贷入 \\(p\\) 元用于购买合约，一年后收入 \\(A\\) 元并偿还 \\(p(1+r) \\) 元，假设你也不存在套利机会（否则这样的合约有多少给我来多少），有：
\\[
A-p(1+r) \le 0, \tag{2}
\\]
由 \\((1),(2)\\) 可知：
\\[
p=\frac{A}{(1+r)}=97.09.
\\]
以上可以看出，无套利假设得以成立，建立在这样几个条件之上：

1. 市场具有足够的流动性（markets are liquid），即市场上有足够多的买方和卖方。
2. 买卖双方都能获得价格信息。
3. 供求双方的竞争会修正实际价格和无套利价格之间的偏差。

这个例子也说明了，未来的 \\(100\\) 元没有当下的 \\(100\\) 元值钱（\\(97.09 \lt 100\\)）。在计算未来现金流的价格时，总要根据利率打点折扣。

上面的例子之所以能给出准确定价，是由于假设了存贷款利率相等，而通常情况下，贷款利率 \\(r_B\\) 总不低于存款利率 \\(r_L\\)（银行也不会白给你套利机会的），这时就只能给出 \\(p\\) 的取值范围：
\\[
\frac{A}{(1+r_B)} \le p \le \frac{A}{(1+r_L)}
\\]
这时候，\\(p\\) 在取值范围内，具体值由供求关系决定。
