---
title: grep的陷阱
author: 瞿伟斌
date: '2016-04-22'
slug: grep的陷阱
categories:
  - R
tags: []
---

>R语言中有一个处理文本的神奇函数grep，其在处理文本数据中屡建奇功，我也多次利用这个函数来查找字符串中的特定字符。但是，在今天精简之前我写的文本挖掘的函数时，却有了一些出乎意料的发现。

&emsp;&emsp;起因源于向量化处理函数，以提高计算速度缩短运行时间。于是我查看了每一个函数所用的时间，令我不解地是在25‘的运转时间里，grep函数耗时居然占到了50%以上。这完全超出了我的预期，在我想象中，最耗时的应该是我连续使用的两个for循环，以及里面的赋值运算。然而现实就是之前我一直以为是处理字符串的利器的grep函数其内在却是一个多吃多占的大户。同时我又一次领会到了向量化编程的重要性。下面的两个代码分别是修改前和修改后的，可以体会一下。

修改前：
```{r}
    for(word in id_table$job_describe){
    if(length(grep(word,stopwords))==0)
      id_keyword = rbind(id_keyword,id_table[grep(paste("^",word,"$",sep = ""),id_table$job_describe),])
    }
```

修改后：
```{r}
    for(word in id_table$job_describe){
    if(sum(word == stopwords)==0)
      id_keyword = rbind(id_keyword,id_table[grep(paste("^",word,"$",sep = ""),id_table$job_describe),])
    }
```

&emsp;&emsp;其中在第一个版本中还有一个grep的语法错误，就是前面的pattern的正则表达式部分，不知道各位有木有看出来。

&emsp;&emsp;今天真是被这个grep函数坑惨了，学艺不精还是要多看书啊。

另：之前夸下海口说每一篇都要翻译成英文，结果对我来说也是一件艰巨任务，尤其是上一篇的1000+的长文，但是话都放出去了，只能硬着头皮做了。所以这篇就写短点。

