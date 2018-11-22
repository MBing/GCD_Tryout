#  GCD Implementation / Ray Wenderlich tutorial

See: https://www.raywenderlich.com/5370-grand-central-dispatch-tutorial-for-swift-4-part-1-2

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

## Handling Background Tasks

See tutorial for steps..
