---
layout: post
title: TA-Lib 中蜡烛图模式识别详解
date: '2017-08-21 12:48:34'
permalink: "/explanation-of-candlesticks-pattern-recognization-in-ta-lib/"
tags:
- ta-lib
- candlesticks
---

最近本人在使用 [TA-Lib](http://ta-lib.org/index.html) 这个库的时候，需要接触到 [蜡烛图](https://zh.wikipedia.org/wiki/K线) 形态识别的部分。由于东西方语言差异的原因，国内许多的量化开发者无法将自己所熟悉的各种蜡烛图术语与 TA-Lib 中 CDL 前缀的函数一一对应，所以在谨慎参考 [「日本蜡烛图技术——古老东方投资术的现代指南」](https://item.jd.com/10090445.html)、网上相关的术语及源代码注释之后，将各个函数所对应形态的详解整理成文，分享给大家，其中包括函数实现中具体的判定条件等，因此这篇文章篇幅很长，建议大家以阅读技术文档的方式来阅读此文。

> 这里我使用的是 TA-Lib 0.4 版本的源代码，如有更新，以最新版本中的说明为准。

在理解、翻译相关判断条件时，本人力求信雅达三点，但仍可能出现错误。如果你发现有相关的错误，欢迎在评论区予以指出。

索引:

- [两只乌鸦 Two Crows](#twocrows)
- [三只乌鸦 Three Black Crows](#threeblackcrows)
- [三内部上涨（下跌） Three Inside Up / Down](#threeinsideupdown)
- [三线打击 Three-Line Strike](#threelinestrike)
- [三外部上涨（下跌） Three Outside Up / Down](#threeoutsideupdown)
- [南方三星形态 Three Stars In The South](#threestarsinthesouth)
- [前进白色三兵 Three Advancing White Soldiers](#threeadvancingwhitesoldiers)
- [弃婴 Abandoned Baby](#abandonedbaby)
- [前方受阻 Advance Block](#advanceblock)
- [捉腰带线 Belt-hold](#belthold)
- [突破 Breakaway](#breakaway)
- [捉腰带线 Closing Marubozu](#closingmarubozu)
- [藏婴吞没 Concealing Baby Swallow](#concealingbabyswallow)
- [反击线、约会线 Counterattack](#counterattack)
- [乌云盖顶 Dark Cloud Cover](#darkcloudcover)
- [十字线 Doji](#doji)
- [十字星 Doji Star](#dojistar)
- [蜻蜓十字、T型十字 Dragonfly Doji](#dragonflydoji)
- [抱线、看涨（跌）吞没 Engulfing Pattern](#engulfingpattern)
- [十字黄昏星 Evening Doji Star](#eveningdojistar)
- [黄昏星 Evening Star](#eveningstar)
- [向上（下）跳空并列白色蜡烛线 Up / Down-gap Side-by-side White Lines](#updowngapsidebysidewhitelines)
- [墓碑十字线、灵位十字线 Gravestone Doji](#gravestonedoji)
- [锤子线 Hammer](#hammer)
- [上吊线 Hanging Man](#hangingman)
- [孕线 Harami Pattern](#haramipattern)
- [十字孕线 Harami Cross Pattern](#haramicrosspattern)
- [风高浪大线 High-Wave Candle](#highwavecandle)
- [陷阱形态 Hikkake Pattern](#hikkakepattern)
- [修正陷阱形态 Modified Hikkake Pattern](#modifiedhikkakepattern)
- [家鸽形态 Homing Pigeon](#homingpigeon)
- [三胞胎乌鸦 Identical Three Crows](#identicalthreecrows)
- [切入线 In-Neck Pattern](#inneckpattern)
- [倒锤子线 Inverted Hammer](#invertedhammer)
- [反冲形态 Kicking](#kicking)
- [反冲形态（由较长光头光脚阴阳线判定） Kicking - Bull / Bear determined by the longer Marubozu](#kickingbullbeardeterminedbythelongermarubozu)
- [梯底 Ladder Bottom](#ladderbottom)
- [长腿十字线 Long Legged Doji](#longleggeddoji)
- [长蜡烛线 Long Line Candle](#longlinecandle)
- [光头光脚阳（阴）线 Marubozu](#marubozu)
- [相同低价 Matching Low](#matchinglow)
- [铺垫 Mat Hold](#mathold)
- [十字启明星 Morning Doji Star](#morningdojistar)
- [启明星 Morning Star](#morningstar)
- [待入线 On-Neck Pattern](#onneckpattern)
- [斩回线、刺透形态 Piercing Pattern](#piercingpattern)
- [黄包车夫线 Rickshaw Man](#rickshawman)
- [上升（下降）三法 Rising / Falling Three Methods](#risingfallingthreemethods)
- [分手线 Separating Lines](#separatinglines)
- [流星线 Shooting Star](#shootingstar)
- [短蜡烛线 Short Line Candle](#shortlinecandle)
- [纺锤线 Spinning Top](#spinningtop)
- [停顿形态 Stalled Pattern](#stalledpattern)
- [条形三明治 Stick Sandwich](#sticksandwich)
- [探水竿 Takuri (Dragonfly Doji with very long lower shadow)](#takuridragonflydojiwithverylonglowershadow)
- [跳空并列阴阳线形态 Tasuki Gap](#tasukigap)
- [插入线形态 Thrusting Pattern](#thrustingpattern)
- [三星形态 Tristar Pattern](#tristarpattern)
- [奇特三川底部形态 Unique 3 River](#uniqueriver)
- [向上跳空两只乌鸦 Upside Gap Two Crows](#upsidegaptwocrows)
- [向上（下）跳空三法形态 Upside / Downside Gap Three Methods](#upsidedownsidegapthreemethods)

两只乌鸦 Two Crows
------

- 函数原型: integer = CDL2CROWS(open, high, low, close)
- 信号: 看跌
- 形态: 反转
- 定义: 第一天是一根坚挺的白色实体，第二天是一根黑色蜡烛线，且与第一天蜡烛线实体之间的形成跳空。第三天是一根黑色蜡烛线，且其开市价位于第二天蜡烛线实体之内，且收于第一天蜡烛线的实体内。此反转形态仅在明显的上升趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDL2CROWS.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long white candle
    * - second candle: black real body
    * - gap between the first and the second candle's real bodies
    * - third candle: black candle that opens within the second real body and closes within the first real body
    * The meaning of "long" is specified with TA_SetCandleSettings
    * outInteger is negative (-1 to -100): two crows is always bearish; 
    * the user should consider that two crows is significant when it appears in an uptrend, while this function 
    * does not consider the trend
    */
```

三只乌鸦 Three Black Crows
------

- 函数原型: integer = CDL3BLACKCROWS(open, high, low, close)
- 信号: 看跌
- 形态: 反转
- 定义: 三根依次下降的黑色蜡烛线，且收市价都应当位于其低点或接近低点的位置，开市价也应位于前一个实体范围之内。第一根黑色蜡烛线收市价应当低于之前最近一根的白色蜡烛线的高点。此反转形态仅在高水平的价格上，或者是经过充分发展的上涨行情中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDL3BLACKCROWS.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - three consecutive and declining black candlesticks
    * - each candle must have no or very short lower shadow
    * - each candle after the first must open within the prior candle's real body
    * - the first candle's close should be under the prior white candle's high
    * The meaning of "very short" is specified with TA_SetCandleSettings
    * outInteger is negative (-1 to -100): three black crows is always bearish; 
    * the user should consider that 3 black crows is significant when it appears after a mature advance or at high levels, 
    * while this function does not consider it
    */
```

三内部上涨（下跌） Three Inside Up / Down
------

- 函数原型: integer = CDL3INSIDE(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 三内部上涨（下跌）：第一天是一根疲软的黑色实体（坚挺的白色实体），第二天的实体较短，并形成被第一天蜡烛线完全吞没的形态，第三天收于第一天蜡烛线的开市价之上（下）。此反转形态仅在明显的下降（上升）趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDL3INSIDE.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long white (black) real body
    * - second candle: short real body totally engulfed by the first
    * - third candle: black (white) candle that closes lower (higher) than the first candle's open
    * The meaning of "short" and "long" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) for the three inside up or negative (-1 to -100) for the three inside down; 
    * the user should consider that a three inside up is significant when it appears in a downtrend and a three inside
    * down is significant when it appears in an uptrend, while this function does not consider the trend
    */
```

三线打击 Three-Line Strike
------

- 函数原型: integer = CDL3LINESTRIKE(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 持续
- 定义: 看涨（跌）三线打击：前三天的蜡烛线形成了前进白色三兵（三只乌鸦）的形态，第四天的开市价位于第三天蜡烛线收市价之上（下），第五天形成一根疲软的黑色实体（坚挺的白色实体），且最终收于第一天蜡烛线的开市价之下（上）。此反转形态仅在与前三天蜡烛线表示的相同趋势中才有意义。
- 最少所需的蜡烛线数量: 4
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDL3LINESTRIKE.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - three white soldiers (three black crows): three white (black) candlesticks with consecutively higher (lower) closes,
    * each opening within or near the previous real body
    * - fourth candle: black (white) candle that opens above (below) prior candle's close and closes below (above) 
    * the first candle's open
    * The meaning of "near" is specified with TA_SetCandleSettings;
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish;
    * the user should consider that 3-line strike is significant when it appears in a trend in the same direction of
    * the first three candles, while this function does not consider it
    */
```

三外部上涨（下跌） Three Outside Up / Down
------

- 函数原型: integer = CDL3OUTSIDE(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 三外部上涨（下跌）：第一天是一根黑色（白色）实体蜡烛线，第二天的实体完全吞没了第一天的实体，第三天蜡烛线继续第二天的走势，收于第二天蜡烛线的上方（下方）。此反转形态仅在明显的下降（上升）趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDL3OUTSIDE.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first: black (white) real body
    * - second: white (black) real body that engulfs the prior real body
    * - third: candle that closes higher (lower) than the second candle
    * outInteger is positive (1 to 100) for the three outside up or negative (-1 to -100) for the three outside down;
    * the user should consider that a three outside up must appear in a downtrend and three outside down must appear
    * in an uptrend, while this function does not consider it
    */
```

南方三星形态 Three Stars In The South
------

- 函数原型: integer = CDL3STARSINSOUTH(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 第一天的蜡烛线为一根疲软的黑色蜡烛线，并有较长的下影线。第二天的开市价高于第一天蜡烛线的收市价但并未超过第一天蜡烛线的高点，且当天的低点介于第一天蜡烛线的低点和收市价之间，最后收成带有一个长下影线的蜡烛线。第三天是一根光头光脚阴线，并形成被第二天蜡烛线的高低点完全吞没的形态。此反转形态仅在明显的下降趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDL3STARSINSOUTH.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black candle with long lower shadow
    * - second candle: smaller black candle that opens higher than prior close but within prior candle's range 
    *   and trades lower than prior close but not lower than prior low and closes off of its low (it has a shadow)
    * - third candle: small black marubozu (or candle with very short shadows) engulfed by prior candle's range
    * The meanings of "long body", "short body", "very short shadow" are specified with TA_SetCandleSettings;
    * outInteger is positive (1 to 100): 3 stars in the south is always bullish;
    * the user should consider that 3 stars in the south is significant when it appears in downtrend, while this function 
    * does not consider it
    */
```

前进白色三兵 Three Advancing White Soldiers
------

- 函数原型: integer = CDL3WHITESOLDIERS(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 由接连三根白色蜡烛线组成，且每一根白色蜡烛线的开市价都位于前一天的白色实体之内，收市价都应该位于当日的最高点或接近最高点处。为了与前方受阻形态所区别，任何一根蜡烛线都不应与前一根蜡烛线有着明显的缩短。此反转形态仅在明显的下降趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDL3WHITESOLDIERS.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - three white candlesticks with consecutively higher closes
    * - Greg Morris wants them to be long, Steve Nison doesn't; anyway they should not be short
    * - each candle opens within or near the previous white real body 
    * - each candle must have no or very short upper shadow
    * - to differentiate this pattern from advance block, each candle must not be far shorter than the prior candle
    * The meanings of "not short", "very short shadow", "far" and "near" are specified with TA_SetCandleSettings;
    * here the 3 candles must be not short, if you want them to be long use TA_SetCandleSettings on BodyShort;
    * outInteger is positive (1 to 100): advancing 3 white soldiers is always bullish;
    * the user should consider that 3 white soldiers is significant when it appears in downtrend, while this function 
    * does not consider it
    */
```

弃婴 Abandoned Baby
------

- 函数原型: integer = CDLABANDONEDBABY(open, high, low, close, penetration=0)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 看涨（跌）弃婴：第一天是一根疲软的黑色（坚挺的白色）蜡烛线，第二天是一根十字线，与第一天蜡烛线的低（高）点形成向下（上）跳空，且两者高低点区域无重叠。第三天是一根坚挺的白色（疲软的黑色）蜡烛线且收市价位于第一天蜡烛线的实体内，与第二天的十字线高（低）点形成向上（下）跳空，且两者高低点区域无重叠。此反转形态仅在明显的趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLABANDONEDBABY.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long white (black) real body
    * - second candle: doji
    * - third candle: black (white) real body that moves well within the first candle's real body
    * - upside (downside) gap between the first candle and the doji (the shadows of the two candles don't touch)
    * - downside (upside) gap between the doji and the third candle (the shadows of the two candles don't touch)
    * The meaning of "doji" and "long" is specified with TA_SetCandleSettings
    * The meaning of "moves well within" is specified with optInPenetration and "moves" should mean the real body should
    * not be short ("short" is specified with TA_SetCandleSettings) - Greg Morris wants it to be long, someone else want
    * it to be relatively long
    * outInteger is positive (1 to 100) when it's an abandoned baby bottom or negative (-1 to -100) when it's 
    * an abandoned baby top; the user should consider that an abandoned baby is significant when it appears in 
    * an uptrend or downtrend, while this function does not consider the trend
    */
```

前方受阻 Advance Block
------

- 函数原型: integer = CDLADVANCEBLOCK(open, high, low, close)
- 信号: 看跌
- 形态: 反转
- 定义: 符合前进白色三兵的定义，每一根蜡烛线的开市价都位于上一根蜡烛线实体之内或者附近。第一根蜡烛线拥有较短或者没有上影线。第二根与第三根蜡烛线、或仅第三根蜡烛线体现出显著的市场力量减弱信号，表现为越来越小的实体和相对越来越长的上影线。此反转形态仅在明显的上升趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLADVANCEBLOCK.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - three white candlesticks with consecutively higher closes
    * - each candle opens within or near the previous white real body 
    * - first candle: long white with no or very short upper shadow (a short shadow is accepted too for more flexibility)
    * - second and third candles, or only third candle, show signs of weakening: progressively smaller white real bodies 
    * and/or relatively long upper shadows; see below for specific conditions
    * The meanings of "long body", "short shadow", "far" and "near" are specified with TA_SetCandleSettings;
    * outInteger is negative (-1 to -100): advance block is always bearish;
    * the user should consider that advance block is significant when it appears in uptrend, while this function 
    * does not consider it
    */
```

捉腰带线 Belt-hold
------

- 函数原型: integer = CDLBELTHOLD(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 看涨（跌）捉腰带线：一根坚挺的白色（疲软的黑色）蜡烛线，其开市价位于当日的最低（高）点或者这跟蜡烛线只有极短的下（上）影线，然后市场一路上扬（下跌）。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLBELTHOLD.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - long white (black) real body
    * - no or very short lower (upper) shadow
    * The meaning of "long" and "very short" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when white (bullish), negative (-1 to -100) when black (bearish)
    */
```

突破 Breakaway
------

- 函数原型: integer = CDLBREAKAWAY(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 看涨（跌）突破：第一天是一根疲软的黑色（坚挺的白色）蜡烛线，紧接着跟随三根高点越来越低（高）、低点越来越低（高）的蜡烛线，它们与第一天蜡烛线的实体形成向下（上）跳空，中间一根颜色无所谓，其余两根为黑色（白色）蜡烛线。第五天一根坚挺的白色（疲软的黑色）蜡烛线回填了跳空缺口。此反转形态仅在与最后一根蜡烛线相反的趋势中才有意义。
- 最少所需的蜡烛线数量: 5
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLBREAKAWAY.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black (white)
    * - second candle: black (white) day whose body gaps down (up)
    * - third candle: black or white day with lower (higher) high and lower (higher) low than prior candle's
    * - fourth candle: black (white) day with lower (higher) high and lower (higher) low than prior candle's
    * - fifth candle: white (black) day that closes inside the gap, erasing the prior 3 days
    * The meaning of "long" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish;
    * the user should consider that breakaway is significant in a trend opposite to the last candle, while this 
    * function does not consider it
    */
```

捉腰带线 Closing Marubozu
------

- 函数原型: integer = CDLCLOSINGMARUBOZU(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 参考捉腰带线。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLCLOSINGMARUBOZU.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - long white (black) real body
    * - no or very short upper (lower) shadow
    * The meaning of "long" and "very short" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when white (bullish), negative (-1 to -100) when black (bearish)
    */
```

藏婴吞没 Concealing Baby Swallow
------

- 函数原型: integer = CDLCONCEALBABYSWALL(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 前两天都是一根光头光脚大阴线，第三天的开市价形成向下跳空但最终收成一根具有较长上影线的蜡烛线，且上影线穿入前一根蜡烛线的实体内。第四天也是一根疲软的黑色蜡烛线，但将前一天的高低点完全吞没。此反转形态仅在明显的下降趋势中才有意义。
- 最少所需的蜡烛线数量: 4
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLCONCEALBABYSWALL.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: black marubozu (very short shadows)
    * - second candle: black marubozu (very short shadows)
    * - third candle: black candle that opens gapping down but has an upper shadow that extends into the prior body
    * - fourth candle: black candle that completely engulfs the third candle, including the shadows
    * The meanings of "very short shadow" are specified with TA_SetCandleSettings;
    * outInteger is positive (1 to 100): concealing baby swallow is always bullish;
    * the user should consider that concealing baby swallow is significant when it appears in downtrend, while 
    * this function does not consider it
    */
```

反击线、约会线 Counterattack
------

- 函数原型: integer = CDLCOUNTERATTACK(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 其他
- 定义: 看涨（跌）反击线：类似刺透（乌云盖顶）形态，但后一个白色（黑色）蜡烛线仅回升（下跌）到前一天的收市价位置。此形态仅在明显的趋势中才有意义。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLCOUNTERATTACK.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black (white)
    * - second candle: long white (black) with close equal to the prior close
    * The meaning of "equal" and "long" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish;
    * the user should consider that counterattack is significant in a trend, while this function does not consider it
    */
```

乌云盖顶 Dark Cloud Cover
------

- 函数原型: integer = CDLDARKCLOUDCOVER(open, high, low, close)
- 信号: 看跌
- 形态: 反转
- 定义: 第一天是一根坚挺的白色实体，第二天开市价超过了第一天的最高价，但收市价位于第一天的蜡烛线的实体中。备注：Greg Morris 期望收市价位于第一天的蜡烛线实体中点以下的位置。此反转形态仅在明显的上升趋势中才有意义。
- 最少所需的蜡烛线数量: 2
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLDARKCLOUDCOVER.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long white candle
    * - second candle: black candle that opens above previous day high and closes within previous day real body; 
    * Greg Morris wants the close to be below the midpoint of the previous real body
    * The meaning of "long" is specified with TA_SetCandleSettings, the penetration of the first real body is specified
    * with optInPenetration
    * outInteger is negative (-1 to -100): dark cloud cover is always bearish
    * the user should consider that a dark cloud cover is significant when it appears in an uptrend, while 
    * this function does not consider it
    */
```

十字线 Doji
------

- 函数原型: integer = CDLDOJI(open, high, low, close)
- 信号: 无法判断
- 形态: 反转或保持原有形态
- 定义: 具有较小实体的蜡烛线。单独的一根十字线并不意味着看涨或者看跌。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLDOJI.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    *
    * Must have:
    * - open quite equal to close
    * How much can be the maximum distance between open and close is specified with TA_SetCandleSettings
    * outInteger is always positive (1 to 100) but this does not mean it is bullish: doji shows uncertainty and it is
    * neither bullish nor bearish when considered alone
    */
```

十字星 Doji Star
------

- 函数原型: integer = CDLDOJISTAR(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 看涨（跌）十字星：第一天是一根坚挺的白色（疲软的黑色）实体，第二天开市价高（低）于了第一天的最高（低）价且收成一根十字星。此反转形态仅在明显的下降（上升）趋势中才有意义。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLDOJISTAR.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long real body
    * - second candle: star (open gapping up in an uptrend or down in a downtrend) with a doji
    * The meaning of "doji" and "long" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish; 
    * it's defined bullish when the long candle is white and the star gaps up, bearish when the long candle 
    * is black and the star gaps down; the user should consider that a doji star is bullish when it appears 
    * in an uptrend and it's bearish when it appears in a downtrend, so to determine the bullishness or 
    * bearishness of the pattern the trend must be analyzed
    */
```

蜻蜓十字、T型十字 Dragonfly Doji
------

- 函数原型: integer = CDLDRAGONFLYDOJI(open, high, low, close)
- 信号: 无法判断
- 形态: 反转
- 定义: 符合十字线定义，且拥有非常短的或者没有上影线，相对的，下影线应该足够的长，以与其他十字星的变种所能够区分。蜻蜓十字并不意味着看涨，要结合原有趋势判断。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLDRAGONFLYDOJI.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    *
    * Must have:
    * - doji body
    * - open and close at the high of the day = no or very short upper shadow
    * - lower shadow (to distinguish from other dojis, here lower shadow should not be very short)
    * The meaning of "doji" and "very short" is specified with TA_SetCandleSettings
    * outInteger is always positive (1 to 100) but this does not mean it is bullish: dragonfly doji must be considered
    * relatively to the trend
    */
```

抱线、看涨（跌）吞没 Engulfing Pattern
------

- 函数原型: integer = CDLENGULFING(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 看涨（跌）吞没：第一天是一根黑色（白色）蜡烛线，第二天是一根白色（黑色）蜡烛线且实体必须覆盖第一根蜡烛线的实体（不一定需要吞没上下影线）。此反转形态仅在明显的下降（上升）趋势中才有意义

- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLENGULFING.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first: black (white) real body
    * - second: white (black) real body that engulfs the prior real body
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish;
    * the user should consider that an engulfing must appear in a downtrend if bullish or in an uptrend if bearish,
    * while this function does not consider it
    */
```

十字黄昏星 Evening Doji Star
------

- 函数原型: integer = CDLEVENINGDOJISTAR(open, high, low, close, penetration=0)
- 信号: 看跌
- 形态: 反转
- 定义: 符合黄昏星形态的定义，但第二根蜡烛线为十字线形态。此反转形态仅在明显的上升趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLEVENINGDOJISTAR.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long white real body
    * - second candle: doji gapping up
    * - third candle: black real body that moves well within the first candle's real body
    * The meaning of "doji" and "long" is specified with TA_SetCandleSettings
    * The meaning of "moves well within" is specified with optInPenetration and "moves" should mean the real body should
    * not be short ("short" is specified with TA_SetCandleSettings) - Greg Morris wants it to be long, someone else want
    * it to be relatively long
    * outInteger is negative (-1 to -100): evening star is always bearish; 
    * the user should consider that an evening star is significant when it appears in an uptrend, 
    * while this function does not consider the trend
    */
```

黄昏星 Evening Star
------

- 函数原型: integer = CDLEVENINGSTAR(open, high, low, close, penetration=0)
- 信号: 看跌
- 形态: 反转
- 定义: 第一天是一根坚挺的白色实体，第二天开市价产生了向上跳空且没有回填缺口，最后收成一根具有较小实体的星线，第三天时出现了一根黑色蜡烛线且最终收市价位于第一根的实体之内。此反转形态仅在明显的上升趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLEVENINGSTAR.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long white real body
    * - second candle: star (short real body gapping up)
    * - third candle: black real body that moves well within the first candle's real body
    * The meaning of "short" and "long" is specified with TA_SetCandleSettings
    * The meaning of "moves well within" is specified with optInPenetration and "moves" should mean the real body should
    * not be short ("short" is specified with TA_SetCandleSettings) - Greg Morris wants it to be long, someone else want
    * it to be relatively long
    * outInteger is negative (-1 to -100): evening star is always bearish; 
    * the user should consider that an evening star is significant when it appears in an uptrend, 
    * while this function does not consider the trend
    */
```

向上（下）跳空并列白色蜡烛线 Up / Down-gap Side-by-side White Lines
------

- 函数原型: integer = CDLGAPSIDESIDEWHITE(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 持续
- 定义: 向上（下）跳空并列阳线：第二天出现了一根向上（下）跳空的白色蜡烛线，紧接着第三天的蜡烛线长度与第二天相似，开盘、收市价也相当接近，且并未回填跳空缺口。此持续形态仅在明显的趋势中才有意义。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLGAPSIDESIDEWHITE.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - upside or downside gap (between the bodies)
    * - first candle after the window: white candlestick
    * - second candle after the window: white candlestick with similar size (near the same) and about the same 
    *   open (equal) of the previous candle
    * - the second candle does not close the window
    * The meaning of "near" and "equal" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) or negative (-1 to -100): the user should consider that upside 
    * or downside gap side-by-side white lines is significant when it appears in a trend, while this function 
    * does not consider the trend
    */
```

墓碑十字线、灵位十字线 Gravestone Doji
------

- 函数原型: integer = CDLGRAVESTONEDOJI(open, high, low, close)
- 信号: 无法判断
- 形态: 其他
- 定义: 符合十字线定义，且拥有非常短的或者没有下影线，相对的，上影线应该足够的长，以与其他十字星的变种所能够区分。此形态并不意味着看涨，要结合原有趋势判断。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLGRAVESTONEDOJI.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    *
    * Must have:
    * - doji body
    * - open and close at the low of the day = no or very short lower shadow
    * - upper shadow (to distinguish from other dojis, here upper shadow should not be very short)
    * The meaning of "doji" and "very short" is specified with TA_SetCandleSettings
    * outInteger is always positive (1 to 100) but this does not mean it is bullish: gravestone doji must be considered
    * relatively to the trend
    */
```

锤子线 Hammer
------

- 函数原型: integer = CDLHAMMER(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 实体位于整个价格区间的上端，上影线尽可能短或者没有，下影线尽可能长，实体位于上一根蜡烛线低点的附近或比它更低。此反转形态仅出现在明显的下降趋势中。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLHAMMER.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - small real body
    * - long lower shadow
    * - no, or very short, upper shadow
    * - body below or near the lows of the previous candle
    * The meaning of "short", "long" and "near the lows" is specified with TA_SetCandleSettings;
    * outInteger is positive (1 to 100): hammer is always bullish;
    * the user should consider that a hammer must appear in a downtrend, while this function does not consider it
    */
```

上吊线 Hanging Man
------

- 函数原型: integer = CDLHANGINGMAN(open, high, low, close)
- 信号: 看跌
- 形态: 反转
- 定义: 实体位于整个价格区间的上端，上影线尽可能短或者没有，下影线尽可能长，实体位于上一根蜡烛线高点的附近或比它更高。此反转形态仅出现在明显的上升趋势中。
- 最少所需的蜡烛线数量: 1
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLHANGINGMAN.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - small real body
    * - long lower shadow
    * - no, or very short, upper shadow
    * - body above or near the highs of the previous candle
    * The meaning of "short", "long" and "near the highs" is specified with TA_SetCandleSettings;
    * outInteger is negative (-1 to -100): hanging man is always bearish;
    * the user should consider that a hanging man must appear in an uptrend, while this function does not consider it
    */
```

孕线 Harami Pattern
------

- 函数原型: integer = CDLHARAMI(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 看涨（跌）孕线：前一个蜡烛线为一个较长的实体的白色蜡烛线，将后一个小实体完全吞没起来，两根蜡烛线实体的颜色可以不一定是不同的颜色，但通常情况下是不同的。此反转形态仅在明显的下降（上涨）趋势中才有意义。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLHARAMI.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long white (black) real body
    * - second candle: short real body totally engulfed by the first
    * The meaning of "short" and "long" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish; 
    * the user should consider that a harami is significant when it appears in a downtrend if bullish or 
    * in an uptrend when bearish, while this function does not consider the trend
    */
```

十字孕线 Harami Cross Pattern
------

- 函数原型: integer = CDLHARAMICROSS(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 看涨（跌）十字孕线：符合看涨（跌）孕线的定义，且第二天蜡烛线为一根十字线。此反转形态仅在明显的下降（上涨）趋势中才有意义。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLHARAMICROSS.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long white (black) real body
    * - second candle: doji totally engulfed by the first
    * The meaning of "doji" and "long" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish; 
    * the user should consider that a harami cross is significant when it appears in a downtrend if bullish or 
    * in an uptrend when bearish, while this function does not consider the trend
    */
```

风高浪大线 High-Wave Candle
------

- 函数原型: integer = CDLHIGHWAVE(open, high, low, close)
- 信号: 无法判断
- 形态: 其他
- 定义: 类似十字线，拥有较小的实体，但有较长的上下影线。此形态并不意味着看涨或者看跌，而是反映了市场中犹豫不决的状态。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLHIGHWAVE.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - short real body
    * - very long upper and lower shadow
    * The meaning of "short" and "very long" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when white or negative (-1 to -100) when black;
    * it does not mean bullish or bearish
    */
```

陷阱形态 Hikkake Pattern
------

- 函数原型: integer = CDLHIKKAKE(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转或保持原有形态
- 定义: 看涨（跌）陷阱：三根蜡烛线依次高点越来越低（高），低点越来越高（低）。在未来的三天内，将出现确认信号：某一天的收市价高（低）于第二天蜡烛线的高（低）点。如果相同的蜡烛线既是确认信号又是陷阱形态的话，将会返回后者的结果。
- 最少所需的蜡烛线数量: 4~6
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLHIKKAKE.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first and second candle: inside bar (2nd has lower high and higher low than 1st)
    * - third candle: lower high and lower low than 2nd (higher high and higher low than 2nd)
    * outInteger[hikkakebar] is positive (1 to 100) or negative (-1 to -100) meaning bullish or bearish hikkake
    * Confirmation could come in the next 3 days with:
    * - a day that closes higher than the high (lower than the low) of the 2nd candle
    * outInteger[confirmationbar] is equal to 100 + the bullish hikkake result or -100 - the bearish hikkake result
    * Note: if confirmation and a new hikkake come at the same bar, only the new hikkake is reported (the new hikkake
    * overwrites the confirmation of the old hikkake)
    */
```

修正陷阱形态 Modified Hikkake Pattern
------

- 函数原型: integer = CDLHIKKAKEMOD(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 看涨（跌）修正陷阱：四根蜡烛线依次高点越来越低（高），低点越来越高（低）。在未来的三天内，将出现确认信号：某一天的收市价高（低）于第三天蜡烛线的高（低）点。如果相同的蜡烛线既是确认信号又是陷阱形态的话，将会返回后者的结果。此反转形态仅在明显的下降（上升）趋势中才有意义。
- 最少所需的蜡烛线数量: 4~6
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLHIKKAKEMOD.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle
    * - second candle: candle with range less than first candle and close near the bottom (near the top)
    * - third candle: lower high and higher low than 2nd
    * - fourth candle: lower high and lower low (higher high and higher low) than 3rd
    * outInteger[hikkake bar] is positive (1 to 100) or negative (-1 to -100) meaning bullish or bearish hikkake
    * Confirmation could come in the next 3 days with:
    * - a day that closes higher than the high (lower than the low) of the 3rd candle
    * outInteger[confirmationbar] is equal to 100 + the bullish hikkake result or -100 - the bearish hikkake result
    * Note: if confirmation and a new hikkake come at the same bar, only the new hikkake is reported (the new hikkake
    * overwrites the confirmation of the old hikkake);
    * the user should consider that modified hikkake is a reversal pattern, while hikkake could be both a reversal 
    * or a continuation pattern, so bullish (bearish) modified hikkake is significant when appearing in a downtrend 
    * (uptrend)
    */
```

家鸽形态 Homing Pigeon
------

- 函数原型: integer = CDLHOMINGPIGEON(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 第一天是一根疲软的黑色实体，第二天是一根小的黑色实体蜡烛线，且实体部分完全被第一天蜡烛线的实体所包裹（吞没）。此反转形态仅在明显的下降趋势中才有意义。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLHOMINGPIGEON.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black candle
    * - second candle: short black real body completely inside the previous day's body
    * The meaning of "short" and "long" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100): homing pigeon is always bullish; 
    * the user should consider that homing pigeon is significant when it appears in a downtrend,
    * while this function does not consider the trend
    */
```

三胞胎乌鸦 Identical Three Crows
------

- 函数原型: integer = CDLIDENTICAL3CROWS(open, high, low, close)
- 信号: 看跌
- 形态: 反转
- 定义: 符合三只乌鸦的定义，但后面两根蜡烛线要求开市价等于或者非常接近于前一天蜡烛线的收市价。此反转形态仅在高水平的价格上，或者是经过充分发展的上涨行情中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLIDENTICAL3CROWS.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - three consecutive and declining black candlesticks
    * - each candle must have no or very short lower shadow
    * - each candle after the first must open at or very close to the prior candle's close
    * The meaning of "very short" is specified with TA_SetCandleSettings;
    * the meaning of "very close" is specified with TA_SetCandleSettings (Equal);
    * outInteger is negative (-1 to -100): identical three crows is always bearish; 
    * the user should consider that identical 3 crows is significant when it appears after a mature advance or at high levels, 
    * while this function does not consider it
    */
```

切入线 In-Neck Pattern
------

- 函数原型: integer = CDLINNECK(open, high, low, close)
- 信号: 看跌
- 形态: 持续
- 定义: 第一天是一根疲软的黑色实体，第二天开市价低于第一天蜡烛线的低点，后续走高，但最终收于第一天蜡烛线实体下部，靠近其收市价。此持续形态仅在下降的趋势中才有意义。
- 最少所需的蜡烛线数量: 2
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLINNECK.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black candle
    * - second candle: white candle with open below previous day low and close slightly into previous day body
    * The meaning of "equal" is specified with TA_SetCandleSettings
    * outInteger is negative (-1 to -100): in-neck is always bearish
    * the user should consider that in-neck is significant when it appears in a downtrend, while this function 
    * does not consider it
    */
```

倒锤子线 Inverted Hammer
------

- 函数原型: integer = CDLINVERTEDHAMMER(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 实体位于整个价格区间的下端，下影线尽可能短或者没有，上影线尽可能长，实体与上一根蜡烛线之间形成向下跳空。此反转形态仅出现在明显的下降趋势中。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLINVERTEDHAMMER.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - small real body
    * - long upper shadow
    * - no, or very short, lower shadow
    * - gap down
    * The meaning of "short", "very short" and "long" is specified with TA_SetCandleSettings;
    * outInteger is positive (1 to 100): inverted hammer is always bullish;
    * the user should consider that an inverted hammer must appear in a downtrend, while this function does not consider it
    */
```

反冲形态 Kicking
------

- 函数原型: integer = CDLKICKING(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 看涨（跌）反冲：第一天是一根光头光脚阴（阳）线，第二天开市价形成向上（下）跳空，收成一根光头光脚阳（阴）线。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLKICKING.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: marubozu
    * - second candle: opposite color marubozu
    * - gap between the two candles: upside gap if black then white, downside gap if white then black
    * The meaning of "long body" and "very short shadow" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish
    */
```

反冲形态（由较长光头光脚阴阳线判定） Kicking - Bull / Bear determined by the longer Marubozu
------

- 函数原型: integer = CDLKICKINGBYLENGTH(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 反转
- 定义: 符合反冲形态的定义，但是看涨看跌由由两根光头光脚阳（阴）线中较长的那根所决定。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLKICKINGBYLENGTH.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: marubozu
    * - second candle: opposite color marubozu
    * - gap between the two candles: upside gap if black then white, downside gap if white then black
    * The meaning of "long body" and "very short shadow" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish; the longer of the two
    * marubozu determines the bullishness or bearishness of this pattern
    */
```

梯底 Ladder Bottom
------

- 函数原型: integer = CDLLADDERBOTTOM(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 前三根蜡烛线符合三只乌鸦的定义。第四根蜡烛线为一根具有较长上影线的黑色蜡烛线。第五根蜡烛线的开市价高于第四根蜡烛线的实体，为一根白色实体的蜡烛线，且最终收于第四根蜡烛线的高点之上。此反转形态仅在明显的下降趋势中才有意义。
- 最少所需的蜡烛线数量: 5
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLLADDERBOTTOM.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - three black candlesticks with consecutively lower opens and closes
    * - fourth candle: black candle with an upper shadow (it's supposed to be not very short)
    * - fifth candle: white candle that opens above prior candle's body and closes above prior candle's high
    * The meaning of "very short" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100): ladder bottom is always bullish; 
    * the user should consider that ladder bottom is significant when it appears in a downtrend, 
    * while this function does not consider it
    */
```

长腿十字线 Long Legged Doji
------

- 函数原型: integer = CDLLONGLEGGEDDOJI(open, high, low, close)
- 信号: 无法判断
- 形态: 其他
- 定义: 符合星型的定义且有一或两根较长的影线。长腿十字线并不意味着看涨，而是显示出市场的不确定性。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLLONGLEGGEDDOJI.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    *
    * Must have:
    * - doji body
    * - one or two long shadows
    * The meaning of "doji" is specified with TA_SetCandleSettings
    * outInteger is always positive (1 to 100) but this does not mean it is bullish: long legged doji shows uncertainty
    */
```

长蜡烛线 Long Line Candle
------

- 函数原型: integer = CDLLONGLINE(open, high, low, close)
- 信号: 无法判断
- 形态: 其他
- 定义: 具有较长实体和上下影线的一根蜡烛线。此形态并不意味着任何看涨或者看跌。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLLONGLINE.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - long real body
    * - short upper and lower shadow
    * The meaning of "long" and "short" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when white (bullish), negative (-1 to -100) when black (bearish)
    */
```

光头光脚阳（阴）线 Marubozu
------

- 函数原型: integer = CDLMARUBOZU(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 持续
- 定义: 光头光脚阳（阴）线：一根坚挺的白色（疲软的黑色）蜡烛线，其开市价位于当日的低（高）点或者只有极短的下（上）影线，然后市场一路上扬（下挫），收市价位于当日的高（低）点或者只有极短的上（下）影线。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLMARUBOZU.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - long real body
    * - no or very short upper and lower shadow
    * The meaning of "long" and "very short" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when white (bullish), negative (-1 to -100) when black (bearish)
    */
```

相同低价 Matching Low
------

- 函数原型: integer = CDLMATCHINGLOW(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 第一天是一根黑色蜡烛线，第二天也是一根黑色蜡烛线，且收市价接近第一天蜡烛线的收市价。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLMATCHINGLOW.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: black candle
    * - second candle: black candle with the close equal to the previous close
    * The meaning of "equal" is specified with TA_SetCandleSettings
    * outInteger is always positive (1 to 100): matching low is always bullish;
    */
```

铺垫 Mat Hold
------

- 函数原型: integer = CDLMATHOLD(open, high, low, close, penetration=0)
- 信号: 看涨
- 形态: 持续
- 定义: 第一天出现一跟坚挺的白色蜡烛线，其次紧挨着三根或以上的依次下降的小实体蜡烛线。第一根实体线为黑色蜡烛线，且与第一天蜡烛线的实体间形成跳空。其余的两根小实体线均收于第一天蜡烛线的实体内。最后一天是同样一根坚挺的白色蜡烛线，且开市价应高于前一天收市价，收于该组合中的最高点。
- 最少所需的蜡烛线数量: 5
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLMATHOLD.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long white candle
    * - upside gap between the first and the second bodies
    * - second candle: small black candle
    * - third and fourth candles: falling small real body candlesticks (commonly black) that hold within the long 
    *   white candle's body and are higher than the reaction days of the rising three methods
    * - fifth candle: white candle that opens above the previous small candle's close and closes higher than the 
    *   high of the highest reaction day
    * The meaning of "short" and "long" is specified with TA_SetCandleSettings; 
    * "hold within" means "a part of the real body must be within";
    * optInPenetration is the maximum percentage of the first white body the reaction days can penetrate (it is 
    * to specify how much the reaction days should be "higher than the reaction days of the rising three methods")
    * outInteger is positive (1 to 100): mat hold is always bullish
    */
```

十字启明星 Morning Doji Star
------

- 函数原型: integer = CDLMORNINGDOJISTAR(open, high, low, close, penetration=0)
- 信号: 看涨
- 形态: 反转
- 定义: 符合启明星形态的定义，但第二根蜡烛线为十字线形态。此反转形态仅在明显的下降趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLMORNINGDOJISTAR.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black real body
    * - second candle: doji gapping down
    * - third candle: white real body that moves well within the first candle's real body
    * The meaning of "doji" and "long" is specified with TA_SetCandleSettings
    * The meaning of "moves well within" is specified with optInPenetration and "moves" should mean the real body should
    * not be short ("short" is specified with TA_SetCandleSettings) - Greg Morris wants it to be long, someone else want
    * it to be relatively long
    * outInteger is positive (1 to 100): morning doji star is always bullish;
    * the user should consider that a morning star is significant when it appears in a downtrend, 
    * while this function does not consider the trend
    */
```

启明星 Morning Star
------

- 函数原型: integer = CDLMORNINGSTAR(open, high, low, close, penetration=0)
- 信号: 看涨
- 形态: 反转
- 定义: 第一天是一根坚挺的黑色实体，第二天开市价产生了向下跳空且没有回填缺口，最后收成一根十字线，第三天时出现了一根坚挺的白色蜡烛线且最终收市价位于第一根的实体之内。此反转形态仅在明显的下降趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLMORNINGSTAR.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black real body
    * - second candle: star (Short real body gapping down)
    * - third candle: white real body that moves well within the first candle's real body
    * The meaning of "short" and "long" is specified with TA_SetCandleSettings
    * The meaning of "moves well within" is specified with optInPenetration and "moves" should mean the real body should
    * not be short ("short" is specified with TA_SetCandleSettings) - Greg Morris wants it to be long, someone else want
    * it to be relatively long
    * outInteger is positive (1 to 100): morning star is always bullish; 
    * the user should consider that a morning star is significant when it appears in a downtrend, 
    * while this function does not consider the trend
    */
```

待入线 On-Neck Pattern
------

- 函数原型: integer = CDLONNECK(open, high, low, close)
- 信号: 看跌
- 形态: 持续
- 定义: 第一天是一根疲软的黑色实体，第二天开市价低于第一天蜡烛线的低点，后续走高，但最终收于第一天蜡烛线低点附近。此持续形态仅在下降的趋势中才有意义。
- 最少所需的蜡烛线数量: 2
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLONNECK.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black candle
    * - second candle: white candle with open below previous day low and close equal to previous day low
    * The meaning of "equal" is specified with TA_SetCandleSettings
    * outInteger is negative (-1 to -100): on-neck is always bearish
    * the user should consider that on-neck is significant when it appears in a downtrend, while this function 
    * does not consider it
    */
```

斩回线、刺透形态 Piercing Pattern
------

- 函数原型: integer = CDLPIERCING(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 第一天是一根疲软的黑色实体，第二天开市价位于第一天实体之中，但收市价向上超越黑色实体的中点。此反转形态仅在下降的趋势中才有意义。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLPIERCING.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black candle
    * - second candle: long white candle with open below previous day low and close at least at 50% of previous day 
    * real body
    * The meaning of "long" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100): piercing pattern is always bullish
    * the user should consider that a piercing pattern is significant when it appears in a downtrend, while 
    * this function does not consider it
    */
```

黄包车夫线 Rickshaw Man
------

- 函数原型: integer = CDLRICKSHAWMAN(open, high, low, close)
- 信号: 无法判断
- 形态: 反转
- 定义: 符合十字线定义，且实体位于上下影线的中点附近。此形态并不意味着任何看涨或者看跌。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLRICKSHAWMAN.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    *
    * Must have:
    * - doji body
    * - two long shadows
    * - body near the midpoint of the high-low range
    * The meaning of "doji" and "near" is specified with TA_SetCandleSettings
    * outInteger is always positive (1 to 100) but this does not mean it is bullish: rickshaw man shows uncertainty
    */
```

上升（下降）三法 Rising / Falling Three Methods
------

- 函数原型: integer = CDLRISEFALL3METHODS(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 持续
- 定义: 上升（下降）三法：第一天出现一跟坚挺的白色（黑色）蜡烛线，其次紧挨着三根或以上的依次下降（上升）的小实体蜡烛线，且这些实体都局限于第一个白色（黑色）蜡烛线的高低点之内；最后一天是同样一根坚挺的白色（疲软的黑色）蜡烛线，且开市价应高（低）于前一根蜡烛线的收市价，收市价也应高（低于）于第一天的收市价。
- 最少所需的蜡烛线数量: 5
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLRISEFALL3METHODS.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long white (black) candlestick
    * - then: group of falling (rising) small real body candlesticks (commonly black (white)) that hold within 
    *   the prior long candle's range: ideally they should be three but two or more than three are ok too
    * - final candle: long white (black) candle that opens above (below) the previous small candle's close 
    *   and closes above (below) the first long candle's close
    * The meaning of "short" and "long" is specified with TA_SetCandleSettings; here only patterns with 3 small candles
    * are considered;
    * outInteger is positive (1 to 100) or negative (-1 to -100)
    */
```

分手线 Separating Lines
------

- 函数原型: integer = CDLSEPARATINGLINES(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 持续
- 定义: 看涨（跌）分手线：第一天是一根黑色（白色）蜡烛线，第二天是一根看涨（跌）捉腰带线，且开市价与第一天蜡烛线的开市价在同一水平位置上。仅当第二天的看涨（跌）捉腰带线与将要来临的趋势方向相同时，此持续形态才有深刻的意义。
- 最少所需的蜡烛线数量: 2
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLSEPARATINGLINES.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: black (white) candle
    * - second candle: bullish (bearish) belt hold with the same open as the prior candle
    * The meaning of "long body" and "very short shadow" of the belt hold is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish;
    * the user should consider that separating lines is significant when coming in a trend and the belt hold has 
    * the same direction of the trend, while this function does not consider it
    */
```

流星线 Shooting Star
------

- 函数原型: integer = CDLSHOOTINGSTAR(open, high, low, close)
- 信号: 看跌
- 形态: 反转
- 定义: 拥有较小的实体，且实体位于整个价格区间的下端，上影线尽可能长，而下影线尽可能短或者没有，此外要与之前蜡烛线的实体产生跳空。此反转形态仅出现在明显的上升趋势中。
- 最少所需的蜡烛线数量: 1
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLSHOOTINGSTAR.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - small real body
    * - long upper shadow
    * - no, or very short, lower shadow
    * - gap up from prior real body
    * The meaning of "short", "very short" and "long" is specified with TA_SetCandleSettings;
    * outInteger is negative (-1 to -100): shooting star is always bearish;
    * the user should consider that a shooting star must appear in an uptrend, while this function does not consider it
    */
```

短蜡烛线 Short Line Candle
------

- 函数原型: integer = CDLSHORTLINE(open, high, low, close)
- 信号: 无法判断
- 形态: 其他
- 定义: 具有较短实体和上下影线的一根蜡烛线。此形态并不意味着任何看涨或者看跌。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLSHORTLINE.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - short real body
    * - short upper and lower shadow
    * The meaning of "short" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when white, negative (-1 to -100) when black;
    * it does not mean bullish or bearish
    */
```

纺锤线 Spinning Top
------

- 函数原型: integer = CDLSPINNINGTOP(open, high, low, close)
- 信号: 无法判断
- 形态: 其他
- 定义: 具有较短实体和长于实体大小的上下影线的蜡烛线。此形态并不意味着任何看涨或者看跌。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLSPINNINGTOP.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - small real body
    * - shadows longer than the real body
    * The meaning of "short" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when white or negative (-1 to -100) when black;
    * it does not mean bullish or bearish
    */
```

停顿形态 Stalled Pattern
------

- 函数原型: integer = CDLSTALLEDPATTERN(open, high, low, close)
- 信号: 看跌
- 形态: 反转
- 定义: 符合前进白色三兵的定义，且在后两根蜡烛线中，前一根为有较短或者没有上影线的白色实体，且开市价在第一天蜡烛线实体之内或者附近。后一根在前一根实体顶部附近形成的微小跳空，最终收成一个小实体的蜡烛线。此反转形态仅在明显的上升趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLSTALLEDPATTERN.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - three white candlesticks with consecutively higher closes
    * - first candle: long white
    * - second candle: long white with no or very short upper shadow opening within or near the previous white real body
    * and closing higher than the prior candle
    * - third candle: small white that gaps away or "rides on the shoulder" of the prior long real body (= it's at 
    * the upper end of the prior real body)
    * The meanings of "long", "very short", "short", "near" are specified with TA_SetCandleSettings;
    * outInteger is negative (-1 to -100): stalled pattern is always bearish;
    * the user should consider that stalled pattern is significant when it appears in uptrend, while this function 
    * does not consider it
    */
```

条形三明治 Stick Sandwich
------

- 函数原型: integer = CDLSTICKSANDWICH(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 第一天是一根黑色蜡烛线，第二天是一根白色蜡烛线，且低点高于第一天蜡烛线的收市价，第三天又是一根黑色蜡烛线，且收市价与第一天蜡烛线的收市价在同一水平上。此反转形态仅在明显的下降趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLSTICKSANDWICH.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: black candle
    * - second candle: white candle that trades only above the prior close (low > prior close)
    * - third candle: black candle with the close equal to the first candle's close
    * The meaning of "equal" is specified with TA_SetCandleSettings
    * outInteger is always positive (1 to 100): stick sandwich is always bullish;
    * the user should consider that stick sandwich is significant when coming in a downtrend, 
    * while this function does not consider it
    */
```

探水竿 Takuri (Dragonfly Doji with very long lower shadow)
------

- 函数原型: integer = CDLTAKURI(open, high, low, close)
- 信号: 无法判断
- 形态: 反转
- 定义: 符合十字线定义，且拥有非常短的或者没有上影线，相对的，下影线应该足够的长。通常与蜻蜓十字相比，下影线要更加的长。此反转形态并不意味着看涨，要结合原有趋势判断。
- 最少所需的蜡烛线数量: 1
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLTAKURI.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    *
    * Must have:
    * - doji body
    * - open and close at the high of the day = no or very short upper shadow
    * - very long lower shadow
    * The meaning of "doji", "very short" and "very long" is specified with TA_SetCandleSettings
    * outInteger is always positive (1 to 100) but this does not mean it is bullish: takuri must be considered
    * relatively to the trend
    */
```

跳空并列阴阳线形态 Tasuki Gap
------

- 函数原型: integer = CDLTASUKIGAP(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 持续
- 定义: 向上（下）跳空并列阴阳线：第二天出现了一根向上（下）跳空的白色（黑色）蜡烛线，紧接着第三天为一根黑色（白色）蜡烛线，长度与第二天相似，开市价位于第二天的实体之中，且并未回填跳空缺口。此持续形态仅在明显的趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLTASUKIGAP.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - upside (downside) gap
    * - first candle after the window: white (black) candlestick
    * - second candle: black (white) candlestick that opens within the previous real body and closes under (above)
    *   the previous real body inside the gap
    * - the size of two real bodies should be near the same
    * The meaning of "near" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish;
    * the user should consider that tasuki gap is significant when it appears in a trend, while this function does 
    * not consider it
    */
```

插入线形态 Thrusting Pattern
------

- 函数原型: integer = CDLTHRUSTING(open, high, low, close)
- 信号: 看跌
- 形态: 持续
- 定义: 第一天是一根疲软的黑色实体，第二天开市价低于第一天蜡烛线的低点，后续走高，但最终收于第一天蜡烛线实体中点之下。为了和切入线区分，收市价不应该非常接近第一天的收市价。此持续形态仅在下降的趋势中才有意义，而在即将到来的上升趋势或是数天内出现多次，可能反而意味着看涨。
- 最少所需的蜡烛线数量: 2
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLTHRUSTING.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black candle
    * - second candle: white candle with open below previous day low and close into previous day body under the midpoint;
    * to differentiate it from in-neck the close should not be equal to the black candle's close
    * The meaning of "equal" is specified with TA_SetCandleSettings
    * outInteger is negative (-1 to -100): thrusting pattern is always bearish
    * the user should consider that the thrusting pattern is significant when it appears in a downtrend and it could be 
    * even bullish "when coming in an uptrend or occurring twice within several days" (Steve Nison says), while this 
    * function does not consider the trend
    */
```

三星形态 Tristar Pattern
------

- 函数原型: integer = CDLTRISTAR(open, high, low, close)
- 信号: 无法判断
- 形态: 其他
- 定义: 由三根十字线组成，中间一根为十字星。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLTRISTAR.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - 3 consecutive doji days
    * - the second doji is a star
    * The meaning of "doji" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish
    */
```

奇特三川底部形态 Unique 3 River
------

- 函数原型: integer = CDLUNIQUE3RIVER(open, high, low, close)
- 信号: 看涨
- 形态: 反转
- 定义: 第一天是一根疲软的黑色实体，第二天是一根黑色蜡烛线，低点低于第一天蜡烛线的低点，且与其组合成孕线形态。第三天是一根小的白色实体，且开市价不低于第二天蜡烛线的低点。最好是第三天的开市价与收市价均低于第二天蜡烛线的收市价。此反转形态仅在明显的下降趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLUNIQUE3RIVER.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: long black candle
    * - second candle: black harami candle with a lower low than the first candle's low
    * - third candle: small white candle with open not lower than the second candle's low, better if its open and 
    *   close are under the second candle's close
    * The meaning of "short" and "long" is specified with TA_SetCandleSettings
    * outInteger is positive (1 to 100): unique 3 river is always bullish and should appear in a downtrend 
    * to be significant, while this function does not consider the trend
    */
```

向上跳空两只乌鸦 Upside Gap Two Crows
------

- 函数原型: integer = CDLUPSIDEGAP2CROWS(open, high, low, close)
- 信号: 看跌
- 形态: 反转
- 定义: 第一天是一根坚挺的白色实体线，第二天为一根黑色实体线与白色实体线之间的跳空。第三天黑色蜡烛线的开市价位于第二天黑色蜡烛线的开市价之上，且收于第一天蜡烛线的开市价之上，完全吞没第二天的黑色蜡烛线。此反转形态仅在明显的上升趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: -100 ~ -1，越小的数值表示越确定该形态
- 源文件名: `src/ta_func/ta_CDLUPSIDEGAP2CROWS.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: white candle, usually long
    * - second candle: small black real body
    * - gap between the first and the second candle's real bodies
    * - third candle: black candle with a real body that engulfs the preceding candle 
    *   and closes above the white candle's close
    * The meaning of "short" and "long" is specified with TA_SetCandleSettings
    * outInteger is negative (-1 to -100): upside gap two crows is always bearish; 
    * the user should consider that an upside gap two crows is significant when it appears in an uptrend, 
    * while this function does not consider the trend
    */
```

向上（下）跳空三法形态 Upside / Downside Gap Three Methods
------

- 函数原型: integer = CDLXSIDEGAP3METHODS(open, high, low, close)
- 信号: 看涨（跌）
- 形态: 持续
- 定义: 向上（下）跳空三法形态：第一天是一根白色（黑色）蜡烛线，第二天是一根白色（黑色）蜡烛线，且第二天蜡烛线的实体与第一天蜡烛线的实体形成向上（下）跳空。第三天是一根黑色（白色）蜡烛线，且开市价位于第二天蜡烛线实体之内，最终收于第一天蜡烛线实体之内，回填了跳空。此反转形态仅在明显的趋势中才有意义。
- 最少所需的蜡烛线数量: 3
- 输出: 1 ~ 100，越大的数值表示越确定看涨的形态；-100 ~ -1，越小的数值表示越确定看跌的形态
- 源文件名: `src/ta_func/ta_CDLXSIDEGAP3METHODS.c`

源文件关键引用：
```
/* Proceed with the calculation for the requested range.
    * Must have:
    * - first candle: white (black) candle
    * - second candle: white (black) candle
    * - upside (downside) gap between the first and the second real bodies
    * - third candle: black (white) candle that opens within the second real body and closes within the first real body
    * outInteger is positive (1 to 100) when bullish or negative (-1 to -100) when bearish;
    * the user should consider that up/downside gap 3 methods is significant when it appears in a trend, while this 
    * function does not consider it
    */
```

### References

- [「日本蜡烛图技术——古老东方投资术的现代指南」](https://item.jd.com/10090445.html)
- [术语表A 蜡烛图技术术语及示意图小词典](http://www.818qihuo.com/book/ribenlazhutu/4149.html)
- [TA-Lib学习笔记-K线模式识别](https://xueqiu.com/2729190594/80199191)
- [QuantShare Programming Language - PDF](https://www.quantshare.com/download/QuantShareLanguage.pdf)