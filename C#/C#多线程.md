youtube：https://www.youtube.com/watch?v=S4lSFmoT44o&list=PL5Agzt13Z4g9KHIyr0xRIrrfqlDPLMp-t

# C#多线程学习

2024/04/21

## Thread

> Thread相关方法
>
> 1. **Start()**: 启动线程，并开始执行线程的入口点方法。
> 2. **Join()**: 阻塞当前线程，直到调用该方法的线程完成执行。
> 3. **Abort()**: 终止线程的执行。这个方法可能会导致线程立即终止，因此谨慎使用。
> 4. **Suspend()**: 暂停线程的执行。已过时，不建议使用，因为可能导致线程死锁或其他问题。
> 5. **Resume()**: 恢复线程的执行。已过时，不建议使用，因为可能导致线程死锁或其他问题。
> 6. **Sleep(int millisecondsTimeout)**: 让当前线程休眠指定的时间（以毫秒为单位）。
> 7. **Interrupt()**: 中断线程的等待状态。如果线程处于等待状态（如`Sleep`、`Join`、`Wait`等），则会抛出`ThreadInterruptedException`异常。
> 8. **IsAlive**: 获取一个值，指示线程是否正在运行。
> 9. **CurrentThread**: 获取当前正在执行的线程的实例。
> 10. **GetHashCode()**: 获取线程的哈希码。
> 11. **Equals(object obj)**: 判断线程是否等于另一个线程。
> 12. **SetName(string name)**: 给线程设置一个名称。
> 13. **GetDomain()**: 获取当前线程所在的应用程序域。
> 14. SpinWait():自旋等待是一种在多线程编程中常用的技术，它用于等待某个条件变为真时，不断地检查条件，而不是立即阻塞线程。`SpinWait`类提供了一种简单的自旋等待机制，可以在一定程度上减少线程切换的开销。





## 多线程中保护共享资源

> 免受并发访问
>
> 三种方式

### 1、Interlocked.Increment()

> 在一些情况下，`Interlocked.Increment`可能比使用锁（`lock`）的方式具有更低的性能消耗。
>
> ```C#
>  static void Addition()
>  {
>      for(int i = 0;i<100000;i++)
>      {
>                 Interlocked.Increment(ref Program.Sum);
> 
>      }
>  }
> ```
>
> 

### 2、Lock方法

> ```C#
>  static void Addition()
>  {
>      for(int i = 0;i<100000;i++)
>      {
>          lock(myLock)
>          {
>              Program.Sum++; 
>          }
> 
>      }
>  }
> ```
>
> 

### 3、Monitor方法

> `monitor.enter` 和 `lock` 在 C# 中都用于实现线程同步，它们本质上是相同的，只是 `lock` 是 `monitor.enter` 和 `monitor.exit` 的一种简化形式。
>
> 具体来说，`lock` 关键字在编译时会被翻译成 `monitor.enter` 和 `monitor.exit` 的调用，同时在编译时会自动处理异常以确保在退出临界区时释放锁。这样做使得代码更加简洁和易读。
>
> 在C#中，`Monitor` 类提供了一些方法用于实现线程同步和互斥访问。这些方法包括：
>
> 1. **Enter**: `Enter` 方法用于进入指定对象的临界区。如果其他线程已经进入了同一个对象的临界区，那么当前线程将会被阻塞，直到临界区可用为止。`Enter` 方法有多个重载形式，其中一些允许你指定超时时间，以便在等待锁时避免无限期地阻塞。
>
> 2. **TryEnter**: `TryEnter` 方法类似于 `Enter` 方法，但是它会立即返回一个布尔值，指示是否成功进入了临界区。你可以选择使用这个方法来尝试进入临界区而不被阻塞。
>
> 3. **Exit**: `Exit` 方法用于退出指定对象的临界区。当线程完成临界区内的操作后，应该调用 `Exit` 方法来释放锁，以便其他线程可以进入临界区。
>
> 4. **Wait**: `Wait` 方法用于使当前线程在指定对象上等待，直到另一个线程调用 `Pulse` 或 `PulseAll` 方法来通知它。`Wait` 方法会释放对象的锁，并将当前线程放入等待队列中，直到收到通知或超时。
>
> 5. **Pulse**: `Pulse` 方法用于通知等待在指定对象上的一个线程，使其从 `Wait` 方法中返回。调用 `Pulse` 方法会唤醒等待队列中的一个线程，但不会释放对象的锁。
>
> 6. **PulseAll**: `PulseAll` 方法与 `Pulse` 方法类似，但它会唤醒等待队列中的所有线程。
>
> 这些方法可以帮助你实现线程之间的同步和互斥访问，确保多个线程之间的操作是安全的。
>
> wait和pulse是一对
>
> `Wait` 和 `Pulse` 是一对的。它们通常一起使用来实现线程间的同步和通信。
>
> - `Wait` 方法用于使当前线程等待在某个对象上，直到另一个线程调用该对象的 `Pulse` 或 `PulseAll` 方法。
> - `Pulse` 方法用于通知等待在某个对象上的一个线程，使其从 `Wait` 方法中返回。





