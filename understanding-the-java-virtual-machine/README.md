# Understanding the Java Virtual Machine

- [Chapter 1. Being familiar with Java](#chapter-1-being-familiar-with-java)
    - [History of JDK](#history-of-jdk)
    - [History of JVM](#history-of-jvm)
    - [Future of Java](#future-of-java)
- [Chapter 2. Automatic Memory Management](#chapter-2-automatic-memory-management)
  - [Runtime Data Areas](#runtime-data-areas)
  - [Runtime Data Areas Explanation](#runtime-data-areas-explanation)
  - [How HotSpot VM stores Objects in memory](#how-hotspot-vm-stores-objects-in-memory)
  - [Real world examples](#real-world-examples)

## Chapter 1. Being familiar with Java

### What is included in java system?

- Java programming language
- JVM
- Standard libs
- Third party libs
- Class file format

What is JDK

- java programming language + JVM + standard libs

### History of JDK

- JDK 1.1(1997-02-19)
    - JAR file format, JDBC, Java beans, RMI
- JDK 1.2(1998-12-04)
    - Divided java system into 3 parts
        - J2SE: for desktop applications
        - J2EE: for enterprise applications
        - J2ME: for mobile devices
    - EJB, Java plugin, Java IDL, Swing
    - Added JIT compiler
    - VMs : Classic VM, HotSpot VM, Exact VM
    - Collection API added
- JDK 1.3(2000-05-08)
    - HotSpot VM became the default
- JDK 1.4(2002-02-13)
    - RegEx, exception chaining, NIO, etc
    - .NET framework appeared on 2002
- JDK 5(2004-09-30)
    - Let's stop calling JDK 1.X
    - Auto boxing, generics, dynamic annotation, enum, varargs ...
- JDK 6(2006-12-11)
    - Rename J2EE, J2SE, J2ME to EE 6, SE 6, ME 6
    - Implement or enhance JVM's lock, concurrency control, garbage collection, class loading etc
    - OpenJDK established => plan to convert java into open source
- JDK 7(2009-02-19)
    - Jigsaw project, support for dynamic languages in JVM, G1 collector etc
    - The primary development shifted from Sun to Oracle
- JDK 8(2014-03-18)
    - Lambda support, remove permgen from hotspot vm
- JDK 9(2017-09-21)
    - module system(Jigsaw project)
    - "Every JDK version should be supported at least 3 years" => REMOVED
        - Every 6th(8, 11, 17...) version will be the LTS version
- JDK 10(2018-03-20)
    - Internal refactoring
        - added little features(type reference etc)
    - Move ownership of Java EE to Eclipse foundation => Jakarta EE
- JDK 11(2018-09-25)
    - test version of ZGC added
    - Open source commercial features => OpenJDK 11 == Oracle JDK 11
        - OpenJDK: Free for development, test and production. But Oracle's support is limited to 6 month
        - Oracle JDK: Free for personal use, but not for production use. Support for 3 years.
- JDK 12(2019-03-19)
    - Responsibility for supporting previous versions is shifted RedHat
- JDK 13(2019-09-17)
    - Re-implement socket API, text blocks preview
- JDK 14(2020-03-17)
    - switch
    - CMS removed
- JDK 15(2020-09-16)
    - ZGC, text blocks
- JDK 16(2021-03-16)
    - enhance metaspace management
    - pattern matching using `instanceof`
    - record class
- JDK 17(2021-09-14)
    - sealed class
    - pseudo random generator
- JDK 18(2022-03-22)
    - `finalize()` became the deprecated target
- JDK 19(2022-09-20)
    - preview for foreign function, memory API, virtual thread, structured concurrency
- JDK 20(2023-03-21)
- JDK 21(2023-09-19)
    - generational ZGC, virtual thread

### History of JVM

- Classic VM
    - the first VM
    - included in JDK 1.0
    - Only interpretation of java code, no JIT => JIT can be supported by using plugin
    - But the supported JIT compilers compiled the whole code => too slow
- Exact VM
    - combination of interpreting and compilation
    - didn't live long enough due to HotSpot VM
- HotSpot VM
    - Developed by LongView technologies
    - Hot code detection
        - notifies the JIT compiler with the code area in which the compilation efficiency could be high
        - then the JIT compiler would compile the code down into native code

### Future of Java

- GraalVM
    - "Run programs faster anywhere"
    - GraalVM is a full-stack cross-language VM built on top of HotSpot VM
        - Java, Kotlin, Scala, Groovy, Python, Javascripp ...
        - Languages that make use of LLVM: C, C++, Rust ...
    - Different languages can access the same memory space, so it's possible to use language specific libraries
    - HotSpot internals
        - interpreter
        - C1 compiler
            - fast compilation, but little optimization
        - C2 compiler
            - slow compilation, but more optimization
            - Graal compiler will replace C2 compiler
- AOT compilation
    - Compile java code into native code before running the application
        - But "write once run everywhere" is now unavailable
- Project Valhalla
    - Allow primitive classes on generics e.g. `List<int>` will become possible without boxing overhead
    - Introduces value types, which are identity-less, inlineable and allow Java to handle data more like
      primitive types but with the structure and flexibility of objects
- Project Amber
    - Local variable type inference(`var` added in JDK 10)
    - Local variable syntax for lambda parameters(added in JDK 11)
    - Switch expression(added in JDK 14)
    - Text blocks(added in JDK 15)
    - Pattern matching for instanceof(added in JDK 16)
    - Records(added in JDK 16)
    - Sealed classes(added in JDK 17)
    - Record patterns(added in JDK 21)
    - Pattern matching for switch(added in JDK 21)
    - String templates(preview in JDK 21)
    - Unnamed patterns and variables(preview in JDK 21)
    - Unnamed classes and instance main methods(preview in JDK 21)
    - Statements before `super()`(preview in JDK 21)
- Project Panama
    - Remove the barrier between JVM and native code. Currently Java uses JNI to call native code
    - Features
        - Foreign function and memory API
            - Use method handle to connect native code with java code(no more JNI glue code)
        - Vector API
            - Hardware accelerated vector computation
        - Jextract
            - Binds native library header with java

## Chapter 2. Automatic Memory Management

### Runtime Data Areas

- Shared by all threads
    - Heap
    - Method area
        - Runtime constant pool
- Private per thread
    - Native or VM Stack
    - PC registers

### Runtime Data Areas Explanation

- PC registers
    - stored in thread private memory
    - points to the bytecode line number of specific thread
    - we need PC registers in order to resume the execution of that specific thread
    - when running native methods, the PC register's value is Undefined
- VM stack
    - whenever java method is called, JVM creates a stack frame and push it to the VM stack(pops the stack frame
      when method finishes)
    - stack frame stores
        - local variable table
            - stores primitive data types, object reference, return address into local variable slot
            - local variable slot is "normally" 32 bit
            - number of local variable slot if determined at compile time
        - operand stack
        - dynamic link
        - method return value
    - 2 types of errors related to VM stack
        - StackOverflowError: when the depth of the stack increase above the JVM's specified limit
        - OutOfMemoryError: when there is not much memory for the increasing stack
- Native stack
    - whenever native method is called
- Heap
    - "Almost" every objects are stored in the heap
        - maybe the day might come when java objects are not stored in the heap
    - Not every GC divides heap into young, old, permanent and etc
        - JVM spec doesn't define how to divide the heap. It depends on the JVM implementation
    - TLAB(Thread Local Allocated Buffer)
        - In order to improve the throughput of object allocation, heap is divided into multiple thread local
          allocated buffers
- Method area
    - used to store type information, constants, static variables and code cache(compiled by JIT)
    - some people call this non-heap(but JVM spec regards method area as part of the heap)
    - method area and permanent generation space
        - HotSpot VM: included the method area into permanent generation space(until JDK 7)
            - from JDK 8, HotSpot VM removed the concept of permanent generation space. Instead, it implemented
              metaspace in native memory
        - BEA JRockit, IBM J9: no concept of permanent generation space
    - runtime constant pool
        - included in the method area
        - stores class version, field, method, interface + literal, symbols
            - when VM loads classes, it stores above information into the runtime constant pool
        - constants can be added to runtime constant pool during "runtime" e.g. String's `intern()` method
- Direct memory
    - `DirectByteBuffer`
        - used in NIO
        - allocate memory on non-heap
        - no need to copy data back and forth between java heap and native heap
    - you should consider `direct memory + heap memory < physical memory`

### How HotSpot VM stores Objects in memory

<b>Object creation</b>

- What happen when we use `new` keyword to create a new object?
- Java compiler will convert `new` keyword into 2 byte codes which is `new` and `invokespecial`
- Bytecode `new`
    1. Checks whether the parameter passed to the new keyword(the name of the class) is a symbolic reference to
       a class inside the constant pool
    2. Checks whether that class is loaded, resolved and initialized
    3. Allocate memory for the object
        - JVM make use of free list, which manages the free memory blocks
        - JVM make use of TLAB to reduce the contention of allocating memory between threads. When the buffer is
          full, it requests to increases the size of the buffer
        - When memory allocation is done, all the allocated space is initialized with 0.
            - thanks to this property, we can use object's field without initializing, cause the fields are
              initialized with 0
    4. Fill object header
        - Class information of the object
        - How to find meta information of the class
        - hashcode(calculated when `Object::hashCode()` is first called)
        - GC age
    5. Object cre
- Bytecode `invokespecial`
    1. `<init>()` method is called -> the constructor method is executed

<b>Object memory layout</b>

Hotspot divides an object into 3 parts

1. Object header
    - Mark word
        - stores runtime data of the object
        - hashcode, GC age, lock state flag etc
    - Class word
        - stores a pointer which points to the metadata of the class(which resides in the metaspace)
        - by using it, we can know the information of the class of an object during runtime
    - Length of array(if the object is an array)
        - JVM retrieves the size of an object from object header
        - Object header only stores the type information of an object(even though it's an array)
            - that's why we need length information in order to calculate the size of an array
2. Instance data
    - stores field information, parent class information etc
3. Alignment padding
    - HotSpot requires every object's memory to start from 8x bytes memory address.
    - If the object's size is not a multiple of 8, then alignment padding is used

<b>How to access objects in heap</b>

- Using a handle
    - reference pointer(stack) -> handle(heap) -> instance data, type data(heap)
    - object's frequently move their position in the heap, but if we use the handle, the address of handle is
      very stable
- Direct pointer
    - reference pointer(stack) -> instance data, type data(heap)
    - fast(no indirection)
    - HotSpot VM uses this method

### Real World Examples

- -Xms, -Xmx
    - controls the min/max size of the heap
- -Xss
    - max stack size
- -XX:PermSize, -XX:MaxPermSize
    - controls the size of the permgen
    - HotSpot removed the permgen from JDK 8. Replaced it with metaspace
- -XX:MaxMetaspaceSize
    - controls the max size of metaspace
    - defaults to -1, which means it's unlimited

<b>Memory Analysis</b>

- Shallow heap vs Retained Heap
    - shallow heap: the amount of memory taken up by just that object, without considering any objects it
      references
    - retained heap: the total amount of memory that would be freed if the object itself, along with all objects
      it exclusively references, were garbage collected
- Overflow in direct memory is not recorded to heap dump

## Chapter 3. Garbage Collector and Memory Allocation Strategies 

### What is dead object? 
- Reference counting? No, it's not used in Java 
  - What happen when object A and object B references each other?(circular reference) => can't remove both objects
  - Instead, Java uses "reachability analysis" algorithm 
- Reachability Analysis Algorithm 
  - Everything starts from GC root objects 
  - If objects can not be reached from GC root(reference chain doesn't exist), the objects can be considered dead 
- GC roots in Java 
  - Objects referenced by JVM stack e.g. arguments, local variables etc 
  - Statically referenced objects by classes in method read 
  - ... 
- Types of references in Java 
  - Strong reference: never garbage collected 
  - Soft reference: if memory overflow might occur, the gc collects soft referenced objects  
  - Weak reference: removed on next gc 
  - Phantom reference: weakest reference 
- Objects which has to run `finalize()` method will be added to the `F-Queue` 
  - JVM will start a thread to run `finalize()` method on the objects in `F-Queue` asynchronously 
  - `finalize()` will only be called once for each object 
  - it's recommended to not use the `finalize()` method 
- Garbage collection in method area 
  - it's not mandatory, and not very cost-effective
  - gc targets: unused constants and classes 

### Garbage Collection Algorithm 


