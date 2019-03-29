# Central Processing Unit

Computer technology is everywhere: whenever you browse the internet, read your mail, watch a movie, give a call to your family, drive your car, everywhere. Computer, and digital technology in general are meant to process data in a very fast manner. The brain of this technology is the **C**​entral **P**​rocessing **U**​nit \(CPU\).

## CPU structure

The CPU is the elements responsible for the computer’s operations. Operations can be arithmetic calculation \(such as _additions_ or _division_\) or logic operations \(such as _AND_ or _OR_\). Operations are executed by the **arithmetic unit**.

Operations have one or two parameters – also known as operands \(e.g. `5 + 2` has two parameters: `5`and `2`\). In order to execute operations, the CPU needs to get \(read\) the parameters and store the results somewhere. This is why a part of the CPU \(the **control unit**\) is responsible for retrieving and saving data in the memory. The data processed and saved can be many things \(e.g. an image, a music file, a word document, the amount of health point your game character has left\), but it will always be _binary_ data – meaning a bunch of 1s and 0s.

Operation parameter\(s\) sometimes might come from **inputs** \(e.g. the keyboard, the mouse or a temperature sensor\). Once the operation is done, we usually want to **output** the final result somewhere \(e.g. print it on the screen, play the music on speakers or send it to the network\). The **control unit** is responsible for dealing with inputs and output \(I/O\).

