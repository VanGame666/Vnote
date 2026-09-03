# Linux 中 `struct thread` 的笔记（位于 `tool` 文件夹）

## 1. 背景
在标准的 Linux 系统编程（glibc/POSIX）中，**不存在**名为 `struct thread` 的原生结构体。  
如果你在项目的 `tool` 文件夹中遇到它，通常来自 **Linux 内核源码中的 `perf` 工具**（路径：`tools/perf/util/thread.h`）。

---

## 2. `struct thread` 的定义（perf 工具内部）
~~~c
struct thread {
    union {
        struct rb_node rb_node;
        struct list_head node;
    };
    struct map_groups *mg;
    pid_t pid_;          /* 进程 ID（部分工具不更新） */
    pid_t tid;           /* 线程 ID */
    pid_t ppid;          /* 父进程 ID */
    int cpu;
    refcount_t refcnt;
    bool comm_set;
    int comm_len;
    bool dead;           /* 线程是否已退出 */
    struct list_head namespaces_list;
    struct rw_semaphore namespaces_lock;
    struct list_head comm_list;
    struct rw_semaphore comm_lock;
    u64 db_id;
    void *priv;
    struct thread_stack *ts;
    struct nsinfo *nsinfo;
    struct srccode_state srccode_state;
#ifdef HAVE_LIBUNWIND_SUPPORT
    void *addr_space;
    struct unwind_libunwind_ops *unwind_libunwind_ops;
#endif
    bool filter;
    int filter_entry_depth;
};
~~~

---

## 3. 用途
- 专用于 **性能分析工具 `perf`**，用于在运行时管理每个线程的上下文信息。
- 存储线程的 PID/TID、地址空间映射（`map_groups`）、通信记录、命名空间、调用栈等。
- 与内核调度无关，纯属 `perf` 用户态内部数据结构。

---

## 4. 与标准线程标识符的区别
| 对比项 | `struct thread` (perf) | `pthread_t` (POSIX) |
|--------|------------------------|----------------------|
| 来源   | `tools/perf/util/thread.h` | `<pthread.h>` |
| 本质   | 显式结构体，包含大量分析字段 | 不透明类型（glibc 中为 `unsigned long`） |
| 用途   | 性能采样、事件追踪 | 应用层多线程控制（创建、等待、同步） |
| 使用场景 | 仅限 `perf` 工具内部 | 所有 POSIX 多线程程序 |

---

## 5. 如何定位你的 `struct thread`
- 在 `tool` 文件夹内使用命令：
  ~~~bash
  grep -r "struct thread" ./tool
  ~~~
- 查找是否有 `perf` 相关的子目录（如 `tool/perf/util/`）。
- 如果是项目自定义结构体，请查阅项目文档。

---

## 6. 关键结论
- `struct thread` 不是系统 API，也不是内核通用结构体（内核用 `task_struct`）。
- 它属于 `perf` 工具的内部数据结构，**不能**直接用于应用层多线程编程。
- 应用层线程编程应使用 `pthread_t` 及 `<pthread.h>` 中的函数。