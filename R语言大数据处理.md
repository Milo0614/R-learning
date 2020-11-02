# R语言大数据处理

本章节主要介绍使用R语言中的程序包`data.table`做大数据处理。该程序包用于在内存中快速处理大数据集（如100GB），包括数据的聚合、连接、增加、删除、修改、读取和存储，并提供一套灵活、自然的语法用于快速开发。

本章所有例子基于2个数据集。第一个数据集是2014年前10个月由纽约出发航班的准点情况数据，由美国运输局收集整理，包含253,316个样本，每个样本包含17个属性，如下表所示

| 属性      | 定义           |
| --------- | -------------- |
| year      | 年份           |
| month     | 月份           |
| day       | 日期           |
| dep_time  | 出发时间       |
| dep_delay | 出发延误分钟数 |
| arr_time  | 到达时间       |
| arr_delay | 到达延误分钟数 |
| carrier   | 航空公司代码   |
| tailnum   | 尾翼编号       |
| flight    | 航班编号       |
| origin    | 出发地         |
| dest      | 目的地         |
| air_time  | 飞行分钟数     |
| distance  | 距离           |
| hour      | 小时           |
| min       | 分钟           |

该数据集的路径为`/opt/data/flights/flights14.csv`。

另一个数据集是航空公司简写与描述的映射数据集，由美国运输局收集整理，包含424个样本，每个样本包含2个属性，如下表所示

| 属性        | 定义         |
| ----------- | ------------ |
| carrier     | 航空公司代码 |
| description | 描述         |

该数据集的路径为`/opt/data/flights/carrier.csv`。

调用函数`library()`载入程序包`data.table`。

调用函数`fread()`读取航班数据集，该函数是程序包`data.table`对R语言内置数据读取函数`read.table()`等更高效的实现，参数列表与函数`read.table()`十分类似。

直接输入读入数据的变量名，查看数据集的前几行和后几行。

```R
library(data.table)
flights <- fread("/opt/data/flights/flights14.csv")
flights
```

![image-20201024124144155](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124144155.png)

再读取航空公司数据集。

```R
carriers <- fread("/opt/data/flights/carrier.csv")
carriers
```

![image-20201024124213418](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124213418.png)

## 数据选择

数据表（`data.table`）提供了增强版的数据框（`data.frame`）数据类型，即数据框的相关函数和用法也可以用于数据表。

### 1.创建数据表

调用函数`data.table()`创建数据表，其中

- 前1个或多个参数`...`形如`<值>`或`<标签>=<值>`，表示变量名称和对应的数据。

```R
library(data.table)
DT = data.table(ID = c("b","b","b","a","a","c"), a = 1:6, b = 7:12, c = 13:18)
DT
class(DT$ID)
```

![image-20201024124422237](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124422237.png)

可以看出，与数据框不同，字符型数据变量不会默认转换为因子型。在显示出来的数据样本中，会默认显示行号与“:”。

### 2.数据表的基本形式

数据表的基本形式为

```R
DT[i, j, by]
```

如果读者有SQL语言的基础，可以理解为

- `i`项：对应SQL中的`where`语句；
- `j`项：对应SQL中的`select`和`update`语句；
- `by`项：对应SQL中的`group by`语句。

另一种通俗的理解方式是：使用`i`选择行，然后计算`j`，并按`by`分组。

### 3.数据行的选择（使用`i`项）

在`i`项中使用变量名和比较运算符做筛选条件，选取航班数据集中出发地（变量`origin`）为`'JFK'`、月份（变量`month`）为6月的数据行。

```R
flights <- fread("/opt/data/flights/flights14.csv")
flights[origin == "JFK" & month == 6]
```

![image-20201024124501435](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124501435.png)

需要注意的是，与数据框不同

- 变量名并不必须添加数据表名称前缀`flights$`，但也完全支持添加；
- 在筛选条件后不必须添加“,”，形如`flights[dest == "JFK" & month == 6L, ]`，但也完全支持添加。

选取航班数据集的前2行数据。

```R
flights[1:2]
```

![image-20201024124522693](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124522693.png)

在数据表的`i`项中调用函数`order()`按出发地（变量`origin`）顺序、目的地（变量`dest`）倒序排列。

```R
flights[order(origin, -dest)]
```

![image-20201024124604712](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124604712.png)

### 4.数据列的选择（使用`j`项）

在数据表的`j`项中直接使用变量名，选取航班到达延误分钟数（变量`arr_delay`），返回结果为**向量**。

```R
flights[1:6, arr_delay]
```

![image-20201024124619157](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124619157.png)

将变量名放在`.()`中，则返回结果为**数据表**。

```R
flights[1:6, .(arr_delay)]
```

![image-20201024124649279](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124649279.png)

也可以在`.()`中包含多个变量名，选取航班到达延误分钟数（变量`arr_delay`）和出发延误分钟数（变量`dep_delay`），则返回包含多列的数据表。

```R
flights[1:6, .(arr_delay, dep_delay)]
```

![image-20201024124823222](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124823222.png)

可以重新命名选取的列，这里将变量`arr_delay`和`dep_delay`重命名为`delay_arr`和`delay_dep`。

```R
flights[1:6, .(delay_arr = arr_delay, delay_dep = dep_delay)]
```

### 5.数据列的计算（使用`j`项）

在数据表的`j`项中调用函数`sum()`，计算航班总延误分钟数（变量`arr_delay`与`dep_delay`之和）小于0的记录数。

