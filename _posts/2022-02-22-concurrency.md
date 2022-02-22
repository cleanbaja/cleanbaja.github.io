## A greater look at Locking

Have you always wondered how multicore systems share the same resources, without causing data-races or corrupted memory?
The answer lays in two key structures, called the mutex and the semaphore, whose purpose is to hold/stall a resource,
until the CPU who is currently using it is done. This practice of making CPUs wait until a resource is available
is called Locking, and together, we'll take a deeper look into how it really works.

---

### Mutex

A Mutex (short for mutually exclusive) is a type of lock, which at the core, has a bit that is mutually exclusive with the task
at hand. Mutexes are most commonly used to protect a resource, such as a area of memory. Only one thread can hold a mutex at once, 
while the other threads are put to sleep, until it becomes available. Then they are awakened by the operating system.

Example python implementation
```python
import threading, time, random
 
mutex = threading.Lock()
class thread_one(threading.Thread):
	def run(self):
		global mutex
		print ("thread_one is sleeping")
		time.sleep(random.randint(1, 5))
		print("thread_one has now finished")
		mutex.release()
 
class thread_two(threading.Thread):
	def run(self):
		global mutex
		print ("thread_two is sleeping")
		time.sleep(random.randint(1, 5))
		mutex.acquire()
		print("thread_two has now finished")

mutex.acquire()
t1 = thread_one()
t2 = thread_two()
t1.start()
t2.start() 
```

Output:
```
thread_one is sleeping
thread_two is sleeping
thread_one has now finished
thread_two has now finished
```

As you can see, mutexes are relativly useful for synchronzining threads. Although we still have to take a look at how mutexes 
are implemented in the linux kernel. For starters, linux's mutex code is located in [include/linux/mutex.h](https://github.com/torvalds/linux/include/linux/mutex.h).
The header contains many functions and structures, but only three structures/functions are relevant to us. These are `struct mutex`, 
`mutex_lock()` and `mutex_unlock()`. Their declarations are as follows.

```c
struct mutex {
	atomic_long_t		owner;
	spinlock_t		wait_lock;
	struct list_head	wait_list;
};

extern void mutex_lock(struct mutex* lock);
extern void mutex_unlock(struct mutex* lock);
```

The structure of a mutex is actually relativly simple, since its only a list of threads that are waiting on the mutex, along with the spinlock 
which the active threads spin on. Next, there is a atomic integer `owner`, which has the purpose of making sure that only the owner can
unlock the mutex. Finally, the two functions `mutex_lock()` and `mutex_unlock()` use the assembly instruction `cmpxchg` on x86, to speed up
the acquiring of the lock.

---

### Semaphore

TODO

