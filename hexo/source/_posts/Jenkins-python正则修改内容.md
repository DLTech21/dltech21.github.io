---
title: Jenkins+python正则修改内容
date: 2018-04-26 10:45:44
tags: [Jenkins,python]
---

在[前面文章](https://dltech21.github.io/2018/04/13/mac-sed%E5%91%BD%E4%BB%A4%E5%AD%A6%E4%B9%A0/)写到shell脚本修改文件内容的方法，现在采用python来实现。

实现了一个通用的py脚本文件：**Jenkins_Replace.py**

```python
import sys
import re

args = sys.argv
regex=args[1]
filename=args[2]
target=args[3]

pattern2 = re.compile(r''+regex)

old_file=filename
fopen=open(old_file,'r')
w_str=""
for line in fopen:
     line=pattern2.sub(target,line)
     w_str+=line
print w_str
wopen=open(old_file,'w')
wopen.write(w_str)
fopen.close()
wopen.close()
```

### 调用方式
shell命令

```bash
python Jenkins_Replace.py "正则规则" "文件路径" "替代字符串"
```

demo地址:[github](https://github.com/DLTech21/jenkins-python)



