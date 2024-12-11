---
title: "周报 #25 - 基于 Crossbell 的个人信息输出与同步系统（重构）"
date: 2023-01-09T19:12:56+08:00
draft: false
tags: ["review", "life", "home", "mood", "miniflux", "rss", "information", "serverless", "pkm", "system", "xlog", "xsync", "crossbell"]
categories: ["Ideas"]
authors:
- "pseudoyu"
---

{{<audio src="audios/here_after_us.mp3" caption="《后来的我们 - 五月天》" >}}

## 前言

本篇是对 `2023-01-01` 到 `2023-01-09` 这周生活的记录与思考。

这是 2023 年的第一篇周报，明明感觉跨年似乎还在昨天，一月上旬却已然结束，大概是心理上对于时间的感知愈加迟钝了吧。

跨年时写了一篇还算详尽的年度总结，将这过去的一年中的发生的种种悉数道来，写完后发现篇幅已经超过预计，再加上新年计划与期待这一块当时也还没有理清思绪，所以略过了，所以也就趁着新年伊始的这篇周报，浅立一下 Flag 了，有些是小小的习惯养成，有些是长远的充满不确定性的规划，也不知道未来的这一年是否能如愿，但列出来了就会更有一些动力，也当作是一种监督了。

宅了快两个月，周末终于决定出门去朋友家蹭饭，度过了快乐的一天（不然感觉自己都已经不知道怎么跟人面对面讲话了）；虽然出片率堪忧，但可算是把照片修出来了，发了两篇摄影集；整理自己的各种软硬件服务（每年的仪式感了，总觉得整理后就会更有动力重新开始）；整理的时候想起之前的一些小计划，搭建了一个网站运行了 [IPFS 版 ZLibrary](https://zlib.pseudoyu.com/)，却得到了意料之外的关注，吓得我连夜优化服务器和线路；以及很多有意思的事。

## 个人服务重构

### 服务管理

跟往年一样。开年就整理了自己的各种服务，发现居然已经有 20 个之多，且半数是 serverless，这一年白嫖功力见长。为了方便管理，用开源的 Uptime Kuma 为搭建了一个监控服务，并且绑定了 Telegram Bot 提醒，放心了许多。

![uptime_kuma_yu_services](https://image.pseudoyu.com/images/uptime_kuma_yu_services.png)

说来有趣，其实自己之前一直觉得用服务器管理网站很麻烦，每次迁移或是服务变动总是很头疼，所以把大部分的服务都托管到了 Railway、Vercel、Supabase 这几个 Serverless 平台，因为大多是一些个人的服务，没什么太高配置需求，安全稳定就够了，就一直没折腾 Nginx 反向代理、https 证书这些。

而之前有提到过最近帮一个二次元同学做 B 站直播的房管和技术支持，就想着用一台白嫖的甲骨文日本机器来专门做直播监控和自动录制。因为有时候朋友也需要能够直接查看和下载，那自然一个好记的域名、国内网络环境下的访问速度、下载带宽等都要考虑在内，Serverless 服务就已经远远不够（也并不太划算了），于是探索了一些方案，选择了 [Nginx Proxy Manager](https://nginxproxymanager.com/) 这一便捷的反向代理工具。

![npm_yu_dashboard](https://image.pseudoyu.com/images/npm_yu_dashboard.png)

我在一台线路比较好（CN2 GIA）的搬瓦工机器上进行部署，托管自己的各项服务，能够保障还不错的访问体验。因为也可以通过通配符匹配的方式直接为自己的 `*.pseudoyu.com` 子域名统一签发 https 证书和自动续期，很省心。配合上述的监控，目前使用了一周，还挺舒服的。

官方文档很清晰详细，配合 docker-compose 这样人性化的容器服务管理方式，操作起来上手很快，不过可能还是会考虑后面出一个教程，让想托管一些像是博客这些小服务的朋友们有所参考。

### RSS 输入

2022 年其实大多都还是专注在博客输出以及自己的 Telegram 频道上，对于输入和各个平台同步这一块其实没花太多心思，导致自己的 RSS 订阅堆积，newsletter 也有些过载，反倒是没能好好过滤输入信息源，于是删除了用了很久的 NetNewsWire，通过 Railway + Supabase 的方式搭建了一个更轻量级的 Miniflux，作为自己的主要阅读器，并且对 RSS 信息源作了筛选，控制在了 52 个，几乎都是个人博客，后续也会继续优化调整。

![miniflux_yu_page](https://image.pseudoyu.com/images/miniflux_yu_page.png)

虽然有了 Miniflux 提供了还不错的阅读体验，但我其实更习惯于点进原文，我总是觉得对于个人博客来说，不仅仅是内容，网站的风格设计、一些相关的文章和主题也都是属于博主不可或缺的一部分，才能带来阅读更完整的享受。

RSS 阅读器对于我更多是作为第一步聚合工具，而由于 Miniflux 是一个基于网站的服务，并没办法做好很即时的提醒，而我每天又高度依赖 Telegram，所以基于 [RSS to Telegram Bot](https://github.com/Rongronggg9/RSS-to-Telegram-Bot) 搭建了自己的 Telegram 提醒，将这些信息源更新推送给我，看到一些感兴趣的标题会留个印象，空闲时统一到 Miniflux 去阅读查看。

![yu_rss_to_tg_bot](https://image.pseudoyu.com/images/yu_rss_to_tg_bot.png)

这样下来也比较不容易错过想看的文章，也不至于造成太多信息堆积，目前这套方案使用下来感觉很不错，顺便每次周末看到各种周报的时候也催更效果显著（~~这周日出去玩了，合理拖更~~）。

### Telegram 输出

我同样基于 Railway + Supabase 方式搭建了一个自己的 n8n 同步服务，将自己的各平台输入同步到频道，详细描述可以看这篇『[周报 #12 - 赛博空间、自我定义与界限](https://www.pseudoyu.com/zh/2022/09/19/weekly_review_20220919/)』。

之前平台基于 [Reorx](https://github.com/reorx) 的方案进行了一些自己的调整，但一直没添加更多信息源，国内源较少。

虽然自己自己目前国内的各个平台以及极少进行分享，但也总共是自己的一部分，再加上新增了少数派作为自己的一些工作效率类文章发表渠道，所以在朋友[涂俊杰 JunJie](https://blog.tujunjie.com/) 推荐了 RSSHub 与 n8n 集成这种形式后，我在服务器上部署了一套 [RSSHub](https://github.com/DIYgod/RSSHub) 服务体验了一下，顿时感觉是很惊艳的解决方案，火速给自己的 Telegram 信息流频道加上了网易云、微博、B 站和少数派的同步支持，内容更加丰富了。

### Crossbell 同步

虽然像是 Twitter、Telegram 已经是比较大的平台，但毕竟是中心化的产物，再加上最近的各种风波，对于自己这些信息源的归集总是不放心 Telegram 作为最终站，尤其是我常常在删消息时差点误点删除全部（奇怪的交互体验），所以信息的同步导出部分也是很重要一环。

自己之前提到过的 Side Project 也算是在做这样的事，不过作为一个 Web3 从业者，自然也是眼馋基于区块链的解决方案很久了。其实毕业设计也是做的[基于 Ethereum 和 IPFS 的数据所有权保护 ÐApp](https://github.com/pseudoyu/uright) 项目，不过我那个纸糊的 Demo 项目自然是没法满足自己的各种需求，而当时的代码写得实在太乱也没有去重构的欲望了，于是开始找寻链上解决方案。

好久之前关注了 [Crossbell](https://crossbell.io/)，也莫名机缘巧合结识了不少 [RSS3](https://rss3.io/) 的朋友，但对 Crossbell 之前的印象还停留在 [Diygod](https://diygod.me/) 在推特上发的 [CrossSync](https://crosssync.app/) 浏览器插件是基于这个链的，当时手机打开的链接，关联钱包并不方便，所以搁置了。

所以想着去官网逛一下，结果发现居然已经有了 [xLog](https://xlog.app/)、[xSync](https://xsync.app/)、[xChar](https://xchar.app/)、[xFeed](https://crossbell.io/feed) 等好几项应用，而最关注的 xSync 居然还刚好支持 Telegram Channel，完美匹配了我的需求。

#### xLog 同步发布博文

于是开始一番配置和装修，首先是将自己个人思考相关的博文同步到了 xLog 上，视觉效果和体验感都不错，且基于 Crossbell 地址能够很方便地进行 follow 和评论。

这是我的 xLog 访问地址：[xlog.pseudoyu.com](https://xlog.pseudoyu.com/)，有兴趣的朋友们也可以关注一下，不过目前出于定制化程度、各种历史文章迁移路由问题、自己各项数据统计服务变动等考虑，还是更多作为一个同步分发渠道，暂不打算把博客彻底迁移过去。

![yu_xlog_homepage](https://image.pseudoyu.com/images/yu_xlog_homepage.png)

自带的 [NFT 展示柜](https://xlog.pseudoyu.com/nft)很不错，应该是集成的 [Unidata](https://unidata.app/)，之前就想集成到我的 Hugo 博客里，但一直没来得及动工（~~有现成的就更懒了~~）。

![yu_xlog_nft](https://image.pseudoyu.com/images/yu_xlog_nft.png)

#### xSync 自动同步 Telegram 和 Twitter

看到 xSync 能够同步 Teleram Channel 数据的时候真的很惊喜，完全不需要再做任何改造就能把我的聚合频道作再一次备份与存档，也很快配置上了，~~瞬间有点想舍弃自己 Side Project 的冲动~~。

![yu_xsync_homepage](https://image.pseudoyu.com/images/yu_xsync_homepage.png)

不过有些遗憾的就是历史数据只同步了一部分，之前没接入时的数据似乎也没有手动备份同步的选项，不知道有没有配置项或者后续功能可以解决，或者有 RSS3 的朋友知道解决方案的可以说一下，感谢！

都配置好后就可以通过 xChar 来查看自己的各项消息了，很完美的解决方案，这是我的 xCharacter 个人主页: [xchar.app/pseudoyu](https://xchar.app/pseudoyu)，也可以查看我的信息流。

![yu_xchar_profile](https://image.pseudoyu.com/images/yu_xchar_profile.png)

另外的一个小插曲就是看到要把 `pseudoyu@crossbell` 放到简介时会心一笑，我当时毕业设计做版权保护 ÐApp 的时候是在 Solidity 合约里使用了 Oraclize API 来访问链下数据，也是抓取的 Youtube 的简介里的唯一标识来作为帐号所有权凭证，有种奇妙的熟悉感哈哈，后面有机会研究一下代码。

这套基于 Crossbell 的信息输入输出解决方案可以说是重构了我原本的个人管理系统，也希望能够结合这个系统做一些自己的尝试。

## 新年计划

似乎每年列一些年度计划已经是不成文的习惯，但是自己过去的那么多年里也少有照做实现的。今年增加了更多的公开表达渠道，似乎能够更加有动力去做践行。

之前看过 [Xuanwo](https://xuanwo.io/) 的一篇『[2022-37: 基于 Github 的公开工作流程](https://xuanwo.io/reports/2022-37/)』，稍微研究了一下 GitHub Projects，觉得简洁却也够用，虽然平时也基于 Logseq 做一些基础的 GTD，但依然很难作为看板来使用，今年会试试，也给自己一些对应的压力。

新年计划的粒度很难把控，就随心所欲了，不写那么大而空的了，更多是一些指标吧，有的是自由探索的想法，有的是一些长期的目标，也有一些短期要完成的事，采取了勾选框这样的形式，也许后面想起来会继续添加，期间完成或新的一年年终总结的时候会来回顾 check 一下。

- [ ] 好好照顾捏捏，保护好她
- [ ] 去日本 or 回香港工作/一份享受其中的远程工作/自由度满意的工作模式，按照优先级三选一吧
- [ ] 至少去 6 个没去过的城市旅游，最好能见见久违的朋友，虽然不多
- [ ] 坚持每周写周报，完成 48 篇
- [ ] 除了周报外，至少再更新 48 篇原创博文，技术为主
- [ ] 多外出拍照，新开的摄影集栏目至少更新 12 篇（元旦已经冲 kpi 发了两篇了），并且深入学习一下构图、色彩与后期
- [ ] 为 GoCN 贡献至少 12 篇译文
- [ ] 少数派发布 10 篇文章，赚猫罐头钱
- [ ] 开始做 B 站 up 与 Youtuber，至少发 10 个视频，不能太水
- [ ] 坚持每周锻炼/跑步至少四天（健身环或 Keep 也算），也同样会在周报中记录打卡
- [ ] 坚持练吉他，录至少 3 首歌并发布
- [ ] 捡回滑板技能，每周至少练习滑两次
- [ ] 读至少 24 本有意义的书，但不能囫囵吞枣，需要在豆瓣等平台发布自己的感想
- [ ] 日语 N2 证书，为了之后日本的一些计划做的筹备，学习进度会在周报里单开一个模块打卡，可能会突击一下选择 7 月的考试，~~实在不行 12 月再来一次~~
- [ ] CKAD 证书，去年就准备了一半，不过后来忘记报名购买考试了，没有了压力果然会偷懒
- [ ] 为更多开源项目贡献代码，不要求量，但希望有更多有意义的提交
- [ ] 为自己的开源工具箱项目『[Yu Tools](https://github.com/pseudoyu/yu-tools)』写一个展示网站，以及为其中的软硬件条目都写使用体验（大工程了），不断优化迭代
- [ ] 完善『[Blockchain Guide](https://guide.pseudoyu.com/)』这一开源指南项目，把过去这一年工作学习的区块链底层与 Web3 相关的项目经验工程经验都多涵盖一点，惭愧的是大部分文章还是在香港读研的时候写的
- [ ] 和朋友一起做的 Side Project 创业项目顺利上线并且不断优化
- [ ] 探索更多有意思的技术，继续享受其中
- [ ] 认识更多有趣的人
- [ ] 好好生活下去

## 个人生活剪影

从 11 月北京疫情严重开始，我就开始了长达两个月的宅家生活，大概多少是有着不错的物理防御属性（指把当时手头上唯一的一点药寄给了朋友，纯靠不出门来隔绝病毒）和幸运点数（每天照常点外卖，中途还有物业来家里处理漏水问题一下午），自己到现在还保持着阴性，已经在决赛圈了。

但随之而来的后果就是已经康复转阴的朋友已经在四处旅游，而我依然连倒个垃圾都全副武装，更不敢出远门了，就这样和猫猫度过了两个月。

虽然确实是宅，但随着疫情放开也确实没个头，所以心态也佛了，这周末应邀（~~并不，只是以携猫拜访的名义去玩~~）去了博译学姐家蹭饭，呼吸到了外面并不那么新鲜的空气（~~毕竟北京~~），也吃上了久违的家常菜，摆了一天，却心安理得且快乐。

![wonderful_meal_with_boyi](https://image.pseudoyu.com/images/wonderful_meal_with_boyi.jpg)

打算 1.18 回杭州了。其实 2022 年回家时间在近几年里已经不算短了，各种调休和假期回家前后加起来可能有 1 个月，只是常常疫情反复，也没来得及回老家一趟。两年前的 1 月外婆离世，困于香港疫情没能回家，去年春节又因为突然的疫情而滞留在京，是该回去看看了，越长大，去的地方越来越多，家却也离我愈发遥远了。

其实前段时间就一直在犹豫回家的事，担心再有什么变故，但还是想回去看看，但这种局势下又不放心给猫舍或者不熟悉的人看管。后来一次开会闲聊时偶然提到，有了解决方案，定了捏捏会寄养在我的项目的小 leader 家里，他女儿眼馋猫猫很久了，安顿好后终于放下了悬着的心。

这样一路折腾估计十有八九是要阳的，得知这个，学姐还给了我豪华抗疫大礼包，感人。

![medicines_from_boyi](https://image.pseudoyu.com/images/medicines_from_boyi.jpg)

然后前段时间博译学姐在灵隐的时候帮我许愿了”2023 都能如愿做自己喜欢的事、能够探索更多爱好“，还带了一个好看的佛珠手饰送给我了，我单方面宣布是天下第一好的学姐了，希望新的一年也都能好好的。

突然想起其实之前大学的时候有一年多一直带着倪给我的一个同样是灵隐的佛珠，直到线快磨断了、珠子摇摇欲坠才收起来，莫名觉得那一年确实幸运了许多，有时可能只是需要一些心静吧。

实现了会去还愿的，双份的愿望。

![wonderful_gift_with_boyi](https://image.pseudoyu.com/images/wonderful_gift_with_boyi.jpg)

## 其他

这个部分会记录一下自己的输入输出以及其他觉得有意思的东西。

这周在 B 站看了两个很有感触的视频，一个是来自我最喜欢的 Up 主『[小鹿 Lawrence](https://space.bilibili.com/37029661)』的『[这是我最拼的一年，却让公司缩小了一半｜2022 年终总结](https://www.bilibili.com/video/BV1Mx4y1G7zW)』，有一些感触：

> 接连着看了好几年了，年终总结这个保留栏目每次也都会看好多好多遍。

> 有过处于同阶段的感同身受，为一些视频而触动；有过动态被鹿哥回复、鼓励了，开心了好多好多天；更多的是陪伴着我度过一个又一个深夜，醒来继续努力生活。可能由于太过熟悉，刚开新工作室门时细微的一点停顿，讲那句“因为曾经家人的支持是你的底气”时的哽咽，花束的 BGM，回顾这一年时的一声苦笑，都让我的情绪随之起伏与落泪。

> “不是你长大了你就变了，而是你长大了，世界才开始对你展露全部的真相”。也许自己常常被描述的少年感、学生气也不过是过去经历的那些幸运的透支，与身边人对我的保护，才能在自己的周报里一次次谈论自我，一次次向往美好。而在 2022 年，一切也都回到原点。所幸，还能保留着『记录』这种习惯，还未丧失『感受』这项能力，微小，却弥足珍贵。

> “这一年失去的东西太多太多，任何一点细小的死亡与崩坏，都会变得不可承受”。是啊，2022 就是很难啊，不可名状。新的一年，也要勉强努力一个人生活下去。

> 感谢鹿哥过去一年的陪伴与带来的感动，新的一年，加油！

还有一个很犀利的 Up 主『[老蒋巨靠谱](https://space.bilibili.com/119801456)』的『[和不可名状的非必要一年说再见——我的新年献词](https://www.bilibili.com/video/BV17M411y7es)』，感想：

> 太喜欢老蒋的思考和叙述风格了，平实、真诚但却大胆而不失尖锐，是看过得最好的新年献词了。

> 2022 就是这样过去了，很多事不可说，很多事正在发生，很多事再也不会发生，不可名状大概也是最好的形容了。

## 总结

2023 年的第一周，今年是个还不错的开端吧。