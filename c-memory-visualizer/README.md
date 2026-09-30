# C Memory Visualizer

A learning tool that shows what a C program does to memory, step by step.

## Files
- `trace.py`: a gdb Python script. It runs the program line by line and writes a JSON trace.
- `node_cards_viewer.html`: open the JSON trace (and the `.c` file) in it. It has the node cards
  view, the memory layout view and the Watch panel.
- `remove_linked_list_2.c`: an example program.

## How to run
1. `gcc -g -O0 prog.c -o prog`
2. `gdb -q -batch -x trace.py ./prog`
3. This makes `prog.json`. Open it in `node_cards_viewer.html`.

## Watchpoints
`trace.py` uses real gdb hardware watchpoints to catch what library calls like `free()`,
`malloc()`, `memset()` or `sscanf()` write into your memory:
- Just before your code calls a library function, it arms up to 4 watchpoints (x86-64 has 4,
  8 bytes each) on the memory that call is likely to write.
- Every write made inside the library is saved in the trace (`"lib"` on each step).
- If the computer can't do hardware watchpoints, the trace still works, without them
  (`"watchpoints": false`).
