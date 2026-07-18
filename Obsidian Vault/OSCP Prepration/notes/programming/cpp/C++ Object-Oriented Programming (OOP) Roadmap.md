## 1. Basics & Syntax

- Structure of a C++ program (Headers, `main()` function)
    
- Variables, Data Types (`int`, `float`, `char`, `bool`, etc.), and Constants
    
- Input/Output streams (`std::cin`, `std::cout`)
    
- Operators (Arithmetic, Relational, Logical, Assignment)
    

## 2. Control Flow

- Conditional Statements (`if`, `else if`, `else`, `switch`)
    
- Loops (`for`, `while`, `do-while`)
    
- Jump Statements (`break`, `continue`, `goto`)
    

## 3. Functions

- Function Declaration, Definition, and Calling
    
- Function Parameters (Pass-by-value vs. Pass-by-reference)
    
- Function Overloading
    
- Inline Functions and Default Arguments
    
- Recursion
    

## 4. Derived Data Types & Memory Management

- Arrays (Single-dimensional and Multi-dimensional)
    
- Pointers (Pointer arithmetic, Pointers to pointers)
    
- References
    
- Dynamic Memory Allocation (`new` and `delete`)
    
- C-style Strings vs. `std::string`
    

## 5. Object-Oriented Programming (OOP)

- Classes and Objects
    
- Access Specifiers (`public`, `private`, `protected`)
    
- Constructors and Destructors (Shallow vs. Deep copy)
    
- Encapsulation and Abstraction
    
- **Inheritance** (Single, Multiple, Multilevel, Hierarchical, Hybrid)
    
- **Polymorphism**
    
    - Compile-time (Function & Operator Overloading)
        
    - Runtime (Virtual Functions, Pure Virtual Functions, Abstract Classes)
        

## 6. Advanced Memory & Structure

- Structures (`struct`), Unions, and Enumerations (`enum`)
    
- `const` keyword correctness
    
- Static Members (Variables and Functions)
    
- Friend Functions and Friend Classes
    
- Smart Pointers (`std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`)
    

## 7. Templates & Standard Template Library (STL)

- Function Templates and Class Templates
    
- **STL Containers:**
    
    - Sequence (`std::vector`, `std::list`, `std::deque`)
        
    - Associative (`std::set`, `std::map`)
        
    - Derived/Adapters (`std::stack`, `std::queue`, `std::priority_queue`)
        
- **STL Iterators**
    
- **STL Algorithms** (`std::sort`, `std::reverse`, `std::find`, etc.)
    

## 8. Exception Handling & File I/O

- Exception Handling (`try`, `catch`, `throw`)
    
- Custom Exceptions
    
- File Handling (`std::ifstream`, `std::ofstream`, `std::fstream`)
    
- Working with Binary and Text files
    

## 9. Modern C++ (C++11 to C++23 Features)

- Type Inference (`auto` and `decltype`)
    
- Lambda Expressions
    
- Move Semantics and Rvalue References (`std::move`)
    
- Concepts and Ranges (Modern C++20+)
    
- Concurrency and Multithreading (`std::thread`, `std::mutex`, `std::async`)
## 1. Classes & Objects (The Foundation)
* **Class Architecture**
    * Defining structural abstractions (Member variables / attributes).
    * Implementing behaviors (Member functions / methods).
* **Object Lifecycle & Instantiation**
    * Stack allocation vs. Heap allocation.
    * Dynamic instantiation via `new` and memory clearance via `delete`.
* **Access Specifiers & Data Hiding**
    * `public`: Accessible from outside the class context.
    * `private`: Restricted strictly to internal class scope.
    * `protected`: Accessible within the class and its derived structures.
* **Data Encapsulation**
    * Hiding raw states and exposing controlled behaviors through Getter and Setter patterns.

## 2. Constructor & Destructor Lifecycle Mechanics
* **Constructors**
    * Default Constructors.
    * Parameterized Constructors.
    * **Member Initialization Lists:** Initializing attributes cleanly before the constructor execution block runs.
* **Resource Management**
    * **Copy Constructor:** Managing shallow copies vs. handling deep copies for resource allocation.
    * **Copy Assignment Operator (`operator=`)**: Dealing with variable re-assignments safely.
    * **Destructors (`~ClassName`)**: Cleaning up allocated entities, closing open files, and avoiding memory leaks.
* **Resource Management Standards**
    * **The Rule of Three:** Managing the Destructor, Copy Constructor, and Copy Assignment Operator as a cohesive unit when allocating heap memory.
    * **The Rule of Five (Modern C++):** Incorporating Move Constructors and Move Assignment Operators to leverage move semantics smoothly.

## 3. The Pillars of OOP

### A. Encapsulation & Abstraction
* Separating structural interfaces (`.h`/`.hpp` header files) from explicit implementation pathways (`.cpp` files).
* Packaging related logic together while hiding complex implementation details.

### B. Inheritance (Code Reuse & Hierarchy)
* **Base Class (Parent)** vs. **Derived Class (Child)** relationships.
* **Visibility Control:** Public, Protected, and Private inheritance models.
* **Structural Topologies:**
    * Single Inheritance.
    * Multiple Inheritance.
    * Multilevel Inheritance.
    * Hierarchical Inheritance.
    * Hybrid Inheritance.
* **The Diamond Problem:** Solving multiple-inheritance ambiguity cleanly using `virtual` base inheritance paths.

### C. Polymorphism (Extensible Design)
* **Compile-Time Polymorphism (Static Binding)**
    * Function Overloading.
    * Operator Overloading (Customizing behaviors like `+`, `-`, `<<`, `==`).
* **Runtime Polymorphism (Dynamic Binding)**
    * **Virtual Functions (`virtual`)**: Overriding base class methods in derived classes.
    * **VTABLE & VPTR Mechanics:** Understanding behind-the-scenes runtime execution routing tables.
    * **Pure Virtual Functions:** Defining interface requirements (`virtual void method() = 0;`).
    * **Abstract Base Classes (ABCs):** Designing interface templates that cannot be directly instantiated.
    * **Virtual Destructors:** Making sure derived object classes tear down cleanly when destroyed via base-class pointers.

## 4. Advanced OOP Concepts
* **Static Class Members**
    * `static` variables: Global shared state across all object instances.
    * `static` functions: Scope-restricted functions that can run without instantiating an object.
* **Friend Utilities**
    * `friend` functions: Granting exterior functions permission to inspect `private` fields.
    * `friend` classes: Allowing external class contexts to read hidden variables.
* **Const Correctness**
    * `const` member functions: Ensuring methods do not inadvertently alter object attributes.
* **Smart Pointers in OOP**
    * Managing modern objects cleanly using `std::unique_ptr` and `std::shared_ptr` to avoid manual deletion issues.
