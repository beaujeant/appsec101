# Programming

[Learning C](https://www.learn-c.org/), or programming in general, is beyond the scope of this course. You are actually expected to understand C and the main concepts of programming to follow this course. This chapter is just a recap and allows me to have references to some concepts later in this course. The chapter [assembly](assembly.md) is meant to match this chapter so that you can see the transformation from C to assembly.

## Hello world!

Usually, when learning a new language, the first code we teach to students is the `Hello World!`program. The goal of this program is to display the text `Hello World!`. In C, the `Hello World!` program looks like this:

{% code title="hello-world.c" %}
```c
#include <stdio.h>

void main ()
{
    printf("Hello World!");
}
```
{% endcode %}

Let’s try it! First, we create a new file `hello.c` with the code above, then execute the following two commands:

```text
$ gcc hello.c -o hello
$ ./hello
```

You should see in your terminal below `./hello` the text `Hello World!`.

Let’s analyse the source code now. The first line, i.e. `#include <stdio.h>`, tells `gcc` to copy the content of the file `stdio.h` and paste it in `hello.c` before it start compiling the code. `stdio.h` is a _header file_that _defines_ a few variables, macros and functions related to input/ouptut, such as `printf()`. _Defining_ doesn’t mean that the source code for `printf()` is in `stdio.h`. Instead, we only have its _prototype_. We will see later in the sub-chapter [functions](programming.md#functions) what a prototype is.

On the third line, we have `void main ()`, this basically declare the function `main`. This line tells that the function `main` doesn’t return any value \(`void`\) and take no arguments \(`()`\). The function is delimited with curly brackets \(on line 4 and 6\).

The syntax in C is quite flexible, i.e. the tabulation is purely aesthetic \(unlike python\) and the curly brackets could also be located on the same line as the function declaration:

```c
void main () {
    printf("Hello World!");
}
```

… or everything in one line:

```c
void main () { printf("Hello World!"); }
```

… or the way I prefer:

```c
void main ()
{
    printf("Hello World!");
}
```

The `main` function is a bit special as it is called by the operating system whenever the program is run. If you remember in chapter [Memory](memory.md) - [ELF/PE file](memory.md#elf-pe-file), the PE file contains the address where to start once the application is loaded in memory. This address is also known as the _entry point_. The _entry point_ contains the address of the function `_start`. Unless explicitly written otherwise in your code, the function `_start` is responsible for setting up some memories \(including the program arguments and the environment variables\) then calls `main`.

This means simply writing a function in your code is not enough to execute it. This function needs to be called somewhere. Either directly in `main`, or in a function called in main \(or function called from a function called… in main\).

Finally, on the 5th line, we have the content of the function `main`, i.e. `printf("Hello World!");`. This call the function `printf` with one argument, the string `Hello World!`. The function `printf` will writes the string \(the argument\) to the standard output \(in this case, the terminal\). As explained in chapter [Memory](memory.md) - [ELF/PE file](memory.md#elf-pe-file), the code for the function `printf` is actually in the library `libc.so`, which is imported by default when compiling with `gcc`. Therefore, there is no need to write the code `printf`.

## Comments

Developers can add notes in their code that will be ignored by the compiler. Comments are typically used to:

* Explain what the code is doing
* Explain how to use a function
* Add note for future changes
* Ignore one or multiple lines a code

Initially, comments in C always started with `/*` and ended with `*/` and could take place over multiple lines, e.g.:

```c
/* 
 * Multiple
 * line
 * comment
 */

printf("Hello World!");

/* One line comment */
```

Later, with the introduction of the C++ language, a new format of comment was used, the double slash comment `//`. Unlike the initial format, double slash can comment only one line each time:

```c
// This is a comment

printf("Hello World");

// This
// is
// a
// comment
```

The comment can also be used at the end of a code line:

```c
printf("Hello World");   // This is a comment
```

## Variables

People create programs to automate and fasten the processing of data. In computer science, everything is about manipulating data. Data is located in memory, which means for the CPU to manipulate the data, you need to provide the address in memory where the data is located. When writing code, it would be very confusing and obscure to use direct memory addresses \(or offsets\) when storing and manipulating data. Instead, when programming in C, we use **variables**, a kind of _aliases_ for memory locations. Variable allows developers to have meaningful name to identify/locate data and simplify the memory allocation.

Before a variable is used, it has to be **declared**. The syntax for declaring a variable is the following: `<type> <variable name>;`. For instance, if you want to declare an _integer_ with the name _number_, you simply need to type:

```c
int number;
```

You will find a list of different types of variable in chapter [CPU](cpu.md). _Integers_ are declared with `int`, _floats_ with `float`, _characters_ with `char` and _booleans_ with `bool`.

{% hint style="info" %}
More types are available, but those are the most used.
{% endhint %}

There is a limited set of types available for variables. However, you are able to define your own custom type with `struct`. Let say for instance you want to create a variable that stores time \(date + time\):

```c
struct Date
{
   int year;
   int month;
   int day;
   int hours;
   int minutes;
   int seconds;
};
```

Now that the type `Date` is defined and initialized, we can declare variables \(`today` for instance\) as a `Date` variable:

```c
struct Date today;
```

It is also possible to declare arrays of a specific type. In this case, we just add `[<size>]` right after the name of the variable. For instance, the following line will declare an array of 11 characters:

```c
char string[11];
```

Declaring a variable tells the compiler \(the tool responsible for the translation from C to machine instructions\) what type of data the variable is expected to handle. With this information, the compiler will be able to do the following:

* Before compiling, it will verify in the code if the data stored in the variable corresponds to the type used when declared
* When using arrays, it will know the right offset in memory between the elements
* It will write instructions to allocate the appropriate amount of bytes in memory based on the type
* Select the right machine instructions based on the type

### Type verification

Let’s consider the following code:

```c
int number;
number = "Hello";
```

The variable `number` is declared as an _integer_, then the string “Hello” is assigned to it \(actually the pointer to the string “Hello”\), which is not an integer. Therefore, when compiling, the compiler throws the following error message:

```text
warning: assignment makes integer from pointer without a cast [-Wint-conversion]
   number = "Hello";
```

Sometime, when possible, the compiler will fix automatically the problem:

```c
int number;
number = 3.2;
```

Here, we assign a _float_ to the variable `number` which is supposed to handle _integer_. When compiling, the code is translated with the following assembly instruction:

```text
mov DWORD PTR [ebp-0xc],0x3
```

It automatically converted the float `3.2` into the integer \(`0x3`\) and copied it in memory where the variable `number` is located \(i.e. `ebp-0xc`\). Don’t worry if you didn’t understand this last example, we will have a closer look at it in the next [chapter](assembly.md).

### Browsing array

Declaring is also important for arrays. Elements in arrays are stored in memory one after the other. But each element takes a specific amount of bytes in memory depending on the type. For instance, let’s consider the following array of 10 integers:

```c
int array[10];
```

Let say the beginning of the array is located at the address `0xbfffef00`. The first element \(`array[0]`\) will be located at `0xbfffef00`, the second element \(`array[1]`\) will be located at `0xbfffef08` \(`0xbfffef00`+ size of _one_ integer\), the third element \(`array[2]`\) will be located at `0xbfffef10` \(`0xbfffef00` + _two_ time the size of one integer\), etc. Now, let’s consider the following array of 10 characters:

```c
char string[10];
```

This array is located at `0xbfffee00`. The first element \(`string[0]`\) will be located at `0xbfffee00`, the second element \(`string[1]`\) will be located at `0xbfffee01` \(`0xbfffee00` + size of _one_ character\), the third element \(`string[2]`\) will be located at `0xbfffee00` \(`0xbfffee02` + _two_ time the size of one character\), etc.

So we can tell the address pointed by `string[2]` is the same as the address calculated from `string` + `(2 * sizeof(char))`:

* `string`: When the variable name of an array is used without `[ ]`, using the name gets you the address in memory of the variable \(first element in array\)
* `sizeof()`: Gets the size of an object or type
* `char`: Type with which the array has been declared

So basically, the type in the declaration of an array is used to determine where the next element is located in memory. In other words, it tells you how many bytes should be skipped to reach the next element.

### Allocating memory

Declaring a variable also tells how much space should be allocated in memory. So for instance, if you declare an integer, 4 bytes will be allocated in the stack:

```c
int number;
```

Will be translated as the following in machine instruction:

```text
sub esp, 0x4
```

The above assembly instruction allocates 4 bytes in the stack. `ESP` points at the top of the stack, and if you look in chapter [memory](memory.md), the stack is growing _backward_, so subtracting `ESP` means increasing the size of the stack.

### Assigning value

Once the variable declared, we can assign data and manipulate it. To assign a value to the variable, you simply need to use the symbol `=`:

```c
int number;
number = 3;
```

In a table, you will need to indicate the index \(first element being `0`\). So the following will set the value `L` at the third element of the array:

```c
char string[11];
string[2] = 'L';
```

{% hint style="info" %}
Single quotes \(`'`\) is interpreted in C as a single character, while double quotes \(`"`\) are interpreted as a `NULL` terminated string.
{% endhint %}

Assigning value on `struct` variables is slightly different. The two most common ways to set the variable is either by using `{}` with the value separated with a coma, or by using the sub-name separated with a `.`. Here is a mix of both:

```c
struct Date today = { 2019, 1, 1, 0, 0, 0 };
today.day = 17;
today.hours = 15;
today.minutes = 6;
today.minutes = 45;
```

Now that the variables are declared and set, we can start manipulating them.

### Pointer

We mentioned at the beginning of this sub-chapter that variables are meant to be some sort of abstraction layer so that we don’t have to deal with memory addresses and offsets and instead we can use a meaningful name to store, move and process data. However, sometime, it could be interesting to actually know and manipulate the memory address of a variable. One reason will be explain later in sub-chapter [function arguments](programming.md#functions).

In order to know the address in memory where the variable is stored, you can use the symbol `&` right in front of the variable name, e.g. `&number`. You can also create variables that are meant to store a memory address. Those variables are called _pointer_ and can be declared as follow: `<type> *<variable name>`. For instance, if we want to create a pointer `pointer` that will store the address of an integer, we will have the following line:

```c
int *pointer;
```

Then, here is how we assign a memory address to a pointer:

```c
int a;
int *pointer;

pointer = &a;
```

Now, if we modify the variable `a`, the address will remain the same, but its value will change:

```c
int a, b;
int *pointer;

a = 7;
b = a;

pointer = &a;

printf("%d", a); // Print "7"
printf("%d", b); // Print "7"
printf("%d", pointer); // Print address location of number in decimal format
printf("%d", *pointer); // Print "7"

a = 14;

printf("%d", a); // Print "14"
printf("%d", b); // Print "7"
printf("%d", pointer); // Print address location of number in decimal format (same as before)
printf("%d", *pointer); // Print "14"
```

When using the pointer name alone, we get the memory address stored in the pointer variable. Whenever we use the `*` in front of the name, we get the value located at the memory address stored in the pointer variable.

As you can see in our last example, whenever we did `b = a;`, we copied the content stored in the variable `a` into the variable `b`. We did not copy the address. So, later, whenever the variable `a` is modified, `b` still has the old value.

## Math/Logic operations

C language has native arithmetic and logic operation available straight out of the box without using additional functions. Below are some examples using the operants `int a = 7` and `int b = 10`, where the result is stored in `signed int res`.

* **Addition**: `res = a + b; // res = 17`
* **Subtraction**: `res = a - b; // res = -3`
* **Multiplication**: `res = a * b; // res = 70`
* **Incrementing**: `res = ++a; // a = 8`
* **Decrementing**: `res = --a; // a = 6`
* **Division**: `res = a / b; // res = 0`
* **AND**: `res = a & b; // res = 2`
* **OR**: `res = a | b; // res = 15`
* **XOR**: `res = a ^ b; // res = 13`
* **NOT**: `res = ~ a; // res = -8`
* **Shift left**: `res = a << b; // res = 7168`
* **Shift right**: `res = a >> b; // res = 0`

{% hint style="info" %}
The division resulted in an incorrect value. This is because we are dividing integer numbers and save the value in an integer variable.
{% endhint %}

## Branching

When programming, you will most likely want at some point to verify or test some variables and do something accordingly. For instance, verifying if the password entered by the user is correct or verify if a file exists. These can be done thanks to **branching**. Branching is whenever the program has to choose between two \(or more\) branches based on one \(or more\) conditions. The most common form of branches is the **if**/**else** statements:

```c
if( pin_code == 1234 )
{
    printf("PIN correct!");
}
else
{
    printf("Wrong PIN!");
}
```

{% hint style="info" %}
The `else` statement is not mandatory.
{% endhint %}

In this example, the condition is `pin_code == 1234`, which verifies whether the variable `pin_code`contains the integer value `1234`. If it is the case, it will execute whatever is in the `if`’s curly brackets, which means it will print the message “PIN correct!”. Otherwise, it will execute whatever is in the `else`’s curly brackets, and thus print “Wrong PIN!”.

The result of a condition is always seen as a boolean in the **if** statement. So the condition could simply be a single variable. If the variable is `0`, the condition will be seen as `false`, any other value will be seen as `true`. Conditions can also be a comparison as seen earlier:

* `==`: returns `true` if both elements are equal
* `!=`: returns `true` if both elements are different
* `>`: returns `true` if the first element is bigger than the first element
* `>=`: returns `true` if the first element is bigger or equal than the second element
* `<`: returns `true` if the first element is smaller than the first element
* `<=`: returns `true` if the first element is smaller or equal than the second element

The boolean result can also be inverted with the operator `!` \(i.e. _NOT_\):

```c
if( ! (pin_code == 1234) )
{
    printf("Wrong PIN!");
}
```

You can have more than one condition concatenated with the operator `&&` \(i.e. _AND_\) or `||` \(i.e. _OR_\). With `&&`, both condition must be true, while when using `||`, either or both must be true.

{% tabs %}
{% tab title="AND" %}
```c
if( (pin_code >  1233) && (pin_code <  1235)  )
{
    printf("PIN correct!");
}
```
{% endtab %}

{% tab title="OR" %}
```c
if( (pin_code <= 1233) || (pin_code >= 1235) )
{
    printf("Wrong PIN!");
}
```
{% endtab %}
{% endtabs %}

## Loop

You may want at some point to repeat multiple times a succession of instructions. Let say you want to initialize a big array with `0`. You could do the following:

```c
int table[100];
table[0] = 0;
table[0] = 0;
table[0] = 0;
table[0] = 0;
...
table[99] = 0;
table[100] = 0;
```

In this example, you would have a long repetitive list of instructions. If you suddenly want to initialize the array with the value `2` instead of `0`, you would have to change 100 lines. Furthermore, what if you don’t know the size of the array and this depends on the user input? You wouldn’t be able to predict the size of the array and write the correct amount of lines. Instead, you could use the `for`, the `while` or the `do-while` loop.  Loops take one or more conditions \(like `if/else`\). The condition dictates whether the loop should reiterate or not.

```c
int table[100];
int i = 0;

while( i < 100 )
{
    table[i] = 0;
    i = i + 1;
}
```

In line 1, we declare an array of 100 integers. In line two, we declare the integer `i`, which will be used later as a counter for the loop. In line 4, we have the beginning of the loop with the condition within the parentheses, then between line 5 and 8 \(the curly brackets\), we have the code to execute at each iteration. In line 6, we have the element of the array initialized, where the counter is used as an index. Finally, in line 7, we have the counter incremented. Whenever the program reaches the end of the loop, it will check the condition and check whether it should go back at the beginning of it \(line 6\).

The structure of a `for` loop allows you to condense a little bit the code that manages the loop variable \(in this case, the variable `i`\):

```c
int table[100];

for( int i = 0; i < 100; i = i + 1)
{
    table[i] = 0;
}
```

As you can see, the condition of the `for` loop is split into 3 parts \(separated with semi-colons\):

* Declaration and assignment of the counter: `int i = 0`
* Condition to match in order to continue the loop: `i < 100`
* The operation to execute at the end of the iteration \(usually in order to update the counter\): `i = i + 1`

There is still one last loop variant: the `do while`. Basically, it works like the `while` loop except that the content of the loop is executed at least once, no matter if the condition is met or not.

```c
int i = 0;

do
{
    printf("This is executed at least once");
    i = i + 1;
} while( i <  10);
```

## Functions

Whenever you have a set of instructions that is used multiple times, you might want to avoid re-writing them over and over again. Usually, programmers group sets of instructions that are used for a specific purpose into functions. Let say for instance you have the following block of code used multiple times in your program:

```c
int january_temperature[] = { -5, -6, -4, -7, -8, -4, -2, 2, 0, -1, 3, 6, 4, 3, 6, 5, 7, 2, 0, 4, 2, 1, 3, -2, -6, -3, -1, 0, 1, 2, 3 };
int february_temperature[] = { 2, 1, 4, 5, 4, 7, 1, 3, 6, 7, 12, 5, 7, 6, 3, 7, 7, 9, 5, 7, 9, 11, 9, 8, 5, 3, 3, 7 };
// ...


// Begin of the block
// ------------------
int day;
int highest;
int lowest;

highest = january_temperature[0];
lowest = january_temperature[0];

for ( day = 1; day < 31; day++ ) 
{
    if ( highest < january_temperature[day] )
        highest = january_temperature[day];

    if ( lowest > january_temperature[day] )
        lowest = january_temperature[day];
}

printf( "Highest temperature: %d", highest );
printf( "Lowest temperature: %d", lowest );

// ------------
// End of block

// Begin of the block
// ------------------
highest = february_temperature[0];
lowest = february_temperature[0];

for ( day = 1; day < 28; day++ ) 
{
    if ( highest < february_temperature[day] )
        highest = february_temperature[day];

    if ( lowest > february_temperature[day] )
        lowest = february_temperature[day];
}

printf( "Highest temperature: %d", highest );
printf( "Lowest temperature: %d", lowest );

// ------------
// End of block

// ...
```

In this example, we have a set of instructions that are meant to find the highest and lowest value in an array and print the result in the terminal. Instead of always re-writing the block of code, we could create a function that re-uses the instructions then whenever we want to execute those blocks of instructions, we simply need to call the function by its defined name.

A function typically takes 0, one or more variables \(or direct values\) as inputs. It processes data, sometimes call other functions, then it returns 0 or one variable \(or direct value\) as output. 

A function is defined by its header, which contains the type of the returned value, the name of the function, then the list of expected arguments:

```c
void get_highest_and_lowest_temperature( int month[], int days_in_month )
{
    // Body
}
```

Here, we can see our new function `get_highest_and_lowest_temperature` has been defined. The function doesn’t return any output \(`void`\) and takes two arguments: one array of integer called `month` and one integer called `days_in_month`. 

Now that the function is defined, we can add the body, which contains the operations to be executed by the function.

```c
void get_highest_and_lowest_temperature( int month[], int days_in_month )
{
    int highest = month[0];
    int lowest = month[0];

    for ( int day = 1; day < days_in_month; day++ ) 
    {
        if ( highest < month[day] )
            highest = month[day];

        if ( lowest > month[day] )
            lowest = month[day];
    }

    printf( "Highest temperature: %d \n", highest );
    printf( "Lowest temperature: %d \n", lowest );
}
```

Now, our function is ready to be used. We can call it in `main` like this:

{% code title="func.c" %}
```c
#include <stdio.h>

void get_highest_and_lowest_temperature( int month[], int days_in_month )
{
    int highest = month[0];
    int lowest = month[0];

    for ( int day = 1; day < days_in_month; day++ ) 
    {
        if ( highest < month[day] )
            highest = month[day];

        if ( lowest > month[day] )
            lowest = month[day];
    }

    printf( "Highest temperature: %d \n", highest );
    printf( "Lowest temperature: %d \n", lowest );
}

void main ()
{
    int january_temperature[] = { -5, -6, -4, -7, -8, -4, -2, 2, 0, -1, 3, 6, 4, 3, 6, 5, 7, 2, 0, 4, 2, 1, 3, -2, -6, -3, -1, 0, 1, 2, 3 };
    int february_temperature[] = { 2, 1, 4, 5, 4, 7, 1, 3, 6, 7, 12, 5, 7, 6, 3, 7, 7, 9, 5, 7, 9, 11, 9, 8, 5, 3, 3, 7 };
    // ...

    get_highest_and_lowest_temperature( january_temperature, 31 );
    get_highest_and_lowest_temperature( february_temperature, 28 );
    // ...
}
```
{% endcode %}

When compiled and executed, we have the following output:

```text
$ gcc func.c -o func
$ ./func
Highest temperature: 7
Lowest temperature: -8
Highest temperature: 12
Lowest temperature: 1
```

### Returned value

As mentioned earlier, functions can also return one variable \(or direct value\). Let’s consider the following example:

```c
int add( int a, int b )
{
    int c;

    c = a + b;

    return c;
}
```

We created the function `add` that takes two arguments \(`a` and `b`\), add both numbers and returns the result. Here is how we can call that function and save the returned value:

```c
#include <stdio.h>

int add( int a, int b )
{
    int c;

    c = a + b;

    return c;
}

void main ()
{
    int result;

    result = add(5, 7);

    printf("Result: %d"); // Result: 12
}
```

Once compiled and executed, the program will simply print `Result: 12`.

### Function arguments

Functions can take 0, one or multiple arguments. These are meant to send data from the _caller_ \(the function calling another function\) to the _callee_ \(the function called by another function\). Unless explicitly declared outside all functions, a variable can be used only in the function where it has been declared. So this means the following won’t work:

{% code title="add.c" %}
```c
#include <stdio.h>

void add()
{
    a = a + b;
}

void main ()
{
    int a = 5;
    int b = 7;

    printf("a: %d \n", a);
    printf("b: %d \n", b);
    
    add();
    
    printf("a: %d \n", a);
    printf("b: %d \n", b);
}
```
{% endcode %}

First of all, you won't be able to compile this code as the variable `a` is not defined in the function `add`. But event if you could, the variable `a` would contain 5 before and after calling the function `add`.

If you want to use a variable that get updated and persist after the call of the function, you can do as follow:

```c
#include <stdio.h>

int A = 5;
int B = 7;

void add()
{
    A = A + B;
}

void main ()
{
    printf("A: %d \n", A); // A: 5
    printf("B: %d \n", B); // B: 7
    
    add();
    
    printf("A: %d \n", A); // A: 12
    printf("B: %d \n", B); // B: 7 
}
```

{% hint style="info" %}
Variables declared outside all functions are called _global_ variable.
{% endhint %}

Although maybe counterintuitive, whenever you send a variable as an argument, you actually send a copy of its content rather than the variable itself. This means if a function modifies the value of an argument, the change will take place only within that function:

```c
#include <stdio.h>

int add( int a, int b )
{
    a = a + b;

    return a;
}

void main ()
{
    int a = 5;
    int b = 7;
    int c;

    c = add( a, b );

    printf( "a: %d \n", a ); // a: 5
    printf( "b: %d \n", b ); // b: 7
    printf( "c: %d \n", c ); // c: 12
}
```

As you can see, although the variable `a` in the function `add` has been changed, the variable `a` in `main`remains the same. This is because `a` from `add` only receive a copy of the value from `a` in main and both `a` variables only exist within the function where they have been declared. Using the the same variable name in both function doesn't makes it persistent and might actually be confusing. You could have used a different name for the variable in the function `add`, it would be the same.

If we want the alter a variable declared in a different function \(without using global variables\), we need to send a pointer to that variable. In this case, we don’t send a copy of the content, but the address in memory where the variable is located. In this case, when we change the content, since we directly edit the value located at the same memory location, the changes will remain even after the function returns.

```c
#include <stdio.h>

void add( int *a, int b )
{
    *a = *a + b;
}

void main ()
{
    int a = 5;
    int b = 7;
    
    printf( "a: %d \n", a ); // a: 5
    printf( "b: %d \n", b ); // b: 7

    add( &a, b );

    printf( "a: %d \n", a ); // a: 12
    printf( "b: %d \n", b ); // b: 7
}
```

### Prototype

If you remember well, we mentioned at the beginning of this chapter that the included file `stdio.h` at the beginning of the `Hello World!` program contains the prototype for the function `printf`. A function prototype declares the function name, its parameters, and its return type to the rest of the program. It is used to tell the compiler what argument\(s\) is expected and what type is returned. Since the code for `printf` is not located in the program source code, without the prototype, the compiler will not be able to determine whether `printf` is expecting a string as the first parameter, and therefore, it won’t be able to flag an error if the programmer used the function in a wrong way.

Function prototypes are also used whenever a call to a function happen before the function is declared in the code. In this case, the prototype of the function should be placed at the beginning of the source code:

```c
#include <stdio.h>

int add( int a, int b ); // Function prototype for add

void main ()
{
    int c;

    c = add( 5, 7 );
    printf( "c: %d", c ); // c: 12
}

int add( int a, int b )
{
    a = a + b;

    return a;
}
```

## Program arguments

Now that we have discussed pointers and function prototypes, we can talk about program arguments. Whenever you want to run a program, you typically execute the following command:

```text
$ ./program-name
```

Sometimes, the program can take arguments, like for instance the Linux command `cp` \(copy\) which accepts — at least — two arguments, the first one being the path to the file we want to copy and the second arguments being the path where we want to copy the file.

```text
$ cp /path/to/source-file /path/to/destination-file
```

Whenever you send arguments to a program, it will be available through an array of pointer.

Here is the actual declaration of `main`:

```c
int main(int argc, char *argv[]);
```

The function `main` is usually expected to return an _exit status_ \(`0` if the program completed successfully\) and it takes two arguments:

* `argc`: an integer which contains the number of arguments sent
* `argv`: an array of `char` pointer which are the actual arguments sent

We’ve seen earlier in chapter [CPU](cpu.md), strings are actually an array of `char` terminated with a NULL character \(`0x00`\). And as we’ve seen in this chapter, whenever we use the name of an array without brackets, this is actually a pointer to the first element of that array. Whenever we deal with a string, we usually send the pointer and not a single element of the array. So the argument `char *argv[]` can be considered as an array of strings, where `argv[0]` is the first argument, `argv[1]` the second, etc.

Here is an example to print all program arguments:

{% code title="read-args.c" %}
```c
#include <stdio.h>

int main (int argc, char *argv[])
{
    int i = 0;

    while(i < argc)
    {
        printf("%d. %s \n", i, argv[i]);
        i++;
    }
}
```
{% endcode %}

When executed:

```text
$ gcc read-args.c -o read-args
$ ./read-args hello world
0. ./args 
1. hello 
2. worl d
```

## Common functions

The following functions are all part of the default `libc.so` library and are common functions used in most programs. These functions are all somewhat related to the vulnerabilities we will cover in this course, so it is very important to understand what’s their purpose and how they work.

### printf

We’ve already used this function earlier in some examples. We know it is meant to print text in the terminal \(standard output\), but let’s have a closer look at it.

Unlike most functions, `printf` can take an _unlimited_ amount of argument. It expects at least one argument, the _format string_. The _format string_ is basically what `printf` will _print_ in the terminal.

```c
printf("Hello World!"); // Hello World!
```

Whenever you want to print a variable, within that text, you can add a _format specifier_ in the _format string_. The function `printf` will then replace the _format specifier_ with the corresponding value sent as an argument to the function. The _format specifier_ has the following structure: `%[flags][width][.precision][length]specifier`.

{% hint style="info" %}
Options in brackets \(`[` `]`\) are optional.
{% endhint %}

The `specifier` is the most important part of the _format specifier_ as it the way the corresponding input is interpreted and displayed. Here is the list of valid _specifier_:

| Specifier | Output | Example |
| :--- | :--- | :--- |
| `d` _or_ `i` | Signed decimal integer | `392` |
| `u` | Unsigned decimal integer | `7235` |
| `o` | Unsigned octal | `610` |
| `x` | Unsigned hexadecimal integer | `7fa` |
| `X` | Unsigned hexadecimal integer \(uppercase\) | `7FA` |
| `f` | Decimal floating point, lowercase | `392.65` |
| `F` | Decimal floating point, uppercase | `392.65` |
| `e` | Scientific notation \(mantissa/exponent\), lowercase | `3.9265e+2` |
| `E` | Scientific notation \(mantissa/exponent\), uppercase | `3.9265E+2` |
| `g` | Use the shortest representation: `%e` or `%f` | `392.65` |
| `G` | Use the shortest representation: `%E` or `%F` | `392.65` |
| `a` | Hexadecimal floating point, lowercase | `-0xc.90fep-2` |
| `A` | Hexadecimal floating point, uppercase | `-0XC.90FEP-2` |
| `c` | Character | `a` |
| `s` | String of characters | `sample` |
| `p` | Pointer address | `0xb8000000` |
| `n` | Nothing printed. The corresponding argument must be a pointer to a signed int. The number of characters written so far is stored in the pointed location. |  |
| `%` | A `%` followed by another `%` character will write a single % to the stream. | `%` |

Here are some examples:

```c
int n = 42;
printf("Variable n (int): %d \n", n);       // Variable n (int): 42
printf("Variable n (hex): %X \n", n);       // Variable n (hex): 2A
printf("Pointer to variable n: %p \n", &n); // Pointer to variable n: 0x--------

n = 1337;
printf("Variable n (int): %d \n", n);       // Variable n (int): 1337
printf("Pointer to variable n: %p \n", &n); // Pointer to variable n: 0x--------

char c = 'a';
printf("Variable c (char): %c \n", c);      // Variable c (char): a
printf("Variable c (int): %X \n", c);       // Variable c (int): 61
printf("Pointer to variable c: %p \n", &c); //Pointer to variable c: 0x--------
```

{% hint style="info" %}
The backslash n \(`\n`\) is meant to go to the next line.
{% endhint %}

So far we’ve printed only one variable, but `printf` can, of course, print more than one. For this, you simply need to add the variable in the argument of the function and add the appropriate _specifier_ in the _format string_. The first _format specifier_ will correspond to the second argument, the second _format specifier_ will correspond to the third one, …

```c
int x = 42;
int y = 1337;
int res;

res = x + y;

printf("%d + %d = %d \n", x, y, res); // 42 + 1337 = 1379
```

Among all _specifier_, there is one particular: `n`. Unlike the other _specifier_, `n` is not meant to print something in the terminal but instead, it writes the number of characters written so far at a given address.

```c
int i;

printf("Hello%n World! \n", &i);      // Hello World!
//           ^ after 5th character

printf("Variable i (int): %d \n", i); // Variable i (int): 5

printf("Hello World!%n \n", &i);      // Hello World!
//                  ^ after 12th character

printf("Variable i (int): %d \n", i); // Variable i (int): 12

int n = 1234567890;
printf("i = %d %n \n", n, &i);        // i = 1234567890
//             ^ after 7th character but i is printed with 10 decimal
//               7 + 10 - 2 (the %d specifier is two characters) = 15

printf("Variable i (int): %d \n", i); // Variable i (int): 15
```

{% hint style="info" %}
The specifier `%n` expect a memory address \(pointer\) and not a variable, hence the `&i`.
{% endhint %}

The _format specifier_ can also contain the following optional sub-specifiers:

* flag
* width
* precision
* Length

For this workshop, we are only interested in the _width_ sub-specifier. If you want more information about the other sub-specifier, you can read the [printf reference](http://www.cplusplus.com/reference/cstdio/printf/).

By default, printf will print the minimum amount of character required to display the variable. For instance, the integer `1234`, although it’s stored in a 32-bit variable, if we want to print it as hexadecimal, we won’t have `000004D2`, but only `4D2`:

```c
int n = 1234;

printf("%d in hex: %X\n", n, n); // 1234 in hex: 4D2
```

If you want the variable to be printed to actually be printed with minimum 8 character, you can use the _width_ sub-specifier:

```c
int n = 1234;

printf("%d in hex: %X\n", n, n); // 1234 in hex: 4D2
```

By default, the padding is a space \( ``\). So here, in this case, the variable `1234` was printed with 5 x space and 3 x hexadecimal values. If you want to pad it with `0`’s, you can use the _flag_ sub-specifier `0`:

```c
int n = 1234;

printf("%d in hex: %08X\n", n, n); // 1234 in hex: 000004D2
```

In our last example, we printed the same variable multiple times. Therefore, we send the same variable multiple times as argument. In order to avoid that, we can use the _position specifier_:

```c
int n = 1234;

printf("%d in hex: %X\n", n, n); // 1234 in hex: 4D2

printf("%1$d in hex: %1$X\n", n); // 1234 in hex: 4D2
```

The `1$` correspond to the second argument, `2$` correspond to the third argument, etc. Here is a final example:

```c
char e = 'E';
char h = 'H';
char l = 'L';
char o = 'O';

printf("%2$c %1$c %3$c %3$c %4$c \n", e, h, l, o); // H E L L O
```

### sprintf

The function `sprintf` is a bit like `printf`, it builds a string based on a _format string_ and the additional arguments, but instead of printing the result, it stores it in a string variable \(char array\). The destination variable is the first argument, the format string is the second and additional format parameters are place after.

```c
#include <stdio.h>

void main()
{

    int a = 42;
    char string[100]; 

    sprintf(string, "Answer = %d", &a);
    
    printf("string: %s \n", string); // string: Answer = 42

}
```

You can find a bit more information about the function in the [sprintf reference](http://www.cplusplus.com/reference/cstdio/sprintf/?kw=sprintf).

### scanf

We’ve seen how to print \(output\) data with `printf`, now we will see how to get \(input\) data from the user with the function `scanf`. The first argument for `scanf` is also a _format string_. In this case, the _format string_ tells how we want to cast the user input. The _format string_ usually contains a single _format specifier_. The `scanf` _format specifier_ has a similar structure as `printf`: `%[*][width][length]specifier`.

For this workshop, we will only use the _format specifier_ without options, i.e. `%specifier`. Here is a list of _specifiers_ we will use in our examples:

| specifier | Description | Characters extracted |
| :--- | :--- | :--- |
| i | Integer | Any number of digits, optionally preceded by a sign \(+ or -\). Decimal digits assumed by default \(0-9\), but a 0 prefix introduces octal digits \(0-7\), and 0x hexadecimal digits \(0-f\). Signed argument. |
| c | Character | The next character. If a width other than 1 is specified, the function reads exactly width characters and stores them in the successive locations of the array passed as argument. No null character is appended at the end. |
| s | String of characters | Any number of non-whitespace characters, stopping at the first whitespace character found. A terminating null character is automatically added at the end of the stored sequence. |
| p | Pointer address | A sequence of characters representing a pointer. The particular format used depends on the system and library implementation, but it is the same as the one used to format %p in `printf`. |

```c
#include <stdio.h>

void main()
{

    int  a;
    char b;
    char c[100]; 

    scanf("%d", &a);
    scanf(" %c", &b);
    scanf(" %s", c);

    printf("a: %d \n", a);
    printf("b: %c \n", b);
    printf("c: %s \n", c);

}
```

{% hint style="info" %}
At the first `scanf` we enter a number in the terminal. In order to “validate” the number, we press `<enter>`. However, this `<enter>`  is also sent to `scanf` as `\n`, which doesn’t match the format specifier, so `scanf` keep it in the buffer. So next time `scanf` is called, the first character received will be that `\n`. In order to discard that newline character, we added a whitespace in our format strings of the following `scanf`. This whitespace tells `scanf` to ignore any “whitespace” element, i.e. space, tabulation, and newline.
{% endhint %}

{% hint style="info" %}
When using `%s`, `scanf` will copy the string until it finds a "whitespace". So, if you type "Hello World!", only the "Hello" will be stored in the variable.
{% endhint %}

That’s all we need to know about `scanf` for this workshop. If you want more information about `scanf`, you can read the [scanf reference](http://www.cplusplus.com/reference/cstdio/scanf/).

### gets

Another similar function to `scanf` is `gets`. The function only takes one argument, the _destination_ where the user input will be stored. Unlike `scanf`, it stops copying the user input only once it reaches a newline character \(or EOF\). So, this means you can store a string that has spaces and tabulations.

```c
#include <stdio.h>

void main()
{
    
    char text[100]; 

    gets(text);

    printf("text: %s \n", text);
}
```

The function is quite simple, but if you want to know more about, you can read the [gets reference](http://www.cplusplus.com/reference/cstdio/gets/).

### strcpy

Now that we know how to read and print strings, let’s see how to manipulate them. The function `strcpy` is meant to copy a string from one address to another. The function takes two arguments, the _destination_ \(first\) and the _source_ \(second\). Both are pointers to a string \(i.e. array of `char`\). `strcpy`copies one by one all character from _source_ to _destination_ until it reaches the `NULL` \(`0x00`\) character that terminates the _source_ string.

```c
#include <stdio.h>
#include <string.h>

void main()
{

    char a[100]; 
    char b[100]; 
    char c[100]; 

    strcpy(a, "Hello World");
    printf("a: %s \n", a);

    scanf("%s", b);
    strcpy(c, b);
    printf("b: %s \n", b);
    printf("c: %s \n", c);

}
```

{% hint style="info" %}
The prototype of the function `strcpy` is located in the header file `string.h`, so you need to `#include` it first at the beginning of your code.
{% endhint %}

There is not much more to say about that function. If you have to read a bit more about it, you can have a look at the [strcpy reference](http://www.cplusplus.com/reference/cstring/strcpy/).

### malloc

We’ve already talked about `malloc` and the reasons why we need it in chapter [Memory](memory.md) - [Heap](memory.md#heap). Basically, `malloc` is used to dynamically allocate memory to store data. The function takes only one argument, the _size_ of the new block you want to allocated \(in byte\) and it returns a pointer to that allocated memory block.

```c
#include <stdio.h>
#include <stdlib.h>

void main()
{

    char month[10];
    int amount;
    int cpt = 0;
    int *pointer;
    int tmp;

    printf("Month: ");
    scanf("%s", month);

    printf("How many days in %s? ", month);
    scanf("%i", &amount);

    pointer = malloc (amount * 4); // Integer are 4 bytes

    // Collect temperature
    while(cpt < amount)
    {
        printf("Day %d temperature: ", cpt+1);
        scanf("%i", &tmp);
        pointer[cpt] = tmp;
        cpt++;
    }

    // Print temperature
    cpt = 0;
    while(cpt < amount)
    {
        printf("%d: %d, ", cpt+1, pointer[cpt]);
        cpt++;
    }

}
```

{% hint style="info" %}
The prototype of the function `malloc` is located in the header file `stdlib.h`, so you need to `#include` it first at the beginning of your code.
{% endhint %}

Here again, that’s pretty much it. If you want further information, you can read the [malloc reference](http://www.cplusplus.com/reference/cstdlib/malloc/).

### free

The function `free` was also covered in chapter [Memory](memory.md) - [Heap](memory.md#heap). `free` is used whenever we no longer need the dynamically allocated memory block. It _releases_ so that it can be re-used for another dynamically allocated memory. The function takes as argument a pointer to a dynamically allocated memory block and returns nothing.

```c
#include <stdio.h>
#include <stdlib.h>

void main()
{

    char month[10];
    int amount;
    int cpt = 0;
    int *pointer;
    int tmp;

    // see the previous example

    pointer = malloc (amount * 4); // Integer are 4 bytes

    // see the previous example

    free(pointer);

}
```

Once again, for more information, you can read the [free reference](http://www.cplusplus.com/reference/cstdlib/free/). We will see more about this function in the chapter [use-after-free](https://beaujeant.github.io/appsec101/programming/#).

That’s it for this big chapter about C programming. In the [next chapter](https://beaujeant.github.io/appsec101/assembly/), we will see how this is translated by the compiler in assembly.

## References

* [Learning C](https://www.learn-c.org/): https://www.learn-c.org/
* [C Tutorial](https://www.codingunit.com/category/c-tutorials): https://www.codingunit.com/category/c-tutorials
* [printf reference](http://www.cplusplus.com/reference/cstdio/printf/): http://www.cplusplus.com/reference/cstdio/printf/
* [sprintf reference](http://www.cplusplus.com/reference/cstdio/sprintf/?kw=sprintf): http://www.cplusplus.com/reference/cstdio/sprintf/?kw=sprintf
* [scanf reference](http://www.cplusplus.com/reference/cstdio/scanf/): http://www.cplusplus.com/reference/cstdio/scanf/
* [gets reference](http://www.cplusplus.com/reference/cstdio/gets/): http://www.cplusplus.com/reference/cstdio/gets/
* [strcpy reference](http://www.cplusplus.com/reference/cstring/strcpy/): http://www.cplusplus.com/reference/cstring/strcpy/
* [malloc reference](http://www.cplusplus.com/reference/cstdlib/malloc/): http://www.cplusplus.com/reference/cstdlib/malloc/
* [free reference](http://www.cplusplus.com/reference/cstdlib/free/): http://www.cplusplus.com/reference/cstdlib/free/
* [Online compiler](https://www.tutorialspoint.com/compile_c_online.php): https://www.tutorialspoint.com/compile\_c\_online.php

