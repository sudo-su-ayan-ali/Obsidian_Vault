## 1. Structure of a C++ Program

Every C++ program follows a specific anatomy. Here is what the most basic program looks like:

C++

```
#include <iostream> // 1. Header File

// 2. The Main Function
int main() {
    // 3. Execution Standard Output
    std::cout << "Hello, World!" << std::endl; 
    
    return 0; // 4. Return Statement
}
```

### The Key Components:

- **Headers (`#include <iostream>`):** These are directives that tell the compiler to include pre-existing code. `<iostream>` stands for _Input/Output Stream_ and is required for printing text to the screen or taking user input.
    
- **The `main()` Function:** This is the entry point of every C++ program. When you run your code, the operating system looks for `main()` and starts executing instructions from the opening curly brace `{` to the closing curly brace `}`.
    
- **`std::cout`:** This stands for "Character Output" (wrapped in the `std` standard namespace). It sends data to the console.
    
- **`return 0;`:** This signals to the operating system that the program ran successfully and finished without errors.
    

## 2. Variables, Data Types, and Constants

Variables are containers for storing data values. In C++, you must explicitly declare the _type_ of data a variable will hold before you can use it.

### Common Data Types:

|**Data Type**|**What it Stores**|**Size (Typical)**|**Example**|
|---|---|---|---|
|`int`|Integers (whole numbers)|4 bytes|`int age = 21;`|
|`double`|Floating-point numbers (decimals)|8 bytes|`double pi = 3.14159;`|
|`float`|Smaller floating-point numbers|4 bytes|`float interest = 5.99f;`|
|`char`|A single character (uses single quotes)|1 byte|`char grade = 'A';`|
|`bool`|Boolean (true or false)|1 byte|`bool isCodingFun = true;`|
|`std::string`|A sequence of text (uses double quotes)|Dynamic|`std::string name = "Alice";`|

### Constants

If you have a value that should **never change** during the program's execution, use the `const` keyword. This prevents accidental bugs.

C++

```
const double LIGHT_SPEED = 299792458; 
// LIGHT_SPEED = 300000000; // Error! The compiler will block this.
```

## 3. Input/Output Streams (`std::cin`, `std::cout`)

C++ uses streams to handle data flowing in and out of your program.

- **`std::cout` (Output):** Uses the **insertion operator** (`<<`) to push data _out_ to the console.
    
- **`std::cin` (Input):** Uses the **extraction operator** (`>>`) to pull data _in_ from the keyboard and store it in a variable.
    

C++

```
#include <iostream>

int main() {
    int favoriteNumber;

    std::cout << "Enter your favorite number: "; // Output
    std::cin >> favoriteNumber;                  // Input

    std::cout << "Your favorite number is " << favoriteNumber << "!" << std::endl;
    return 0;
}
```

> **Tip:** `std::endl` does two things: it moves the cursor to a new line and flushes the buffer (ensures the text actually appears on the screen immediately). You can also use `"\n"` for a faster new-line break.

## 4. Operators

Operators are symbols that tell the compiler to perform specific mathematical or logical manipulations.

### Arithmetic Operators

Used to perform basic mathematical operations.

- `+` (Addition), `-` (Subtraction), `*` (Multiplication), `/` (Division)
    
- `%` (**Modulo**): Returns the _remainder_ of a division (e.g., `5 % 2` evaluates to `1`).
    

### Relational Operators

Used to compare two values. They always return `true` or `false`.

- `==` (Equal to) — _Note: Do not confuse with `=` which assigns a value!_
    
- `!=` (Not equal to)
    
- `>` (Greater than), `<` (Less than)
    
- `>=` (Greater than or equal to), `<=` (Less than or equal to)
    

### Logical Operators

Used to combine multiple conditions.

- `&&` (**Logical AND**): Returns true only if _both_ conditions are true (e.g., `score > 50 && score < 100`).
    
- `||` (**Logical OR**): Returns true if _at least one_ condition is true (e.g., `hasCash || hasCard`).
    
- `!` (**Logical NOT**): Reverses the logical state (e.g., `!true` becomes `false`).
    

### Assignment Operators

Used to assign values to variables.

- `=` : Simple assignment (`x = 10;`)
    
- `+=`, `-=`, `*=`, `/=` : Compound assignments. `x += 5;` is just a shorthand way of writing `x = x + 5;`.