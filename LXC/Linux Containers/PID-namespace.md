# PID namespaces

- The Process ID (PID) namespaces provide the ability for a process to have an ID that already exists in the default namespace, 
for example an ID of `1`. This allows for an init system to run in a container with various other processes, without causing a collision with the rest of the PIDs on the same OS.

To demonstrate this concept, open up `pid_namespace.c`:

```
#define _GNU_SOURCE 
#include <stdlib.h> 
#include <stdio.h> 
#include <signal.h> 
#include <sched.h> 
 
static int childFunc(void *arg) 
{ 
    printf("Process ID in child  = %ld\n", (long) getpid()); 
} 

```
First, we include the headers and define the childFunc function that the `clone()` system call will use. 
The function prints out the child PID using the `getpid()` system call:
```
static char child_stack[1024*1024]; 
 
int main(int argc, char *argv[]) 
{ 
    pid_t child_pid; 
 
    child_pid = clone(childFunc, child_stack + 
    (1024*1024),      
    CLONE_NEWPID | SIGCHLD, NULL); 
 
    printf("PID of cloned process: %ld\n", (long) child_pid); 
    waitpid(child_pid, NULL, 0); 
    exit(EXIT_SUCCESS); 
} 

```

In the `main()` function, we specify the stack size and call `clone()`, passing the child function childFunc, 
the stack pointer, the CLONE_NEWPID flag, and the SIGCHLD signal. The CLONE_NEWPID flag instructs `clone()` to create a new PID namespace
and the `SIGCHLD` flag notifies the parent process when one of its children terminates. The parent process will block on `waitpid()` if the child process 
has not terminated.

Compile and then run the program with the following:

```
root@server:~# gcc pid_namespace.c -o pid_namespace
root@server:~# ./pid_namespace
PID of cloned process: 17705
Process ID in child  = 1
root@server:~#

```

From the output, we can see that the child process has a PID of 1 inside its namespace and 17705 otherwise.

# Note
```
Note that error handling has been omitted from the code examples for brevity.
```
