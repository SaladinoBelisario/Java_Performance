# **Java Performance Notes**
Java versions covered: 8, 11 and eventually 17.

## **Index**

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

![JIT compiler for the JVM](E:\GitHub\Java_Performance\img\Compilers_JVM.svg "JIT_Compilers")

> We can see when the C2 compiler is used in the code as the method is called if we use the
> flags:
>
> _-XX:+UnlockDiagnosticVMOptions_ 
>
> _-XX:+LongCompilation_


### Code cache

The compiled code from the JIT compiler is going to be allocated in the code cache, as a cache
the reason for using it is to improve the performance of the I/O operations 
(is faster than main memory), but has a little problem, the amount of cache memory is very 
limited, so we can only optimize a little fraction of our code.

If you have too much code for optimization you will see a message like this:

`VM warning: CodeCache is full. Compiler has been disabled. `

meaning that even though there are code to be optimized, the compiler won't optimize such code
because cannot use the code cache (is already full).

> We can see the code cache size and other basic information if we use the flag:
>
> _-XX:+PrintCodeCache_

> We specify the initial, maximum and growing rate for the code cache with the flags:
>
> _-XX:InitialCodeCacheSize=[size]_ 
> 
> _-XX:ReservedCodeCacheSize=[size]_ 
> 
> _-XX:CodeCacheExpansionSize=[size]_
> 
> the size can be provided in Kilobytes (k) or Megabytes(m or M)

**We can use JConsole to monitor the code cache in a remote way, just connect the process
you want to monitor and select memory menu.**