---
title: "SQL-last entry that are not NULL"
layout: post
date: 2019-07-10 22:48
image: /assets/images/markdown.jpg
headerImage: True
tag:
- sql
category: blog
author: bobzhai
description: How to select the last entry that are not null in sql
---



今天在处理数据的时候，发现了个问题，搜索了国内的各种网站，没有发现解决方案，当然也有可能是我搜索的问题。不过anyway，发现了个外国人写的文章解释了怎么处理。由于我的sql技术太低了，发现虽然看懂了，不过不能灵活运用，所以打算搬运一下这篇文章，翻译的同时加深自己的理解。

想直接看原文的朋友直接看这里：[英文原文](https://www.itprotoday.com/sql-server/last-non-null-puzzle)

问题是这样的，我今天处理的房贷数据是美国政府国民抵押贷款协会（Ginnie Mae）的，他们的数据有个特点，如果一个贷款还完了，他们最后的某几列会变成Null，但我需要把每个房贷的属性提取出来，比如说他们的LTV（Loan To Value）是多少，发行的利率是多少。一般来说，如果只有最后一个是Null的话还好说，我可以直接max()，然后就能得到想要的数据，关键是他们的贷款中间还有变化，我们经过研究，决定取最新的数据（当然这里得把最后的Null给去掉）。废话少说，这篇文章就是教我们怎样取每一列的最后一个不是Null的单元。

我这里直接用原文作者的例子：

```excel
id      col1    lastval
----------- ----------- -----------
2       NULL    NULL
3       10      10
5       -1      -1
7       NULL    -1
11      NULL    -1
13      -12     -12
17      NULL    -12
19      NULL    -12
23      1759    1759
```

他这个例子比我想要得到的结果和我想要的结果很类似，他的要求是给出id 和 col1，要求生成一列，在这一列里面，如果当前的值是NULL，要取前一位不是NULL的数。

在这个表格里面，id代表了顺序。

作者用了两个方法去实现这个过程，时间关系，我先只写一种：用Concatenation来实现。

这个方法分两步走：

## 第一步

```sql
SELECT id, col1, binstr,
  MAX( binstr ) OVER( ORDER BY id
          ROWS UNBOUNDED PRECEDING ) AS mx
FROM dbo.T1
  CROSS APPLY ( VALUES( CAST(id AS BINARY(4)) + CAST(col1 AS BINARY(4)) ) )
    AS A(binstr);
```

做完之后可以得出下面的表格：

```excel
id             col1    					binstr        		 mx
----------- ----------- ------------------ ------------------
2       				NULL    NULL           						NULL
3       					10    0x000000030000000A 		0x000000030000000A
5       					-1    0x00000005FFFFFFFF 		0x00000005FFFFFFFF
7       				NULL    NULL           				0x00000005FFFFFFFF
11      				NULL    NULL           				0x00000005FFFFFFFF
13      				 -12    0x0000000DFFFFFFF4 		0x0000000DFFFFFFF4
17      				NULL    NULL           				0x0000000DFFFFFFF4
19      				NULL    NULL           				0x0000000DFFFFFFF4
23      				1759    0x00000017000006DF 		0x00000017000006DF
```

这一步什么意思呢？因为我们要找上一个不是NULL的数，这里有个巧妙的地方是任何东西+NULL = NULL  [详情](https://docs.microsoft.com/en-us/sql/t-sql/statements/set-concat-null-yields-null-transact-sql?view=sql-server-2017)。所以先算出来binstr，就是当前行的值。这个值是由id转成2进制String，然后col1的值也转成2进制string，然后合在一起（concatenate）。（btw，我也不是很懂什么是二进制string，sql基础薄弱）

不过从上面的例子可以看出来，是3的16进制表达 合 10的16进制表达：

i.e.

| 十进制 —> | 十六进制（4个byte） | 十进制 | 十六进制   |
| --------- | ------------------- | ------ | ---------- |
| 3         | 0x00000003          | 10     | 0x0000000A |

合在一起就是上面的：0x000000030000000A

这样做有什么好处呢？一个原因是因为有个NULL，上下取最大的时候会取到不是NULL。另外一个原因是：id前面说到是顺序，这样子就可以保证是上一个。还有一个原因是后面可以unpack，这个后面说。

这个cross apply我之前也没用过，不过看这里的用法，比join高级的是因为可以在第二个query里面做运算，然后符合的拿出来，而且第二个query可以直接用到前面query的变量，例如id和col1.

最后得出最右边这一列然后再转换回原来的值：

```sql
SELECT id, col1,
  CAST(
    SUBSTRING(
  MAX( CAST(id AS BINARY(4)) + CAST(col1 AS BINARY(4)) )
    OVER( ORDER BY id
      ROWS UNBOUNDED PRECEDING ),
  5, 4)
    AS INT) AS lastval
FROM dbo.T1;
```

 因为他这里是按byte来算，所以最后就是substring(5, 4)，从第五位起，取四位，看上面例子就很make sense了，不熟悉进制的想一下，十六进制2位就是一个byte。

至于为什么很厉害，作者从执行的方面讲了下，有兴趣的可以自己去看看。链接再给一下：[链接](https://www.itprotoday.com/sql-server/last-non-null-puzzle)  

这个方法套用在我的例子，我还需要多一步，就是需要group by，不过只需要取最后一个值

今天有点累了，具体请看：[例子](https://dba.stackexchange.com/questions/208935/how-to-select-the-set-of-last-non-null-values-per-column-over-a-group)

