# 写一个 bash 脚本以统计一个文本文件 words.txt 中每个单词出现的频率。

为了简单起见，你可以假设：
* words.txt只包括小写字母和 ' ' 。
* 每个单词只由小写字母组成。
* 单词间由一个或多个空格字符分隔。

示例:

假设 words.txt 内容如下：

```cpp
the day is sunny the the
the sunny is is
```

你的脚本应当输出（以词频降序排列）：

```cpp
the 4
is 3
sunny 2
day 1
```

说明:

不要担心词频相同的单词的排序问题，每个单词出现的频率都是唯一的。
你可以使用一行 Unix pipes 实现吗？

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/word-frequency
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解决1

`cat words.txt | tr -s ' ' '\n' | sort -r | uniq -c | sort -r | awk '{print $2,$1}'`

## 解决2

```java
mubi@mubideMacBook-Pro tmp_work $ cat words.txt | xargs
the day is sunny the the the sunny is is
```

`cat words.txt | xargs -n 1 | sort -r | uniq -c | sort -r | awk '{print $2,$1}'`

取top3

`cat words.txt | xargs -n 1 | sort | uniq -c | sort -r |  head -3 | awk '{print $2,$1}'`
