Assignment
PROBLEM SOLVING THROUGH C PROGRAMMING
1. Variables in C programming?
2. Symbolic constants and give one example.
3. Difference between while and do-while loops.
4. A function prototype in C?
5. Define recursion with an example.
6. How pointer declared?
7. Define an algorithm and list its key properties.
8. What is a flowchart? Mention two common flowchart symbols.
9. List the types of program errors in C.
10. Define a structure in C. Give an example.
11. The purpose of printf() function?
12. File handling in C programming.
13. Explain the steps in algorithm development and its importance in
problem solving.
14. Detailed note on data types, operators, and expressions in C. Explain
with examples how operator precedence and type conversions affect
program results
15. The types of errors in C programs and methods to debug them.
16. Control flow statements in C with examples.
17. The use of functions in modular programming and scope rules.
18. How pointers and arrays are related in C.
19. The steps for reading and writing files in C using fopen(), fprintf(),
and fscanf()
20. Function recursion and its advantages in C programming
21. The concept of pointers, arrays, and structures in C. Discuss pointer
arithmetic, arrays of structures, and pointers to functions with examples
22. The concept of structures and unions with examples.
23. The use of formatted input/output functions in C programming.
24. The process of program design and development. Discuss the stages
— algorithm creation, flowcharting, coding, testing, debugging, and
documentation with examples.


---


# Q.1
A **variable** is a named memory location that stores a value of a specific data type.

**Example:**

int age = 25; float price = 19.99; char grade = 'A';


---


# Q.2
In C programming, a **symbolic constant** is a name that is given to a constant value. Instead of using a literal value (like `3.14159` or `100`) directly in your code, you use a descriptive name.

---


# Q.3
- **`   while` loop:** An **entry-controlled** or **pre-test** loop. It checks the condition **before** executing the loop's body. This means the loop body might not execute at all if the condition is false from the very beginning.
- **`do-while` loop:** An **exit-controlled** or **post-test** loop. It executes the loop's body **at least once** and then checks the condition. This means the loop body is guaranteed to execute a minimum of one time.


---

# Q.4
A function prototype in C is a declaration that informs the compiler about a function's name, return type, and the number and types of its parameters, without including the function's body.

---


# Q.5
Recursion in C is a programming technique where a function calls itself to solve a problem by breaking it down into smaller, similar subproblems.


---


# Q.6
In C programming, a pointer is declared by specifying its data type followed by an asterisk (*) and the pointer's name.


---


# Q.7
An algorithm is a finite set of unambiguous instructions designed to solve a specific problem or perform a particular task within a finite amount of time.


---

# Q.8
A flowchart is a graphical representation of an algorithm or process that uses standardized symbols connected by arrows to illustrate the sequence of steps and the flow of execution.
### 🔹 **Two Common Flowchart Symbols**

| Symbol | Name                      |
| ------ | ------------------------- |
| ⬛ / ⏹  | **Terminal (Start/Stop)** |
| ▭      | **Process Box**           |

---


# Q.9
The primary types of program errors in C are syntax errors, runtime errors, linker errors, logical errors, and semantic errors.

---
# Q.10
A structure in C is a user-defined data type that groups together variables of different data types under a single name, allowing for the organization of related data in a logical and easy-to-understand way.

### Ex:
	struct Student{char name;int id;float gpa;};


---
# Q.11
The `Printf ` function in C is used to print formatted output to the standard output stream, which is typically the console screen.

---

# Q.12
File handling in C programming is the process of performing operations like creating, opening, reading, writing, and closing files to persist data beyond the runtime of a program.

---

# Q.13
**Algorithm development steps:**

1. Define the problem
    
2. Analyze requirements
    
3. Design the algorithm
    
4. Test with examples
    
5. Implement in code
    
6. Evaluate results
    

**Importance +6 
Algorithm development ensures logical, efficient, and error-free solutions. It simplifies complex problems, improves program structure, saves time, and enhances reliability and performance in problem solving before coding.


---


# Q.14
In **C programming**, **data types** define the kind of data a variable can store, such as `int`, `float`, `double`, and `char`. **Operators** perform operations on data, including arithmetic (`+`, `-`, `*`, `/`), relational (`>`, `<`, `==`), logical (`&&`, `||`), and assignment (`=`, `+=`). **Expressions** combine variables, constants, and operators to produce values. **Operator precedence** determines the order of evaluation, while **type conversions**—implicit or explicit—ensure correct results, prevent data loss, and maintain precision in calculations.


