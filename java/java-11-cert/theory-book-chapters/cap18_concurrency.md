# Chapter 18 - Concurrency

> 1.- Given an instance of a `Stream s` and a `Collection c`, which are valid ways of creating a paralell stream?

The method defined in the `Stream` class to create a parallel stream from an existing stream is `parallel()`; therefore the following is correct:

```
s.parallel()
```

The method defined in the `Collection` class to create a parallel stream from a collection is `parallelStream()`; therefore the following is correct:

```
c.parallelStream()
```

> 2.- Given that the sum of the numbers from 1 (inclusive) to 10 (exclusive) is 45, what are the possible results of executing the following program?

```
import java.util.concurrent.locks.*;
import java.util.stream.*;
public class Bank {
    private Lock vault = new ReentrantLock();
    private int total = 0;
    public void deposit(int value) {
        try {
            vault.tryLock();
            total += value;
        } finally {
            vault.unlock();
        }
    }
    public static void main(String[] unused) {
        var bank = new Bank();
        IntStream.range(1, 10).parallel()
            .forEach(s -> bank.deposit(s));
        System.out.println(bank.total);
    }
}
```

The code may print 45 or may throw an exception.

The `tryLock()` method returns immediately with a value of `false` if the lock cannot be acquired. Unlicke `lock()`, it does not wait for a lock to become available. This code fails to check the return value, resulting in the protected code being entered regardless of whether the lock is obtained.

In some executions (when the `tryLock()` returns `true` on every call), the code will complete sucessfully and print `45` at runtime. On other executions (when `tryLock()` returns `false` at least once), the `unlock()` method will throw an `IllegalMonitorStateException` at runtime.

The `ReentrantLock` class maintains a counter of the number of times a lock has been given to a thread. To release the lock for other threads to use, `unlock()` must be called the same number of times the lock was granted. 

Besides always making sure to release a lock, you also need to make sure that you only release a lock that you actually have. If you attempt to release a lock that you don't have, you will get an exception at runtime.

> 3.- Which of the following statements about the `Callable call()` and `Runnable run()` methods are correct?

The following are correct:

- Both can throw unchecked exceptions
- `Callable` can throw a checked exception
- Both can be implemented with lambda expressions
- `Callable` returns a generic type

First, rember that all methods are capable of throwing unchecked exceptions.

Only `Callable` is capable of throwing checked exceptions, remember it's method signature:

```
V call() throws Exception
```

Both `Runnable` and `Callable` are functional interfaces that can be implemented with lambda expression.

Finally, `Runnable` returns `void` and `Callable` returns a generic type.

> 4.- Which lines need to be changed to make the code compile?

```
ExecutorService service =   // w1
    Executors.newSingleThreadScheduledExecutor();
service.scheduleWithFixedDelay(() -> {
    System.out.println("Open Zoo");
    return null;    // w2
}, 0, 1, TimeUnit.MINUTES);
var result = service.submit(() ->   // w3 
    System.out.println("Wake Staff"));
System.out.println(result.get());   // w4
```

The code does not compile due lines `w1` and `w2`.

The first problem is that although a `ScheduledExecutorService` is created, it is assigned to an `ExecutorService`. The type of the variable on line `w1` would have to be updated to `ScheduledExecutorService` for the code to compile. We could store an instance of `ScheduledExecutorService` in an `ExecutorService` variable, although doing so would mean we'd have to cast the object to call any scheduled methods. Since this code does not cast the object and make use of a scheduled method, line `w1` does not compile.

The second problem is that `scheduleWithFixedDelay()` supports only `Runnable`, not `Callable`, and any attempt to return a value is invalid in a `Runnable` lambda expression, therefore line `w2` will also not compile.

Remember the method definition for `scheduleWithFixedDelay()`:

```
scheduleWithFixedDelay(
    Runnable command,
    long initialDelay, long delay,
    TimeUnit unit)
```

The rest of the lines compile without issue.

> 5.- What statement about the following code is true?

```
var value1 = new AtomicLong(0);
final long[] value2 = {0};
IntStream.iterate(1, i -> 1).limit(100).parallel()
    .forEach(i -> value1.incrementAndGet());
IntStream.iterate(1, i -> 1).limit(100).parallel()
    .forEach(i -> ++value2[0]);
System.out.println(value1+" "+value2[0]);
```

The output cannot be determined ahead of time.

