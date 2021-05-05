---
layout: post
title: Reservá 小记
date: '2016-09-19 12:47:58'
tags:
- app-tag
---

Reservá 是我做的一款用于检测 Apple Store 的 iPhone 7 & 7 Plus 供货状况的一个 App。做这个 App 的原因也很简单，就是给自己预约 iPhone 使用，后来思考了一下，其实可以公开出来给大家使用，于是就重点在某论坛推广了一下，也取得了月活 500+、日活 150+ 的一点微小的工作。后来由于开发者账号到期，就没有继续维护 IPA 版本，源码依旧正在更新。由于没有上架，所以大多数用户都是越狱设备直接安装 IPA 的，但其中也有不少使用 Xcode 重新编译安装或者使用 Sideload 大法的用户。

![Reserva_Best_MAU](https://static.austinchou.com/reserva_best_mau.png)

具体的代码在 [https://github.com/EvianZhow/Reserva](https://github.com/EvianZhow/Reserva)

最开始是没有打算支持香港 Apple Store 的预约的，后来发现 Apple 的 API 设计是非常通用化的，导致只需要修改一小部分即可，就随手加上了。界面的设计主要按照用户所关心的 iPhone 型号，到供货列表。供货列表可以选择 “全部” 和 “有库存” 两种，方便搜索。点击列表项即可直接进入预约，无须重新选择。

当然也想过以某一个 Apple Store 具体的供货情况来进行展示，但是考虑到用户在购买的时候往往已经确定了目标型号，就没有再将此需求深挖下去。后续修改的部分大概就是，当用户选中“仅有货”的 Tab 时，如果该型号的库存情况有变化，将使用震动通知。

![Reserva_Screenshot_1](https://static.austinchou.com/reserva_screenshot_1.PNG)
![Reserva_Screenshot_2](https://static.austinchou.com/reserva_screenshot_2.PNG)
![Reserva_Screenshot_3](https://static.austinchou.com/reserva_screenshot_3.PNG)

再来看看大家关心的 iPhone 型号，以 9/17/16 的数据为例：

![Reserva_Best_Seller_Models](https://static.austinchou.com/reserva_best_seller_models.png)

- MNH22CH/A - iPhone 7 128GB 亮黑色  - 10% 
- MNFP2CH/A - iPhone 7 Plus 128GB 黑色  - 9% 
- MNH72CH/A - iPhone 7 256GB 亮黑色  - 7% 
- MNFU2CH/A - iPhone 7 Plus 128GB 亮黑色  - 5% 
- MNFV2CH/A - iPhone 7 Plus 256GB 黑色  - 4% 
- MNFQ2CH/A - iPhone 7 Plus 128GB 银色  - 4% 
- MNFT2CH/A - iPhone 7 Plus 128GB 玫瑰金色  - 3% 
- MNFR2CH/A - iPhone 7 Plus 128GB 金色  - 3% 
- MNRJ2CH/A - iPhone 7 Plus 32GB 黑色  - 3% 
- MNG02CH/A - iPhone 7 Plus 256GB 亮黑色  - 3%

可以看到新出的亮黑色和黑色是大家最喜欢的颜色，基本上也就是加价最狠的那几款，7 Plus 128G 亮黑色在 16 日发售当天收购价就突破了 12k...

另外一个比较有意思的数据是这样的：

![DAU_Price_Ratio.png](https://static.austinchou.com/dau_price_ratio.png)

这是我采集了 Reservá 日活用户与苹果团报价后生成的一个折线图，蓝色的线为当日的 DAU 除以此段时间平均的 DAU 得到的一个比率，绿色和黄色的线均为该型号 128G 黑色国行现货晚报价除以国行售价的比率，可以看到，传言的**“国行的 iPhone 7 跌破发行价”**应该是属实的，而 Plus 系列由于产能的低下，始终处于一个价格较高的状态。而最近如果有访问预约系统的朋友都应该知道，如果你是早晨 8:00 准时预约的话，iPhone 7 连亮黑色都是有现货的，但同样的，iPhone 7 Plus 全线几乎无货。

如果你对原始数据感兴趣的话，可以访问这里[下载](https://static.austinchou.com/dau_price_ratio.pdf)到。