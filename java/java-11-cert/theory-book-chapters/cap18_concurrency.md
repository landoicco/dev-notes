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

> 11.- What statements about the following code are true?

```
Integer i1 = List.of(1, 2, 3, 4, 5).stream().findAny().get();
syncronized(i1) {   // y1
    Integer i2 = List.of(6, 7, 8, 9, 10)
        .parallelStream()
        .sorted()
        .findAny().get();   // y2
    System.out.println(i1 + " " + i2);
}
```

The following statement is correct: *The output cannot be determined ahead of time.*

The code compiles and run without any issue.

First, note that synchronizing on the first variable does not actually impact the results of the code since only one thread is being used.

Second, sorting on a parallel stream does not mean that `findAny()` will return the first record. The `findAny()` method will return the value from the first thread that retrieves a record. Therefore, the output is not guaranteed.

**Note:** Remember that even on serial streams, the `findAny()` method is free to select any element.

> 12.- Assuming `takeNap()` is a method that takes five seconds to execute without throwing an exception, what is the expected result of executing the following code snippet?

```
ExecutorService service = null;
try {
    service = Executors.newFixedThreadPool(4);
    service.execute(() -> takeNap());
    service.execute(() -> takeNap());
    service.execute(() -> takeNap());
} finally {
    if (service != null) service.shutdown();
}
service.awaitTermination(2, TimeUnit.SECONDS);
System.out.println("DONE!");
```

The code will pause for 2 seconds and then print `DONE!`.

The code snippet submits three tasks to an `ExecutorService`, shuts it down, and the waits for the results. The `awaitTermination()` method waits the specified amount of time for all tasks to complete, and the service to finish shutting down.

Since each five-seconds task is still executing, the `awaitTermination()` method will return a value of `false` after two seconds but not throw an exception.

**Note:** The `awaitTermination()` method waits the specified amount of time to complete all tasks, returning sooner if all tasks finish or an `InterruptedException` is detected.

> 13.- What statements about the following code are true?

```
System.out.print(List.of("duck", "flamingo", "pelican")
    .parallelStream().parallel()    // q1
    .reduce(0, 
        (c1, c2) -> c1.length() + c2.length(),  // q2
        (s1, s2) -> s1 + s2));  // q3
```

The code will not compile because of line `q2`.

The problem here is that `c1` is an `int` and `c2` is a `String`, so the code fails to combine on line `q2`, since calling `length()` on an `int` is not allowed.

The rest of the lines compile without issue. Note that calling `parallel()` on an already parallel stream is allowed, and it may in fact return the same object.

**Note:** The stream operation `reduce()` combines a stream into a single object. The following is the signature of the method.

```
<U> U reduce(U identity,
    BiFunction<U,? super T, U> accumulator,
    BinaryOperator<U> combiner)
```

> 14.- What statements about the following code snippet are true?

```
Object o1 = new Object();
Object o2 = new Object();
var service = Executors.newFixedThreadPool(2);
var f1 = service.submit(() -> {
    synchronized(o1) {
        synchronized(o2) { System.out.print("Tortoise); } 
    }
});
var f2 = service.submit(() -> {
    synchronized(o2) {
        synchronized(o1) { System.out.print("Hare"); }
    }
});
f1.get();
f2.get();
```

The following statements are correct:

- If the code does output anything, the order cannot be determined.
- The code compiles but may produce a deadlock at runtime

The code compiles without any issue. Since both tasks are submitted to the same thread executor pool, the order cannot be determined. The key here is that the order in which the resources `o1` and `o2` are synchronized could result in a deadlock. For example, if the first thread gets a lock on `o1` and the second thread gets a lock on `o2` before either thread can get their second lock, the code will hang on runtime.

Note that the code cannot produce a livelock, since both threads are waiting. Remember that on a livelock, the threads are active and trying to complete their task. In this case, both threads are waiting.

Finally, if a deadlock does occur, an exception will not been thrown since no exceptions are thrown when a deadlock occurs.

> 15.- Which statement about the following code snippet is correct?

```
2: var cats = Stream.of("leopard", "lynx", "ocelot", "puma")
3:    .parallel();
4: var bears = Stream.of("panda", "grizzly", "polar").parallel();
5: var data = Stream.of(cats, bears).flatMap(s -> s)
6:    .collect(Collectors.groupingByConcurrent(
7:      s -> !s.startsWith("p")));
8: System.out.println(data.get(false).size()
9:      + " " + data.get(true).size());
```

The code compiles and run without any issue and it outputs `3 4`.

The `collect()` operation groups the animals into those that do and do not start with the letter `p`. Note that there are four animals that do not start with the letter `p` and three animals that do. The logical complement operator (!) before the `startsWith()` method means that the result are reversed, so the output is `3 4`.