The code compiles and and runs without throwing an exception or entering an infinite loop. The key here is that the increment operator `++` is not atomic. While the first part of the output will always be `100`, the second part is nondeterministic. It could output any value from `1` to `100`, because threads can overwrite each other's work.

> 6.- Which statements about the following code are correct?

```
public static void main(String[] args) throws Exception {
    var data = List.of(2,5,1,9,8);
    data.stream().parallel()
        .mapToInt(s -> s)
        .peek(System.out::println)
        .forEachOrdered(System.out::println);
}
```

The following are correct:

- The `peek()` method will print the entries in an order that cannot be determined ahead of time.
- The `forEachOrdered()` method will print the entries in the order: 2, 5, 1, 9 ,8

The code compiles. The `peek()` method on a parallel stream will process the elements concurrently, so the order cannot be determined ahead of time.

The `forEachOrdered()` method will process the elements in the order they are stored in the stream.

> 7.- Fill in the blanks:_____________ occur(s) when two or more threads are blocked forever but both appear active. ______________ occur(s) when two or more threads try to complete a related task at the same time, resulting in invalid or unexpected data.

The following can fill in the blanks: *Livelock, Race conditions*

Remember that *Liveness* is the ability of an application to be able to execute in a timely manner. Liveness problems are those in which the application becomes unresponsive or in some kind of "stuck" state. The following are Liveness problems:

- **Deadlock:** Deadlock occurs when two or more threads are blocked forever, each waiting on each other.
- **Starvation:** Starvation occurs when a single thread is perpetually denied access to a shared resource or lock. The thread is still active, but it's unable to complete its work as a result of other threads constantly taking the resource that they are trying to access.
- **Livelock:** Livelock occurs when two or more threads are conceptually blocked forever, although they are still active and trying to complete their task. Livelock is a special case of resource starvation in which two or more threads actively try to acquire a set of locks, are unable to do so, and restart part of the process. Livelock is often a result of two threads trying to resolve a Deadlock.
- **Race conditions:** A race condition is an undesirable result that occurs when two tasks, which should be completed sequentially, are completed at the same time.

> 8.- Assuming this class is accessed by only a single thread at a time, what is the result of calling the `countIceCreamFlavors()` method?

```
import java.util.stream.LongStream;
public class Flavors {
    private static int counter;
    public static void countIceCreamFlavors() {
        counter = 0;
        Runnable task = () -> counter++;
        LongStream.range(1, 500)
            .forEach(m -> new Thread(task).run());
        System.out.println(counter);
    }
}
```

The method consistently prints 499. The method looks like it executes a task concurrently, but it actually runs synchronously. In each iteration of the `forEach()` loop, the process waits for the `run()` method to complete before moving on. For this reason, the code is actually thread-safe. It executes a total of 499 times, since the second value of `range()` excludes the 500.

Note that if `start()` had been used instead of `run()`, or the stream was parallel, then the output would be indeterminate.

**Note:** Remember that calling `run()` on a `Thread` or `Runnable` does not actually start a new thread. Such code will wait until the `run()` method is complete before moving on to the next line.

To properly create a new thread, you should use the `start()` method.

> 9.- Which happens when a new task is submitted to an `ExecutorService`, in which there are not threads available.

The following statement is correct:

*The executor adds the task to an internal queue and completes when there is an available thread*

If a task is submitted to a thread executor, and the thread executor does not have any available threads, the call to the task will return immediately with the task being queued internally by the thread executor.

> 10.- What is the result of executing the following code snippet?

```
List<Integer> lions = new ArrayList<>(List.of(1,2,3));
List<Integer> tigers = new CopyOnWriteArrayList<>(lions);
Set<Integer> bears = new ConcurrentSkipListSet<>();
bears.addAll(lions);
for (Integer item : tigers) tigers.add(4);  // x1
for (Integer item : bears) bears.add(5);    // x2
System.out.println(lions.size() + " " + tigers.size()
    + " " + bears.size());
```

This code outputs: `3, 6, 4`

The code compiles without any issue. The `CopyOnWriteArrayList` class is designed to preserve the original list on iteration, so the first loop will be executed exactly three times and, in the process, will increase the size of `tigers` to six elements.

The `ConcurrentSkipListSet` class allows modifications, and since it enforces uniqueness of its elements, the value `5` is added only once leading to a total of four elements in `bears`.

Finally, despite using the elements of `lions` to populate the collections, `tigers` and `bears` are not backed by the original list, so the size of `lions` is 3 throughout this program.

> 11.- 