# awk

## awk阿里原题（2020-12-24）

```java
trace.log

2018-03-29 14:40:58.963|0ab251b315223056589343953d0ec9|LOGIN_MESSAGE_havana_unity_login|29
2018-03-29 14:40:58.963|0a9683c815223056589441906d0715|LOGIN_MESSAGE_havana_unity_login|20
2018-03-29 14:40:58.970|0b83e32215223056588558603e7614|TRADE_SUB_trade_created_done|13
2018-03-29 14:40:58.972|0b83e32215223056588558603e7614|trade_created_done_main_order|2
  
  
管道线分隔的各列值释义：时间戳|traceid|事件code|rt，其中traceid由小写字母和数字组成，长度固定为在30位
请找出满足以下条件的所有记录行，并(只)打印其traceid
1. 时间在2018-03-29 14:00:00.000 ~ 2018-03-29 14:30:00.000之间
2. 事件code为LOGIN_MESSAGE_havana_unity_login
3. rt超过100
4. traceid第24,25位同为数字0
```

## -F "分隔符号"

```bash
mubi@mubideMacBook-Pro tmp_work $ cat trace.log | awk -F "|" '{print $1}'
2018-03-29 14:40:58.963
2018-03-29 14:40:58.963
2018-03-29 14:40:58.970
2018-03-29 14:40:58.972
mubi@mubideMacBook-Pro tmp_work $
```

```bash
mubi@mubideMacBook-Pro tmp_work $ cat trace.log | awk -F "|" '{print $4}'
29
20
13
2
mubi@mubideMacBook-Pro tmp_work $
```

## 判断加条件

```bash
mubi@mubideMacBook-Pro tmp_work $ cat trace.log | awk -F "|" '$4 >= 20 {print $4}'
29
20
mubi@mubideMacBook-Pro tmp_work $
```

## `$0`表示所有行

```bash
mubi@mubideMacBook-Pro tmp_work $ cat trace.log | awk -F "|" '$4 >= 20 {print $0}'
2018-03-29 14:40:58.963|0ab251b315223056589343953d0ec9|LOGIN_MESSAGE_havana_unity_login|29
2018-03-29 14:40:58.963|0a9683c815223056589441906d0715|LOGIN_MESSAGE_havana_unity_login|20
mubi@mubideMacBook-Pro tmp_work $
```

## NF:一条记录的字段的数目; NR:已经读出的记录数，就是行号，从1开始

```bash
mubi@mubideMacBook-Pro tmp_work $ cat trace.log | awk -F "|" '$4 >= 20 {print NR,$0}'
1 2018-03-29 14:40:58.963|0ab251b315223056589343953d0ec9|LOGIN_MESSAGE_havana_unity_login|29
2 2018-03-29 14:40:58.963|0a9683c815223056589441906d0715|LOGIN_MESSAGE_havana_unity_login|20
mubi@mubideMacBook-Pro tmp_work $
```

## awk 内置函数

算数函数、字符串函数、其它一般函数、时间函数
