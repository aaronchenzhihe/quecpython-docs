# FAQ on Multi-thread Development

### **Is there any limit to the number of threads that can be created? How many can be created at maximum?**

It is 16 at maximum.

### **What is the size of thread stack ?**

The default is 8192 bytes, the Unisoc platform supports a maximum of 65535 bytes, and the ASR platform and CAT-M platform do not set ceiling.

### **How to determine whether the thread status is normal?**

When programming multiple threads, it is recommended to monitor other sub-threads in the main thread to avoid exceptions caused by them, resulting in stopping running. 

### **When there is no semaphore related interface in the multi-thread function, how to implement the semaphore function?**
The implementation of QuecPython's multi-thread module is originated from the thread of standard Python. As a result, the semaphore-related interfaces are not provided. 

If is is needed to implement a semaphore-like function, the mutex  ([`_thread.allocate_lock`](../../api_reference/stdlib/_thread.md#Mutex) method) or message queue ([`queue.Queue`](../../api_reference/syslib/Queue.md) method) can be applied. 

If the maximum value of the semaphore is only 1, the mutex or message queue can be applied to implement the semaphore function; if the value of the semaphore is greater than 1, the message queue will be valid to implement the semaphore function.

The basic logic of using a mutex to implement the semaphore function is:

- Create a mutex object
- After that, please lock immediately (The mutex can be obtained at this time, and it will return immediately after calling interface)
- Call lock interface in place where is needed to wait semaphore (The lock cannot be obtained at this time since the interface is blocked and wait for others to release the lock to simulate the behavior of waiting for the semaphore)
- When an event occurs, call the interface that releases the mutex (The mutex is released then. the place waiting for the semaphore acquires the lock, which can be acquired in which waiting for the semaphore. Additionally, the interface returns to simulate the behavior of releasing the semaphore)

The basic logic to implement the semaphore function via message queue:

- Create a message queue object (The size of the queue is the maximum value of the semaphore)

- In place where you need to wait for the semaphore, call the interface waiting for the message

- In place where an event occurs, call the interface for sending messages (If it is a simple semaphore and no need to care about the message content, please send a message with a value of `None`. )

### **Will it cause memory leakage and exception in system after the thread is executed and exits directly? **

The thread can exit directly after it is finished, and the bottom layer of the system will automatically release resources without causing memory leakage or system exception.