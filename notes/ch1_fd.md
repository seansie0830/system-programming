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

parameter explaination
- pathname :
- open flags
- permission
  - if you open **unexist** file with `O_CREAT` mode , `open()` would create a new file based on your **given permissions**
  - if the file has been **exist** , the permission would be ignored
 
### example
```c
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    // O_WRONLY: Open for writing
    // O_CREAT: Create the file if it does not exist
    // 0644: Read/Write for owner, Read for group and others
    int fd = open("new_file.txt", O_WRONLY | O_CREAT, 0644);

    if (fd == -1) {
        perror("Error opening file");
        return 1;
    }

    // Use the file descriptor here...
    printf("the fd I get is %d\n",fd);
    close(fd);
    return 0;
}
```
 output as followiing
 ```text
$ ./ch1a
the fd I get is 3
```
> why the fd is 3? , because the pre-assigned fd are stdin (0) , stdout(1) , stderr(2) , and mostly the OS arrange the fd incrememtally , so this is why the fd are usually 3 , but this is not guranteed in every enviroment especially the system where the program has feteched a few fd recently
 

