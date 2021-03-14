# Lab environment

In this course, we will do a little bit of _reversing_, _debugging_ and _compiling_.

**Reversing**, short for **reverse engineering**, is the process of reading machine language instructions and making sense out of it. This process usually involve the usage of a **disassembler**, a tool that converts machine instructions to their corresponding assembly representation. In this course, we will use the built-in command-line disassembler/debugger **GDB** \(see below\). Many disassemblers are much better than GDB, such as [IDA](https://www.hex-rays.com/products/ida/support/download_freeware.shtml), however, for what we need from it in the exercices, GDB will be sufficient.

In the context of this course, **Debugging** means analysing the binary application while it is running thanks to a _debugger_. A _debugger_ allows you to set _breakpoints_ in the debugged running application. A _breakpoint_ can be set on one or several instructions. Once the instruction with the breakpoint is reached and is about to be processed by the CPU, the application will pause the program. While being paused, the analyst can read and edit instructions, the memory and _registers_ \(see more about registers in chapter [memory](memory.md#registers)\). In this course, we will use **GDB** \(see below\).

**Compiling** is the process of transforming computer code written in one programming language \(the source language\) into another programming language \(the target language\). In the context of this course, this means transforming C code in a binary application format using machine language instructions \(i.e. compiling and linking\). The compiler used in this course is **GCC**.

## Operating System

For this course, we decided to use the standard **Ubuntu 32-bit Desktop** distribution. We could have found a much lighter Operating System \(OS\) which would have been sufficient for what we need, however, we thought this would be easier to install configure and maintain. Furthermore, Ubuntu is free and widely used, so there is a high chance you already encountered and used it, so you should feel already comfortable with it.

As mentioned in the [introduction](./), this course cover 32-bit only. Although it is possible to compile, run and debug x386 applications on 64-bit operating systems, using a 32-bit OS will reduce the dependencies and environment complexity.

### Download

Ubuntu is not available in 32-bit since 18.04, so we need to install the [16.04](http://releases.ubuntu.com/16.04/) version instead.

Download [Ubuntu 16.04.5 Desktop 32-bit](http://releases.ubuntu.com/16.04/ubuntu-16.04.5-desktop-i386.iso) \(ISO\)

### VM deployment

You can either run Linux natively on your hardware or in a virtual environment. If you have Linux already installed on your computer, you can skip this part.

In order to keep this course free, we recommend using [VirtualBox](https://www.virtualbox.org/) as your virtual environment. Once the ISO downloaded and VirtualBox installed, you can follow this [tutorial](https://medium.com/@tushar0618/install-ubuntu-16-04-lts-on-virtual-box-desktop-version-30dc6f1958d0). The given specs are sufficient, i.e.:

* 1024 Mb RAM
* 10 Gb Hard disk \(VDI - Dynamically allocated\)

{% hint style="info" %}
During the Ubuntu installation, make sure you select the right keyboard \(use “Detect keyboard layout”\). For the rest, you can choose whatever name, username, password, and language.
{% endhint %}

Once the VM deployed, you can remove all links from the launcher \(left panel\) and add “Terminal”. This will be the only tool you will need for this course. Now that you have your VM ready, the first thing to do is to update your Linux. For this, type the following command in youe Terminal:

```text
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install virtualbox-guest-additions-iso
```

Now, let’s install the VirtualBox Guest Addition. This will allow you to copy/paste from the host to the VM, which will come handy to copy the source code for instance. For this, you simply need to go in the VirtualBox menu “Devices” and select “Install Guest Additions CD image…”. Then, you just need to follow the instructions. At the end of the process, go in the VirtualBox menu “Machine” and select “Settings…”. In the tab “General” &gt; “Advanced”, set the “Shared Clipboard” value to “Bidirectional”. You now should be able to copy/paste from the VM to your host and vice-versa.

Finally, we want to disable **A**ddress **S**pace **L**ayout **R**andomization \(ASLR\) on our VM. **ASLR** is a security mechanism that makes it more complicate to exploit memory corruption vulnerabilities such as _buffer overflow_ or _format string_. When ASLR is activated on your operating system, the process responsible for loading the executable into memory will place  key data areas \(e.g. stack, heap and libraries\) in unpredictable location. We will see later in chapter buffer overflow the impact of ASLR.

ASLR can be configured with the file `/proc/sys/kernel/randomize_va_space`. When the file contains `0`, this means ASLR is disabled, when it contains `1` or `2`, ASLR is enabled.

```text
$ cat /proc/sys/kernel/ramdomize_va_space
2
```

You can use a text editor and change the content to `0` to disable ASLR, however, the value will be reset to `2` after the next boot. To change the configuration permanently, you can add the following line in the file `/etc/sysctl.conf`:

```text
kernel.randomize_va_space = 0
```

You will need sudo privileges to add that line:

```text
$ echo "kernel.randomize_va_space = 0" | sudo tee -a /etc/sysctl.conf
```

Once done, you will need to run the following command to take the changes into effect:

```text
$ sudo sysctl -p
```

## Tools

For this course, we only need 2 tools: **GDB** and **GCC**. Both tools are already pre-installed in Ubuntu Desktop.

### GCC

Let’s compile our first C code for this course. Use your favourite text editor and create the following `mul.c` file:

{% code title="mul.c" %}
```c
#include <stdio.h>

int add(int, int);
int mul(int, int);

int main()
{
    int four, three, result;

    four = 4;
    three = 3;

    result = mul(four, three);

    printf("%d x %d = %d\n", four, three, result);

    return 0;
}


int add(int a, int b)
{
    int first, second, result;

    first = a;
    second = b;

    result = first + second;

    return result;
}


int mul(int x, int y)
{
    int number, multiplier, result, i;

    number = x;
    multiplier = y;
    result = 0;
    i = 0;

    while (i < multiplier)
    {
        result = add(result, number);
        i = i + 1;
    }

    return result;
}
```
{% endcode %}

Learning C is beyond the scope of this course, so we assume you understand what this code does, but just to make sure we are on the same page: it multiplies `4` by `3` by using only the addition operation then it prints the result.

Now, in order to compile this code, you simply need to execute the following command:

```text
$ gcc mul.c -o mul
```

This will compile `mul.c` and create the binary application `mul`. Now, in order to run the application, you simply need to execute the following command:

```text
$ ./mul
```

When compiling vulnerable source code for buffer overflow examples, we will also need to remove the **stack protector** \(also known as **canary**\) added by default with GCC. The stack protector is a mechanism that verify if some specific memory areas haven't been altered during execution. To disable the stack protector, we need to use `-fno-stack-protector` as argument with GCC:

```text
$ gcc mul.c -o mul -fno-stack-protector
$ ./mul
```

{% hint style="info" %}
We will see later in chapter [buffer overflow](buffer-overflow.md) how the stack protector works and make it more complicate the exploitation of buffer overflow.
{% endhint %}

When exploiting buffer overflows, we will inject in the stack malicious instructions. By default, the stack is non-executable \(also known as NX\), which means the CPU will not executed instructions located in the stack. In oder to disable this security feature, we will use the option `-z execstack` when compiling with GCC:

```text
$ gcc mul.c -o mul -fno-stack-protector -z execstack
$ ./mul
```

By default, GCC will build a _symbol table_  \(.symtab\) in the final binary that contains the list of all functions name from in the application. This include `main`, `add` and `mul`, but also function not explicitly written by the developer that were added by the compiler such as `_start`.

The symbol table is really useful when reversing an application as function usually give a short description of what the function does. Furthermore, it allows to use the function name as alias instead of memory address in the debugger.

The symbol table can be listed using `readelf`:

```text
$ readelf -s ./mul

Symbol table '.dynsym' contains 7 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     [...]

Symbol table '.symtab' contains 65 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000238     0 SECTION LOCAL  DEFAULT    1
    [...]
    46: 0000000000000695    38 FUNC    GLOBAL DEFAULT   14 add
    [...]
    59: 000000000000064a    75 FUNC    GLOBAL DEFAULT   14 main
    60: 00000000000006bb    77 FUNC    GLOBAL DEFAULT   14 mul
```

### GDB

As mentioned earlier, `GDB` is a disasembler and a debugger. The main feature of a debugger is to set breakpoints and read memory areas and registers. Let’s debug our first application:

```text
$ gdb mul -q
```

{% hint style="info" %}
The option `-q` is to avoid printing introducery and copyright message.
{% endhint %}

Now, we want to disassemble the `main()` function to look at the listing of instructions and decide where to set a breakpoint. For this, we can use the command `disassemble`.

```text
(gdb) disassemble main
Dump of assembler code for function main:
   0x0804840b <+0>:    lea    0x4(%esp),%ecx
   0x0804840f <+4>:    and    $0xfffffff0,%esp
   0x08048412 <+7>:    pushl  -0x4(%ecx)
   0x08048415 <+10>:    push   %ebp
   0x08048416 <+11>:    mov    %esp,%ebp
   0x08048418 <+13>:    push   %ecx
   0x08048419 <+14>:    sub    $0x14,%esp
   0x0804841c <+17>:    movl   $0x4,-0x14(%ebp)
   0x08048423 <+24>:    movl   $0x3,-0x10(%ebp)
   0x0804842a <+31>:    sub    $0x8,%esp
   0x0804842d <+34>:    pushl  -0x10(%ebp)
   0x08048430 <+37>:    pushl  -0x14(%ebp)
   0x08048433 <+40>:    call   0x8048483 <mul>
   0x08048438 <+45>:    add    $0x10,%esp
   0x0804843b <+48>:    mov    %eax,-0xc(%ebp)
   0x0804843e <+51>:    pushl  -0xc(%ebp)
   0x08048441 <+54>:    pushl  -0x10(%ebp)
   0x08048444 <+57>:    pushl  -0x14(%ebp)
   0x08048447 <+60>:    push   $0x8048550
   0x0804844c <+65>:    call   0x80482e0 <printf@plt>
   0x08048451 <+70>:    add    $0x10,%esp
   0x08048454 <+73>:    mov    $0x0,%eax
   0x08048459 <+78>:    mov    -0x4(%ebp),%ecx
   0x0804845c <+81>:    leave  
   0x0804845d <+82>:    lea    -0x4(%ecx),%esp
   0x08048460 <+85>:    ret    
End of assembler dump.
```

When use with a single argument, the function surrounding that address is disassembled.

{% hint style="info" %}
Most of the time, you can use the command `help` to get more information about the usage of GDB. For instance, `help disassemble`.
{% endhint %}

If your resolution is not big enough, it is possible that `GDB` prints the following message:

```text
---Type <return> to continue, or q <return> to quit---
```

In this case, you simply need to type `ENTER` to see the rest of the disassembled code.

As explained in the [CPU](cpu.md) and [memory](memory.md) chapters, machine code instructions are basically a group of binary values that signify a specific instruction for the CPU. E.g. `b8 00 00 00 00` means moving `0x00000000` in the register `EAX`. However, it is usually easier for humans to read pseudo-English rather than hexadecimal value, therefore disassembler translates the binary values \(machine code\) in human-readable code \(assembly\). Here in this case, `b8 00 00 00 00` is translated as `mov $0x0, %eax` \(see instruction at the address `0x08048454`\). This representation is using the AT&T syntax. However, it exists different ways to represent assembly code, the most known one being “Intel”. In Intel syntax, `b8 00 00 00 00` is translated as `mov eax, 0x0`. Personally, I think the Intel syntax is easier to read than AT&T, therefore this course will be using Intel syntax. To change the syntax in GDB, you can run the following command:

```text
(gdb) set disassembly-flavor intel
```

The disassembly of `main` will now look like this:

```text
(gdb) disassemble main
Dump of assembler code for function main:
   0x0804840b <+0>:    lea    ecx,[esp+0x4]
   0x0804840f <+4>:    and    esp,0xfffffff0
   0x08048412 <+7>:    push   DWORD PTR [ecx-0x4]
   0x08048415 <+10>:    push   ebp
   0x08048416 <+11>:    mov    ebp,esp
   0x08048418 <+13>:    push   ecx
   0x08048419 <+14>:    sub    esp,0x14
   0x0804841c <+17>:    mov    DWORD PTR [ebp-0x14],0x4
   0x08048423 <+24>:    mov    DWORD PTR [ebp-0x10],0x3
   0x0804842a <+31>:    sub    esp,0x8
   0x0804842d <+34>:    push   DWORD PTR [ebp-0x10]
   0x08048430 <+37>:    push   DWORD PTR [ebp-0x14]
   0x08048433 <+40>:    call   0x8048483 <mul>
   0x08048438 <+45>:    add    esp,0x10
   0x0804843b <+48>:    mov    DWORD PTR [ebp-0xc],eax
   0x0804843e <+51>:    push   DWORD PTR [ebp-0xc]
   0x08048441 <+54>:    push   DWORD PTR [ebp-0x10]
   0x08048444 <+57>:    push   DWORD PTR [ebp-0x14]
   0x08048447 <+60>:    push   0x8048550
   0x0804844c <+65>:    call   0x80482e0 <printf@plt>
   0x08048451 <+70>:    add    esp,0x10
   0x08048454 <+73>:    mov    eax,0x0
   0x08048459 <+78>:    mov    ecx,DWORD PTR [ebp-0x4]
   0x0804845c <+81>:    leave  
   0x0804845d <+82>:    lea    esp,[ecx-0x4]
   0x08048460 <+85>:    ret    
End of assembler dump.
```

Now, let say you want to set a breakpoint at the `mov eax, 0x0` \(located at the address `0x08048454`\), you simply need to enter the following command:

```text
(gdb) break *0x08048454
Breakpoint 1 at 0x08048454
```

In order to reach this instruction, you need to `run` the application:

```text
(gdb) run
Starting program: /home/lab/mul
4 x 3 = 12

Breakpoint 1, 0x08048454 in main ()
```

As you can see, at this stage, the application already did the multiplication and printed the result. Once the breakpoint is reached, the application pauses. At this point, you can read \(and write\) memory, instructions and registers. To view the registers, you can run the command:

```text
(gdb) info registers
eax            0xb    11
ecx            0x7ffffff5    2147483637
edx            0xb7fbc870    -1208235920
ebx            0x0    0
esp            0xbfffef20    0xbfffef20
ebp            0xbfffef38    0xbfffef38
esi            0xb7fbb000    -1208242176
edi            0xb7fbb000    -1208242176
eip            0x8048454    0x8048454 <main+73>
eflags         0x282    [ SF IF ]
cs             0x73    115
ss             0x7b    123
ds             0x7b    123
es             0x7b    123
fs             0x0    0
gs             0x33    51
```

To read memory locations, including instructions \(since instruction are located in memory\), you can use the command `x` \(for e**X**amine\). The command `x` has the following format:

```text
x/nfu addr
```

* **n** _\(optional\)_ is the repeat count: the repeat count is a decimal integer; the default is 1. It specifies how much memory \(counting by units u\) to display. If a negative number is specified, memory is examined backward from addr.
* **f** _\(optional\)_ is the display format: the display format is one of the formats used by print \(‘_x_’, ‘_d_’, ‘_u_’, ‘_o_’, ‘_t_’, ‘_a_’, ‘_c_’, ‘_f_’, ‘_s_’\), and in addition ‘_i_’ \(for machine instructions\). The default is ‘_x_’ \(hexadecimal\) initially. The default changes each time you use either x or print.
* **u** _\(optional\)_ is the unit size:
  * _b_: Bytes
  * _h_: Halfwords \(two bytes\)
  * _w_: Words \(four bytes\) \(default\)
  * _g_: Giant words \(eight bytes\)
* _addr_ is the starting display address: _addr_ is the address where you want GDB to begin displaying memory. The expression need not have a pointer value

Source and more info here: [onlinedocs](https://sourceware.org/gdb/onlinedocs/gdb/Memory.html).

We now have reached the breakpoint located at `0x8048454`. So if we use the command `x/i 0x8048454`, we should see the instruction `mov eax, 0x0`:

```text
(gdb) x/i 0x8048454
=> 0x8048454 <main+73>:    mov    eax,0x0
```

Now, if we want to see the opcode instead of the instruction, we have to change the format from `i`\(machine instruction\) to `x` \(hexadecimal representation\). Furthermore, it would be more readable to print the opcode byte by byte, instead of a full word \(4 bytes\), which is the default unit size, so we will use the unit size `b`. Lastly, the instruction `mov eax, 0x0` is 5 bytes long, therefore, the repeat counter should be set to `5`:

```text
(gdb) x/5xb 0x8048454
0x8048454 <main+73>:    0xb8    0x00    0x00    0x00    0x00
```

{% hint style="info" %}
Unlike ARM, x386 instruction length is variable, so we first need to read the first byte to know how long will be the instruction in opcode. For instance, `0xb8` means “move the next 4 bytes \(word\) in EAX”. So the entire instruction is 1 byte \(`0xb8`\) + 4 bytes \(value to copy in EAX\) = 5 bytes. Whilst `0x89` means moving the value of a register into another register. The following byte will indicate which is the source register and the destination register. For instance `0xc8` indicates that the source is ECX and destination is EAX”. If we put the pieces together, `89 c8` means `mov eax, ecx`. So the entire instruction is 1 byte \(`0x89`\) + 1 byte \(source/destination register\) = 2 bytes.
{% endhint %}

Now that we know how to read memory, let’s navigate through the instructions. The four most important commands to remember are `run`, `continue`, `nexti` and `stepi`:

* `run` will start the application. It is recommended to set breakpoint before executing the command `run` otherwise, it will just execute the application and you might not have the time to pause it to investigate the memory, registers, and instructions. You can add arguments after the command. For instance, if our application `mul` was expecting a number as argument, we could have executed the command `run 123`.
* `continue` resume the program execution. So once you reached a breakpoint and you want to resume the execution until the next breakpoint or the end of the program, you can use the command `continue`.
* `nexti`, short for _next instruction_, execute one machine instruction, but if it is a function call, proceed until the function returns.
* `stepi`, short for _step instruction_, execute one machine instruction, then stop and return to the debugger.

The difference between `nexti` and `stepi` might not be clear, so here is an example. Let’s first set two additional breakpoint: one at the `call 0x8048461 <mul>` and one at the `call 0x80482e0 <printf@plt>`then restart the app:

```text
(gdb) break *0x08048433
Breakpoint 2 at 0x8048433
(gdb) break *0x0804844c
Breakpoint 3 at 0x804844c
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/lab/mul

Breakpoint 2, 0x08048433 in main ()
```

{% hint style="info" %}
Since the application was already running, you should have a warning asking you whether you really want to start from the beginning. You simply need to type y to confirm.
{% endhint %}

Now we reached the breakpoint at `call 0x8048461 <mul>`. If you want to, you can execute the command `disassemble main` again to see where we are. The `call` instruction will redirect the execution flow to the function `mul`. If you want the function to execute and go straight to the next instruction \(i.e. `add esp,0x10`\) you can use the command `nexti`. But if you want to follow the execution flow and jump in the function `mul`, you can use the command `stepi`. Let’s use the later:

```text
(gdb) stepi
0x08048483 in mul ()
```

As you can see, we now are in the function `mul` located at `0x08048483`. The debugger executed one single instruction, then automatically paused the program without the need to set another breakpoint.

Now let’s have a look at the next 4 instructions. Instead of retyping the address where are, you could use the register _EIP_, which is the instruction pointer \(see [memory](memory.md) chapter\):

```text
(gdb) x/4i $eip
=> 0x8048483 <mul>:    push   ebp
   0x8048484 <mul+1>:    mov    ebp,esp
   0x8048486 <mul+3>:    sub    esp,0x10
   0x8048489 <mul+6>:    mov    eax,DWORD PTR [ebp+0x8]
```

We can now use either `stepi` or `nexti` to execute the current instruction and move to the next one:

```text
(gdb) nexti
0x08048484 in mul ()
(gdb) nexti
0x08048486 in mul ()
(gdb) nexti
0x08048489 in mul ()
```

Now, let’s go to the next breakpoint. For this, we can use the command `continue`, so that the debugger will continue the program and until it reaches a breakpoint or the application close.

```text
(gdb) continue
Continuing.

Breakpoint 3, 0x0804844c in main ()
```

Now we have reached the instruction `call 0x80482e0 <printf@plt>`. The function `printf` is a native function from `libc`, so we don’t want to debug/reverse that function, we know already what it does. So we just want to move to the instruction after the call. For this, we simply need to execute the command `nexti`:

```text
(gdb) nexti
4 x 3 = 12
0x08048451 in main ()
```

As you can see, `printf` has been executed and printed in the terminal `4 x 3 = 12`.

Now, let’s `continue` the execution until the last breakpoint at the instruction `mov eax,0x0`, then print the registers to see the value of _EAX_ before modification, then execute the instruction and print again the registers so that we can see the new value of _EAX_:

```text
(gdb) continue
Continuing.

Breakpoint 1, 0x08048454 in main ()
(gdb) info registers
eax            0xb    11
ecx            0x7ffffff5    2147483637
edx            0xb7fbc870    -1208235920
ebx            0x0    0
esp            0xbfffef20    0xbfffef20
ebp            0xbfffef38    0xbfffef38
esi            0xb7fbb000    -1208242176
edi            0xb7fbb000    -1208242176
eip            0x8048454    0x8048454 <main+73>
eflags         0x282    [ SF IF ]
cs             0x73    115
ss             0x7b    123
ds             0x7b    123
es             0x7b    123
fs             0x0    0
gs             0x33    51
(gdb) nexti
0x08048459 in main ()
(gdb) info registers
eax            0x0    0
ecx            0x7ffffff5    2147483637
edx            0xb7fbc870    -1208235920
ebx            0x0    0
esp            0xbfffef20    0xbfffef20
ebp            0xbfffef38    0xbfffef38
esi            0xb7fbb000    -1208242176
edi            0xb7fbb000    -1208242176
eip            0x8048459    0x8048459 <main+78>
eflags         0x282    [ SF IF ]
cs             0x73    115
ss             0x7b    123
ds             0x7b    123
es             0x7b    123
fs             0x0    0
gs             0x33    51
```

{% hint style="info" %}
You can print a single register by using `print $reg`, for instance `print $eax`.
{% endhint %}

Finally, let’s have a look at the breakpoints we have set with the command `info breakpoints`:

```text
(gdb) info breakpoints
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   0x08048454 <main+73>
    breakpoint already hit 1 time
2       breakpoint     keep y   0x08048433 <main+40>
    breakpoint already hit 1 time
3       breakpoint     keep y   0x0804844c <main+65>
    breakpoint already hit 1 time
```

Let say you want to delete the breakpoint at the instruction `mov eax,0x0`. You simply need to execute the command `delete` together with the breakpoint index number \(`Num`\). In this case, the index is `1`\(the instruction `mov eax,0x0` is located at the address `0x08048454`\):

```text
(gdb) delete 1
(gdb) info breakpoints
Num     Type           Disp Enb Address    What
2       breakpoint     keep y   0x08048433 <main+40>
    breakpoint already hit 1 time
3       breakpoint     keep y   0x0804844c <main+65>
    breakpoint already hit 1 time
```

That’s enough commands for this chapter, I would recommend to play around the application, setting breakpoint, read memory/registers, navigate through the instructions. Once done, you can quit `GDB`with the following command:

```text
(gdb) quit
A debugging session is active.

    Inferior 1 [process 2674] will be killed.

Quit anyway? (y or n) y
```

In the next exercises, we will go a bit deeper with the usage of `GDB`.

## References

* [IDA](https://www.hex-rays.com/products/ida/support/download_freeware.shtml): https://www.hex-rays.com/products/ida/support/download\_freeware.shtml
* [16.04](http://releases.ubuntu.com/16.04/): http://releases.ubuntu.com/16.04/
* [Ubuntu 16.04.5 Desktop 32-bit](http://releases.ubuntu.com/16.04/ubuntu-16.04.5-desktop-i386.iso): http://releases.ubuntu.com/16.04/ubuntu-16.04.5-desktop-i386.iso
* [VirtualBox](https://www.virtualbox.org/): https://www.virtualbox.org
* [tutorial](https://medium.com/@tushar0618/install-ubuntu-16-04-lts-on-virtual-box-desktop-version-30dc6f1958d0): https://medium.com/@tushar0618/install-ubuntu-16-04-lts-on-virtual-box-desktop-version-30dc6f1958d0
* [onlinedocs](https://sourceware.org/gdb/onlinedocs/gdb/Memory.html): https://sourceware.org/gdb/onlinedocs/gdb/Memory.html

