请访问: https://doctording.github.io/sword_at_offer/

对我，主要是复习巩固【学习目的】，自己原本打算找人的，所以还是算了。有兴趣的小伙伴可以检查补充，有问题可以issue或email。可以补充实现JS,C/C++,PHP等等语言版本，方便后续的复习。

待补充如下内容，以便深入理解和使用Java

* JVM
* Java8
* 并发编程
* NIO

---

# 剑指offer刷题说明

刷题地址: http://www.nowcoder.com/ta/coding-interviews?page=1

需要找到最优解法，参考学习牛客网**左程云**视频、牛客网算法讨论、《剑指offer》图书等。

直接编辑器敲出代码 =》在纸上直接写出代码（注意时间/空间复杂度）

剑指offer整体难度都不高，不过如果手撕代码的话，还是很锻炼的。

# 学习目的

1. python基础语法巩固

2. Java各种数据结构巩固 + Java深入

3. 剑指offer基础算法巩固

4. simiki 操作使用

# simiki 部署使用

```bash
$ simiki g # 生成output
$ simiki p # 本地运行：http://127.0.0.1:8000/sword_at_offer/
```

```bash
# 部署到 https://doctording.github.io/sword_at_offer/
$ ghp-import -p -m "Update output documentation" -r origin -b gh-pages output
```

可参考： https://blog.csdn.net/u013041398/article/details/73958706

# tool.py处理git提交

```bash
$ python tool.py ["本次commit的描述信息"]
```

如
```bash
$ python tool.py "更新了README文档"
```

或者，默认的commit描述信息为："updated"
```bash
$ python tool.py
```

图片使用例子

```bash
![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/solved_by_java/imgs/circle.png)
```
