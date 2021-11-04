---
title: 两岸差异用词爬虫
date: 2017-09-10 14:14:14
categories: 数据
tags: Java
---

写在开头 - **Java 大法好**

昨天我在把一个电影字幕从繁体转成简体时，发现转完后有一些台湾地区的用词（繁体字幕大部分都是台湾的），所以突发灵感想写个翻译程序。

首先得有个词库，[Chinese Linguipedia][2] 这个网站，里边有个两岸差异用词，看上去挺全的，就选它了。

<!-- more -->

词语有三级分类，每页最多十行，链接如此：
`http://chinese-linguipedia.org/clk/diff?class=二级分类编号&classNa=三级分类名称&page=页面`

编号范围是 1\~34，其中 1\~5 是五个一级分类的编号。因为爬下来的数据要细分，所以就忽略大分类，直接从分类 6~34 每一页遍历过去获取。
同时，只有 6, 7, 8 三个二级分类下有三级分类，这些分类我手打进了一个 [json][3] 里。

然后是网页处理部分

每一页都有几段这样的 html：

```html
<td align='center';>同實異名</td>
<td class="head3" style="font-family:'微軟正黑體';font-size:1.1em;"><a href=""><a href="http://chinese-linguipedia.org/clk/search/刀具/0/0?srchType=1">刀具</a></a></td>
<td class="head3" style="font-family:'微軟正黑體';font-size:1.1em;"><a href=""><a href="http://chinese-linguipedia.org/clk/search/刀具/0/0?srchType=1">刀具</a>/<a href="http://chinese-linguipedia.org/clk/search/刃具/0/0?srchType=1">刃具</a></a></td>
```

首先通过正则表达式 `<td align='center';>(.*?)</td>\r\n(.*?)\r\n(.*)`
取出 $1 “同異類別”、$2 “臺灣語詞” 的元素、$3 “大陸語詞” 的元素
然后用另一个正则表达式 `1\">([\s\S]*?)</a` 匹配 $2 和 $3，得到每个“臺灣語詞”和“大陸語詞”，之后以`/`为分隔符合并
向 *output.csv* 文件写入一行：`一级分类,二级分类,三级分类,同異類別,臺灣語詞,大陸語詞`

有人可能注意到上面用了一个 `[\s\S]`，这里是因为网站的数据有点问题，有一条记录的词里边有个换行，导致获取不到这条记录。

合并字符串集合或数组，用到了一个 Java 8 String 类的新方法 String.join，第一个参数是分隔符字符串，第二个参数是一个集合或者数组

至于翻译程序，晚点再来弄，目前计划是检测到一个台湾用词，就显示出相对应的所有大陆用词，可以自定义更改的词语。

需要考虑到一个词语重叠问题，和类似于“簡訊地址”这样的词语的翻译问题，应该需要翻译为“联系方式”。

还有一些这个词库里没有涉及到的词语，例如“聯絡”和“联系”，网站上面显示的是大陆“今不用”此词语，可以通过增加的方法来完善。

**同时这个网站支持通配符查询功能，应该可以将整部字典都爬下来，只是一个时间问题。**

若有哪位大神比较熟悉于这方面的问题，我们可以互相讨论一下，来共同完成这个项目。

本项目的 GitHub：
[https://github.com/yescallop/ChineseLinguipediaCrawler][1]

PS: 微软系列产品不能正确识别无 BOM 头的 UTF-8 编码文件，需要在写入数据之前在文件头写入 BOM 头 `0xEFBBBF`

PPS: 这个网站早已大改，我对 Java 的记忆也已变淡，谨以此篇留作纪念。

[1]: https://github.com/yescallop/ChineseLinguipediaCrawler
[2]: http://chinese-linguipedia.org/clk/diff
[3]: https://github.com/yescallop/ChineseLinguipediaCrawler/blob/master/src/structure.json
