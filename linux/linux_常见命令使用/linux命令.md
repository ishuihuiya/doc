# Linux各项命令使用

## 排序sort 

### 命令格式

```shell
sort [选项]... [文件]...
```

### 常见的选项

| 选项 | 说明                                                 |
| ---- | ---------------------------------------------------- |
| -f   | 忽略大小写                                           |
| -n   | 使用数字进行排序(数字默认会被看成字符处理)           |
| -h   | 对易读性结果进行排序(M G T等计量单位的排序)          |
| -r   | 将结果进行反转(降序排列)                             |
| -k   | 指定某一个区间进行排序(哪一列，从什么开始到什么结束) |
| -t   | 指定sort输入输出的分隔符                             |
| -o   | 将结果输出到指定文件                                 |
| -u   | 将排序结果进行去重                                   |

### 示例

* 对`/etc/passwd`文件按照用户名进行排序

  ```shell
   [kala@uosPro Desktop]$ sort /etc/passwd 
  _apt:x:103:65534::/nonexistent:/usr/sbin/nologin
  avahi:x:112:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
  backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
  bin:x:2:2:bin:/bin:/usr/sbin/nologin
  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
  ...
  ```

* 对`/etc/passwd`文件按照`UID`进行排序

  ```shell
  [kala@uosPro Desktop]$ sort -t ":" -nk3 /etc/passwd 
  root:x:0:0:root:/root:/bin/bash
  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
  bin:x:2:2:bin:/bin:/usr/sbin/nologin
  sys:x:3:3:sys:/dev:/usr/sbin/nologin
  sync:x:4:65534:sync:/bin:/bin/sync
  games:x:5:60:games:/usr/games:/usr/sbin/nologin
  man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
  lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
  mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
  news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
  uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
  ...
  
  # 这里面有两个注意的点
  # 第一个就是分隔符， 默认空格，需要用-t指定使用:作为分隔符
  # 第二个就是需要使用-n将数字字符转换为数字
  ```

* 对局域网Ip地址的后两位进行排序，如果第三位相同就对第四位进行排序

  示例文本：

  ```shell
  192.168.100.10  tz:BB:CC:DD
  192.168.20.1   gg:BB:CC:DD
  192.168.20.14  gg:BB:CC:DD
  192.168.20.22 gg:BB:CC:DD
  192.168.20.2 gg:BB:CC:DD
  192.168.30.20   kA:BB:CC:DD
  192.168.110.2   ss:BB:CC:DD
  192.168.20.21   bb:BB:CC:DD
  192.168.100.2   cc:BB:CC:DD
  192.168.100.23   cc:BB:CC:DD
  192.168.100.21   cc:BB:CC:DD
  192.168.100.110   cc:BB:CC:DD
  192.168.10.80   ee:BB:CC:DD
  192.168.100.1   hA:BB:CC:DD
  ```

  排序：

  ```shell
  [kala@uosPro Desktop]$ sort -n -t "." -k 3 -nk 4,4  1.txt 
  192.168.10.80   ee:BB:CC:DD
  192.168.20.1   gg:BB:CC:DD
  192.168.20.14  gg:BB:CC:DD
  192.168.20.2 gg:BB:CC:DD
  192.168.20.21   bb:BB:CC:DD
  192.168.20.22 gg:BB:CC:DD
  192.168.30.20   kA:BB:CC:DD
  192.168.100.1   hA:BB:CC:DD
  192.168.100.10  tz:BB:CC:DD
  192.168.100.110   cc:BB:CC:DD
  192.168.100.2   cc:BB:CC:DD
  192.168.100.21   cc:BB:CC:DD
  192.168.100.23   cc:BB:CC:DD
  192.168.110.2   ss:BB:CC:DD
  
  。。。。好像有问题，后面在研究一下。。。
  ```

  python的勉强实现:

  ```python
  import pandas as pd
  
  # 新建字典
  dict={}
  # 分割
  ipmacsList = ipmacs.split("\n")
  for i in ipmacsList:
      if i !="":
          dict[i.split("\t")[0]] = i.split("\t")[1]
  
  # 字典转为dataframe
  df = pd.DataFrame({"ip":dict.keys(), "mac":dict.values()})
  
  # ip分列 合并
  df2 =pd.concat([df, df["ip"].str.split(".", expand=True).astype("int")],axis=1)
  
  # dataframe排序
  df2 = df2.sort_values(by=[0,1,2,3])
  # 输出
  print(df2.loc[:,["ip", "mac"]])
  ```

  

## 去重 uniq

准确意义上来说，uniq仅仅是合并文件中相邻的相同的行，同时也支持显示出现的总次数。所以uniq一般都是会搭配sort一起使用的，先排序然后uniq。

### 命令格式

```shell
uniq [OPTION]… [INPUT [OUTPUT]]
```

### 常见的选项

| 选项 | 说明                             |
| ---- | -------------------------------- |
| -c   | 取出重复项，并计算有多少个重复项 |
| -i   | 忽略大小写                       |
| -u   | 仅打印没有重复的行               |
| -d   | 仅打印重复的行                   |

示例：

统计ip出现的次数

```shell
[kala@uosPro Desktop]$ cat 1.txt  |sort |uniq -c
      2 192.168.100.10  tz:BB:CC:DD
      7 192.168.100.110   cc:BB:CC:DD
      9 192.168.100.1   hA:BB:CC:DD
     15 192.168.100.21   cc:BB:CC:DD
     16 192.168.100.23   cc:BB:CC:DD
     16 192.168.100.2   cc:BB:CC:DD
      7 192.168.10.80   ee:BB:CC:DD
     11 192.168.110.2   ss:BB:CC:DD
      2 192.168.20.14  gg:BB:CC:DD
      2 192.168.20.1   gg:BB:CC:DD
     14 192.168.20.21   bb:BB:CC:DD
      4 192.168.20.22 gg:BB:CC:DD
     11 192.168.20.2 gg:BB:CC:DD
     11 192.168.30.20   kA:BB:CC:DD

```

## find

find就是用户系统文件的查找。

命令格式

```shell
find   path   -option   [   -print ]   [ -exec   -ok   command ]   {} \;
```

常见的选项:

| 选项   | 说明                                           |
| ------ | ---------------------------------------------- |
| -depth | 搜索文件的深度，默认是递归查找所有文件夹       |
| -name  | 文件名，通常使用*进行模糊搜索                  |
| -iname | 文件名忽略大小写                               |
| -size  | 文件大小                                       |
| -type  | 文件类型 文本、目录、 套接字、管道、连接。。。 |
|        |                                                |

其他的一些查找条件：

* 组合条件
  * 与  -a
  * 或  -o
  * 非 -not







## grep 





## sed







## awk 

