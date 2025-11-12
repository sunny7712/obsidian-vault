
## Thread Creation

- Two ways of creating

![[Pasted image 20250427211216.png]]
- You can only extend from one class whereas you can implement multiple classes. So you can choose any method depending on the situation.

![[Screenshot 2025-04-27 at 9.15.03 PM.png]]

### 1. Implementing Runnable interface

![[Pasted image 20250427212325.png]]

1. Create a class that implements **Runnable** interface and override the *run* method defining the task which the thread has to do.
2. Start the thread.
	1. You can do this by passing the Runnable object to the **Thread Constructor**.
	2. Then start the thread by calling `start()` method of the object returned by the thread constuctor.
	 ![[Pasted image 20250427212759.png]]

### 2. Extending Thread class

![[Screenshot 2025-04-27 at 9.31.51 PM.png]]

1. Create a class and extend **Thread** class.
	1. Override the *run* method to define the task that the thread has to perform.
2. Create an instance of the subclass and call the `start()` method. 

## Thread Lifecycle

![[Pasted image 20250427213731.png]]

![[Pasted image 20250427214649.png]]

- Whenever thread is blocked or waiting, it releases all MONITOR LOCKS. But when in Timed waiting, it does not.
- When the thread is running, it puts the monitor lock.

### Monitor Lock

- It helps to make sure that only 1 thread goes inside the particular section of code (a synchronized block or method). Monitor lock is put on an object.
- When a thread gets a lock it starts executing the method. After executing, it will release the lock. Other threads will wait to acquire the lock.
- When you put `synchronized` to a method it allows only one thread to access it at a time. But the catch is that, this only works **when all the threads has the same instance of the object which has this method**.