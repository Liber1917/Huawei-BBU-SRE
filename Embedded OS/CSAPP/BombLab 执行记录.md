配置 VSCode 插件：[x86 and x86_64 Assembly - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=13xforever.language-x86-64-assembly)
安装[[调试工具：pwndbg]]：`curl --proto '=https' --tlsv1.2 -LsSf 'https://install.pwndbg.re' | sh -s -- -t pwndbg-gdb`
Bootstrap 参考：[更适合北大宝宝体质的 Bomb Lab 踩坑记](https://arthals.ink/blog/bomb-lab)

---
### 反汇编调试
#### 环境限制
`glibc` 库 `/lib64/ld-linux-x86-64.so.2` 是必须要的，故而选择 debian 系列环境。如果你的电脑不支持，可以参考[[在MacOS运行BombLab]]。
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

#### phase_1
根据 `bomb.c`，先简单 `b phase_1` 设置断点，然后跑。
```
0000000000400ee0 <phase_1>:
400ee0: 48 83 ec 08 subq $0x8, %rsp
400ee4: be 00 24 40 00 movl $0x402400, %esi # imm = 0x402400
400ee9: e8 4a 04 00 00 callq 0x401338 <strings_not_equal>
400eee: 85 c0 testl %eax, %eax
400ef0: 74 05 je 0x400ef7 <phase_1+0x17>
400ef2: e8 43 05 00 00 callq 0x40143a <explode_bomb>
400ef7: 48 83 c4 08 addq $0x8, %rsp
400efb: c3 retq
└───────────────────────────────────────────────────────────────┘

pwndbg> x/s 0x402400
0x402400:       "Border relations with Canada have never been better."
```
这里发现了 `strings_not_equal`，所以直接使用上述命令取值。
#### phase_2
```
0000000000400efc <phase_2>:
0x400efc: push   %rbp          ; 保存调用者的 rbp
0x400efd: push   %rbx          ; 保存调用者的 rbx
0x400efe: sub    $0x28, %rsp   ; 栈顶往下移 40 字节，放 6 个 int（4×6=24，多出空间对齐）
0x400f02: mov    %rsp, %rsi    ; rsi = 当前栈顶地址（作为参数传给 read_six_numbers）
0x400f05: call   read_six_numbers  ; 读 6 个数，写到 rsp 开始的内存
调用后，栈上从 rsp 开始排着 6 个 int：
rsp+0:  输入的第 1 个数
rsp+4:  输入的第 2 个数
rsp+8:  输入的第 3 个数
rsp+12: 输入的第 4 个数
rsp+16: 输入的第 5 个数
rsp+20: 输入的第 6 个数
0x400f0a: cmpl   $0x1, (%rsp)  ; 比较 1 和 第一个数（rsp 指向的位置）
0x400f0e: je     0x400f30      ; 如果相等，跳去初始化循环
0x400f10: call   explode_bomb  ; 不相等 → 炸
0x400f15: jmp    0x400f30      ; （炸了就执行不到这里）
条件：第 1 个数必须等于 1。 否则引爆。
; 循环初始化（跳转目标）
0x400f30: lea    0x4(%rsp), %rbx   ; rbx = &第 2 个数（指针指向第二个）
0x400f35: lea    0x18(%rsp), %rbp  ; rbp = &第 6 个数再往后一个（结束标记）
                                     ; 0x18 = 24 = 6×4，所以 rbp 指向数组末尾之后
0x400f3a: jmp    0x400f17          ; 跳进循环体
; 循环体
0x400f17: mov    -0x4(%rbx), %eax  ; eax = rbx 前面那个数（前一个数）
0x400f1a: add    %eax, %eax       ; eax = eax + eax = 前一个数 × 2
0x400f1c: cmp    %eax, (%rbx)     ; 比较 前一个数×2 和 当前 rbx 指向的数
0x400f1e: je     0x400f25         ; 相等 → 继续
0x400f20: call   explode_bomb     ; 不相等 → 炸

0x400f25: add    $0x4, %rbx       ; rbx += 4（移动到下一个数）
0x400f29: cmp    %rbp, %rbx       ; 比较 rbx 和结束标记 rbp
0x400f2c: jne    0x400f17         ; 没到末尾 → 继续循环
0x400f2e: jmp    0x400f3c         ; 循环结束，跳去清理
循环逻辑：当前数 = 前一个数 × 2，从第 2 个数开始一直检查到第 6 个。
0x400f3c: add    $0x28, %rsp   ; 恢复栈顶
0x400f40: pop    %rbx          ; 恢复调用者的 rbx
0x400f41: pop    %rbp          ; 恢复调用者的 rbp
0x400f42: ret                  ; 返回
```
#### phase_3
