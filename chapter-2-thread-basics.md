# Chapter 2: Thread Basics and Synchronization

## 1. Theoretical Explanations

### 1.1 Thread Lifecycle
1. New: Thread object is created but not yet started
2. Runnable: Thread is ready to run and waiting for CPU time
3. Running: Thread is currently executing
4. Blocked/Waiting: Thread is temporarily inactive (e.g., waiting for I/O or synchronization)
5. Terminated: Thread has completed execution or been stopped

### 1.2 Thread Priorities
- Threads can be assigned priorities to influence scheduling
- Higher priority threads are generally favored by the scheduler
- Priorities should be used sparingly and not relied upon for correctness

### 1.3 Thread Synchronization
- Mechanism to control access to shared resources
- Prevents race conditions and ensures data consistency
- Achieved through various constructs like synchronized methods, blocks, and locks

### 1.4 Thread Safety
- Property of code that can be safely executed by multiple threads concurrently
- Achieved through proper synchronization and immutable shared state

### 1.5 Thread Local Storage
- Allows each thread to have its own private copy of variables
- Useful for maintaining thread-specific data without synchronization overhead

## 2. Code Examples

### 2.1 Thread Lifecycle Demonstration

```java
public class ThreadLifecycleDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            try {
                System.out.println("Thread is running");
                Thread.sleep(2000);
                System.out.println("Thread woke up and is terminating");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        System.out.println("Thread state after creation: " + thread.getState());

        thread.start();
        System.out.println("Thread state after starting: " + thread.getState());

        Thread.sleep(1000);
        System.out.println("Thread state while sleeping: " + thread.getState());

        thread.join();
        System.out.println("Thread state after termination: " + thread.getState());
    }
}
```

### 2.2 Synchronized Method Example

```java
public class SynchronizedCounter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized void decrement() {
        count--;
    }

    public synchronized int getCount() {
        return count;
    }

    public static void main(String[] args) throws InterruptedException {
        SynchronizedCounter counter = new SynchronizedCounter();
        Thread incrementThread = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        Thread decrementThread = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.decrement();
            }
        });

        incrementThread.start();
        decrementThread.start();

        incrementThread.join();
        decrementThread.join();

        System.out.println("Final count: " + counter.getCount());
    }
}
```

### 2.3 ThreadLocal Example

```java
public class ThreadLocalExample {
    private static final ThreadLocal<Integer> threadLocalValue = new ThreadLocal<>();

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(() -> {
            threadLocalValue.set(1);
            System.out.println("Thread 1: " + threadLocalValue.get());
        });

        Thread thread2 = new Thread(() -> {
            threadLocalValue.set(2);
            System.out.println("Thread 2: " + threadLocalValue.get());
        });

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println("Main thread: " + threadLocalValue.get());
    }
}
```

## 3. Important Use Cases and Scenarios

1. **Banking Systems**: Ensuring atomic transactions and consistent account balances
2. **E-commerce Platforms**: Managing concurrent user sessions and orders
3. **Caching Systems**: Implementing thread-safe caches for improved performance
4. **Logging Frameworks**: Providing thread-safe logging capabilities
5. **Resource Pools**: Managing shared resources like database connections
6. **Game Engines**: Handling multiple game objects and physics simulations concurrently

## 4. Best Practices and Common Pitfalls

### Best Practices:
1. Use `synchronized` sparingly and with a clear scope
2. Prefer higher-level concurrency utilities over low-level synchronization
3. Always acquire locks in a consistent order to prevent deadlocks
4. Use `volatile` keyword for simple flags that are read/written by multiple threads
5. Implement proper exception handling in threaded code
6. Use thread-safe collections when sharing data structures between threads

### Common Pitfalls:
1. Overusing synchronization, leading to contention and poor performance
2. Forgetting to synchronize access to shared mutable state
3. Creating too many threads, causing resource exhaustion
4. Incorrect use of `wait()` and `notify()` methods
5. Ignoring `InterruptedException`, breaking the interruption mechanism
6. Assuming that operations on primitive types are atomic (except for `long` and `double`)

## 5. References for Further Reading and Practical Exercises

### Books:
1. "Java Concurrency in Practice" by Brian Goetz
2. "Concurrent Programming in Java: Design Principles and Patterns" by Doug Lea
3. "Java Threads and the Concurrency Utilities" by Jeff Friesen

### Online Resources:
1. Java Documentation on Thread Synchronization: [https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html](https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html)
2. Baeldung's Guide to Java Synchronization: [https://www.baeldung.com/java-synchronization](https://www.baeldung.com/java-synchronization)

### Practical Exercises:
1. Implement a thread-safe singleton pattern
2. Create a custom thread pool with a fixed number of worker threads
3. Develop a multi-threaded file search program
4. Implement a concurrent hash map from scratch
5. Create a thread-safe object cache with timeout functionality
