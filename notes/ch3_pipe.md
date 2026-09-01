# Pipe
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/1ec05402-7ef4-4d63-a408-1a27f9519d17" />
## type of pipe

## making a anonymous pipe

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    int fd[2];

    if (pipe(fd) == -1) {
        perror("pipe failed");
        return 1;
    }

    printf("Read FD: %d, Write FD: %d\n", fd[0], fd[1]);

    return 0;
}
```
where
- `fd[0]` is for read , just like the `stdin` is 0
- `fd[1]` is for write , just like the `stdout` is 1

## sealing the pipe
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/1aa7b65f-2664-4c7b-84e6-a2367bf077ac" />

since when a pipe is created , the related process would get the fd at the both end , and `read()` function would return 0 to notify the reader that the pipe is empty **only if** all the **write fd** is closed , which means that we are supposed to close one end once the pipe is created.
```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/wait.h>

int main() {
    int fd[2];
    pid_t pid;
    char buffer[100];

    if (pipe(fd) == -1) {
        perror("pipe failed");
        return 1;
    }

    pid = fork();
    if (pid < 0) {
        perror("fork failed");
        return 1;
    }

    if (pid == 0) {
        // Child process: reads from the pipe
        close(fd[1]); // Seal (close) the unused write end

        int n = read(fd[0], buffer, sizeof(buffer) - 1);
        if (n > 0) {
            buffer[n] = '\0';
            printf("Child received: %s\n", buffer);
        }
        
        close(fd[0]); // Close the read end when finished
        exit(0);
    } else {
        // Parent process: writes to the pipe
        close(fd[0]); // Seal (close) the unused read end

        char *msg = "Hello through the sealed pipe!";
        write(fd[1], msg, strlen(msg));
        
        close(fd[1]); // Seal (close) the write end to send EOF to the reader

        wait(NULL); // Wait for the child process to complete
    }

    return 0;
}
```
## making a named pipe
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/d7a5072e-34a4-4836-be94-f667fafd70d1" />

name pipe is a special file for IPC (interProcess communication) , especially for the communication among the process with no parent-child relationship.

since it is just a file , we can tread it like a file by using file api ( `open` `read` `write` by the file descriptor point at that pipe)

by the way , this pipe is basically a FIFO queue , and it is allowed to be read by multiple process , however , once the data is read by the specific process , this data would be comsumed which means that the data cant be shared among the comsuming processes.

> note that the named pipe require both the reader and writer exist if you open with default config.
### creating named pipe by system call
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    const char *path = "/tmp/my_named_pipe";

    // Create a FIFO named pipe with read/write permissions for the owner (0666)
    if (mkfifo(path, 0666) == -1) {
        perror("mkfifo failed");
        exit(1);
    }

    printf("Named pipe created successfully at %s\n", path);
    return 0;
}

```
### create the named pipe by shell command
```bash
mkfifo /tmp/my_shell_pipe
```
