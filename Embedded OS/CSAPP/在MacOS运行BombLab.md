### 拉取 docker 容器
选择 debian 系列
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