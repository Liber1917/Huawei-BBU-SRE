配置 VSCode 插件：[x86 and x86_64 Assembly - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=13xforever.language-x86-64-assembly)
安装[[调试工具：pwndbg]]

---
### 反汇编
BombLab 设计者提供了 `bomb.c` 作为答案参考。
所以采用 `objdump -d ./bomb > bomb.asm` 进行反汇编。