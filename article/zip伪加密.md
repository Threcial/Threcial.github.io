
## 基础知识

对zip文件进行二进制分析，可以发现一般zip文件都由三种部分组成

- 压缩源文件数据区 ***record***
- 压缩源文件目录区 ***dirEntry***
- 压缩源文件目录结束标志 ***endLocator***

一般来说 ***record*** 和 ***dirEntry*** 可以存在多个，而 ***endLocator*** 只存在一个

接下来对各部分作进一步分析

### record

`50 4B 03 04` 是 `record` 的标志，