### 制取 docker 容器
#### 背景
- bomb 是 Linux ELF x 86-64 二进制，macOS 跑不了
- 一开始想装 pwndbg 失败（网络问题 + 需要 Linux 环境）
- 最终方案：OrbStack (Docker) + Debian 容器 + GDB
#### 前置条件
1. 安装 [OrbStack](https://orbstack.dev)（macOS 的 Docker 运行时，替代 Docker Desktop）
2. 启动：orb start（或打开 OrbStack App）
3. 国内网络需要镜像加速（Docker Hub 被墙）
镜像加速配置（~/.orbstack/config/docker.json）：
```
{
  "registryMirrors": ["https://docker.m.daocloud.io"]
}
```
#### 构建 bomb-gdb 镜像
当时用的 Dockerfile（内容从镜像历史还原）：
```
FROM debian:stable-slim
RUN apt-get update && apt-get install -y gdb
WORKDIR /bomb
```
构建命令：
```
docker build -t bomb-gdb .
```
验证命令：
```docker images          # 应看到 bomb-gdb:latest (138 MB)```
为什么 Debian 不用 Alpine：Alpine 用 musl libc，gcompat 无法完全模拟 glibc，bomb 动态链接会出问题。Debian 原生 glibc，零兼容问题。
#### 创建 dbg.sh 包装脚本
/Users/Zhuanz/Desktop/CSAPP/bomb/dbg.sh（已存在，见上）——核心是一个命令模板：

```docker run --rm -v "$BOMB_DIR:/bomb:ro" bomb-gdb gdb ...```

#### 日常使用
```
./dbg.sh run              # 跑炸弹（读 solution.txt）
./dbg.sh asm phase_6      # 反汇编某个函数
./dbg.sh                  # 交互式 GDB（TUI，需真实终端）
./dbg.sh "x/s 0 x 4025 cf"   # 执行任意 GDB 命令（batch 模式）
```
### `dbg.sh`
```shell
#!/bin/bash
# GDB debug wrapper for CSAPP bomb lab
# Usage:
# ./dbg.sh - Interactive GDB session with TUI layout
# ./dbg.sh <cmd> - Run GDB command non-interactively
# ./dbg.sh run - Just run the bomb (with solution.txt)
# ./dbg.sh asm <func> - Disassemble a function (e.g., ./dbg.sh asm phase_1)
  
BOMB_DIR="$(cd "$(dirname "$0")" && pwd)"
IMAGE="bomb-gdb"
	case "${1:-}" in
	run)
		docker run --rm -v "$BOMB_DIR:/bomb:ro" "$IMAGE" sh -c '/bomb/bomb < /bomb/solution.txt'
		;;
	asm)
		shift
		FUNC="${1:-phase_1}"
		docker run --rm -v "$BOMB_DIR:/bomb:ro" "$IMAGE" \
			gdb --batch -ex "file /bomb/bomb" -ex "disas $FUNC" -ex quit
		;;
	"")
		echo "Starting interactive GDB (TUI requires a real terminal)..."
		echo "Hint: Run manually: docker run --rm -it -v \"$BOMB_DIR:/bomb:ro\" $IMAGE gdb -x /bomb/.gdbinit /bomb/bomb"
		docker run --rm -it -v "$BOMB_DIR:/bomb:ro" "$IMAGE" \
			gdb -x /bomb/.gdbinit /bomb/bomb
		;;
	*)
		docker run --rm -v "$BOMB_DIR:/bomb:ro" "$IMAGE" \
			gdb --batch -ex "file /bomb/bomb" -ex "$*" -ex quit
		;;
esac
```
