---
title: R语言画含九段线的中国地图
author: 瞿伟斌
date: '2024-11-14'
slug: r
categories:
  - R
tags: []
---

用R画中国地图是我最开始接触R时就一直渴望实现的功能，从早期跟着谢益辉的[教程](https://yihui.org/cn/2007/09/china-map-at-province-level/)画着偏平的中国地图，到后面使用ggplot2的坐标系进行矫正，再发展到使用[echarts4r](https://cosx.org/2021/12/introduction-to-echarts4r/)来进行美观和可交互的地图。时光荏苒，这么多年过去了，R不断进化更新，成为了时空统计，空间地理统计中的利器，新的`sf`包也较原先的地理包强大许多。
但是仍然有一个困扰我许久的问题，那就是如何画出带有南海九段线小框的地图。众所周知，在当下这个时间（2024年），把南海九段线一并画出来并不是一件多么困难的事情。甚至通过设置投影参数可以得到十分美观的中国地图：

```r
library(sf)
```

```
## Linking to GEOS 3.9.3, GDAL 3.5.2, PROJ 8.2.1; sf_use_s2() is TRUE
```

```r
library(terra)
```

```
## terra 1.7.39
```

```r
library(dplyr)
```

```
## 
## 载入程辑包：'dplyr'
```

```
## The following objects are masked from 'package:terra':
## 
##     intersect, union
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(ggplot2)

china_map = read_sf('D:/work/map/中华人民共和国.json')
china_populatino = rio::import('D:/work/数据资料/七普人口-省份.xlsx')
```



```r
mycrs <- "+proj=aea +lat_1=25 +lat_2=47 +lon_0=105 +x_0=0 +y_0=0 +ellps=GRS80 +datum=WGS84 +units=m +no_defs" # mycrs设定为最佳Albers投影参数
china_map|>
  st_transform(mycrs)|>
  ggplot()+
  geom_sf(aes(fill=人口))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />


如果只是进行研究分析或者图像展示，这样的图像已经没有问题了，但是在日常写报告中，这样的图片实在太长了，尤其是南海诸岛及九段线。
为此市面上常见的中国地图多会截取一部分南海诸岛和九段线，将其放在大陆的右下角，以小图的方式显示。
但就是这样的一个需求却一直困扰着我，虽然用echarts也不是不行，但本着R画图就是方便的思路，还是想试一试通过R结合`sf`和`ggplot2`实现含有南海及九段线小图的方案。
在统计之都论坛的讨论中，对于使用`echarts4r`能够画这种图的可能性普遍持否定的[看法](https://d.cosx.org/d/422867-echarts4r)。
而在我看来，这样的实现应该就是大图加小图的形式，基于`ggplot2`的语法，想要实现这个功能应该还是比较简单的，果不其然，我在搜索的时候就找到了成功通过`ggplot2`实现该功能的，甚至还附有[源代码](https://rstata.duanshu.com/#/course/377485bd8d2b44eda8caa017a71dc5be)。

那么话不多说，直接上代码：


```r
# 读取数据
prov = read_sf("D:/work/map/2023_map/2023年省级/2023年省级.shp")
provline = prov|>
  st_transform(mycrs)
jdx = read_sf("D:/work/map/2023_map/九段线/九段线.shp")|>
st_transform(mycrs)
```




先把中国各省地图数据与九段线数据分开读取，然后将其转化为最佳投影，并将其与人口数据统一合并：

```r
# 结合人口数据
provline = provline|>left_join(china_populatino,by='省份')
# 合并上面的文件
tempdf = provline|>
  bind_rows(jdx)|>
  st_sf()

tempdf
```

```
## Simple feature collection with 44 features and 11 fields
## Geometry type: GEOMETRY
## Dimension:     XY
## Bounding box:  xmin: -2625586 ymin: 362589.8 xmax: 2206965 ymax: 5921583
## Projected CRS: +proj=aea +lat_1=25 +lat_2=47 +lon_0=105 +x_0=0 +y_0=0 +ellps=GRS80 +datum=WGS84 +units=m +no_defs
## # A tibble: 44 x 12
##    省           省级码 省类型 ENG_NAME     VAR_NAME   FIRST_GID FIRST_TYPE year 
##  * <chr>        <chr>  <chr>  <chr>        <chr>      <chr>     <chr>      <chr>
##  1 北京市       110000 直辖市 Beijing      Běi Jīng   110000    Municipal~ 2023 
##  2 天津市       120000 直辖市 Tianjin      Tiān Jīn   120000    Municipal~ 2023 
##  3 河北省       130000 省     Hebei        Hé Běi     130000    Province   2023 
##  4 山西省       140000 省     Shanxi       Shān Xī    140000    Province   2023 
##  5 内蒙古自治区 150000 自治区 Neimenggu    Nèi Měng ~ 150000    Autonomou~ 2023 
##  6 辽宁省       210000 省     Liaoning     Liáo Níng  210000    Province   2023 
##  7 吉林省       220000 省     Jilin        Jí Lín     220000    Province   2023 
##  8 黑龙江省     230000 省     Heilongjiang Hēi Lóng ~ 230000    Province   2023 
##  9 上海市       310000 直辖市 Shanghai     Shàng Hǎi  310000    Municipal~ 2023 
## 10 浙江省       330000 省     Zhejiang     Zhè Jiāng  330000    Province   2023 
## # i 34 more rows
## # i 4 more variables: geometry <MULTIPOLYGON [m]>, 省份 <chr>, 人口 <dbl>,
## #   FID <dbl>
```

现在画图的数据有了，接下来就是最重要的主图和小地图的大小框定，这里借鉴上文提到的那篇[教程](https://rstata.duanshu.com/#/course/377485bd8d2b44eda8caa017a71dc5be)进行绘制

```r
# 主图
main_bbox = st_bbox(c(xmin = -2625586,
                      xmax = 2206965, 
                      ymax = 5921583,
                      ymin = 1836814.1),
                    crs = st_crs(mycrs))|>
  st_as_sfc()

mainchina = tempdf |>
  st_intersection(main_bbox)
```

```
## Warning: attribute variables are assumed to be spatially constant throughout
## all geometries
```

```r
# 南海九段线小地图
small_bbox = st_bbox(c(xmin = 120000,
                       xmax = 1766004.1,
                       ymax = 2557786.0,
                       ymin = 320000),
                     crs = st_crs(mycrs))|>
  st_as_sfc()
nanhai = tempdf|> 
  st_intersection(small_bbox)|>
  mutate(geometry = geometry * 0.5 + c(2100000, 1665139))|>
  st_set_crs(mycrs)
```

```
## Warning: attribute variables are assumed to be spatially constant throughout
## all geometries
```

```r
small_bboxsf = small_bbox |>
  as_tibble()|> 
  st_sf()|> 
  mutate(geometry = st_cast(geometry, "MULTILINESTRING"))|>
  mutate(geometry = geometry * 0.5 + c(2100000, 1665139))|>
  st_set_crs(mycrs)
```

现在就差最后一步了，直接使用所融合的数据画图

```r
# 绘图
map = mainchina|>
  ggplot()+ 
  geom_sf(data = mainchina,aes(fill = 人口),
          color = "black", size = 0.1)+ 
  geom_sf(data = nanhai, aes(fill = 人口),color = "black", size = 0.2)+
  geom_sf(data = small_bboxsf, fill = NA,
          color = "black", size = 0.2)+
  theme_void()+
  theme(legend.position = "none")
map
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />

这样一副完美投影结合南海小地图的中国地图就生成了，对比前面的长图，不得不说这样的图在展示上确实美观许多。这里面最让人困惑的就是小地图的位置问题，不知这样的设置是否符合国家标准规范。其余堪称完美，以后就可以使用`sf`和`ggplot2`美美地画中国地图了。
