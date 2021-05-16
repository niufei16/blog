---
title: 确定IP地理位置的2个例子
date: 2021-05-16 21:24:40
tags:
	- IPv4
---

从IP的traceroute(Windows为tracert)说起。
#### 例子一：12.23.4.7
```shell
$ tracert 12.23.4.7

通过最多 30 个跃点跟踪到 12.23.4.7 的路由

  1    <1 毫秒   <1 毫秒   <1 毫秒 192.168.31.1
  2     1 ms    <1 毫秒   <1 毫秒 192.168.18.1
  3     4 ms     5 ms     2 ms  100.74.0.1
  4     *        2 ms     2 ms  dns73.online.tj.cn [117.8.159.73]
  5     *        *        *     请求超时。
  6    37 ms    34 ms    34 ms  219.158.101.173
  7    35 ms    43 ms    34 ms  219.158.19.66
  8     *       37 ms    39 ms  219.158.20.222
  9   200 ms   198 ms   198 ms  219.158.98.50
 10   545 ms     *      545 ms  la2ca21crs.ip.att.net [12.122.128.182]
 11   545 ms   545 ms   545 ms  la2ca21crs.ip.att.net [12.122.128.182]
 12   541 ms   542 ms     *     12.123.30.49
 13   610 ms   613 ms   609 ms  12.123.30.50
 14     *        *      616 ms  slkut21crs.ip.att.net [12.122.1.186]
 15     *      617 ms     *     dvmco22crs.ip.att.net [12.122.28.45]
 16   615 ms     *      613 ms  cgcil21crs.ip.att.net [12.122.28.78]
 17     *        *      609 ms  cgcil22crs.ip.att.net [12.122.2.54]
 18     *      616 ms     *     n54ny21crs.ip.att.net [12.122.2.238]
 19   604 ms   601 ms   603 ms  cr85.n54ny.ip.att.net [12.122.115.46]
 20     *        *        *     请求超时。
 21     *        *        *     请求超时。
```
第10跳 12.122.128.182 的host为 la2ca21crs.ip.att.net，分析该host中的关键字la2ca可知该IP在美国加利福尼亚州(ca)的城市洛杉矶(la)，第14跳 12.122.1.186 的host为 slkut21crs.ip.att.net，分析关键字slkut可知为美国犹他州(ut)的城市盐湖城(slk)，最后断在了美国纽约州(ny)。

#### 例子二：13.229.188.59
```shell
$ tracert 13.229.188.59

通过最多 30 个跃点跟踪
到 www.github.com [13.229.188.59] 的路由:

  1    <1 毫秒   <1 毫秒   <1 毫秒 192.168.31.1
  2    <1 毫秒   <1 毫秒    6 ms  192.168.18.1
  3     2 ms     2 ms     2 ms  100.74.0.1
  4     3 ms     3 ms     3 ms  dns53.online.tj.cn [117.8.157.53]
  5     4 ms     4 ms     3 ms  dns21.online.tj.cn [117.8.222.21]
  6    22 ms    23 ms    21 ms  219.158.114.97
  7    75 ms    71 ms    72 ms  219.158.19.82
  8    39 ms    46 ms    40 ms  219.158.19.77
  9   216 ms   215 ms   195 ms  219.158.40.178
 10   178 ms   195 ms   196 ms  ae-4.r31.tokyjp05.jp.bb.gin.ntt.net [129.250.3.57]
 11   271 ms   290 ms   306 ms  ae-4.r23.sngpsi07.sg.bb.gin.ntt.net [129.250.2.242]
 12   270 ms   265 ms   249 ms  ae-1.r01.sngpsi07.sg.bb.gin.ntt.net [129.250.4.94]
 13   281 ms   292 ms   263 ms  ae-1.amazon.sngpsi07.sg.bb.gin.ntt.net [116.51.18.166]
 14     *        *        *     请求超时。
 15   396 ms   415 ms   396 ms  52.93.11.81
 16     *        *        *     请求超时。
 17     *        *        *     请求超时。
 18   285 ms   281 ms   278 ms  203.83.223.27
 19     *        *        *     请求超时。
 20     *        *        *     请求超时。
 21     *        *        *     请求超时。
 22     *        *        *     请求超时。
 23     *        *        *     请求超时。
 24     *        *        *     请求超时。
 25   270 ms   258 ms   241 ms  www.github.com [13.229.188.59]

跟踪完成。
```
第10跳 129.250.3.57 的host为 ae-4.r31.tokyjp05.jp.bb.gin.ntt.net，分析该host中的关键字tokyjp可知该IP在日本东京(jp)的城市东京(toky)，第13跳 116.51.18.166 的host为 ae-1.amazon.sngpsi07.sg.bb.gin.ntt.net，分析该host中的关键字sngpsi可知该IP在新加坡(sngp)，同时从该host中的amazon猜测此IP可能是机房的接入口。
最后综合其他信息可判断13.229.188.59是在新加坡。

host里关键字的一般规律是：
 - 二字代码（国家、州、城市）
 - 三字代码（所在城市的机场代码、胡乱拼凑的三字城市代码）
 - 五字代码
 - 六字代码
 - 其他(随便写的一些城市代码)

大部分时候凭借上下IP地理位置和对关键字的猜测基本可以确定IP的地理位置