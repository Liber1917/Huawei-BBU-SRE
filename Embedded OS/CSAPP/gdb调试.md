```embed
title: "CSCI 2021 Quick Guide to gdb: The GNU Debugger"
image: "https://www-users.cse.umn.edu/~kauffman/tutorials/gdb-tui.png"
description: ""
url: "https://www-users.cse.umn.edu/~kauffman/tutorials/gdb"
favicon: ""
aspectRatio: "98.60627177700349"
```


介绍性： [ Guide to Faster, Less Frustrating Debugging](https://heather.cs.ucdavis.edu/matloff/public_html/UnixAndC/CLanguage/Debug.html)
入门文档：[CSCI 2021 Quick Guide to gdb: The GNU Debugger](https://www-users.cse.umn.edu/~kauffman/tutorials/gdb) + https://beej.us/guide/bggdb/

```shell
读汇编拆弹速成（实战向）
1. 你看到的指令长这样
0x400ef2 <+18>:    callq  0x40143a <explode_bomb>
  ↑地址    ↑偏移    ↑指令    ↑目标地址     ↑函数名
GDB 默认是 AT&T 语法：指令 源, 目标
mov    $0x402400, %esi    ← 把 0x402400（立即数）送到 %esi 寄存器
        ↑立即数$前缀       ↑寄存器%前缀
2. 拆弹只需要认识这几条指令
数据移动：
mov   a, b     →  b = a
movl            → 操作 4 字节
movq            → 操作 8 字节
leaq  (a), b   →  b = &a （计算地址，不读内存）
比较（最关键）：
cmp   a, b     →  计算 b - a，设置条件码，不存结果
test  a, a     →  计算 a & a，检查 a 是否为 0/正/负
跳转（跟着比较后面用）：
je   label    →  if (等于) 跳转
jne  label    →  if (不等于) 跳转
jg   label    →  if (大于) 跳转
jl   label    →  if (小于) 跳转
jge  label    →  if (≥) 跳转
jle  label    →  if (≤) 跳转
jmp  label    →  无条件跳转
其他：
call func     →  调用函数
ret           →  返回
add/sub       →  加减
xor  a, a     →  清零（a = 0，比 mov $0 更快）
3. 实战读法——以你的 Phase 1 为例
phase_1:
   0x400ee4:  mov    $0x402400,%esi        ← 把地址 0x402400 放进 esi（第二个参数）
   0x400ee9:  callq  strings_not_equal      ← 调用字符串比较函数
   0x400eee:  test   %eax,%eax              ← 检查返回值是否为 0
   0x400ef0:  je     0x400ef7               ← 如果为 0（相等），跳走不炸
   0x400ef2:  callq  explode_bomb           ← 否则爆炸
   0x400ef7:  retq                          ← 返回
读法三步：
步骤	做什么
① 找 cmp/test + 条件跳转	这就是核心判断
② 找 call 了什么函数	strings_not_equal → 在比字符串；sscanf → 在解析输入；read_six_numbers → 要 6 个数字
③ 看到奇怪地址就 x/s 探	0x402400 用 x/s 看 → "Border relations..."
4. 常见汇编模式
if 模式：
cmp  a, b        ← if (a > b)
jle  else_part   ←   跳 else
then_part:
   ...
   jmp end
else_part:
   ...
end:
循环模式（你的 Phase 2）：
   mov  $0, %eax       ← i = 0
loop:
   cmp  $5, %eax       ← if (i > 5)
   jg   end            ←   跳出
   ...                 ← 循环体
   add  $1, %eax       ← i++
   jmp  loop
end:
switch 模式（你的 Phase 3）：
   cmpq  $6, %rax      ← 输入值 > 6？
   ja    default       ←   跳默认
   jmp   *jump_table(,%rax,8)  ← 跳转表
5. 一句口诀
看 call 知意图，看 cmp 知条件，看跳转知分支，x/s 探地址
```