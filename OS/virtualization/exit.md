## exit

`exit` 用于**正常终止当前进程**。调用 `exit(status)` 后，当前进程结束运行，并把退出状态 `status` 返回给父进程。

父进程可以通过 `wait` / `waitpid` 获得子进程的退出状态。


```c
#include <sys/wait.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        exit(5);
    } else {
        int status;
        wait(&status);

        if (WIFEXITED(status)) {
            printf("child exit code = %d\n", WEXITSTATUS(status));
        }
    }
}
```

可能输出：

```text
child exit code = 5
```

其中：

| 宏                     | 含义                             |
| --------------------- | ------------------------------ |
| `WIFEXITED(status)`   | 子进程是否正常调用 `exit` / `return` 退出 |
| `WEXITSTATUS(status)` | 取得子进程的退出码                      |

> 在 Unix/Linux 中，父进程通常只能通过 `wait` 系列函数获得子进程的退出信息。

---

## exit 会做哪些清理

`exit` 是 C 库函数，退出前会做一系列用户态清理工作。

| 清理内容              | 说明                   |
| ----------------- | -------------------- |
| 调用 `atexit` 注册的函数 | 按注册的相反顺序执行           |
| 刷新标准 I/O 缓冲区      | 例如 `printf` 缓冲内容会被写出 |
| 关闭标准 I/O 流        | 如 `FILE *` 层面的流      |
| 调用底层退出系统调用        | 最终让内核终止进程            |

例如：

```c
#include <stdio.h>
#include <stdlib.h>

void goodbye() {
    printf("atexit function\n");
}

int main() {
    atexit(goodbye);

    printf("main end\n");
    exit(0);
}
```

输出：

```text
main end
atexit function
```

> `exit` 是“有礼貌地退出”：先做 C 运行库层面的清理，再真正终止进程。

---

## return 与 exit 的关系


| 位置       | `return` 的作用 | `exit` 的作用 |
| -------- | ------------ | ---------- |
| `main` 中 | 结束进程         | 结束进程       |
| 普通函数中    | 只返回调用者       | 直接结束整个进程   |

> `return` 是“函数返回”，`exit` 是“进程退出”。

> 只有在 `main` 中，`return status` 才可以看作进程以 `status` 状态退出。

---

## exit 与 _exit

`exit` 和 `_exit` 都能终止进程，但层次不同。

| 对比项              | `exit`      | `_exit`                  |
| ---------------- | ----------- | ------------------------ |
| 层次               | C 库函数       | 系统调用封装                   |
| 是否刷新 `stdio` 缓冲区 | 是           | 否                        |
| 是否调用 `atexit` 函数 | 是           | 否                        |
| 是否关闭文件描述符        | 进程终止后内核都会回收 | 进程终止后内核都会回收              |
| 常见使用场景           | 普通程序正常退出    | `fork` 后子进程异常退出、避免重复刷新缓冲 |

`exit` 大致可以理解为：

$$
\text{exit} = \text{用户态清理} + \text{内核终止进程}
$$

`_exit` 大致可以理解为：

$$
\text{\_exit} = \text{直接进入内核终止进程}
$$

例如：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    printf("hello");   // 没有换行，可能还在缓冲区

    if (fork() == 0) {
        exit(0);
    }

    exit(0);
}
```

由于 `fork` 会复制用户态缓冲区，如果 `printf("hello")` 的内容还没刷新，父子进程都调用 `exit` 时，可能都刷新一次，于是可能输出两次 `hello`。

更安全的子进程异常退出写法是：

```c
if (fork() == 0) {
    // execve 失败时
    _exit(1);
}
```

> `fork` 后子进程如果不打算正常走完整个 C 运行库退出流程，常用 `_exit`，避免把父进程继承来的缓冲区又刷新一遍。

---

## exit 后内核做什么

当进程真正退出时，内核会回收该进程的大部分资源。

| 资源         | 退出后处理             |
| ---------- | ----------------- |
| 用户态地址空间    | 释放                |
| 打开的文件描述符   | 关闭，打开文件对象引用计数减少   |
| 信号量、锁等内核对象 | 按具体机制处理           |
| 进程状态       | 变为终止状态            |
| 退出码        | 暂时保存，等待父进程读取      |
| PID 和进程表项  | 在父进程 `wait` 后完全释放 |

进程退出后，并不是马上“彻底消失”。在父进程读取退出状态之前，子进程会保留一个很小的进程表项，形成**僵尸进程**。

$$
\text{子进程 exit} \rightarrow \text{僵尸状态} \rightarrow \text{父进程 wait} \rightarrow \text{彻底回收}
$$

> `wait` 的作用不是让子进程停止，而是回收已经结束或将要结束的子进程，并取得退出状态。

---

