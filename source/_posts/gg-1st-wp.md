---
title: 1st PKU GeekGame Writeup
date: 2021-12-13 16:16:16
updated: 2021-12-17 20:54:20
categories: Writeup
tags: CTF
---

未完待续.

<!-- more -->

## →签到←

用浏览器打开文件，全选复制，得到两行文本：

```text Original Text
fa{aeAGetTm@ekaev!
lgHv__ra_ieGeGm_1}
```

将两行字符交错置于一行，得 `flag{Have_A_Great_Time@GeekGame_v1!}`.

## 小北问答 Remake

1. 北京大学燕园校区有理科 1 号楼到理科 X 号楼，但没有理科 (X+1) 号及之后的楼。X 是？

    `5`. 搜索可得.

1. 上一届（第零届）比赛的总注册人数有多少？

    `407`. 见[相关报道](https://news.pku.edu.cn/xwzh/203d197d93c245a1aec23626bb43d464.htm).

1. geekgame.pku.edu.cn 的 HTTPS 证书曾有一次忘记续期了，发生过期的时间是？

    `2021-07-11T08:49:53+08:00`. 见 [Censys](https://search.censys.io/). 注意时区.

1. 2020 年 DEFCON CTF 资格赛签到题的 flag 是？

    `OOO{this_is_the_welcome_flag}`. 见 [Scoreboard](https://scoreboard2020.oooverflow.io/).

1. 在大小为 672328094 * 386900246 的方形棋盘上放 3 枚（相同的）皇后且它们互不攻击，有几种方法？

    `2933523260166137923998409309647057493882806525577536`. 归纳可得通项.

1. 上一届（第零届）比赛的“小北问答1202”题目会把所有选手提交的答案存到 SQLite 数据库的一个表中，这个表名叫？

    `submits`. 见 [GitHub](https://github.com/PKU-GeekGame/geekgame-0th/blob/main/src/choice/game/db.py).

1. 国际互联网由许多个自治系统（AS）组成。北京大学有一个自己的自治系统，它的编号是？

    `AS59201`. 见 [IPIP.net ASN Search](https://whois.ipip.net/search/PEKING). 排除 CERNET2 节点 `AS24349`.

1. 截止到 2021 年 6 月 1 日，完全由北京大学信息科学技术学院下属的中文名称最长的实验室叫？

    `区域光纤通信网与新型光通信系统国家重点实验室`. 在该学院[网站](https://eecs.pku.edu.cn/)下搜索并手动比较可得.
