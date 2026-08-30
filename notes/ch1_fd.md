# Understanding File Descriptors

## Everything is a File

<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/ab3377ee-4756-4908-8bbb-82abcbc4562d" />

Modern operating systems handle countless complex tasks, but at their core, computing boils down to a simple cycle: **read input, process data, and write output**.

This simplicity is made possible by the Unix philosophy: **"Everything is a file."**

By treating almost all I/O operations through the unified abstraction of a "file," developers can interact with heterogeneous resources using the exact same standard interface (`open`, `read`, `write`, `close`). These resources include:

- **Regular files** on disk
- **Hardware devices** (keyboards, serial ports)
- **Pipes** (data streams between processes)
- **Network sockets** (remote machine communication)
- **Virtual devices** (`/dev/zero`, `/dev/random`, `/dev/null`)
- **Standard I/O streams** (`stdin`, `stdout`, and `stderr`)
## Lets get a file descriptor!
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/279e2a78-0014-45c3-a04a-32e5cb4b747a" />

