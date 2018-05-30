请访问:

https://doctording.github.io/sword_at_offer/

# 剑指offer刷题说明

刷题地址: http://www.nowcoder.com/ta/coding-interviews?page=1

需要找到最优解法，参考学习牛客网**左程云**视频、牛客网算法讨论、《剑指offer》图书等。

直接编辑器敲出代码 =》在纸上直接写出代码（注意时间/空间复杂度）

# 学习目的

* python基础语法巩固

* 剑指offer基础算法巩固

* simiki 操作使用

# simiki 部署使用

```bash
$ simiki g # 生成output
$ simiki p # 本地运行：http://127.0.0.1:8000/sword_at_offer/
```

```bash
# 部署到 https://doctording.github.io/sword_at_offer/
$ ghp-import -p -m "Update output documentation" -r origin -b gh-pages output
```

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