**Notes:** 
- Remember that a `flatMap()` returns a stream consisting of the results of replacing each element of this stream with the contents of a mapped stream produced by applying the provided mapping function to each element. That said, a `flatMap()` produces an arbitrary number (zero or more) of values for each input value.
- The method `Colectors.groupingByConcurrent()` returns a concurrent `Collector` implementing a "group by" operation on input elements, grouping elements according to a classification function that is provided to the method as a parameter. The classification function maps elements to some key type `K`. The collector produces a `ConcurrentMap<K, List<T>>` whose keys are the values resulting from applying the classification function to the input elements, and whose corresponding values are Lists containing the input elements which map to the associated key under the classification function.

> 16.- Which statements about methods in `ReentrantLock` are correct?

None of the above.

To keep in mind:

- The `lock()` method will wait indefinitely for a lock.
- The method name to attempt to acquire a lock is `tryLock()`.
- By default, a `ReentrantLock` fairness is set to `false` and must be enabled by using an overloaded constructor.
- A thread that holds the lock mat have called `lock()` or `tryLock()` multiple times. A thread needs to call `unlock()` once for each call to `lock()` and `tryLock()` to release a resource so that other threads can obtain the lock.

> 17.- What is the result of calling the following method?

```
3: public void addAndPrintItems(BlockingQueue<Integer> queue) {
4:    queue.offer(103);
5:    queue.offer(20, 1, TimeUnit.SECONDS);
6:    queue.offer(85, 7, TimeUnit.HOURS);
7:    System.out.print(queue.poll(200, TimeUnit.NANOSECONDS));
8:    System.out.print(" " + queue.poll(1, TimeUnit.MINUTES));
9: }
```

The code does not compile.

The methods on line 5, 6, 7 and 8 each throw `InterruptedException`, which is a checked exception; therefore the code does not compile.

If `InterruptedException` was declared in the method signature on line 3, then the answer will be that the output cannot be determined ahead of time, because adding items to the `queue` may be blocked at runtime. In this case, the queue is passed into the method, so there could be other threads operating on it.

**Notes:** 
- An `InterruptedException` is thrown when a thread is waiting, sleeping, or otherwise occupied, and the thread is interrupted, either before or during the activity.
- A `BlockingQueue` is an interface, just as a regular `Queue`, except that it includes methods that will wait a specific amount of time to complete an operation.

> 18.- Which of the following are valid `Callable` expressions?

The following are valid `Callable` expressions:
- `() -> 5`
- `() -> "The" + "Zoo`
- `() -> {System.out.println("Giraffe"); return 10;}`

Remember that a `Callable` lambda expression takes no values and returns a generic type.

> 19.- What is the result of executing the following application?

```
import java.util.concurrent.*;
import java.util.stream.*;
public class PrintConstants {
    public static void main(String[] args) {
        var s = Executors.newScheduledThreadPool(10);
        DoubleStream.of(3.14159, 2.71828)   // b1
            .forEach(c -> s.submit(     // b2
                () -> System.out.println(10*c)));   // b3
        s.execute(() -> System.out.println("Printed"));     // b4
    }
}
```

The code compiles, but the result cannot be determined ahead of time. Compiles but waits forever at runtime.

The application compiles without throwing an exception. Even though the stream is processed in sequential order, the task are submitted to a thread executor, which may complete the task in any order. Therefore the output cannot be determined ahead of time.

The thread executor is never shut down; therefore, the code will run but it will never terminate.

**Note:** Calling `forEach()` on an infinite stream does not terminate. Since there is no return value, there is no reduction.

Notice that this is the only terminal operation with a return type of `void`. The method signature is as follows:
```
void forEach(Consumer<? super T> action)
```

Remember that you use a `Consumer` when you want to do something with a parameter but not return anything. The interface is defined as follows:
```
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    // Ommited default method
}
```

> 20.- What is the result of executing the following program?

```
import java.util.*;
import java.util.concurrent.*;
import java.util.stream.*;
public class PrintCounter {
    static int count = 0;
    public static void main(String[] args) throws
            InterruptedException, ExecutionException {
        ExecutorService service = null;
        try {
            service = Executors.newSingleThreadExecutor();
            var r = new ArrayList<Future<?>>();
            IntStream.iterate(0, i -> i+1).limit(5).forEach(
                i -> r.add(service.execute(() -> {count++;}))   // n1
            );
            for (Future<?> result : r) {
                System.out.print(result.get()+" ");     // n2
            }
        } finally {
            if(service != null) service.shutdown();
        }}}
```

The code will not compile because of line `n1`.

The key to solving this question is to remember that the `execute()` method return `void`, no a `Future` object or `null`. Therefore, line `n1` does not compile.

If the `submit()` method had been used instead of `execute()`, then the code would print `null null null null null` as the output of the `submit(Runnable)` task is a `Future<?>` object that can only return `null` on its `get()` method.

**Note:** Remember that the `submit(Runnable)` method has the following definition:
```
Future<?> submit(Runnable task)
``` 
And submits a `Runnable` task for execution and returns a `Future` representing that task. The Future's get method will return `null` upon successful completion.