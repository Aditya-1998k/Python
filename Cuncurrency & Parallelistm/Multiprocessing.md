Python's GIL prevents `multiple threads` from executing bytecode simultnously.
For `CPU bound` tasks (Heavey computation), threading does not help.
Eg: Heavy computation, Image Processing, ML Training etc

Multiprocessing achieves true parallelism by using seperate processes.
Each process has its own `interpreter and memory` space.

**Key concepts:**
1. `Process` → independent execution unit with its own memory.
2. `Pool` → manages a fixed number of worker processes.
3. `IPC (Inter-Process Communication)` → since processes don’t share memory, use Queue, Pipe, or Manager.
