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
### Parameter Breakdown

- **`pathname`**: The path to the target file or device.
  - **Absolute paths** (e.g., `/var/log/syslog`) resolve starting from the root directory (`/`).
  - **Relative paths** (e.g., `new_file.txt` or `./config.json`) resolve relative to the current working directory (CWD) of the running process.
- **`flags`**: Controls how the file is opened and manipulated. It is formed by bitwise OR-ing (`|`) values together:
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/0264b3e7-159a-4070-9643-fb986ea82900" />

  - **Access Modes (Must specify exactly one):**
    - `O_RDONLY`: Open for read-only access.
    - `O_WRONLY`: Open for write-only access.
    - `O_RDWR`: Open for both reading and writing.
  - **Creation & Action Flags (Optional modifiers):**
    - `O_CREAT`: Creates the file if it does not exist (requires the `mode` argument).
    - `O_EXCL`: Used with `O_CREAT` to ensure the call fails if the file already exists (prevents race conditions).
    - `O_TRUNC`: Truncates an existing regular file to length 0 (overwriting all contents).
    - `O_APPEND`: Appends writes to the end of the file instead of starting at byte offset 0.
  - **Operating Mode Flags (Optional):**
    - `O_NONBLOCK`: Opens the file/socket in non-blocking mode.
    - `O_CLOEXEC`: Automatically closes the file descriptor when `exec()` is called.
- **`mode` (Permissions)**: Defines access permissions (e.g., `0644` or `S_IRUSR | S_IWUSR`):
  - If you open a **non-existent file** with the `O_CREAT` flag, `open()` creates the file using the specified permissions.
  - If the file **already exists**, this parameter is simply ignored.
 
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
> note that  return value of the unix system api mostly indicate the status , the convention is that if the return value is `-1` , the procedure fails , always check after invoke the UNIX api to avoid undefined operation based on failure operation.
### result 
 output as followiing
 ```text
$ ./ch1a
the fd I get is 3
```
> why the fd is 3? , because the pre-assigned fd are stdin (0) , stdout(1) , stderr(2) , and mostly the OS arrange the fd incrememtally , so this is why the fd are usually 3 , but this is not guranteed in every enviroment especially the system where the program has feteched a few fd recently
## 📚Read like a bookworm🪱
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/410c9f3c-e9c0-49b5-8c1a-29e125b15eef" />

### example code
using while loop , check if the return value is less than 1 as exception.
coping with the exception
- bytesRead == 0 : successfully reach the end of the file
- bytesRead < 0 : an error occurs.
```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>

#define PAGE_SIZE 64

int main() {
    int fd = open("test.txt", O_RDONLY);
    if (fd < 0) {
        perror("Error opening file");
        return 1;
    }

    char buffer[PAGE_SIZE];
    ssize_t bytesRead;
    int i=0;
    // Loop until read() returns 0 (EOF) or a negative value (Error)
    while ((bytesRead = read(fd, buffer, PAGE_SIZE)) > 0) {
        i++;
        printf("\n\npage %d , offset %ld\n" ,i, bytesRead);
        // --- Process your data here ---
        // Remember: buffer is NOT null-terminated!
        // If you are writing it to stdout, use write():
        write(STDOUT_FILENO, buffer, bytesRead);

    }

    // Check if the loop terminated because of an error or actual EOF
    if (bytesRead < 0) {
        perror("Error reading file");
    } else {
        printf("\n--- End of File Reached Successfully ---\n");
    }


    close(fd);
    return 0;
}
```
## let write into the file

<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/5d83a20f-74d3-4ccd-823a-b37bd601d0cc" />

 
### example
`printf` is just a wrapped and cross-platform write() function , let use the `write` instead to print something on the screen on `stdout`.

remember , treat everythings as a file in unix, the `stdout` is definitely a write-only special file , if you want to output something on screen , write something into this **file**.

note that the fd is 1 by default

```c
#include<unistd.h>
#include <string.h>
#define STDOUT 1
int main(){
	char *letters = "abcdefghijklmnopqrstuvwxyz\n";
	ssize_t size = write(STDOUT , letters , strlen(letters));
	return 0;
}

```
### result
```
abcdefghijklmnopqrstuvwxyz
```
## 🛸seeking to anywhere of the file!
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/e85c5c59-1723-4a7b-ac1f-65ff67cf3ab3" />

## misc
### `fsync`
### `fdatasync`
