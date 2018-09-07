#Git 命令行中文显示为字符编码问题

* 在确认客户端编码支持中文的情况下，尝试以下命令

```bash
#不对0x80以上的字符进行quote，解决git status/commit时中文文件名乱码
git config --global core.quotepath false
```