![CPU-Memory-I/O](https://beaujeant.github.io/AppSec101/resources/images/cpu-memory-io.png)

As you can see, the **control unit** is doing a lot in the CPU. It orchestrates the four elements \(i.e. memory, the arithmetic unit, input, and output\) and routes the data across them. The control unit doesn’t magically work on its own. It needs instructions to follow. Instructions such as “take the value located here in memory and send it to the arithmetic unit to execute the following operation”. Those sequences of instruction are the program itself. Whenever a programmer compiles code, they actually create a binary application that contains a list of CPU instructions that the control unit will execute. When the program is running, instructions are loaded in memory and read by the control unit. Once executed, the control unit will look at the instruction located right after in memory – unless the instruction executed redirected the execution flow somewhere else \(with a jump for instance\).

Now, let’s focus on the **arithmetic unit** and its principal component: the **A**​rithmentic **L**​ogic **U**​nit \(ALU\). ALU can be represented as follow:

![ALU](https://beaujeant.github.io/AppSec101/resources/images/alu.png)

It has four inputs \(_input A_, _input B_, _command_ and the _status_\) and two outputs \(_output_ and _status output_\):

* _Command_ is the arithmetic or logic operation to execute \(subtraction, multiplication, XOR, NOT, etc\). The _command_ is a binary value that will select the right area in the ALU to be triggered.
* _Status_ is the _status output_ from the previous operation.
* _Input A_ and _input B_ are the parameters of the operation.
* _Output_ is the result of the operation.
* _Status output_ indicates how the operation went. It could have several meanings depending on the operation executed. For instance, with subtraction, the _status output_ can tell whether the _output_value is positive or negative. The _status output_ is important as it is used for taking a decision.

For instance, the `9 + 33` would represented as follow:

![Addition](https://beaujeant.github.io/AppSec101/resources/images/addition.png)

{% hint style="info" %}
The input status has been removed because it was not relevant in this example.
{% endhint %}

We’ve seen earlier, the control unit read instructions and coordinate the memory, input/output, and arithmetic unit accordingly. But it also takes decisions based on operation output \(and status output\). For instance, let’s consider an online transaction. Before executing the transaction, the bank server will first subtract the amount of the transaction to the current account balance. If the subtraction \(which took place in the ALU\) returns a negative value \(indicated by the _status output_\), this would mean the user doesn’t have enough money for the transaction, and thus when the _control unit_ reads the instructions “go to _cancel transaction_ if negative otherwise proceed with transaction” it will be able to will take the right decision.

So in summary, the CPU is the brain of the computer. It reads and executes instructions from a program. The instruction tells the **control unit** how to coordinate with the different connected elements, i.e. **arithmetic unit**, **memory**, **input** and **output**. It fetches data from memory, executes operations with the **ALU**, saves data in the memory and interacts with the input/output. Based on the operation executed, the **control unit** can take decisions that will alter the execution flow of the program.

## Data/information

CPU is executing operations on data. As mentioned earlier, this data – which could also be called _information_ – could be anything, such as:

* Movie
* Music
* Picture
* 3D models
* Novel
* Weather forecast
* Website
* Food recipe
* Health status
* …

Humanity didn’t wait for computer to store and processing such data. Civilization developed language to communicate between each other, the printing industry made it even easier to store and share knowledge, computers are just the next step that facilitates the transfer and the processing of that data, with the major benefit being the high increase of speed. Data can be stored, transferred, copied and processed at a very faster pace thanks to computers.

In order for the data to be processed by a computer, it needs to be converted in a format that the computer can read. For instance, computers cannot process a picture from a [roll film](https://en.wikipedia.org/wiki/Roll_film), it first needs to convert the picture to a digital format, which means converting in a meaningful series of `1`s and `0`s. For this, the computer will need some electronic sensors that will isolate a small portion of the picture, one sensor will evaluate the percentage of red and send to the computer the digital value from 0 \(no red\) to 255 \(100% red\) in binary. Then it will do the same for the color green and blue. Once done, it will move to the portion right next to it, and continue until the entire picture has been scanned. The computer will then reconstruct the red/green/blue \(R/G/B\) portions together in a digital format so that the picture can be displayed, edited or copied.

{% hint style="info" %}
There are many ways to digitalized a picture, this was just a simplified example of how to do it.
{% endhint %}

Digitalizing data often \(but not necessary\) involved loss of information. For instance, with our picture, the sensor isolated a small portion and evaluate the percentage of red/green/blue for that portion. But that portion itself also had a nuance of red/green/blue within it. So in order to be more accurate, the sensor should have taken smaller portions, however, the sensor itself is limited by its own capability. Depending on the quality and technology used, the sensor will be more or less accurate. As of now, we don’t have a color sensor with atomic precision, which means we will lose precision and thus information in the digitalization of pictures.

Not all digitalization processes induce loss of information. It depends on the nature of the collected information. For instance, a text that is digitalized, if properly done, will not lose any information. For sure we won’t have the color of the page, the grain of the paper, the smell of the book, the exact font used; but the text itself can be digitalized with 0% loss of information.

## Binary numeral system

Computers use electricity to store, transfer and process data. Data is basically electricity or magnetic charge. Some engineers tried initially to use voltage-levels to transmit different information, i.e. 0-1V = `0`, 1-3V = `1`, 3-5V = `2`, etc. \[[1](https://hashnode.com/post/why-do-computers-understand-only-0-and-1-logic-cj4zcejn100kuomwu3z70cjan)\] However, they noticed it was complicate to use such implementation. The two main reasons being:

* The design of ALU and logic operation is more complex
* The technical complexity of generating a stable voltage

Engineers decided to use a simpler binary implementation, i.e. ~0V = `0` and ~5V = `1`. When stored, for instance in a hard drive, data is represented in a form of magnetic polarity. One polarity = `0` and opposite = `1`. This is why, in the computer realm, everything is only `1`s and `0`s. So for instance, if you want to execute the operation `5 + 2`, the ALU will receive `00000000000000000000000000000101` as the first input and `00000000000000000000000000000010` as the second input.

{% hint style="info" %}
The ALU needs to receive input of a fixed length. Here this example, we have a 32 bits ALU. This means the inputs are always 32 bits \(one bit is a value that can be either 0 or 1\) long and the output will also be 32 bit long.
{% endhint %}

The common numeral system we are using daily is the decimal system \(also known as base 10\). This means once we reached the 10th value, we increase by one the value positioned before. Here is an example:

| Decimal |
| :--- |
| `0000` |
| `0001` |
| `0002` |
| `0003` |
| `0004` |
| `0005` |
| `0006` |
| `0007` |
| `0008` |
| `0009` |

Here, the value located at the 4th position is about to reach its 10th iteration. Since we are in a base 10, the value at the 3rd position will be incremented and the value at the 4th position will be reset to 0.

| Decimal |
| :--- |
| `0010` |
| `0011` |
| `0012` |
| `....` |
| `0098` |
| `0099` |

Here, the value located at the 4th position is about the reach again its 10th iteration \(since reset to 0\). That means at the next iteration, we should increment the value at the 3rd position. However, the value at the 3rd position is also about to reach its 10th incrementation, so this means we have to increment the value at the position before \(i.e. 2nd position\) and we reset the 3rd and 4th position.

| Decimal |
| :--- |
| `0100` |
| `0101` |
| `0102` |
| `....` |

{% hint style="info" %}
Usually, the leading 0 are not shown. So instead of 0001, 0002, etc, we usually display 1, 2, 3, etc.
{% endhint %}

I know it’s odd to explain something such basic, but this process is the exact same one as with the binary numeral system \(base 2\), but instead of incrementing the value positioned before when we reach its 10th iteration, we increment it at the 2nd iteration:

| Binary | Decimal |
| :--- | :--- |
| `0000` | `0000` |
| `0001` | `0001` |

Here we already have reached the 2nd iteration, so this means we increment the value located at the 3rd position and reset the 4th position.

| Binary | Decimal |
| :--- | :--- |
| `0010` | `0002` |
| `0011` | `0003` |

Now we once again reached the 2nd iteration for the value located at the 4th position but also the one located at the 3rd position, so this means we increment the value at the 2nd position and reset the 3rd and 4th value.

| Binary | Decimal |
| :--- | :--- |
| `0100` | `0004` |
| `0101` | `0005` |
| `0110` | `0006` |
| `0111` | `0007` |
| `1000` | `0008` |
| `1001` | `0009` |
| `1010` | `0010` |
| `1011` | `0011` |
| `1100` | `0012` |
| `1101` | `0013` |
| `1110` | `0014` |
| `1111` | `0015` |

In base 2, four binary values can represent a number from 0 to 15 \(decimal\), while in base 10, 4 decimal values can represent a number from 0 to 9999. This makes it quite tedious to print values in binary, this is why when we want to represent binary value, we usually use the hexadecimal numeral system \(base 16\):

| Hexadecimal | Decimal |
| :--- | :--- |
| `0000` | `0000` |
| `0001` | `0001` |
| `0002` | `0002` |
| `0003` | `0003` |
| `0004` | `0004` |
| `0005` | `0005` |
| `0006` | `0006` |
| `0007` | `0007` |
| `0008` | `0008` |
| `0009` | `0009` |

So far, so good. While the base 10 will already increment the position before, we haven’t reached the 16th iteration in the hexadecimal system so we can keep incrementing the value at the 4th position. However, we don’t have a unique symbol to represent the number 10 \(we use the [Arabic numerals](https://en.wikipedia.org/wiki/Arabic_numerals)\), so we decided to re-use the Latin letter `a` \(could be uppercase or lowercase\), 11 will be `a` and so on, up to 15 being `f`.

| Hexadecimal | Decimal |
| :--- | :--- |
| `000a` | `0010` |
| `000b` | `0011` |
| `000c` | `0012` |
| `000d` | `0013` |
| `000e` | `0014` |
| `000f` | `0015` |
| `0010` | `0016` |
| `0011` | `0017` |
| `0012` | `0018` |
| `....` | `....` |
| `00ff` | `0255` |
| `0100` | `0256` |
| `0101` | `0257` |
| `....` | `....` |

So, why using base 16 and not 10 you may ask. There are multiple reasons but the main ones are:

* It is easier to convert a binary number in hexadecimal and the other way around. Four binary values can always be represented with one hexadecimal value. So this means if you want to convert from hexadecimal to binary, you first need to chunk the number character by character and convert them in their respective 4 binary values.
* This is related to the first reason, but hexadecimal value aligns with its binary equivalent. For instance, the number 9 \(decimal\) is 1001 in binary. One symbol in decimal for four symbols in binary. Now the number 12 \(decimal\) is 1100 in binary. Two symbols in decimal but still four symbols in binary. Now 18 \(decimal\) is 10010 in binary. Two symbols in decimal and 5 in binary. All this long example to explain that there is not a direct correlation with the amount a symbol in decimal and binary. However, since both hexadecimal and binary are bases that are powers of 2, it all aligns. 1-4 symbols in binary will always be 1 symbol in hexadecimal. 5-8 symbols in binary will always be 2 symbols in hexadecimal, etc.

![Bin to Hex](https://beaujeant.github.io/AppSec101/resources/images/bin-hex.gif)

This might not seem interesting at first sight, but with experience, you will notice hexadecimal comes handy.

{% hint style="info" %}
Hex \(short for hexadecimal\) values are usually represented with a leading `0x`. For instance, the binary `10100100` in hex would be `0xa4`.
{% endhint %}

## Data types

Now that we covered binary and hexadecimal, let’s discuss data type. This course covers 32-bit architecture. This means the memory addresses and registers \(see in the chapter [memory](memory.md)\) are 32-bits long. 32-bits long data is called a **double word** – also known as **long word**. A **word** is thus 16 bits, and a **byte** – also known as **octet** – is 8 bits. Those are type name for data of a specific size \(in bits\).

![Data types](https://beaujeant.github.io/AppSec101/resources/images/data-types.png)

{% hint style="info" %}
In computing, the most significant bit \(MSB, also called the high-order bit\) is the bit position in a binary number having the greatest value. The MSB is sometimes referred to as the high-order bit or left-most bit due to the convention in the positional notation of writing more significant digits further to the left. Therefore, the least significant bit \(LSB\) is the bit position in a binary integer giving the units value, that is, determining whether the number is even or odd. \[[2](https://en.wikipedia.org/wiki/Bit_numbering)\]
{% endhint %}

So this means a **double word** \(32 bits\) can be represented in hexadecimal with 8 symbols. E.g. `00010010001101001010101111001101` = `1234abcd`.

As mentioned \(many many times\) already, the binary data stored in memory can be anything and have any size \(not only the ones mentioned\). Here are the most seen types.

### Instruction

Instructions are the operation read and executed by the CPU. This could be moving data in registers/memory locations, arithmetic operation, logic operation, decision making and redirection of execution flow. The size of an instruction usually varies between 1 to 15 bytes. Once the instruction is executed, the CPU read and execute the instruction right after unless execution flow has been redirected with the previous instruction. So instructions should be seen a single block executed straight, but rather as multiple instruction blocks that usually coincide with functions.

### Integer

An integer is a whole number \(not fractional\) that can be positive, negative or 0. Integers can be of different size, but the most used one is 32-bit integer \(defined `int` in C\). An integer can be defined as _signed_ or _unsigned_. `unsigned int` integer are only positive \(or 0\) an goes from 0 \(`00000000000000000000000000000000`\) to 4,294,967,295 \(`11111111111111111111111111111111`\). `signed int`can be positive, negative or 0. The most significant bit is used to determine whether the number is positive or negative \(i.e. if the first bit is `0`, the integer is positive; if the first bit is `1`, the integer is negative\). Integers will described in more details in chapter Integer overflow.

### Float

Floats represent fractional values. You typically have the 32-bits _single precision_ \(`float` in C\) and the 64-bit _double precision_ \(`double` in C\) float values. A float is composed of 3 parts: the sign \(1 bit\), the exponent \(8 bits for _single precision_ and 11 bits for _double precision_\) and the fraction \(23 bits for the single precision and 52 bits for the _double precision_\).

Here is the explanation for the conversion of [single precision](https://en.wikipedia.org/wiki/Single-precision_floating-point_format#Converting_from_single-precision_binary_to_decimal) and [double precision](https://en.wikipedia.org/wiki/Double-precision_floating-point_format#Exponent_encoding). You can play with float \(single and double\) here: [float toy](https://evanw.github.io/float-toy/).

### Character

A character \(`char` in C\) is typically one byte long. It \(usually\) contains one printable character using the ASCII encoding standard.

```text
$ man ascii
...
The hexadecimal set:

00 nul   01 soh   02 stx   03 etx   04 eot   05 enq   06 ack   07 bel
08 bs    09 ht    0a nl    0b vt    0c np    0d cr    0e so    0f si
10 dle   11 dc1   12 dc2   13 dc3   14 dc4   15 nak   16 syn   17 etb
18 can   19 em    1a sub   1b esc   1c fs    1d gs    1e rs    1f us
20 sp    21  !    22  "    23  #    24  $    25  %    26  &    27  '
28  (    29  )    2a  *    2b  +    2c  ,    2d  -    2e  .    2f  /
30  0    31  1    32  2    33  3    34  4    35  5    36  6    37  7
38  8    39  9    3a  :    3b  ;    3c  <    3d  =    3e  >    3f  ?
40  @    41  A    42  B    43  C    44  D    45  E    46  F    47  G
48  H    49  I    4a  J    4b  K    4c  L    4d  M    4e  N    4f  O
50  P    51  Q    52  R    53  S    54  T    55  U    56  V    57  W
58  X    59  Y    5a  Z    5b  [    5c  \    5d  ]    5e  ^    5f  _
60  `    61  a    62  b    63  c    64  d    65  e    66  f    67  g
68  h    69  i    6a  j    6b  k    6c  l    6d  m    6e  n    6f  o
70  p    71  q    72  r    73  s    74  t    75  u    76  v    77  w
78  x    79  y    7a  z    7b  {    7c  |    7d  }    7e  ~    7f del
...
```

So for instance, the character `A` \(uppercase\) is `0x41` in hexadecimal and the ASCII character `1` is not `0x01` but `0x31` in hexadecimal.

### Array

An array is basically a collection of variables of the same type. So for instance, an array of integers will simply be integers places one after each other in memory.

### String

A string is an array of char that is terminated with the ASCII `NULL` character \(`0x00` in hexadecimal\). For instance, the string “Hello World” has the following structure.

| Byte | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 |
| ---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Hex | `0x48` | `0x65` | `0x6c` | `0x6c` | `0x6f` | `0x20` | `0x57` | `0x6f` | `0x72` | `0x6c` | `0x64` | `0x21` | `0x00` |
| ASCII | `H` | `e` | `l` | `l` | `o` |  `` | `W` | `o` | `r` | `l` | `d` | `!` | `NULL` |

{% hint style="info" %}
When calculating the length of a string, the `NULL` character is counted. So the string “Hello World” is 13 characters long.
{% endhint %}

When you want to write a mutli-line text, you can use the ASCII `\n` character \(`0x0a` in hexadecimal\) and/or `\r` character \(`0x0d` in hexadecimal\). Unix systems only need `\n`, Windows systems need both `\r` and `\n` \(in that order\). So the following string:

| Byte | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 |
| ---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Hex | `0x48` | `0x65` | `0x6c` | `0x6c` | `0x6f` | `0x0a` | `0x57` | `0x6f` | `0x72` | `0x6c` | `0x64` | `0x21` | `0x00` |
| ASCII | `H` | `e` | `l` | `l` | `o` | `\n` | `W` | `o` | `r` | `l` | `d` | `!` | `NULL` |

… will print the following text on a Unix system:

```text
Hello
World
```

### Boolean

A boolean type can have one of two values, either `1` \(`true`\) or `0` \(`false`\). C doesn’t have a boolean type by default. So it is not uncommon to see boolean using 32 bits. The only value that matters is LSB \(least significant bit\).

### Variable pointer

You cannot provide a variable name to the CPU, it will not understand where to find the value in memory. Instead, the CPU receives directly the address of the variable in memory. So that means whenever you ask the CPU to add the variable `a` with the variable `b`, you will need to provide to the CPU the instruction `add`, the address where the variable `a` is located in memory, and the address where the variable `b` is located in memory. Since this course covers only i386 \(32-bit\) architecture, memory addresses are always 32 bits long. Variable pointers can also point to an array or a structure variable that contains multiple variables of different types.

### Function pointer

We will see more about it in chapters [memory](memory.md) and assembly, but the instructions sent to the CPU are also located in memory alongside the variables \(although usually located in different sections\). So whenever a function is called, it is merely a jump to another area of the memory where the function’s instructions are located. A function pointer is an address \(32 bits\) of the memory where instructions are located.

### Handle

Unlike pointers, which are memory addresses, a handle is an abstraction of a reference which is managed externally; its opacity allows the referent to be relocated in memory by the system without invalidating the handle, which is impossible with pointers. Typical usages of handles are for _file descriptors_, _network sockets_, _database connections_, _registry key \(Windows\)\_\_, and \_process identifiers_. Let’s take the example of an opened file, whenever you use `fopen("filename.ext", "r+");`\(in C\), the function will return a handle for that file. Next time the application will access \(e.g. read\) the file, it will send the handle \(instead of the filename or an address in memory\), and it will be up to the operating system to find the file based on the handle used as an index, access it and returns whatever the program requested. A pointer is usually an integer that is incremented each time a new handle is generated.

| Handle | Type | Name |
| :--- | :--- | :--- |
| `6` | `File` | /home/user/debug.txt |
| `7` | `File` | /tmp/JeIfwW |
| `8` | `Directory` | /tmp/ |

### Binary object

We’ve just seen that whenever an application opens a file, it receive a handle instead of loading the file directly in memory. Accesses and changes to the actual file are done via the handle. However, in some case, it is necessary to have the file in memory. For instance, Photoshop needs to load the picture in memory so that it can display it within the application interface and perform changes \(changing luminosity, contrast, adding text/shape, etc\). Once the modifications are done, the user can save the picture. So the application uses the handle to write the actual files with the pictures loaded in memory \(so it overwrite the initial picture with the new one\). In this example, the binary object was a picture, but it could be many other things: image, video, sound, excel document, text file, etc.

### Mis-typed data

Data in memory are pure binary. So if you run a program and start looking at the memory, there is no way you can know for sure the boundaries of all variables stored and their type. By this, I mean once the application stored a variable in the memory, this variable has a meaning in the context of the function that uses it \(e.g. an integer that represent the size of a file\). The pointer to this variable will be saved somewhere and the next time the function will read the variable, it will expect a value that represents the size of a file. But what if for some reason, in the meantime, this value has been entirely or partially overwritten with a string, e.g. “BAD”. This would mean the variable, which initially contains the size of a file, has been overwritten with the value `0x42414400`.

| Byte | 1 | 2 | 3 | 4 |
| :--- | :--- | :--- | :--- | :--- |
| Hex | `0x42` | `0x41` | `0x44` | `0x00` |
| ASCII | `B` | `A` | `D` | `NULL` |

Next time the function will access the variable, it will have no way to figure out the value has been overwritten and that the new value was initially a string. It will simply read it as an integer and proceed as if nothing happened.

{% hint style="info" %}
Of course, the developer could add some additional checks to verify the integrity of the data, but by default, C program will not enforce it.
{% endhint %}

While this example could cause some functionality error, this might not be too dangerous from a security point of view. However, what if data in memory is overwritten with malicious instructions and that a function pointer is overwritten with the address to those malicious instructions. Next time the initial function is called, the malicious instructions will be executed, which could compromise the computer. This what we will try to explain in this course:

* How to read and overwrite data
* How to leverage overwritten technique for malicious purposes

## Logic operation

We’ve seen that the CPU \(actually the ALU\) is executing mathematic operations. It receives the command \(mathematical operation to execute\) and the two inputs. Commands can be basics maths, such as addition or division but also _logic operations_. In fact, all mathematical operations executed are based on _logic operation_, even a simple addition.

_Logic operations_ are mathematical operation in which the variable and result are boolean, i.e. either `true` \(`1`\) or `false` \(`0`\). The variable\(s\) are processed through a _gate_ which will execute a mathematical operation. The basic logic operations are: AND, OR, NOT. Based on those operations, other common operation can be built such as NAND, NOR, and XOR.

### AND operation

The _AND_ operation satisfies the following conditions:

* **if** a = b = `1` **then** \(a _AND_ b\) = `1`
* **otherwise** \(a _AND_ b\) = `0`

| a | b | output |
| :--- | :--- | :--- |
| `0` | `0` | `0` |
| `0` | `1` | `0` |
| `1` | `0` | `0` |
| `1` | `1` | `1` |

Basically, the _AND_ gate returns `true` only if both a and b are `true`.

### OR operation

The _OR_ operation satisfies the following conditions:

* **if** a = b = `0` **then** \(a _OR_ b\) = `0`
* **otherwise** \(a _OR_ b\) = `1`

| a | b | output |
| :--- | :--- | :--- |
| `0` | `0` | `0` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `1` |

Basically, the _OR_ gate returns `true` when at least one of the two arguments is `true`.

### NOT operation

The _NOT_ operation only takes one argument and satisfies the following conditions:

* **if** a = `0` **then** _NOT_ a = `1`
* **if** a = `1` **then** _NOT_ a = `0`

| a | output |
| :--- | :--- |
| `0` | `1` |
| `1` | `0` |

Basically, the _NOT_ gate inverts the value of a.

### NAND operation

The _NAND_ is a combination of _NOT_ and _AND_ operations:

* a _NAND_ b = _NOT_ \(a _AND_ b\)

| a | b | output |
| :--- | :--- | :--- |
| `0` | `0` | `1` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `0` |

Basically, the _NAND_ gate returns the invert of the _AND_ gate.

### NOR operation

The _NOR_ is a combination of _NOT_ and _OR_ operations:

* a _NOR_ b = _NOT_ \(a _OR_ b\)

| a | b | output |
| :--- | :--- | :--- |
| `0` | `0` | `1` |
| `0` | `1` | `0` |
| `1` | `0` | `0` |
| `1` | `1` | `0` |

Basically, the _NOR_ gate returns the invert of the _OR_ gate.

### XOR operation

The _XOR_ is a combination of _OR_, _AND_ and _NOT_ operations:

* a _XOR_ b = \(a _OR_ b\) _AND_ \(_NOT_ \(a _AND_ b\)\)

| a | b | output |
| :--- | :--- | :--- |
| `0` | `0` | `0` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `0` |

Basically, the _XOR_ gate returns `true` only when a and b are different.

This was the last logic operation for this course and the last section for this chapter. I hope you now have a better understanding of what a CPU is. In the next chapter, we will see how the memory is structured.

## References

* [roll film](https://en.wikipedia.org/wiki/Roll_film) https://en.wikipedia.org/wiki/Roll\_film
* \[[1](https://hashnode.com/post/why-do-computers-understand-only-0-and-1-logic-cj4zcejn100kuomwu3z70cjan)\] https://hashnode.com/post/why-do-computers-understand-only-0-and-1-logic-cj4zcejn100kuomwu3z70cjan
* [Arabic numerals](https://en.wikipedia.org/wiki/Arabic_numerals) https://en.wikipedia.org/wiki/Arabic\_numerals
* \[[2](https://en.wikipedia.org/wiki/Bit_numbering)\] https://en.wikipedia.org/wiki/Bit\_numbering
* [single precision](https://en.wikipedia.org/wiki/Single-precision_floating-point_format#Converting_from_single-precision_binary_to_decimal) https://en.wikipedia.org/wiki/Single-precision\_floating-point\_format\#Converting\_from\_single-precision\_binary\_to\_decimal
* [double precision](https://en.wikipedia.org/wiki/Double-precision_floating-point_format#Exponent_encoding) https://en.wikipedia.org/wiki/Double-precision\_floating-point\_format\#Exponent\_encoding
* [float toy](https://evanw.github.io/float-toy/) https://evanw.github.io/float-toy/

