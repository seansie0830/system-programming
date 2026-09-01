# Process
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/d078dde4-0801-4962-bccd-59af4cc83dbc" />

for every UNIX process have

- its own independency memory space
- process execute its instruction indepeneency among all the processes.
- by scheduling from OS , multiple process can run at the same time even in single core cpu
> note that the overhead of scheduling to switch among process is costly since they have independent component to sync and restore. please take this into account if performance really matters.
## create a new process
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/1909226c-4657-42a9-a4e7-612b2c50e9ab" />


in the convention of the UNIX system , we create the process from the parent process by `fork()` function , after executing , the child process would have the same memory state to the parent process , which means that both parent and child proc are running exact same instructions after fork.

to differentiate who  the parent or the child is , we should examine the pid , 0 is for the child process, while pid>0 is for the parent process which is the pid of child process.

introducing the special type in this chapter `pid_t` , it is just a platform-dependent integer type with the meaning of the process id .

you can simply tread it as the integer representing the process id.
### example
```c
#include<unistd.h>
#include<stdio.h>

int main(){
  pid_t pid = fork();
  if(pid <0) {
    perror("fork");
  }
  if(pid ==0 ){
    printf("hello from child\n");
  }
  else if (pid >0){
    sleep(3); // use hardcoded synchronization
    printf("hello from parent\n");
  }
}
```

# managing the process from the parent process
in the prev example ,in order to avoid that the parent process terminate before the child prcess ,  we use the hard-coded synchronization `sleep(3)` to ensure this.

however , this approach have some significant drawbacks . For example , the execution time is nondeterministic by the latency of I/O , network and user operation , which cause underestimate and overestimation causing performacne degradation or the inappropriately kill the in-executation child process.

So in this situation , we introducing the way to manage the process from the parent.
## wait()
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/8b7fe359-4dc4-43bf-913a-39caab5165c6" />

 prototype `pid_t wait(int *stat_loc);` @ `sys/wait.h`

where
- `pid_t` returns the very first terminated child process
- `stat_loc` is the pointer point to the area to store the status of the designed child process to manage.

### example 1 single child management from parent proc with wait()
```c
#include <unistd.h>
#include <stdio.h>
#include <sys/wait.h> // Required header for wait()

int main() {
    pid_t pid = fork();

    if (pid < 0) {
        perror("fork");
        return 1;
    }

    if (pid == 0) {
        // Child process execution
        printf("hello from child\n");
    } 
    else {
        // Parent process execution
        wait(NULL); // Blocks parent until the child process terminates
        printf("hello from parent\n");
    }

    return 0;
}

```

### example2 multiple child process management from the parent with wait()
```c
#include <unistd.h>
#include <stdio.h>
#include <sys/wait.h>

#define NUM_CHILDREN 5

int main() {
    // Phase 1: Fork multiple child processes
    for (int i = 0; i < NUM_CHILDREN; i++) {
        pid_t pid = fork();

        if (pid < 0) {
            perror("fork failed");
            return 1;
        }

        if (pid == 0) {
            // Child process execution
            printf("Hello from child %d (PID: %d)\n", i + 1, getpid());
            return 0; // Terminate child immediately so it does not loop and fork
        }
    }

    // Phase 2: Parent waits for ALL child processes to finish
    printf("Parent (PID: %d) waiting for children...\n", getpid());
    
    for (int i = 0; i < NUM_CHILDREN; i++) {
        pid_t finished_pid = wait(NULL); // Blocks until any one child finishes
        printf("Parent: Child with PID %d has finished\n", finished_pid);
    }

    printf("hello from parent\n");
    return 0;
}
```
## waitpid
<img width="512" height="512" alt="image" src="https://github.com/user-attachments/assets/0a6c3961-c4a3-427a-a931-bd72fe0cc7c3" />

this is the `wait()` with specificity version , targeting at the specific pid .

prototype 
```c
#include <sys/wait.h>
pid_t waitpid(pid_t pid, int *status, int options);
```



