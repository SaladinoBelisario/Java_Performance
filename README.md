# **`Java Performance Notes`**
Java versions covered: 8, 11 and eventually 17.

# **Table of contents**
- [**JVM and the Bytecode**](#jvm-and-the-bytecode)
    - [Bytecode](#bytecode)
    - [JIT Compiler vs Interpreter](#jit-compiler-vs-interpreter)
        - [Interpreter](#interpreter)
        - [JIT Compiler](#jit-compiler)
    - [The C1 and C2 Compilers](#the-c1-and-c2-compilers)
    - [Code cache](#code-cache)
- [**Selecting the JVM**](#selecting-the-jvm)
  - [32 vs 64 bit JVM](#32-vs-64-bit-jvm)
  - [Tiered Compilation](#tiered-compilation)
  - [Tuning the native compiler](#tuning-the-native-compiler)

## **JVM and the Bytecode**

### Bytecode

The Java bytecode is the instruction set that Java Virtual Machine can interpret, as assembler in
bare metal machines can be executed, the JVM executes this bytecode, to do this JVM use two 
different mechanisms:

    - The Interpreter
    - The Just-In-Time Compiler

### JIT Compiler vs Interpreter

When we compile our Java program (e.g., using the javac command), we'll end up with our source code compiled into the binary representation 
of our code (JVM bytecode).

To be able to run a Java program, **the JVM interprets the bytecode**. Since interpreters are
usually a lot slower than native code executing on a real processor, the **JVM can run another
compiler which will now compile our bytecode into the machine code that can be run by the
processor**, this so-called just-in-time compiler.

The _de-facto_ standard for executing Java Bytecode has been Interpreting, but in recent years
the JIT Compilation has been gaining popularity because improves the performance of the applications
in most cases.

![Bytecode generation and interpretation](img/JVM_Bytecode.svg "JVM_Bytecode")

#### Interpreter
An Interpreter normally reads the code line by line and translates the code to machine code 
as it reads the lines.

#### JIT Compiler
The Just-In-Time compiler is similar to the Interpreter in most cases, but has a significant 
difference: the JIT Compiler would compile the **most used code** to **native machine code**, for
the rest is similar because it reads line by line the code. To know what code to compile it saves 
a count variable for every method that is actualized with every call, so the compiler is triggered 
in some point to compile the method to native machine code to avoid re-interpreting that code again.

The process of JIT  compiling is performed in a **separated thread** in the JVM (the JVM it's by 
itself a multi-thread application) so the normal execution it's not interrupted by the JIT compiler.

**EXAMPLE**

The code for this example is in [Performance_Example_1](examples/Java11/PerformanceExample1) :

> In order to see which methods are being compiled by the JIT compiler we use the
> flag:
> 
> _-XX:+PrintCompilation_ 

When you compile this code you can see something like this:

`156   60   %    4       java.lang.String::hashCode (49 bytes)   made not entrant`

this means that your code has been compiled to the most optimal compilation level (level 4)

### The C1 and C2 Compilers

The JDK implementation contains two conventional JIT-compilers: the client compiler, also 
called C1 and the server compiler, called opto or C2.

**C1 is designed to run faster and produce less optimized code**, **while C2, on the other hand,
takes a little more time to run but produces a better-optimized code**. The client compiler is
a better fit for desktop applications since we don't want to have long pauses for the 
JIT-compilation. The server compiler is better for long-running server applications that can
spend more time on the compilation.

The JVM would decide which compile level would be applied to your code from 1 to 4 level 
(4 is the most optimized level).

![JIT compiler for the JVM](img/Compilers_JVM.svg "JIT_Compilers")

We can see when the C2 compiler is used in the code as the method is called if we use the
flags:

    -XX:+UnlockDiagnosticVMOptions
    -XX:+LongCompilation


### Code cache

The compiled code from the JIT compiler is going to be allocated in the code cache, as a cache
the reason for using it is to improve the performance of the I/O operations 
(is faster than main memory), but has a little problem, the amount of cache memory is very 
limited, so we can only optimize a little fraction of our code.

If you have too much code for optimization you will see a message like this:

`VM warning: CodeCache is full. Compiler has been disabled. `

meaning that even though there are code to be optimized, the compiler won't optimize such code
because cannot use the code cache (is already full).

We can see the code cache size and other basic information if we use the flag:

    -XX:+PrintCodeCache

We specify the initial, maximum and growing rate for the code cache with the flags:

    -XX:InitialCodeCacheSize=[size]

    -XX:ReservedCodeCacheSize=[size]

    -XX:CodeCacheExpansionSize=[size]

the size can be provided in Kilobytes (k) or Megabytes(m or M)

**We can use JConsole to monitor the code cache in a remote way, just connect the process
you want to monitor and select memory menu.**

## **Selecting the JVM**

### 32 vs 64 bit JVM
Here are some key difference between 32-bit and 64-bit Java Virtual Machine

1. 64-bit JVM, you can specify more memory for heap size than 32-bit JVM, like in 
32-bit JVM, the theoretical limit for maximum memory in 32-bit is 4G, but 64-bit is 
much higher.

2. 64-bit JVM is particularly useful for Java applications with large heaps, like 
applications that use more than 100G for max memory.

3. The same Java application will take memory while running in 64-bit JVM then 32-bit 
because of the increased size of OOP (Ordinary Object pointer), from 32 to 64 bits. Though
you can get away with this by using -XXCompressedOOP JVM option, which tells JVM to use 
32-bit pointers.

4. Both 32-bit and 64-bit JVM have a separate installer.

5. One more thing that changed in the 64 bit JVM architecture is object header size; it is 
now 12 bytes in comparison to 8 bytes headers in 32 bit JVM. Another thing that changed is 
the size of internal references that means it can go a maximum of up to 8 bytes wherein 32 
bit JVM up-to 4 bytes.

So, from these two points, you can conclude that an application running on 64 bit JVM will
consume more space in comparison when the same application runs on the 32-bit version.

In order to force the JIT compiler to use only a type of compiler we can use the flags:

    -client for client compiler(C1)
    
    -server for server compiler(C2)

    -d64 for the 64 bit compiler version

### Tiered Compilation

As we mentioned in the previous section, our Java program, compiled by **javac**, starts its 
execution in an interpreted mode. The JVM tracks each frequently called method and compiles 
them. In order to do that, it uses C1 for the compilation. But, the HotSpot still keeps an
eye on the future calls of those methods. If the number of calls increases, the JVM will 
recompile these methods once more, but this time using C2.

This is the default strategy used by the HotSpot, called **tiered compilation**.

We can force the JVM to run in interpreter-mode only with the flag:

    -XX:-TieredCompilation (we turn off the flag)

### Tuning the native compiler
We can specify the number of threads to perform a compilation with the flag:

    -XX:-CICompilerSize=[size] (default 3, minimum 2)

We also can specify the  threshold number for the C2 compiler to be triggered:

    -XX:CompileThreshold=[threshold_value]

## **Memory allocations: Heap vs Stack**

Our starting point is to understand the terms of the stack and the heap when our applications
run they need access to some of our computers' memory, for example, to store the objects 
that we create and hold a memory.

### The stack

The stack is a very efficient data structure, which is managed effectively by the Java virtual 
machine.

One important aspect of the stack is that Java knows exactly when data on the stack can be 
destroyed (garbage collection).

In Java **all local variables are created on the stack**, and they are automatically popped from
the stack when you reach the close of the block that created that variable. All this happens 
within the Java Virtual Machine.

### The heap

The second area of Java's memory is called the heap. Although the stack is a very efficient
data structure, it can't be used to store complex data types such as an **object**.

## **Passing objects methods**

### Passing variables vs passing references

All object references in Java are passed by value. This means that a copy of the value will
be passed to a method. But the trick is that passing a copy of the value also changes the real
value of the object.

For objects passed to methods, the **Reference** to the object is passed by **Value**

**_final_ keyword**

Real meaning of the final keyword is not that the variable can never be changed, but 
that the variable can only be assigned once. Once the variable has been assigned, it can never 
be altered.

The _final_ keyword **doesn't stop** the object value from changing, only prevents the stack pointer
from changing to the actual object.

## **The Metaspace and internal JVM memory optimisation**

In general, the heap is absolutely huge compared to the stacks and the metaspace, which are
quite small. So the metaspace is used primarily to store metadata. That's going to be 
information about classes, methods, which methods, for example, have been compiled into 
bytecode and which should be compiled native code in general.

But there are other uses for the better space which are more interesting. 

And **the first** of these is it's where **static variables are stored**. We can think of the metaspace
as having the same role as a stack for any object or any variable that we declare as a static
variable so static primitives are stored entirely in the metaspace and **static objects
are stored on the heap**, but with the object pointer or **reference held in the metaspace**.

So if, for example, we declared a _static int_, that's a primitive core global variable that
would exist solely in the meta space if we created a static object in this case, it would be
a _hashmap_. That map would be created on the heap, but with the variable reference in this
case called settings in the metaspace.

Unlike a stack where variables can be popped off when they go out of scope, variables in the 
matter space are permanently there, as we would expect for static variables. They never reach
a state where they can no longer be referenced. **So any objects on the heap which are referenced
from the metaspace will never be garbage collected**.

**All classes and all threads within a Java program have access to the metaspace**, and that's 
why static variables can be accessed by any piece of code we write in our application, because 
the thread running that code can access the letter space, so it can access any variables that
are declared to be living in the metaspace.

### PermGen vs Metaspace

#### PermGen

PermGen (Permanent Generation) is a special heap space separated from the main memory heap.

The JVM keeps track of loaded class metadata in the PermGen. Additionally, the JVM stores all
the static content in this memory section. This includes all the static methods, primitive 
variables, and references to the static objects.

Furthermore, it contains data about bytecode, names, and JIT information. Before Java 7, the
_String Pool_ was also part of this memory. 

The default maximum memory size for 32-bit JVM is 64 MB and 82 MB for the 64-bit version.

However, we can change the default size with the JVM options:

    -XX:PermSize=[size] is the initial or minimum size of the PermGen space

    -XX:MaxPermSize=[size] is the maximum size

Most importantly, **Oracle completely removed this memory space in the JDK 8 release**.
Therefore, if we use these tuning flags in Java 8 and newer versions, we'll get the following
warnings:

`>> java -XX:PermSize=100m -XX:MaxPermSize=200m -version
OpenJDK 64-Bit Server VM warning: Ignoring option PermSize; support was removed in 8.0
OpenJDK 64-Bit Server VM warning: Ignoring option MaxPermSize; support was removed in 8.0
...`

With its limited memory size, PermGen is involved in generating the famous _OutOfMemoryError_. 
Simply put, the class loaders weren't garbage collected properly and, as a result, generated
a memory leak.

Therefore, we receive a memory space error; this happens mostly in the development environment
while creating new class loaders.

#### Metaspace

Simply put, Metaspace is a new memory space (starting from the Java 8 version); **it has replaced
the older PermGen memory space**. The most significant difference is how it handles memory 
allocation.

Specifically, **this native memory region grows automatically by default**.

We also have new flags to tune the memory:

* MetaspaceSize and MaxMetaspaceSize: we can set the Metaspace upper bounds.
* MinMetaspaceFreeRatio: is the minimum percentage of class metadata capacity free after
garbage collection
* MaxMetaspaceFreeRatio: is the maximum percentage of class metadata capacity free after a
garbage collection to avoid a reduction in the amount of space

Additionally, the garbage collection process also gains some benefits from this change. The
garbage collector now automatically triggers the cleaning of the dead classes once the class
metadata usage reaches its maximum metaspace size.

Therefore, **with this improvement, JVM reduces the chance to get the OutOfMemory error**.

**Despite all of these improvements, we still need to monitor and tune the metaspace to avoid 
memory leaks.**