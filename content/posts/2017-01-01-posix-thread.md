---
title: Posix Thread 基础
date: 2017-01-01 00:14:03
tags: [Linux, Thread]
tegories: [Coding]
---

### 线程概念

一个线程包含在进程中表示一个 execution context 的必要信息，包括 *thread ID*, a set of register values, a stack, a scheduling priority and policy, a signal mask, an `errno` variable, and thread-specific data.

进程中的所有东西都是线程间共享的，包括 text of the executable program, the programs's global and heap memory, the stacks, and the file descriptors. <!-- more -->

### 线程 ID

每个进程都有一个 *系统内* 唯一的ID，由`pid_t`类型表示，类似的，每个线程都一个 *进程内* 唯一的ID, 由`pthread_t`类型表示。`pthread_t`的具体类型由实现决定，可能是整数、指针或者结构体，所以不能之间判断两个`pthread_t`是否相等，需要用`pthread_equal`函数：

```c
int pthread_equal(pthread_t tid1, pthread_t tid2);
				Returns nonzeron if equal, 0 otherwise.
```

一个线程可以通过`pthread_self`函数获取自己的ID:

```c
pthread_t pthread_t_self(void);
```

### 线程创建

```c
int pthread_create(pthread_t *tidp,
                  const pthread_attr_t *attr,
                  void *(*start_routine)(void *),
                   void *arg);
									Returns: 0 if OK, error number on failure
```

*tidp*被设置为新创建的线程的ID，*attr*参数用来控制线程的属性，这里用`NULL`表示使用默认的属性。

新创建的线程从 *start_routine*函数处开始执行，这个函数接受一个无类型的指针 `void* arg`.

> 无类型指针（通用指针），只记录了内存的起始地址，指向的目标的大小并不知道，所以不能dereference，只能将它强制转换为其他类型的指针，此时目标的大小已经知道，所以可以dereference，然后对内存做各种操作。

当线程创建完成后，并没有保证是调用`pthread_create`的线程先执行还是新创建的线程先执行。所以如果`pthread_create`的第一个参数是一个全局变量，那么当新线程开始执行时，主线程可能还有从`pthread_create`中返回，也就是说全局变量可能还没有赋值，所以新线程访问到的全局变量可能是未初始化的变量。

### 线程终止

任何一个线程调用`exit`, `_exit` 或 `Exit`, 都会导致整个进程终止。当默认行为是终止进程时，比如一个信号发给一个线程，也会终止整个进程。

一个线程可以以三种方式终止而不用终止整个进程：

1. 线程routine执行到return语句，return值会作为线程的exit code
2. 线程被同一个进程中的其他线程 cancel
3. 线程自己调用pthread_exit，其参数作为 exit code

```c
int pthread_exit(void *rval_ptr)
```

exit code 可以被其他调用 `pthread_join`的线程获取：

```c
int pthread_join(pthread_t thread, void ** rval_ptr);
								Return: 0 if OK, error number on failure
```

调用`pthread_join`的线程会阻塞直到其等待的线程退出。如果被等待的线程正常退出(return)，*rval_ptr*将是 return code。如果被等待的线程是被 canceled，*rval_ptr*会被设为 `PTHREAD_CANCELED`(-1).

>  线程可以cancel自己吗？

通过调用`pthread_join`，被等待的线程会被置为 *detached state*, 所以其资源？。如果被等待的线程已经是 *detached state*, 那么`pthread_join`会失败，返回`EINVAL`。 



一个线程可以调用`pthread_cancel`来请求同一个进程中的另一个线程 be canceled：

```c
int pthread_cancel(pthread_t tid);
									Returns: 0 if OK, error number on failure
```

默认情况下，`pthread_cancel`会导致 `tid`指定的线程就像自己以参数`PTHREAD_CANCELED`调用`pthread_exit`一样。但是，一个线程可以选择忽略或者控制自己如果 cancel。`pthread_cancel`函数并不等待线程`tid`退出，它仅仅发出这个cancel请求。

线程退出时做清理：

```c
void pthread_cleanup_push(void (*routine)(void *),void *arg);
void pthread_cleanup_pop(int execute);
```



默认情况下，线程的退出状态会被一直保留，直到为它调用`pthread_join` ，而一个 *detached* 线程的储存空间可能会被立即回收。如果一个线程已经*detached*, 那么我们无法为其调用`pthread_join`，为一个*detached* thread 调用 `pthread_join`会导致未定义行为。

通过`pthread_detach`，我们可以将一个线程 *detach*：

```c
int pthread_deatch(pthread_t tid);
```

### 线程同步

如果所有的操作都只需要一个 memory cycle，那么就不会有不一致的问题。如果修改变量的操作是原子的，那么也不会有竞争的问题。那么也不需要同步。但实际情况不是这样。

1. Mutex

   一个 *mutex* 是一个互斥锁，当我们访问一个共享资源时设置它，当访问完成时释放它。当 mutex is set, 所有其他试图set它的线程都会被block直到有人release它。当它被释放时，所有等待它的线程都变为可运行的，第一个运行的 will be able to set the lock，其他的线程继续等到它available again。这样，每次只能有一个线程 proceed。

   这种*互斥*只有在所有线程都遵守相同的访问规则时才有效，比如说所有人在访问变量之前都必须获取锁。如果允许一个线程在访问前不获取锁，那么即使其他线程正在持有锁，它也依然可以访问到变量，这可能造成不一致。

   一个互斥变量由 `pthread_mutex_t`表示，使用前必须初始化。要么初始化为常量`PTHREAD_MUTEX_INITIALIZER` (只用于静态分配的mutex)，要么使用`pthreat_mutex_init` (动态分配的mutex，free mutex前必须调用`pthread_mutex_destroy`):

   ```c
   int pthread_mutex_init(pthread_mutex_t *mutex,
                   const pthread_mutexattr_t *attr);
   int pthread_mutex_destroy(pthread_mutex_t *mutex);
   ```

   要锁定一个 mutex，我们调用 `pthread_mutex_lock`，如果这个 mutex 已经被锁定，那么调用的线程被阻塞，直到 mutex 被解锁。解锁一个 mutex 我们调用 `pthread_mutex_unlock` ：

   ```c
   int pthread_mutex_lock(pthread_mutex_t *mutex);
   int pthread_mutex_unlock(pthread_mutex_t *mutex);
   int pthread_mtuex_trylock(pthread_mutex_t *mutex);
   ```

   如果线程不想被阻塞，它可以调用 `pthread_mutex_trylock`，如果 mutex 没有被锁，那么调用 `pthread_mutex_trylock`会锁定这个 mutex，返回0，否则调用会失败，返回`EBUSY`，不阻塞，且不会锁定这个mutex。

### 死锁避免

一个线程试图锁定一个 mutex 两次会造成死锁。

如果程序中使用了多个 mutex， 当一个线程持有一个 mutex 的锁并且试图锁定第二个 mutex，同时另外一个线程持有第二个 mutex的锁并且试图锁定一个 mutex 时，就会造成死锁。

小心的控制加锁的顺序可以避免死锁。

如果一个线程持有一个锁，并且调用`pthread_mutex_trylock`成功，那么线程可以继续，否则线程就释放自己持有的所有的锁，clean up, try again later。



### 参考资料

1. APUE.3e