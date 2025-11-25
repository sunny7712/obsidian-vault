## Chapter 9 of Inside Java Virtual Machine Book
#java #books #garbage_collection

- The Java Virtual Machine's heap stores all objects created by a running Java application.
- Garbage collection is the process of automatically freeing objects that are no longer referenced by the program.
- In addition to freeing unreferenced objects, a garbage collector may also combat **heap fragmentation**.
- Garbage detection is ordinarily accomplished by defining a set of roots and determining *reachability* from the roots.
- While different JVM implementations may have slightly different definitions of what constitutes the root set, there are certain elements that must always be included:
	1. **Object references in local variables**: These are the variables defined within methods that are currently on the call stack[^1]. When a method is executing, any objects referenced by its local variables must be considered live.
	2. **Object references in the operand stack**: The JVM is a stack-based machine, and when executing bytecode instructions, it uses an operand stack to hold intermediate values. Any object references temporarily placed on this stack during method execution must be considered part of the root set.
	3. **Object references in stack frames**: A stack frame is created whenever a method is called, containing the method's local variables, operand stack, and other execution state. All object references in any active stack frame must be included.
	4. **Object references in class variables**: These are static variables belonging to loaded classes. Since these variables exist independently of any specific object instance, they must always be considered part of the root set.

	The key point is that while JVM implementations might add additional sources to their root set definition, these four categories represent the minimum required elements that all implementations must include to properly identify which objects are still in use and which can be safely garbage collected.

- Another source of roots are any object references, such as strings, in the constant pool of loaded classes. The constant pool of a loaded class may refer to strings stored on the heap, such as the class name, superclass name, superinterface names, field names, field signatures, method names, and method signatures.
- Any object referred to by a root is reachable and is therefore a live object. Additionally, any objects referred to by a live object are also reachable. The program is able to access any reachable objects, so these objects must remain on the heap. Any objects that are not reachable can be garbage collected because there is no way for the program to access them.
- In java, sometimes, a primitive[^2] type like an `int` can sometimes **contain a value that coincidentally matches a memory address** of an object in the heap. This can create confusion for certain types of garbage collectors.
	- A **conservative garbage collector** is a type of garbage collector that **does not strictly differentiate between real object references and accidental look-alikes**. Instead, it assumes that **any value that looks like an object reference might be a real reference**.
- Two basic approaches for distinguishing live objects from garbage are *reference counting* and *tracing*.
	1. **Reference counting**: Reference counting garbage collectors distinguish live objects from garbage objects by keeping a count for each object on the heap. The count keeps track of the number of references to that object.
	2. **Tracing**:  Tracing garbage collectors actually trace out the graph of references starting with the root nodes. Objects that are encountered during the trace are marked in some way. After the trace is complete, unmarked objects are known to be unreachable and can be garbage collected.

### Reference Counting Collectors
- Early gc strategy.
- In this approach, a reference count is maintained for each object on the heap.
- When an object is first created and a reference to it is assigned to a variable, the object's reference count is set to one. When any other variable is assigned a reference to that object, the object's count is incremented.
- When a reference to an object goes out of scope or is assigned a new value, the object's count is decremented.
- Any object with a reference count of zero can be garbage collected. When an object is garbage collected, any objects that it refers to have their reference counts decremented.
	- Pros:
		- Any object with a reference count of zero can be garbage collected. When an object is garbage collected, any objects that it refers to have their reference counts decremented.
	- Cons:
		- A disadvantage is that reference counting does not detect cycles: two or more objects that refer to one another. These objects will never have a reference count of zero even though they may be unreachable by the roots of the executing program.
		- Another disadvantage of reference counting is the overhead of incrementing and decrementing the reference count each time.
### Tracing Collectors
- Tracing garbage collectors trace out the graph of object references starting with the root nodes.
- Objects that are encountered during the trace are marked in some way. Marking is generally done by either setting flags in the objects themselves or by setting flags in a separate bitmap.
- After the trace is complete, unmarked objects are known to be unreachable and can be garbage collected. This is called "Mark and sweep."
- In the Java Virtual Machine, the sweep phase must include finalization of objects.
### Compacting Collectors
- Garbage collectors of Java Virtual Machines will likely have a strategy to combat heap fragmentation.
- Two strategies commonly used by mark and sweep collectors:
	- **Compacting**: 
		- Slide live objects over free memory space toward one end of the heap. In the process the other end of the heap becomes one large contiguous free area. All references to the moved objects are updated to refer to the new location.
		- Instead of referring directly to objects on the heap, object references refer to a table of object handles. The object handles refer to the actual objects on the heap. When an object is moved, only the object handle must be updated with the new location. All references to the object in the executing program will still refer to the updated handle, which did not move.
	- **Copying**:
		- 
## Footnotes

[^1]: The call stack is essentially a stack data structure that keeps track of all active function or method calls in a program. When a function is called, the system creates a new frame on top of the call stack to store information related to that function call. Each frame on the call stack typically contains:
	1. Return address
	2. Local variables
	3. Parameters
	4. Other context information
	The call stack has a fixed size in most implementations, which is why deeply nested or recursive method calls can cause a "stack overflow" error if they consume too much stack space.

[^2]: In Java, objects are stored in the **heap memory**, and Java uses **object references** (pointers in a managed way) to access them. Primitive data types, like `int`, `float`, and `char`, are not stored on the heap but directly in the stack or other memory regions.