---


# Q.15

In **C programming**, errors are of three main types:

1. **Syntax Errors** – Detected by the compiler when code violates language rules, e.g., missing semicolons or invalid declarations. These prevent compilation.
    
2. **Logical Errors** – Program runs but produces incorrect output due to wrong logic or algorithm, e.g., using subtraction instead of addition.
    
3. **Runtime Errors** – Occur during execution, e.g., division by zero or invalid memory access, potentially crashing the program.
    

**Debugging methods** include compiler messages, `printf()` statements to trace values, debuggers like GDB for step-by-step execution, code reviews, test cases, and static analysis tools to detect potential issues before running.


---


# Q.16

In **C programming**, **control flow statements** determine the sequence of execution in a program, enabling decision-making, looping, and branching.

**Decision-making statements**:

- `if` executes a block if a condition is true.
    
- `if-else` chooses between two blocks based on a condition.
    
- `else-if ladder` handles multiple conditions.
    
- `switch` selects one case among many.
    

**Looping statements**:

- `for` loop iterates a known number of times.
    
- `while` loop continues while a condition is true.
    
- `do-while` executes at least once before checking the condition.
    

**Jump statements**:

- `break` exits a loop or switch.
    
- `continue` skips the current iteration.
    
- `return` exits a function, optionally returning a value.


---



# Q.17

In **C programming**, functions are blocks of code that perform specific tasks, enabling **modular programming**. They divide a program into smaller, manageable modules, improving **readability, reusability, and maintainability**. Functions help avoid code repetition, simplify debugging, and make complex programs easier to understand.

**Scope rules** determine where variables or functions are accessible:

- **Local scope**: variables inside a function or block, accessible only there.
    
- **Global scope**: variables declared outside functions, accessible anywhere.
    
- **Block scope**: variables inside `{}` accessible only within the block.
    
- **Function scope**: labels accessible within the function.

```c
#include <stdio.h>

void greet() { printf("Hello!\n"); } // function

int main() {
    greet();           // call function
    return 0;
}
```


---


# Q.18
In **C programming**, **pointers and arrays are closely related** because the name of an array acts as a **pointer to its first element**.

### **Key Relationships**

1. **Array name as a pointer**
    


```c
int arr[5] = {1,2,3,4,5};
int *ptr = arr; // ptr points to arr[0]
  ```

2. **Accessing elements using pointers**
    

```c
printf("%d", *(ptr + 2)); // prints arr[2] → 3
```

3. **Pointer arithmetic**
    

- `ptr + 1` moves to the next element based on the element’s data type size.
    

4. **Arrays of pointers**
    

- Can store addresses of multiple variables or strings.

---


# Q.19
In C, reading and writing files involves these steps:

1. **Open the file** using `fopen()` with modes like `"r"` (read), `"w"` (write), or `"a"` (append).
    
2. **Write to the file** using `fprintf()` for formatted output.
    
3. **Read from the file** using functions like `fscanf()` or `fgets()`.
    
4. **Close the file** using `fclose()` to release resources.
    

**Example:**

```c
FILE *fp = fopen("data.txt", "w");
fprintf(fp, "Hello, World!\n");
fclose(fp);```


---


# Q.20

**Recursion** occurs when a **function calls itself** either directly or indirectly to solve a problem. It usually includes:

1. **Base condition** – stops the recursion.
    
2. **Recursive call** – reduces the problem toward the base condition.
### **Advantages of Recursion**

1. **Simplifies code** for problems like factorial, Fibonacci, or tree traversal.
    
2. **Reduces code length** by avoiding complex loops.
    
3. **Easier problem modeling** for divide-and-conquer algorithms.
    
4. **Enhances readability** by expressing problems naturally.

---


# Q.21
**pointers, arrays, and structures in C**, along with **pointer arithmetic, arrays of structures, and pointers to functions**, with examples:



## **1. Pointers**

A **pointer** is a variable that stores the **address of another variable**.