```R
flights[, sum((arr_delay + dep_delay) < 0)]
```

![image-20201024124955092](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124955092.png)

在数据表的`i`项中做筛选条件，并在数据表的`j`项中调用函数`mean()`，计算航班数据集中满足筛选条件的航班的平均到达延误分钟数（变量`arr_delay`）和平均出发延误分钟数（变量`dep_delay`）。

```R
flights[origin == "JFK" & month == 6L, .(m_arr = mean(arr_delay), m_dep = mean(dep_delay))]
```

![image-20201024124914611](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024124914611.png)

在数据表的`j`项中调用函数`length()`，计算满足筛选条件的记录数。

```R
flights[origin == "JFK" & month == 6L, length(dest)]
```

![image-20201024125017560](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024125017560.png)

另一种方法是使用`.N`，即表示该分组的记录数。需要注意的是，在数据表的`j`项中没有指定输出变量名，则自动命名为`N`。

```R
flights[origin == "JFK" & month == 6L, .N]
```

![image-20201024125028206](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024125028206.png)

## 数据聚合

### 1.使用`by`关键字

在数据表的`j`项中使用`.N`，并在数据表的`by`项中使用`by`关键字指定按出发地（变量`origin`）分组，计算各出发地的记录数。

```R
library(data.table)
flights <- fread("/opt/data/flights/flights14.csv")
flights[, .N, by = origin]
```

![image-20201024125054274](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024125054274.png)

也可以继续在`i`项中做筛选条件，并在数据表的`by`项中指定多个变量，计算各不同出发地（变量`origin`）、目的地（变量`dest`）组合中，航空公司代码（变量`carrier`）为“AA”的记录条数。

```R
flights[carrier == "AA", .N, by = .(origin,dest)]
```

![image-20201024125206175](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024125206175.png)

以下例子计算了航空公司代码为“AA”的航班，在各不同出发地、目的地和月份组合中，平均到达和出发延误分钟数。需要注意的是，在数据表的`j`项中没有指定输出变量名，则自动命名为`V1`和`V2`。

```R
flights[carrier == "AA", .(mean(arr_delay), mean(dep_delay)), by = .(origin, dest, month)]
```

![image-20201024125226206](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024125226206.png)

### 2.使用`keyby`关键字

程序包`data.table`默认在聚合时保持原始数据的顺序，但也有时候，我们希望能够按照分组变量的顺序排列结果。

在数据表的`by`项中使用`keyby`关键字指定分组变量。可以看出，结果按照分组变量顺序排列。

```R
flights[carrier == "AA", .(mean(arr_delay), mean(dep_delay)), keyby = .(origin, dest, month)]
```

![image-20201024125246537](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024125246537.png)

### 3.级联操作

考虑以下2步操作：

1. 首先计算各不同出发地（变量`origin`）、目的地（变量`dest`）组合中，航空公司代码（变量`carrier`）为“AA”的记录条数；
2. 再按按出发地（变量`origin`）顺序、目的地（变量`dest`）倒序排列。

```R
DT <- flights[carrier == "AA", .N, by = .(origin, dest)]
DT[order(origin, -dest)]
```

![image-20201024125305285](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024125305285.png)

以上操作需要将第1步的结果存储在一个中间变量中，浪费了内存资源。

可以通过级联操作直接得到结果。

```R
flights[carrier == "AA", .N, by = .(origin, dest)][order(origin, -dest)]
```

![image-20201024125305285](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024125305285.png)

## 数据引用

在前几节中，所有操作都产生了一个新的数据集。而在处理大数据集时，有时需要我们在原始数据集上直接做数据列的增加、修改和删除。

首先需要区分R语言中浅拷贝和深拷贝的概念。

- 浅拷贝：仅拷贝数据集列向量的指针，而没有拷贝实际数据，即按引用拷贝；
- 深拷贝：拷贝所有数据，即按值拷贝。

考虑以下数据框中的例子。

```R
DF <- data.frame(ID = c("b","b","b","a","a","c"), a = 1:6, b = 7:12, c = 13:18)
DF
```

![image-20201024125359443](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024125359443.png)

在R语言3.1版本之后，

- 以下取代整列的操作使用浅拷贝；

```R
DF$c <- 18:13
```

- 以下对于数据列中某些行的赋值操作使用深拷贝。

```R
DF$c[DF$ID == "b"] <- 15:13
```

### 1.`:=`操作符

程序包`data.table`在`j`项中提供`:=`操作符，用于数据表中列的无拷贝更新，其基本形式有2种：

1. `<左边项> := <右边项>`的形式

```R
DT[, c("<列1>", "<列2>", ...) := list(<值1>, <值2>, ...)]
```

如对之前DT表进行处理：

```R
DT = data.table(ID = c("b","b","b","a","a","c"), a = 1:6, b = 7:12, c = 13:18)
DT
DT[,c("b","a") := list(1,2)] 
DT
```

![image-20201024153558325](/Users/wangchenming/Library/Application Support/typora-user-images/image-20201024153558325.png)

2. 函数形式

```R
DT[, :=(<列1> = <值1>, <列2> = <值2>, ...)]
```

如对之前DT表进行处理：

```R
DT = data.table(ID = c("b","b","b","a","a","c"), a = 1:6, b = 7:12, c = 13:18)
DT
DT[, :=(a=1,b=2)] 
DT
```











