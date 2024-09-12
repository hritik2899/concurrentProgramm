# Chapter 3: Locks and Deadlocks

## 1. Theoretical Explanations

### 1.1 Locks
- Synchronization mechanism for controlling access to shared resources
- More flexible than `synchronized` keyword
- Types: ReentrantLock, ReadWriteLock, StampedLock

### 1.2 Lock Interface
- `lock()`: Acquire the lock, blocking if necessary
- `unlock()`: Release the lock
- `tryLock()`: Attempt to acquire the lock without blocking
- `newCondition()`: Create a new condition variable associated with the lock

### 1.3 Deadlocks
- Situation where two or more threads are unable to proceed because each is waiting for the other to release a lock
- Four necessary conditions (Coffman conditions):
  1. Mutual Exclusion
  2. Hold and Wait
  3. No Preemption
  4. Circular Wait

### 1.4 Livelock
- Threads are actively performing operations, but these operations do not move the state of the system forward

### 1.5 Starvation
- A thread is unable to gain regular access to shared resources and is unable to make progress

## 2. Code Examples

### 2.1 ReentrantLock Example

```java
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockExample {
    private final ReentrantLock lock = new ReentrantLock();
    private int count = 0;

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }

    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReentrantLockExample example = new ReentrantLockExample();
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                example.increment();
            }
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                example.increment();
            }
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Final count: " + example.getCount());
    }
}
```

### 2.2 ReadWriteLock Example

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockExample {
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private int data = 0;

    public int readData() {
        rwLock.readLock().lock();
        try {
            return data;
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public void writeData(int newValue) {
        rwLock.writeLock().lock();
        try {
            data = newValue;
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public static void main(String[] args) {
        ReadWriteLockExample example = new ReadWriteLockExample();
        
        // Multiple readers can access simultaneously
        new Thread(() -> System.out.println("Reader 1: " + example.readData())).start();
        new Thread(() -> System.out.println("Reader 2: " + example.readData())).start();
        
        // Writer blocks all other operations
        new Thread(() -> example.writeData(42)).start();
        
        // This will wait until write is complete
        new Thread(() -> System.out.println("Reader 3: " + example.readData())).start();
    }
}
```

### 2.3 Deadlock Example

```java
public class DeadlockExample {
    private static final Object RESOURCE_A = new Object();
    private static final Object RESOURCE_B = new Object();

    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            synchronized (RESOURCE_A) {
                System.out.println("Thread 1: Holding Resource A...");
                try { Thread.sleep(100); } catch (InterruptedException e) {}
                System.out.println("Thread 1: Waiting for Resource B...");
                synchronized (RESOURCE_B) {
                    System.out.println("Thread 1: Holding Resource A and Resource B");
                }
            }
        });

        Thread thread2 = new Thread(() -> {
            synchronized (RESOURCE_B) {
                System.out.println("Thread 2: Holding Resource B...");
                try { Thread.sleep(100); } catch (InterruptedException e) {}
                System.out.println("Thread 2: Waiting for Resource A...");
                synchronized (RESOURCE_A) {
                    System.out.println("Thread 2: Holding Resource B and Resource A");
                }
            }
        });

        thread1.start();
        thread2.start();
    }
}
```

## 3. Important Use Cases and Scenarios

1. **Database Management Systems**: Implementing row-level locking for concurrent transactions
2. **File Systems**: Managing concurrent access to files and directories
3. **Cache Systems**: Implementing efficient read/write operations on cached data
4. **Resource Pools**: Managing shared resources like database connections or thread pools
5. **Concurrent Data Structures**: Implementing thread-safe collections and data structures
6. **Game Engines**: Managing access to shared game state and resources

## 4. Best Practices and Common Pitfalls

### Best Practices:
1. Always release locks in a `finally` block to ensure they are released even if an exception occurs
2. Use `tryLock()` with a timeout to avoid indefinite waiting
3. Keep critical sections (locked regions) as small as possible
4. Use ReadWriteLock when you have operations that don't modify the resource
5. Consider using higher-level concurrency utilities (e.g., `java.util.concurrent`) instead of low-level locks
6. Acquire locks in a fixed, global order to prevent deadlocks

### Common Pitfalls:
1. Forgetting to release locks, leading to deadlocks
2. Holding locks for too long, reducing concurrency
3. Using the wrong type of lock for the use case (e.g., using an exclusive lock when a read lock would suffice)
4. Nesting locks incorrectly, potentially causing deadlocks
5. Ignoring the return value of `tryLock()`
6. Using locks when a simpler synchronization mechanism would suffice

## 5. References for Further Reading and Practical Exercises

### Books:
1. "Java Concurrency in Practice" by Brian Goetz
2. "The Art of Multiprocessor Programming" by Maurice Herlihy and Nir Shavit
3. "Concurrent Programming in Java: Design Principles and Patterns" by Doug Lea

### Online Resources:
1. Oracle's Java Tutorial on Locks: [https://docs.oracle.com/javase/tutorial/essential/concurrency/newlocks.html](https://docs.oracle.com/javase/tutorial/essential/concurrency/newlocks.html)
2. Baeldung's Guide to the Java Lock API: [https://www.baeldung.com/java-concurrent-locks](https://www.baeldung.com/java-concurrent-locks)

### Practical Exercises:
1. Implement a thread-safe bounded buffer using ReentrantLock and Condition variables
2. Create a custom ReadWriteLock implementation
3. Develop a deadlock detection algorithm
4. Implement a resource hierarchy solution to prevent deadlocks in a system with multiple resources
5. Create a concurrent hash map using fine-grained locking
