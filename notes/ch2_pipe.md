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