### 面试题

> 交替打印字符
>
> ```C#
> using System;
> using System.Threading;
> 
> class Program
> {
>     static object lockObj = new object(); // 用于同步的对象
> 
>     static void Main(string[] args)
>     {
>         Thread threadA = new Thread(PrintA);
>         Thread threadB = new Thread(PrintB);
> 
>         threadA.Start();
>         threadB.Start();
> 
>         threadA.Join();
>         threadB.Join();
>         
>         Console.WriteLine("\nPrinting completed.");
>     }
> 
>     static void PrintA()
>     {
>         for (int i = 0; i < 50; i++)
>         {
>             lock (lockObj)
>             {
>                 Console.Write('A');
>                 Monitor.Pulse(lockObj); // 通知 PrintB 线程可以运行
>                 Monitor.Wait(lockObj); // 等待 PrintB 线程打印完成
>             }
>         }
>     }
> 
>     static void PrintB()
>     {
>         for (int i = 0; i < 50; i++)
>         {
>             lock (lockObj)
>             {
>                 Console.Write('B');
>                 Monitor.Pulse(lockObj); // 通知 PrintA 线程可以运行
>                 if (i < 49)
>                     Monitor.Wait(lockObj); // 等待 PrintA 线程打印完成
>             }
>         }
>     }
> }
> 
> ```
>
> 





## ManualResetEvent

> `ManualResetEvent` 是 .NET 中用于线程同步的一个类，它允许一个或多个线程等待，直到收到信号为止。具体来说，`ManualResetEvent` 可以在两种状态之间切换：有信号和无信号。
>
> - 当 `ManualResetEvent` 处于有信号状态时，调用 `WaitOne` 方法的线程会立即继续执行。
> - 当 `ManualResetEvent` 处于无信号状态时，调用 `WaitOne` 方法的线程会被阻塞，直到收到信号或超时。
>
> `ManualResetEvent` 提供了以下主要方法和属性：
>
> 1. **WaitOne**: 当 `ManualResetEvent` 处于无信号状态时，调用此方法的线程将被阻塞，直到 `ManualResetEvent` 处于有信号状态或超时。
> 2. **Set**: 将 `ManualResetEvent` 设置为有信号状态，唤醒所有处于阻塞状态的线程。一旦调用 `Set` 方法，`ManualResetEvent` 将保持有信号状态，直到调用 `Reset` 方法。
> 3. **Reset**: 将 `ManualResetEvent` 设置为无信号状态，导致调用 `WaitOne` 方法的线程进入阻塞状态，直到 `Set` 方法再次被调用。
> 4. **WaitHandle**: 获取一个 `WaitHandle` 对象，允许将 `ManualResetEvent` 用于等待任何 `WaitHandle` 上的信号。





## AutoResetEvent

