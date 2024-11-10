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
    - What happen when object A and object B references each other?(circular reference) => can't remove both
      objects
    - Instead, Java uses "reachability analysis" algorithm
- Reachability Analysis Algorithm
    - Everything starts from GC root objects
    - If objects can not be reached from GC root(reference chain doesn't exist), the objects can be considered
      dead
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

### Garbage Collection Algorithms

- Generational based collection theory
    - 2 premises + 1 additional premise
        - Most of the objects die fast
        - Long survived objects tend to live longer
        - intergenerational reference hypothesis: number of intergenerational references are much less than
          intragenerational references
- Related terms
    - minor GC, major G, full GC
        - minor GC: only the young generation
        - major GC: only the old generation
        - mixed GC: young + old generation
        - full GC: java heap + method area
    - mark-sweep, mark-copy, mark-compact algorithms
    - young and old generations
- Algorithms
    - Mark and Sweep
        - How it works
            1. Marks the objects to be deleted(or should survive)
            2. Sweep the garbage objects
        - Cons
            - When there are too many objects in heap, it takes too much time. Execution time varies depending
              on the usage of the heap
            - Memory fragmentation after the sweep.
    - Mark and Copy
        - Introduced to overcome the cons(not efficient when the number of garbage increases) of Mark and Sweep
          algorithm
        - How it works
            - Divide the memory into 2 parts
            - Copy the survived objects into another part of the memory
                - can remove memory fragmentation
                - but... if there are too many survived objects, the cost of copying can increase
        - Most of the JVM use this algorithm
        - How memory is divided in real world JVMs?(Appel style collection)
            - Young generation is divided into eden + 2 survivor spaces
            - Only use the eden space + 1 survivor space when allocating object into heap
                - After the GC, survived objects are moved into another survivor space
            - HotSpot VM's default is eden(80%), s1(10%), s2(10%)
            - But what if survived object's size is bigger than single survivor space?
                - moved to another region in memory, most of which are the old generation
        - Mark and Compact algorithm
            - Appropriate for long lived objects
            - Similar to mark and sweep, but mark and compact reallocates all the survived object sequentially
              in memory
            - GC execution time will increase, but it can improve the allocation and object access efficiency

### HotSpot Implementation

- Root node enumeration
    - Every garbage collectors must stop during the root node enumeration(stop the world)
    - HotSpot uses `OopMap` to efficiently enumerate the GC root nodes
        - doesn't create `OopMap` for every instruction, rather for safe points only
            - garbage collectors doesn't stop the program until it reaches the safe point
    - How does garbage collectors stop the threads?
        - Preemptive suspension
            - Interrupt running threads until they reach the safe point
            - Not much JVMs use this approach
        - Voluntary suspension
            - Use a flag bit
            - When the flag bit is set, the thread stops at the nearest safe point
    - Safe region
        - Is using safe point enough? => NO
            - What if there are blocked or waiting threads. What happen when they wake up?
        - A safe region is a code block where the reference doesn't change, so we can run garbage collection
          without worries
        - Garbage collector doesn't have to wait for threads that are running in the safe region.
- Remembered set and Card table
    - Remembered set is a data structure used to keep track of references from objects in one memory region to
      objects in another
        - usually from old generation to young generation
    - The primary purpose of the remembered set is to track intergenerational references. By doing this, the
      garbage collector can avoid scanning the entire old generation when performing a minor garbage collection
      on the young generation
    - When an object in the old generation references an object in the young generation, that reference is
      recorded in the remembered set
    - During a minor GC, the garbage collector uses the remembered set to identify only those old generation
      objects with references to young generation objects
    - Card tables
        - remembered set is often implemented using a card table
            - a table that divides memory into small fixed size cards
        - when a reference crosses generational boundaries, the corresponding card is marked, allowing the
          collector to identify and scan only marked cards rather than the whole memory
- Write Barrier
    - A mechanism in GC used to intercept changes to object references
        - A small, efficient code snippet that executes whenever an object reference is updated(like an AOP)
    - Whenever an object reference is updated(especially when an old-generation object starts referencing a
      young generation object), the write barrier activates it. It marks the corresponding card in the card
      table, indicating a potential intergenerational reference.
- Tri-color marking
    - White
        - Not reachable objects
    - Gray
        - Objects that have been discovered as reachable but whose references have not been fully processed yet.
          Still needs to be scanned to mark other objects they reference
    - Black
        - Nodes that have been marked as reachable and whose references have been fully processed.
        - The object marked as black will not be revisited or reprocessed in this collection cycle

### Classic Garbage Collector

- Garbage collectors were classified by their purpose: for new generation or old generation
- GC for new generation
    - Serial collector
        - stop the world when running GC thread
        - uses mark and copy algorithm
    - Parnew collector
        - Everything is the same as serial collector except it uses multiple GC threads
    - PS collector(Parallel Scavenge collector)
        - focus on the throughput of the JVM
            - throughput = (time spent running user code) / (time spent running user code) + (time spent running
              GC)
- GC for old generation
    - Serial old
        - same as serial collector but for the old generation
        - uses mark and compact algorithm
    - Parallel old
        - used with the PS collector
        - uses multithreading to collect garbage in the old generation
    - CMS
        - runs mark phase and sweep phase concurrently with the user thread
        - focuses on reducing the stop the world
        - how it works
            1. Initial mark(STW): Identifies and marks all live objects that are "directly" reachable from the
               root references(so this is fast). "STW" occurs.
            2. Concurrent mark: Performs marking phase while the application threads continue to run. Scan the
               entire reference chain. This phase is prone to missing references updated by active application
               threads.
            3. Remark(STW): To account for any objects that may have been missed during the concurrent marking
               phase due to application activity, CMS performs another "STW" phase to finalize marking. This
               phase is to ensure that all live objects are correctly marked before reclamation.
            4. Concurrent sweep: Sweeps through the heap and reclaims memory occupied by unmarked objects.
        - Cons
            - Prone to processor resources. By sharing cpu resources while GC, application throughput is
              reduced.
            - 'Floating garbage' problem. Because CMS runs concurrently runs marking phase with application
              thread running, new garbage can be created after the marking phase. It has to wait until the next
              GC to remove those garbage.
            - Because it uses mark and sweep algorithm, memory fragmentation occurs
                - `-XX:+UseCMSCompactAtFullCollection`: remove memory fragmentation by compaction, but because
                  it has to move objects, CMS is no longer ale to run concurrently with the application thread
        - Removed from JDK 14
    - G1(Garbage First)
        - has been default since JDK 9
        - Pause prediction model
            - when target pause time is set to M, it limits GC time below M
        - Heap layout when using G1
            - Regions
                - Unlike older GC that split the heap into 2 main generation, G1 divides the heap into regions
                  of equal size, typically between 1MB ~ 32MB
                - Allows G1 to manage memory by collecting specific regions based on needs and GC goals rather
                  than collecting an entire generation at once.
            - Young and Old regions
                - Young region
                    - eden region: where new objects are created
                    - survivor region: after minor GC, survived objects move from eden to survivor region
                - Old region
                    - region that store objects that have survived multiple GC cycles
                    - collected incrementally to avoid long pauses
                - Humongous region
                    - large objects that exceed half the size of a regular region are stored in humongous
                      regions
                    - typically contiguous and managed separately from other regions
        - How it works
            1. Initial mark(short STW)
                - mark objects directly referenced by GC roots
                - modify TAMS pointer value
                  => create snapshot
            2. Concurrent marking
                - Scan the reference chain
            3. Remark(short STW)
                - Check for any changes and mark those objects
            4. Copy and clean(STW)
                - Select regions to collect garbage(by referencing the statistic)
                - Require STW cause live objects has to move to another region(collected region is cleaned)

### Low Latency Garbage Collectors

- G1 GC and CMS still require STW when moving the objects, initial marking phase, remarking phase
- Shenandoah and ZGC limits GC pauses below 10 ms

- Shenandoah
    - Share some code with G1, heap layout etc
        - divide heap into regions, support humongous region, collect cost effective regions first
    - What differs from G1?
        - Collecting garbage while running application thread
        - No concept of generation until JDK 21
        - Use "connection matrix" to solve interregional reference(instead of remembered set)
            - connection matrix is a simple 2D matrix which marks when "A" region references "B" region
    - How it works?
        1. Initial mark(STW)
            - marks objects that are directly referenced by the GC roots
            - short STW
        2. Concurrent mark phase
            - marks all the live objects by using the reference chain from GC roots
        3. Final mark(remark) phase
            - briefly STW to ensure all reachable objects are marked
            - addresses any objects that may have been modified or newly created since the concurrent marking
              phase begin
        4. Concurrent cleanup
            - optimal cleanup phase to reclaim regions that have no live objects
        5. Concurrent evacuation phase
            - compacts memory by relocating live objects to new memory regions
            - done concurrently with the application
            - uses read barrier and forwarding pointer(brooks pointer) to manage references to relocated objects
        6. Initial update references phase(STW)
            - ensure that concurrent evacuation phase is over
            - short STW
        7. Concurrent references update
            - updates any references to objects that were relocated during the evacuation phase
            - ensures that all object references in the heap are up-to-date and point to correct memory location
        8. Final reference update(STW)
            - update the references of GC roots
        9. Concurrent free phase
            - reclaims the memory of regions that are no longer in use

    - What is forwarding pointer?
        - a technique used to handle references to objects that have been relocated during memory compaction or
          evacuation
        - this technique allows Shenandoah to move live objects without stopping the application thread
        - Forwarding pointer is placed within each object's headers
            - allows Shenandoah to redirect references from the old location of the object to its new one
        - Shenandoah prevent concurrent access to forwarding pointers, otherwise what can happen in below case?
            1. GC thread creates copy of the live object
            2. Application thread modifies field of the live object
            3. GC thread updates the forwarding pointer address

- ZGC
    - first version of ZGC is ...
        - region based heap memory layout without generation concept
        - targets low latency
            - in JDK 17, ZGC is able to reduce the average GC latency under 1ms
        - utilizes read barrier, colored pointer, multi memory mapping to implement concurrent mark and compact
          algorithm
    - Concepts
        - Memory layout
            - region based(or Page/ZPage)
            - dynamically sized
                - for x86-64 platform, region size is small(2MB)/medium(32MB)/large(N x 2MB)
                    - small: 2MB fixed size, stores objects smaller than 256KB
                    - medium: 32MB fixed size, stores objects smaller than 4MB
                    - large
                        - dynamically sized(but it has to be multiple of 2MB), for objects greater than 4MB
                        - can be smaller than medium region though
                        - don't reallocate large regions(because the copying cost is high)
        - Colored pointers
            - ZGC encodes metadata directly in "object pointers"(not on the objects)
            - how is the object pointer(64 bit) comprised of?
                - first 16bit - not used
                - Finalizable
                - Remapped
                - Marked1
                - Marked0
                - last 44 bit - 16TB address space
    - How it works
        1. Initial marking(STW): marks the objects directly referenced by the GC roots
        2. Concurrent marking phase
            - marks live objects while the application threads continue running
        3. Final marking(STW)
        4. Prepare for concurrent relocation
            - Choose regions to clean up -> create a relocation set
        5. Concurrent relocation
            - compact the memory by moving objects around
            - references are updated using the colored pointers
            - when application thread access objects that are being relocated
                - memory barrier comes into play
                - forwards to new memory location(where the object has been moved)
                - object pointer is updated as well -> "self healing"
        6. Remapping pointers
            - Update all the not up-to-date pointers in the relocation set
            - Not urgent because ZGC has self healing mechanism
            - This phase can be merged with the next concurrent marking phase
- Generational ZGC
    - Weak generational hypothesis still works on ZGC
      - the reason why ZGC didn't include weak generational hypothesis in the first place is because of its implementation complexity
    - How it handles big objects 
      - allocate in young generation region 
      - when it survives longer, the region itself becomes old 

### How to select the right Garbage Collector 

- Epsilon Garbage Collector 
  - Doesn't perform garbage collection 
  - Useful for running performance test or stress test 
- Standards for choosing GC 
  - Is throughput important or latency important 
  - Hardware spec 
  - Which JDK provider are you going to use? 
- If you are using Oracle JDK or OpenJDK
  - If the application handles data which is below 100MB -> serial collector 
  - If the application runs on single processor and latency isn't important -> serial collector  
  - If the application throughput is the utmost important + latency is not that importance -> JVM's default collector or parallel collector 
  - Latency is important -> G1 
  - Latency is super important -> (Generational) ZGC 

# Related Links

[Postings related to GC](https://tschatzl.github.io/)