```c
int a = 10; int *ptr = &a;   // ptr stores address of  a 
printf("%d", *ptr); // dereference to get value → 10```
```

### **Pointer Arithmetic**

Pointers can be **incremented or decremented** to traverse memory locations based on the data type size.

```c
int arr[3] = {1,2,3};
int *p = arr;
printf("%d", *(p + 1)); // prints 2
```



## **2. Arrays**

An **array** is a collection of elements of the same type stored **contiguously**.

```c
int arr[5] = {1,2,3,4,5}; 
printf("%d", arr[2]); // 3
```

- **Relationship with pointers:** Array name acts as a **pointer to the first element**.
    



## **3. Structures**

A **structure** groups **different data types** under a single name.

```c
struct Student {     
	int id;     
	char name[20];     
	float marks; 
};  
struct Student s1 = {101, "Rahul", 89.5};
```

### **Arrays of Structures**

Store multiple structure variables in an array.

```c
struct Student students[2] = {     
	{101, "Rahul", 89.5},     
	{102, "Anita", 92.0} 
};
 printf("%s", students[1].name); // Anita
```



## **4. Pointers to Functions**

A **pointer to function** stores the address of a function, allowing dynamic function calls.

```c
#include <stdio.h>  
void greet() { printf("Hello\n"); }  
int main() {     
	void (*funcPtr)() = greet; // pointer to function     
	funcPtr();                  // call via pointer     
	return 0; 
}
```

- Useful for **callback functions** or selecting functions dynamically.

---

# Q.22


### **Structures in C**

A **structure** is a **user-defined data type** that groups **different types of variables** under a single name. Structures are useful to represent real-world entities like students, employees, or books.

**Example of Structure:**

```c
#include <stdio.h>

struct Student {
    int id;
    char name[20];
    float marks;
};

int main() {
    struct Student s1 = {101, "Rahul", 89.5};
    printf("ID: %d, Name: %s, Marks: %.2f\n", s1.id, s1.name, s1.marks);
    return 0;
}

```
- **Memory:** Each member has its own memory.
    

---

### **Unions in C**

A **union** is similar to a structure, but **all members share the same memory location**. Only one member can hold a value at a time.

**Example of Union:**

```c
#include <stdio.h>

union Data {
    int i;
    float f;
    char ch;
};

int main() {
    union Data d;
    d.i = 10;
    printf("i = %d\n", d.i);
    d.f = 3.5;
    printf("f = %.2f\n", d.f); // overwrites previous value
    return 0;
}

```
- **Memory:** Size equals the largest member.
    
- **Use:** Saves memory when only one member is used at a time.


---

# Q.23 
### **Formatted Input/Output in C**

In **C programming**, formatted input/output functions allow programs to **read and display data in a specific format**. They provide more control than unformatted functions like `getchar()` or `puts()`.

## **1. `printf()` Function (Formatted Output)**

- Used to **display data** on the screen.
    
- Supports **format specifiers** to control output.
## **2. `scanf()` Function (Formatted Input)**

- Used to **read input** from the user.
    
- Format specifiers must match the variable type.


---
# Q.24
Program design and development is a systematic process to create **efficient and error-free programs**. It involves several stages to convert a problem into a working program.



## **1. Algorithm Creation**

An **algorithm** is a step-by-step procedure to solve a problem.  
**Example:** Sum of two numbers:

1. Start
    
2. Input `a` and `b`
    
3. `sum = a + b`
    
4. Display `sum`
    
5. End
    



## **2. Flowcharting**

A **flowchart** is a **graphical representation** of an algorithm using symbols. It helps visualize program logic.

**Example Symbols:**

- **Oval** → Start/End
    
- **Parallelogram** → Input/Output
    
- **Rectangle** → Process
    
- **Diamond** → Decision
    



## **3. Coding**

Writing the program in a **programming language (C)**.
```c
#include <stdio.h>
int main() {
    int a, b, sum;
    scanf("%d %d", &a, &b);
    sum = a + b;
    printf("Sum = %d", sum);
    return 0;
}

```



## **4. Testing**

Running the program with **different input values** to check correctness and ensure it meets requirements.



## **5. Debugging**

Identifying and fixing **errors** such as syntax, logical, or runtime errors.

- Tools: Compiler errors, `printf()` statements, GDB debugger.
    



## **6. Documentation**

Recording details about program logic, inputs, outputs, and functions for future reference or maintenance.

- **Internal:** Comments in code
    
- **External:** User manuals, flowcharts, and design notes
    

---