> `AutoResetEvent` 类似于 `ManualResetEvent`，也是用于线程同步的一个类，但它的行为略有不同。
>
> 与 `ManualResetEvent` 不同的是，`AutoResetEvent` 在收到信号后会自动将自身重置为无信号状态，而不需要手动调用 `Reset` 方法。这意味着，每次调用 `WaitOne` 方法的线程只会收到一次信号。
>
> 具体来说，`AutoResetEvent` 提供了以下主要方法和属性：
>
> 1. **WaitOne**: 当 `AutoResetEvent` 处于无信号状态时，调用此方法的线程将被阻塞，直到 `AutoResetEvent` 处于有信号状态或超时。一旦 `AutoResetEvent` 处于有信号状态，线程会收到信号，并将 `AutoResetEvent` 自动重置为无信号状态。
> 2. **Set**: 将 `AutoResetEvent` 设置为有信号状态，唤醒一个处于阻塞状态的线程。与 `ManualResetEvent` 不同，`AutoResetEvent` 在收到信号后会自动将自身重置为无信号状态。
> 3. **WaitHandle**: 获取一个 `WaitHandle` 对象，允许将 `AutoResetEvent` 用于等待任何 `WaitHandle` 上的信号。
>
> `AutoResetEvent` 通常用于一次性通知机制，即一个线程等待另一个线程执行某种操作，并在操作完成后继续执行。与 `ManualResetEvent` 相比，`AutoResetEvent` 更适合只需要一次信号的情况。
>
> 在 `ManualResetEvent` 和 `AutoResetEvent` 中，初始化为 `true` 或 `false` 主要影响对象的初始状态，即对象创建后立即设置的状态。
>
> 1. **初始化为 `true`**：
>    - 对象被创建时，处于有信号状态。
>    - 调用 `WaitOne` 方法的线程不会被阻塞，因为对象初始状态为有信号，可以继续执行。
>    - 在调用 `Set` 方法后，对象保持有信号状态。
> 2. **初始化为 `false`**：
>    - 对象被创建时，处于无信号状态。
>    - 调用 `WaitOne` 方法的线程会被阻塞，因为对象初始状态为无信号，需要等待信号才能继续执行。
>    - 在调用 `Set` 方法后，对象会变为有信号状态，唤醒一个处于阻塞状态的线程，然后自动将自身重置为无信号状态，等待下一次信号。
>
> ==在有信号状态下调用 `WaitOne` 方法的线程可以继续执行，而在无信号状态下调用 `WaitOne` 方法的线程会被阻塞。==





## Mutex

> `Mutex` 是 .NET 中用于线程同步的一种同步原语，用于控制对共享资源的访问，确保在任何给定时刻只有一个线程可以访问资源。
>
> `Mutex` 具有以下特点：
>
> 1. **互斥性（Mutual Exclusion）**：`Mutex` 提供了互斥锁功能，确保同一时间只有一个线程可以持有它。这意味着只有一个线程可以访问由 `Mutex` 保护的共享资源，其他线程必须等待直到该线程释放了 `Mutex`。
>
> 2. **可命名性**：`Mutex` 可以具有一个名称，使得它可以跨进程共享。不同进程中的线程可以通过使用相同名称的 `Mutex` 来实现跨进程的同步。
>
> 3. **递归性**：`Mutex` 可以是递归的，这意味着同一个线程可以多次获取同一个 `Mutex` 而不会死锁。但是要注意，递归的 `Mutex` 必须在同一个线程中正确释放相同次数，否则可能会导致死锁。
>
> `Mutex` 提供了以下主要方法：
>
> - **WaitOne**: 当调用 `WaitOne` 方法时，如果 `Mutex` 当前没有被任何其他线程持有，则该方法立即返回，并将 `Mutex` 分配给调用线程。如果 `Mutex` 已经被另一个线程持有，则调用线程会阻塞，直到 `Mutex` 可用为止。
>
> - **ReleaseMutex**: 用于释放由当前线程持有的 `Mutex`。一旦调用 `ReleaseMutex` 方法，其他线程可以获取该 `Mutex`。
>
> 使用 `Mutex` 可以确保对共享资源的访问是线程安全的，但要注意正确地释放 `Mutex`，以免出现死锁或资源泄漏的情况。



