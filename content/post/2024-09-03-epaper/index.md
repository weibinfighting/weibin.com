---
title: 墨水屏日历
author: 瞿伟斌
date: '2024-09-03'
slug: epaper
categories:
  - code
---

在当前的时间下，使用日历已经是非常小众的一个需求了，在过去因为想要让每一天都过得更加重视，所以总是会使用一些翻页或者撕页的日历，让每一天都过得更有仪式感一点。然而令人遗憾的是，往常一向稳定的单向历今年迎来了重大的改版，从原先每天爽快一撕的直接到这次小心翼翼依然一撕过一个月的荒诞设计，让我们大感苦恼。有时日历放在那里，已经很久不曾动过。
机缘巧合下看到网上有人用墨水屏定制了一个[电子日历](https://sspai.com/post/82704)，看到墨水屏在日历这种低功耗（一天只需要刷新一次）、低清晰度（不需要特别高的分辨率）场景下确实有着独特的优势。考虑到最近正好在考虑送女朋友礼物的事宜，外加网友提供了不少[开源代码](https://github.com/Ymriri/esp32_7color)，二话不说我就下单了了一块墨水屏，开始了制作墨水屏日历之路。

## 需求确定
基于我对自身实力的了解（菜且懒），我没有选择大家通用的后端运行每日更新的模式，而是偷懒选择了离线模式，只需要制作好每一天的日历照片，然后每天定时更新就好了，省时省力（偷懒有方），为此只需要达成几个步骤就可以实现最后的目标：

1. 用于展示的墨水屏设备
2. 每天的日历信息获取
3. 日历配图获取
4. 日历配文获取
5. 生成用于展示的图片
6. 导入墨水屏并完成每日刷新设置

本以为是一个非常简单的事情，没想到最终花费了远超预期的时间才顺利完成。

## 日历制作流程

### 1.墨水屏设备

首先是采购，基于需求出发，结合实际展示效果，因为要配彩色的图，所以我还是希望能够用能显示彩色的墨水屏，而微雪刚好有一款非常符合需求的[产品](https://www.waveshare.net/shop/PhotoPainter.htm)，虽然价格不太美丽，但考虑到省时省力的需求，还是下单了。

### 2.日历信息获取

日历信息获取相对来说则容易许多，因为日期是不会变的，所以只需要根据给定日期获取到当前信息如阴历、节日、节气等即可，然后按照格式输出即可，这一用python的相关库就可以很容易的完成，代码如下：

```python

import datetime
from borax.calendars.lunardate import LunarDate
from borax.calendars.festivals2 import FestivalLibrary, WrappedDate

def get_lunar_and_weekday(date):
    library = FestivalLibrary.load_builtin()
    week_list = ["星期一","星期二","星期三","星期四","星期五","星期六","星期日"]
    lunar_month_list = ["正月","二月","三月","四月","五月","六月","七月","八月","九月","十月","十一月","腊月"]
    lunar_day_list = ["初一","初二","初三","初四","初五","初六","初七","初八","初九","初十","十一","十二","十三","十四","十五","十六","十七","十八","十九","二十","廿一","廿二","廿三","廿四","廿五","廿六","廿七","廿八","廿九","三十","卅一"]

    # 获取星期几
    weekday = date.strftime("%A")

    # 获取农历日期
    lunar_date = LunarDate.from_solar_date(date.year, date.month, date.day)
    lunar_year = lunar_date.cn_year
    lunar_month = lunar_month_list[lunar_date.month-1]
    lunar_day = lunar_date.cn_day
    lunar_week = '星期'+lunar_date.cn_week
    lunar_term = lunar_date.term #节气
    if len(library.get_festival_names(date.date()))==0:
        festival_names = ''
    else:
        festival_names = library.get_festival_names(date.date())[0]
    return {'weekday':lunar_week,'lunar_year':lunar_year,'lunar_month':lunar_month,'lunar_day':lunar_day,'term':lunar_term,'festival_names':festival_names}
```

### 3.日历内容获取

相比于前两步只需要花钱或调包就能解决的问题，日历配图这一步确实是难倒我了。本来是想着根据网友经验，使用[大都会博物馆在线随机获取](https://metmuseum.github.io/)，但是随机获取了几张图片后发现存在一些问题，首先就是图片的主题往往不能很好地与日期相匹配，总不能在七夕节放一张《苏格拉底之死》。其次就是不同时期的油画有不同的风格，往往很难统一，而如果进行人工筛选，则工作量过大，与初衷相反，为此最好的方法还是另寻他途。

感谢[网友使用为你读诗公众号](https://github.com/Ymriri/esp32_7color)的封面图和标题为我提供了思路。刚好我有关注与其类似的微信公众号————读首诗再睡觉，风格也符合我的喜欢，更重要的是，他们有自己的[网站](https://bedtimepoem.com/)。这么一来二去，爬虫技巧时隔半年再次启动！好在“读睡”的网站做的还是比较简单的，爬起来没有太大的难度，而且因为时间跨度较大，我也就贪心了一下，一口气爬了6年的数据有备无患！

```python
def download_image(url, filename):
    """
    下载图片并保存到指定文件名。
    参数：
    url (str): 图片的 URL。
    filename (str): 保存图片的文件名。
    """
    # 发送请求获取图片数据
    img_data = requests.get(image_url,headers=headers).content
    # 将图片数据写入文件
    with open(filename, 'wb') as f:
        f.write(img_data)
    file_size = os.path.getsize(filename)
    if file_size < 350:
        wait_time = random.randint(1, 3)
        time.sleep(wait_time)
        img_data = requests.get(image_url,headers=headers).content
        with open(filename, 'wb') as f:
            f.write(img_data)

def get_next_url(soup,url=base_url):
    """
    获取下一页的链接。
    参数：
    soup (BeautifulSoup): 解析后的网页内容。
    返回值：
    str: 下一页的链接。
    """
    try:
        next_page = soup.find_all('ul',class_='next-prev-post-nav')[0].find('a',rel='next')['href']
    except:
        wait_time = random.randint(1, 5)
        time.sleep(wait_time)
        response = requests.get(base_url,headers=headers)
        soup = BeautifulSoup(response.text, 'html.parser')
        next_page = soup.find_all('ul',class_='next-prev-post-nav')[0].find('a',rel='next')['href']
    return next_page
```

![读首诗再睡觉的数据](https://image-home-qwb.oss-cn-shanghai.aliyuncs.com/image/20241018103635.png)

### 生成图片

现在日历、配图、配文都有了，下一步就是把这些文件都用python生成一张图片，然后再上传到墨水屏上。这一步就大量参考开源的代码，因为网友使用的是5寸的墨水屏，而我实际使用的是7.5寸的墨水屏，考虑到实践显示效果，还需要对参数进行微调。包括图片比例，字体大小等，然后再在页面上加一些小小的意外惊喜。

![生成的图片](https://image-home-qwb.oss-cn-shanghai.aliyuncs.com/image/20241018105222.png)

### 导入墨水屏

现在图片有了，墨水屏也有了，
然后就是将图片直接拷贝到墨水屏中，就大功告成了！

**才怪！**

在设置墨水屏的刷新格式，改为每24小时刷新后，我直接一口气导入了两年的日历，结果墨水屏直接刷新无效，在翻阅了[产品说明](https://www.waveshare.net/wiki/PhotoPainter)后，依旧没法解决这个问题。最后不得不求助于微雪的官方客服，在和技术人员沟通了一番后才明白，出场设置的设备默认是有数量限制的，同时对于图片的文件名也有严格的要求，要求尽量用数字标识，而不要用汉字和其他符号。

在经过一番折腾后，终于成功导入了几千张日历图片，并且每24小时刷新一次，从成品结果来看简直完美！

![成品图](https://image-home-qwb.oss-cn-shanghai.aliyuncs.com/image/18839f2a1757c6d516fdad95b073248.jpg)

### 感想

这次日历本想着在各位网友无私贡献的代码下能够很快的完成，结果没想到中间遭遇了诸多波折，先是图片获取存在困难，原先爬取的几百张图片无一能用，再是图片格式再遇挫折，不同的墨水屏尺寸导致对日历图片中各元素的位置要精确调控，再来就是基于现成的固件，不懂还是要多问啊。
好在历时月余的彩色日历制作之路总算是顺利结束，从最终的结果来看，也是相对满意的（各个方面）。

另墨水屏的硬件还是有待提高，虽然看起来不错，但是显示效果和刷新率以及价格实在谈不上完美。
而在我写完这篇文章后发现墨水屏又降价了……

希望日后能有机会再次优化一下这个日历方案吧。