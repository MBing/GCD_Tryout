#  GCD Implementation (Ray Wenderlich tutorial)

See: 
[Part 1](https://www.raywenderlich.com/5370-grand-central-dispatch-tutorial-for-swift-4-part-1-2) [Part 2](https://www.raywenderlich.com/5371-grand-central-dispatch-tutorial-for-swift-4-part-2-2)

## Parallelism
Multi-core devices, on the other hand, execute multiple threads at the same time via parallelism.
It’s important to note that parallelism requires concurrency, but concurrency does not guarantee parallelism.
Basically, concurrency is about structure while parallelism is about execution.


## Queues
GCD operates on dispatch queues through a class aptly named `DispatchQueue`. FIFO order
Dispatch queues are thread-safe which means that you can access them from multiple threads simultaneously.
The key to this is to choose the right kind of dispatch queue and the right dispatching function to submit your work to the queue.
Queues can be either serial or concurrent. Serial queues guarantee that only one task runs at any given time. 

GCD provides 3 main types of queues:

- **Main queue**: runs on the main thread and is a serial queue.
- **Global queues**: concurrent queues that are shared by the whole system. There are four such queues with different priorities : high, default, low, and background. The background priority queue has the lowest priority and is throttled in any I/O activity to minimize negative system impact.
- **Custom queues**: queues that you create which can be serial or concurrent. Requests in these queues actually end up in one of the global queues.


Usage of these queues:

- **Main Queue**: This is a common choice to update the UI after completing work in a task on a concurrent queue. To do this, you code one closure inside another. Targeting the main queue and calling `async` guarantees that this new task will execute sometime after the current method finishes.
- **Global Queue**: This is a common choice to perform non-UI work in the background.
- **Custom Serial Queue**: A good choice when you want to perform background work serially and track it. This eliminates resource contention and race conditions since you know only one task at a time is executing. Note that if you need the data from a method, you must declare another closure to retrieve it or consider using `sync`.

When sending tasks to the global concurrent queues, you don’t specify the priority directly. Instead, you specify a *Quality of Service* (QoS) class property. This indicates the task’s importance and guides GCD in determining the priority to give to the task.

The QoS classes are:

- **User-interactive**: This represents tasks that must complete immediately in order to provide a nice user experience. Use it for UI updates, event handling and small workloads that require low latency. The total amount of work done in this class during the execution of your app should be small. This should run on the main thread.
- **User-initiated**: The user initiates these asynchronous tasks from the UI. Use them when the user is waiting for immediate results and for tasks required to continue user interaction. They execute in the high priority global queue.
- **Utility**: This represents long-running tasks, typically with a user-visible progress indicator. Use it for computations, I/O, networking, continuous data feeds and similar tasks. This class is designed to be energy efficient. This will get mapped into the low priority global queue.
- **Background**: This represents tasks that the user is not directly aware of. Use it for prefetching, maintenance, and other tasks that don’t require user interaction and aren’t time-sensitive. This will get mapped into the background priority global queue.

## Synchronous vs Asynchronous
A *synchronous* function returns control to the caller after the task completes.
Schedule a unit by calling: `DispatchQueue.sync(execute:)`.

An *asynchronous* function returns immediately, ordering the task to start but not waiting for it to complete. 
Schedule a unit by calling `DispatchQueue.async(execute:)`.

## Managing Tasks
Each task you submit to a `DispatchQueue` is a `DispatchWorkItem`. You can configure the behavior of a `DispatchWorkItem` such as its QoS class or whether to spawn a new detached thread.

## Delaying Tasks
DispatchQueue allows you to delay task execution. Don’t use this to solve race conditions or other timing bugs through hacks like introducing delays. Instead, use this when you want a task to run at a specific time.

Why not use `Timer`? You could consider using it if you have repeated tasks which are easier to schedule with `Timer`. Here are two reasons to stick with dispatch queue’s `asyncAfter()`.

One is readability. To use `Timer` you have to define a method, then create the timer with a selector or invocation to the defined method. With `DispatchQueue` and `asyncAfter()`, you simply add a closure.

Timer is scheduled on run loops so you would also have to make sure you scheduled it on the correct run loop (and in some cases for the correct run loop modes). In this regard, working with dispatch queues is easier.

## Managing Singletons
There are two thread safety cases to consider: during initialization of the singleton instance and during reads and writes to the instance.

Initialization turns out to be the easy case because of how Swift initializes static variables. It initializes static variables when they are first accessed, and it guarantees initialization is atomic. That is, Swift treats the code performing initialization as a critical section and guarantees it completes before any other thread gets access to the static variable.

A critical section is a piece of code that must not execute concurrently, that is, from two threads at once. This is usually because the code manipulates a shared resource such as a variable that can become corrupt if it’s accessed by concurrent processes.

## Handling Background Tasks

See tutorial for steps..

## Handling the Reader - Writers Problem
This is the classic software development [Readers-Writers Problem](http://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem). GCD provides an elegant solution of creating a [read/write lock](http://en.wikipedia.org/wiki/Read/write_lock_pattern) using dispatch barriers. Dispatch barriers are a group of functions acting as a serial-style bottleneck when working with concurrent queues.

When you submit a `DispatchWorkItem` to a dispatch queue you can set flags to indicate that it should be the only item executed on the specified queue for that particular time. This means that all items submitted to the queue prior to the dispatch barrier must complete before the `DispatchWorkItem` will execute.
Once finished, the queue returns to its default implementation.

*Use caution when using barriers in global background concurrent queues as these queues are shared resources. 
Using barriers in a custom serial queue is redundant as it already executes serially. 
Using barriers in custom concurrent queue is a great choice for handling thread safety in atomic or critical areas of code.*

You need to be careful though. Imagine if you call `sync` and target the current queue you’re already running on. This will result in a **deadlock** situation.

Here’s a quick overview of when and where to use `sync`:

- **Main Queue**: Be VERY careful for the same reasons as above; this situation also has potential for a deadlock condition. This is especially bad on the main queue because the whole app will become unresponsive.
- **Global Queue**: This is a good candidate to sync work through dispatch barriers or when waiting for a task to complete so you can perform further processing.
- **Custom Serial Queue**: Be VERY careful in this situation; if you’re running in a queue and call `sync` targeting the same queue, you’ll definitely create a deadlock.

## Dispatch Groups

With dispatch groups you can group together multiple tasks and either wait for them to complete, or receive a notification once they complete. Tasks can be asynchronous or synchronous and can even run on different queues.

`DispatchGroup` manages dispatch groups. The `wait` method blocks your current thread until all the group’s enqueued tasks finish.

Dispatching asynchronously to another queue then blocking work using wait is clumsy. Fortunately, there is a better way. `DispatchGroup` can instead notify you when all the group’s tasks are complete.

## Canceling Dispatch Blocks

*Be aware that you can only cancel a DispatchWorkItem before it reaches the head of a queue and starts executing.*

## Testing Asynchronous Code

Tests run on the main thread, so you can assume that every test happens in a serial manner. As soon as a given test method completes, **Xcode** considers the test to have finished and moves on to the next test. This means that any asynchronous code from the previous test will continue to run while the next test is running.

### Semaphores
If you want to learn more about semaphores, check out this [detailed discussion](https://greenteapress.com/wp/semaphores/) on semaphore theory. If you're the academic type, you may want to check out [Dining Philosophers Problem](http://en.wikipedia.org/wiki/Dining_philosophers_problem), which is a classic software development problem that uses semaphores.

### Note
When you implement asynchronous tests in your code, look at `XCTWaiter` first before going down to these low-level APIs. XCTWaiter's APIs are much nicer and provide a lot of powerful technology for asynchronous testing.

## Dispatch Sources
 You can use a dispatch source to monitor for some type of event. Events can include Unix signals, file descriptors, Mach ports, VFS Nodes, and other obscure stuff.

Debug example.. See last commit.
You could use this to debug an object and display data whenever you resume the app. You could also give your app custom security logic to protect itself (or the user's data) when malicious attackers attach a debugger to your application.

## Next steps
Beyond GCD, I recommend you check out [Operation and OperationQueue Tutorial in Swift](https://www.raywenderlich.com/?p=76341), a concurrency technology that is built on top of GCD. In general, it's best practice to use GCD if you are using simple fire-and-forget tasks. `Operation` offers better control, an implementation for handling maximum concurrent operations, and a more object-oriented paradigm at the cost of speed.

You should also take a look at our[ iOS Concurrency with GCD and Operations](https://www.raywenderlich.com/3648) video tutorial series, which covers a lot of the same topics that we've covered in this tutorial.


------------------------
[Apple Docs Dispatch](https://developer.apple.com/documentation/dispatch)
[Apple Docs Dispatch work item](https://developer.apple.com/documentation/dispatch/dispatchworkitem)
...