### 和Lock区别

> `Mutex` 和 `lock` 都用于实现线程同步，但它们之间有一些重要的区别：
>
> 1. **范围**：
>    - `lock` 是 C# 中的关键字，只能用于单个应用程序域（AppDomain）内的线程同步，无法跨越进程边界。
>    - `Mutex` 是一个系统对象，可以用于在同一台计算机上不同进程之间进行线程同步。
> 2. **粒度**：
>    - `lock` 的粒度较细，只能锁定一个代码块或对象，而且只允许同一线程多次获取同一个锁。
>    - `Mutex` 的粒度较粗，它是系统级别的同步原语，可以用于控制对任意资源的访问，并且允许多个线程或进程共享一个 `Mutex` 对象。
> 3. **性能**：
>    - 在单线程环境中，`lock` 的性能可能更好，因为它是由编译器和运行时进行优化的。
>    - 在多线程环境中，`Mutex` 的性能可能较差，因为它需要通过系统调用来实现同步，而且由于它可以跨越进程边界，可能会引入额外的开销。
> 4. **用法**：
>    - `lock` 更简单易用，特别适用于同一个线程内的局部变量或对象的同步。
>    - `Mutex` 更灵活，适用于需要跨进程同步的场景，但需要显式地创建和释放 `Mutex`，并处理可能出现的异常情况。
>
> 综上所述，`lock` 适用于单个应用程序域内的线程同步，而 `Mutex` 适用于跨进程的线程同步，但在性能和粒度方面有所差异。选择哪个取决于你的具体需求和场景。





## Semaphore

> `Semaphore` 是用于线程同步的一种同步原语，允许多个线程同时访问共享资源，但是通过控制同时访问的线程数量来限制并发访问。
>
> `Semaphore` 通常用于限制资源的并发访问数量，比如有限数量的连接池或者线程池。
>
> `Semaphore` 具有以下主要特点：
>
> 1. **计数器**：`Semaphore` 内部维护一个计数器，该计数器表示可用的资源数量。当一个线程进入临界区时，计数器会减少，当线程离开临界区时，计数器会增加。
>
> 2. **信号量**：`Semaphore` 中的计数器被称为信号量，它表示可用的资源数量。当信号量大于零时，线程可以进入临界区；当信号量等于零时，线程需要等待直到有资源可用。
>
> 3. **等待队列**：`Semaphore` 维护一个等待队列，当信号量等于零时，等待队列中的线程会被阻塞，直到有资源可用。
>
> `Semaphore` 提供了以下主要方法：
>
> - **WaitOne**: 当调用 `WaitOne` 方法时，如果信号量大于零，则计数器减一并允许线程进入临界区；如果信号量等于零，则线程会被阻塞，直到有资源可用为止。
>
> - **Release**: 用于释放一个资源并增加信号量。调用 `Release` 方法后，等待队列中的一个线程会被唤醒，可以进入临界区。
>
> `Semaphore` 是一种灵活的同步原语，可以用于控制对共享资源的访问数量，适用于需要限制并发访问的场景。
>
> `Semaphore` 允许你指定初始的信号量值，该值表示可用的资源数量。当线程调用 `WaitOne` 方法时，如果信号量大于零，则表示有可用的资源，线程可以进入临界区；如果信号量等于零，则线程需要等待，直到有资源可用。
>
> 假设你创建了一个初始信号量值为 3 的 `Semaphore`，表示最多允许 3 个线程同时进入临界区。如果有 5 个线程同时调用 `WaitOne` 方法，前 3 个线程将获得资源并进入临界区，剩余的 2 个线程将被阻塞，直到有资源可用。当其中一个线程调用 `Release` 方法释放资源后，被阻塞的线程中的一个将被唤醒，可以进入临界区，依此类推。
>
> 因此，`Semaphore` 可以有效地控制并发访问的数量，允许在一定程度上限制对共享资源的访问。
