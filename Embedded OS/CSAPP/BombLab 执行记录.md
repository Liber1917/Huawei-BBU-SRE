配置 VSCode 插件：[x86 and x86_64 Assembly - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=13xforever.language-x86-64-assembly)
安装[[调试工具：pwndbg]]
Bootstrap 参考：[更适合北大宝宝体质的 Bomb Lab 踩坑记](https://arthals.ink/blog/bomb-lab)

---
### 反汇编调试
#### 准备工作
`bomb.c` 可以 `read_line()` 读取一行输入 ，故而 `touch solution.txt` 以免重复输入。
由于 Windows 下 VSCode 采用了 Windows 换行符 `\r\n`，我们可以在右下角将 `CRLF` 改为 `LF`。也可以 `dos2unix solution.txt`。

 `bomb.c` 中并没有告诉我们该输入什么以免爆炸，所以采用 `objdump -d ./bomb > bomb.asm` 进行反汇编。
 
为了方便在命令行视图观察 asm 和 regs，直接 `touch .gdbinit`，复制以下命令，不用像[安全化炸弹](https://arthals.ink/blog/bomb-lab#%E5%AE%89%E5%85%A8%E5%8C%96%E7%82%B8%E5%BC%B9)中那么复杂：
```bash
layout asm
layout regs
set args solution.txt
b phase_1
b phase_2
b phase_3
b phase_4
b phase_5
b phase_6

```

#### 阅读机器码
[第 3 章：程序的机器级表示 | 深入理解计算机系统（CSAPP）](https://hansimov.gitbook.io/csapp/part1/ch03-machine-level-representing-of-programs)
