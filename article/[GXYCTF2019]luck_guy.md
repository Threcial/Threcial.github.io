## 查壳
ELF文件，GCC编译，无壳

## IDA分析

追踪 `patch_me` 可疑函数，发现 `get_flag` 函数，对其进一步分析可以得到 `flag` 的生成步骤

* 给出变量 `s` 并赋值到 `f2`
* 对 `f2`  进行变换
* 组合 `f1` 和 `f2` 成为 `flag`


关键字段 `s = 9180147350284624745LL;`   `f1 = GXY{do_not_`

**其中LL表示long long 类型,**

**由于ELF文件采用小端序储存数据，IDA采用大端序解读数据，**
**真实数据应颠倒前后顺序**

源数据转为十六进制为 `0x7F666F6067756369LL`

脚本复刻步骤生成最终 `f2`

```python
a = '7F666F6067756369'
b = []
for i in [0,2,4,6,8,10,12,14]:
    b.insert(0, int(a[i]+a[i+1], 16))
for i in range(8):
    if i % 2 == 1:
        print(chr(b[i]-2), end="")
    else:
        print(chr(b[i]-1), end="")
```
输出 `hate_me}`

最终结果为 `GXY{do_not_hate_me}`
