# Assembly

Assembly is a vast topic. We could spend hours, days, month and we still would have plenty to say about. Nevertheless, this is not a reverse engineering course and I’m myself not particularly fluent in assembly, so this course will cover the bare minimum to understand and exploit the vulnerabilities listed for this course, i.e. buffer overflow, format string, use after free and integer over/underflow.

As seen in the [CPU](central-processing-unit.md) chapter, instructions are sent to the CPU in the form of binary values of one or multiple bytes. Each binary instruction correspond to one assembly instruction. It is therefore possible to translate binary instructions directly into assembly instructions and the other way round. There is a one-to-one relation between binary instructions sent to CPU and assembly instructions, which is not the case for the relation between C and binary instruction. For instance, this single code line `c = 4 + 2;` in C can be translated in several ways and it will be composed of more than one binary instruction \(CPU operation\).

{% hint style="info" %}
The translation from binary instructions to its assembly representation is called _to disassemble_, while the other way, i.e. the translation from assembly instructions to their binary instructions is called _to assemble_.
{% endhint %}

Programs can be developped entirely in assembly. However, if you simply translate assembly instructions to their corresponding binary instructions, the final binary file won't be executable. For this, you need to build a [ELF file](memory.md#elf-pe-file), embed the translated instructions together with _initialized data_ and _global variables_, map the data, build the import table, etc. This is the role of the _compiler_ and _linker_ to do this, but this is another story. In this course we don't want to learn assembly to build application, instead, we want to learn how to read and understand compiled code, which is different exercice and requires slightly different knowledge and skills in assembly.

## Hello world!

Let see how our `hello-world.c` application has been translated by the compiler:

{% code-tabs %}
{% code-tabs-item title="hello-world.c" %}
```c
#include <stdio.h>

void main(int argc, char** argv)
{
    printf("Hello World!");
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```text
$ gcc hello-world.c -o hello
$ gdb -q hello
Reading symbols from hello...(no debugging symbols found)...done.
(gdb) set disassembly-flavor intel
(gdb) disassemble main
Dump of assembler code for function main:
   0x0804840b <+0>:     lea    ecx,[esp+0x4]
   0x0804840f <+4>:     and    esp,0xfffffff0
   0x08048412 <+7>:     push   DWORD PTR [ecx-0x4]
   0x08048415 <+10>:    push   ebp
   0x08048416 <+11>:    mov    ebp,esp
   0x08048418 <+13>:    push   ecx
   0x08048419 <+14>:    sub    esp,0x4
   0x0804841c <+17>:    sub    esp,0xc
   0x0804841f <+20>:    push   0x80484c0
   0x08048424 <+25>:    call   0x80482e0 <printf@plt>
   0x08048429 <+30>:    add    esp,0x10
   0x0804842c <+33>:    nop
   0x0804842d <+34>:    mov    ecx,DWORD PTR [ebp-0x4]
   0x08048430 <+37>:    leave  
   0x08048431 <+38>:    lea    esp,[ecx-0x4]
   0x08048434 <+41>:    ret    
End of assembler dump.
```

Here, we only disassembled the instructions from the function `main` and we already ended up with 16 assembly instructions. For now, no need to understand why the compiler translated the code like this, just try to understand what each line does independently. Let's dive into it!

#### Line 1: lea ecx,\[esp+0x4\]

```text
   0x0804840b <+0>:     lea    ecx,[esp+0x4]
```

Here is how to read this first line of assembly:

* `0x0804840b` is the address in memory where this binary instruction is located.
* `<+0>` is the offset – in byte – from the beginning of the function `main`. Since we are at the first instruction of `main`, the offset is `0`. Note that on the second line, the offset is `4`, which means the first instruction takes 4 bytes in memory, while on the thrid line, the offset is `7`, which means the second instruction takes 3 bytes in memory \(`7`-`4`\).
* `lea ecx,[esp+0x4]` is the assembly instruction. The assembly syntax start with the instruction followed by zero, one or two operands. Operands are seperated with a coma \(`,`\).

{% hint style="info" %}
This course is using the intel syntax, which [differs](http://web.mit.edu/rhel-doc/3/rhel-as-en-3/i386-syntax.html) slightly from the AT&T syntax.
{% endhint %}

`lea` is the _load in effective address_ instruction. What it does is basically calculating what is inside the bracket of the second operand and write the result \(4 bytes\) in the first operand. So the instruction at `0x0804840b` is reading the content of the register `esp`, add 4, and store the result in the register `ecx`.

{% hint style="info" %}
If you remember from chapter memory, the register `esp` is pointing to the top of the stack.
{% endhint %}

```text
(gdb) break *0x0804840b
Breakpoint 1 at 0x804840b
(gdb) run
Starting program: /home/lab/hello 

Breakpoint 1, 0x0804840b in main ()
(gdb) info registers esp
esp            0xbfffef4c	0xbfffef4c
```

When the instruction is executed, the register `esp` contains the value `0xbfffef4c`. So after execution, `ecx` stores `0xbfffef50` \(`0xbfffef4c` + `0x04`\).

Let's verify it:

```text
(gdb) info registers ecx
ecx            0xdff71327	-537455833
(gdb) nexti
0x0804840f in main ()
(gdb) info registers ecx
ecx            0xbfffef50	-1073746096
```

So, all went as expected. But why copying that value in `ecx`? What is even stored at `[esp+0x4]`? 

First of all, remember that the stack is "growing" **downward**, which means if you add new content to the stack, the stack pointer, which always point to the top of the stack, will **decrease** \(see chapter [memory](memory.md#stack)\). So `[esp+0x4]` is pointing to a variable inside the stack at an offset of 4 bytes from the top of it. As seen in chapter [Memory](memory.md) - [Stack](memory.md#stack), functoin arguments are \(usually\) pushed to the stack prior to call the function. When the function is called, the _return address_ is pushed to the stack \(right above the arguments\).

At this point, when the CPU reached the first instruction from the function `main`, the stack looks like this:

![Stack at the beginning of the main function](.gitbook/assets/stack-main.gif)

{% hint style="info" %}
Note that the data is stored in little-endian, which means if the value `0xb7e21637` is stored at the address `0xbfffef4c`, it actually means `0x37` is stored at 0xbfffef4c, `0xef` is stored at `0xbfffef4d`, `0xff` is stored at `0xbfffef4e` and `0xbf` is stored at `0xbfffef4f`.
{% endhint %}

This means `[esp]` point to the _return address_ `0xb7e21637` and `[esp+0x4]` points to `argc`,  the first argument of main. So, even if we don't use it in our C code, our compiled program has an instruction that saves the address where the first argument is located into the `ecx` registre. I don't know why, but it doesn't matter for now, let's move to the next instruction.

#### Line 2: and esp,0xfffffff0

`and` is doing exactly what the name implies: it executes the logic _AND_ operation with the two operands. The result \(4 bytes\) is then stored in the first operand. Furthermore, the following flags \(see chapter [memory](memory.md#flags-register)\) are updated after the operation:

* `CF`: carry flag is cleared
* `OF`: overflow flag is cleared
* `PF`: parity flag is set according to the result
* `ZF`: zero flag is set according to the result
* `SF`: sign flag is set according to the result
* `AF`: state of ajust flag after AND opertation is undefined

```text
(gdb) info registers esp
esp            0xbfffef4c	0xbfffef4c
(gdb) info registers eflags
eflags         0x296	[ PF AF SF IF ]
(gdb) nexti
0x08048412 in main ()
(gdb) info registers esp
esp            0xbfffef40	0xbfffef40
(gdb) info registers eflags
eflags         0x282	[ SF IF ]
```

In the first four lines of this GDB commad output, we read the content of `esp` and the `eflag` register, then we execute the `and` instruction, and finally, in the four last lines, we read again the content of `esp` and the `eflag` registers to see the changes done by the AND operation.

Once the instruction executed, the stack looks like this:

![](.gitbook/assets/stack-and.png)

{% hint style="info" %}
AND operation on the `esp` register are often done at the beginning of the `main` function to re-align the stack address, i.e. be a mutliple of `0x10` \(16\) \[[1](https://stackoverflow.com/a/28475252)\]
{% endhint %}

#### Line 3: push DWORD PTR \[ecx-0x4\]

The `push` instruction adds the operand value onto the top of the stack and updates `esp` accordingly \(decrement by 4\). The operand is `DWORD PTR [ecx-0x4]`:

* `ecx-0x4`: Takes the value stored in `ecx` and subtract `0x4` \(4 in decimal\).
* `PTR [` `]`: Consider the value calculated inside the bracket as a memory address and the `push` instruction is actually storing into the stack the value pointed by this address \(instead of the address itself\). 
* `DWORD`: Stands for _D_ouble _WORD_ \(4 bytes\). This means the pointed data should be considered as a 4 bytes variable.

`ecx` contains the memory address where `argc` is stored in the stack \(see [line 1](assembly.md#line-1-lea-ecx-esp-0-x4)\). Since addresses are 4-bytes long, subtracting four is basically looking at the variable stored before `argc`, which in this case is the _return address_ `0xb7e21637`. So, the `push` instruction will add to the stack a copy of the _return address_:

![](.gitbook/assets/stack-push-ret.png)

#### Line 4: push ebp

When used without bracket, the instruction `push` with a register as operand simply pushes \(add\) on top of the stack the value stored in the register and update `esp` accordingly \(decrement by 4\). 

The register `ebp` is used as a **b**ase **p**ointer for the stack frame \(see chapter [Memory](memory.md) - [Stack](memory.md#stack)\).

![](.gitbook/assets/stack-push-ebp.png)

#### Line 5: mov ebp,esp

The `mov` instruction is one of the most used instruction. When the operands are two registers, it copies the content of the second operand and overwrite the content of the first operand with it. In this case, when the CPU reaches that instruction, the stack pointer \(`esp`\) contains `0xbfffef38`. Once the instruction executed, the base pointer \(`ebp`\) will also contain `0xbfffef38`.

```text
(gdb) info registers esp
esp            0xbfffef38	0xbfffef38
(gdb) info registers ebp
ebp            0x0	0x0
(gdb) nexti
0x08048418 in main ()
(gdb) info registers esp
esp            0xbfffef38	0xbfffef38
(gdb) info registers ebp
ebp            0xbfffef38	0xbfffef38
```

The registers `ebp` and `esp` are now both pointing to the top of the stack.

#### Line 6: push ecx

We've seen the `push` instruction twice already, so you should know what it does by now, i.e. it will pushes the content of the register `ecx` on the stack. If you remember, in [line 1](assembly.md#line-1-lea-ecx-esp-0-x4), `ecx` contains the stack address where the the first argument of main \(`argc`\) is saved.

![main stack after line 6](.gitbook/assets/main-stack-1.png)

#### Line 7: sub esp,0x4

`sub` stands for _subtracting_. It takes the first operand and subtract the second one to it. The result is than saved in the first operand. This instruction could be basically interpreted as `esp = esp - 0x4`.

```text
(gdb) info registers esp
esp            0xbfffef34	0xbfffef34
(gdb) nexti
0x0804841c in main ()
(gdb) info registers esp
esp            0xbfffef30	0xbfffef30
```

{% hint style="info" %}
Subtracting from `esp` is basically allocating memory in the stack for local variable\(s\). However, bear in mind that the newly allocated memory memory space is not initialized, so this memory area has unknown data from previous stack usage.
{% endhint %}

![main stack after line 7](.gitbook/assets/main-stack-2.png)

#### Line 8: sub esp,0xc

Surprisingly enough, the next instruction also subtract a static number to `esp`. I don't why this is done over two instruction instead of, for instance, this single equivalent instruction: `sub esp, 0x10`.

```text
(gdb) info registers esp
esp            0xbfffef30	0xbfffef30
(gdb) nexti
0x0804841f in main ()
(gdb) info registers esp
esp            0xbfffef24	0xbfffef24
```

![main stack after line 8](.gitbook/assets/main-stack-3.png)

#### Line 9: push 0x80484c0 

When `push` is used with a direct value instead of a register, the value iself is pushed to the stack. But what is `0x80484c0`? Why is it pushed to the stack? Let's have a look at the section mapping to see from which section this address belong to:

```text
(gdb) maintenance info sections 
Exec file:
    `/home/lab/hello', file type elf32-i386.
 ...
 [12]     0x8048300->0x8048308 at 0x00000300: .plt.got ALLOC LOAD READONLY CODE HAS_CONTENTS
 [13]     0x8048310->0x80484a2 at 0x00000310: .text ALLOC LOAD READONLY CODE HAS_CONTENTS
 [14]     0x80484a4->0x80484b8 at 0x000004a4: .fini ALLOC LOAD READONLY CODE HAS_CONTENTS
 [15]     0x80484b8->0x80484cd at 0x000004b8: .rodata ALLOC LOAD READONLY DATA HAS_CONTENTS
 ...
 [24]     0x804a014->0x804a01c at 0x00001014: .data ALLOC LOAD DATA HAS_CONTENTS
 [25]     0x804a01c->0x804a020 at 0x0000101c: .bss ALLOC
 [26]     0x0000->0x0035 at 0x0000101c: .comment READONLY HAS_CONTENTS
```

The address `0x80484c0` is part of the `.rodata` section, which contains read-only initialized static variables \[[1](https://en.wikipedia.org/wiki/Data_segment)\]. Let's have a look at the content pointed by this variable:

```text
(gdb) x/x 0x80484c0
0x80484c0:	0x6c6c6548
```

If you some experience with reversing, you should quickly noticed that each bytes of this 32 bits value are in the range between `0x20` and `0x7d`, which are all printable ASCII characters. So `0x80484c0` might actually be pointing to a ASCII string:

```text
(gdb) x/s 0x80484c0
0x80484c0:	"Hello World!"
```

So `0x80484c0` is actually a pointer to the string "Hello World!" stored in the `.rodata` section. The instruction in line 9 is pushing that address to the stack.

![main stack after line 9](.gitbook/assets/main-stack-4.png)

#### Line 10: call 0x80482e0 printf@plt 

The `call` instruction redirects the execution flow by changing the instruction pointer \(`EIP`\) with the address mentioned as operand. Usually, this operand is an address to a function. However, before it does that, it `push` to the stack the address of the following instruction. This address saved in the stack so that later, whenever the function if done, the program knows where to return.

{% hint style="info" %}
The instruction is actually `call 0x80482e0`, but GDB ❤ is kind enough to let us know that `0x80482e0` is actually the address of `printf` in the `plt` table.
{% endhint %}

Let see the instruction `call` in action:

```text
(gdb) x/3i $eip
=> 0x8048424 <main+25>:	call   0x80482e0 <printf@plt>
   0x8048429 <main+30>:	add    esp,0x10
   0x804842c <main+33>:	nop
(gdb) x/3x $esp
0xbfffef20:	0x080484c0	0xbfffefe4	0xbfffefec
(gdb) stepi 
0x080482e0 in printf@plt ()
(gdb) x/3i $eip
=> 0x80482e0 <printf@plt>:	jmp    DWORD PTR ds:0x804a00c
   0x80482e6 <printf@plt+6>:	push   0x0
   0x80482eb <printf@plt+11>:	jmp    0x80482d0
(gdb) x/3x $esp
0xbfffef1c:	0x08048429	0x080484c0	0xbfffefe4
```

In lines 1-4, I print the current and the next two instructions. We can see that the instruction pointer is not pointing to the `call 0x80482e0`. In line 5-6, I print the first 3 words on top of the stack. As seen in the [previous instruction](assembly.md#line-9-push-0x-80484-c0), the last `push` to the stack was with the pointer to "Hello World!", i.e. `0x080484c0`. We then execute the `call` instruction in line 7 with the GDB command `stepi` and **not** `nexti` \(see the difference in chater [lab](lab-environment.md#gdb)\). One the command executed, we can see in lines 9-12 that the instruction pointer is now pointer to `0x80482e0`, the first operand of the executed `call` instruction. Finally, we see in lines 13-14 that the address of the instruction located right \(i.e. `0x08048429`\) after the call instruction has been pushed to the stack.

![Stack at the beginning of printf](.gitbook/assets/main-stack-5.png)

Here, we will not follow the `printf@plt` execution flow, instead we will tell GDB to continue the execution until we get back to `main` function:

```text
(gdb) finish
Run till exit from #0  0x080482e0 in printf@plt ()
0x08048429 in main ()
(gdb) x/3i $eip
=> 0x8048429 <main+30>:	add    esp,0x10
   0x804842c <main+33>:	nop
   0x804842d <main+34>:	mov    ecx,DWORD PTR [ebp-0x4]
```

We are now back in the `main` function, at the instruction located right after the `call 0x80482e0`.

```text
(gdb) x/3x $esp
0xbfffef20:	0x080484c0	0xbfffefe4	0xbfffefec
```

![main stack after the call printf](.gitbook/assets/main-stack-4.png)

#### Line 11: add esp,0x10 

As its name implies, the instruction `add` operates an addition with the first operand and the second operand. The result is then saved in the first operand.

```text
(gdb) info registers esp
esp            0xbfffef20	0xbfffef20
(gdb) nexti
0x0804842c in main ()
(gdb) info registers esp
esp            0xbfffef30	0xbfffef30
```

{% hint style="info" %}
Adding to `esp` is basically cleaning the stack and remove memory local variable\(s\) from the stack.
{% endhint %}

![main stack after line 11](.gitbook/assets/main-stack-2.png)

#### Line 12: nop 

The `nop` is an interesting instruction since it does absolutly nothing. `nop` stands for **N**o **OP**eration. When the CPU reach that instruction, it won't do anything and just move to the next one. The `nop` instruction is stored in memory as the machine instruction  `0x90`, one single byte. So if we want to be very accurate, the `nop` instruction is actually doing something:  incrementing the instruction pointer by one:

```text
(gdb) info registers eip
eip            0x804842c	0x804842c <main+33>
(gdb) nexti
0x0804842d in main ()
(gdb) info registers eip
eip            0x804842d	0x804842d <main+34>
```

#### Line 13: mov ecx,DWORD PTR \[ebp-0x4\] 

We've seen it already, the mov instruction is copying the content of the second operand into the first operant. The first operand \(moving destination\) is pretty clear, i.e. the register `ecx`. However, the first operand \(moving source\) might need a few explanation:

* `ebp-0x4`: Takes the value stored in `ebp` and subtract `0x4` \(4 in decimal\).
* `PTR [` `]`: Consider the value calculated inside the bracket as a memory address and the `mov` instruction is actually copying the value pointed by this address \(instead of the address itself\). 
* `DWORD`: Stands for _D_ouble _WORD_ \(4 bytes\). This means the pointed data should be considered as a 4 bytes variable.

The register `ebp` was set in [line 5](assembly.md#line-5-mov-ebp-esp) and is used as the base pointer of the stack frame for the function `main`. When accessing data with a **negative** offset to `ebp`, we usually try to access local variable of the function to which the stack frame belong to. While if we try to access data with **positive** offset to `ebp`, this will most likely by argument to the function to which the stack frame belong to.

So here, `ebp-0x4` is the address where the register `ecx` was saved earlier in [line 6](assembly.md#line-6-push-ecx). So basically, in line 6, we saved `ecx` in the stack, then we do something \(between line 7 and 13\) which migh alter `ecx`, and later in line 14, we restore the initial value saved in line 6 back in `ecx`.

```text
(gdb) info registers ecx
ecx            0x804b014	134524948
(gdb) x/x $ebp-4
0xbfffef34:	0xbfffef50
(gdb) nexti
0x08048430 in main ()
(gdb) info registers ecx
ecx            0xbfffef50	-1073746096

```

#### Line 14: leave

The instruction `leave` is meant to reset the stack as it was at the beginning of the function. For this, is copies the content of the base pointer \(`ebp`\) in the stack pointer \(`esp`\), which releases the stack space allocated to the stack frame \(e.g. local variables\), then it copies the the variable on top of the stack in the base pointer \(`ebp`\). It is basically the equivalent of the following instructions:

```text
mov esp, ebp
pop ebp
```

{% hint style="info" %}
We haven't seen the `pop` instruction yet. `pop` is kind of the opposite of `push`. It takes the 4-bytes value on top of the stack and stores it in the first operant. Then it updates the the stack pointer accordingly, i.e. subtract 4 from `esp`.
{% endhint %}

#### Line 15: lea esp,\[ecx-0x4\]

The instruction `lea` was the [first one](assembly.md#line-1-lea-ecx-esp-0-x4) we've seen. It actually works a bit like `mov`, except that it has a different syntax. When using bracket for the second operand, `lea` calculates what is inside and copies the result in the first operand; while `mov` calculates what is inside the bracket and copies the value pointed by the resulted address in the first operand.

So here, we take the value stored in `ecx`, subtact `0x4` and save the result in `esp`:

```text
(gdb) info registers ecx
ecx            0xbfffef50	-1073746096
(gdb) info registers esp
esp            0xbfffef3c	0xbfffef3c
(gdb) nexti
0x08048434 in main ()
(gdb) info registers esp
esp            0xbfffef4c	0xbfffef4c
```

![main stack after line 14](.gitbook/assets/main-stack-6.png)

At this point of time, `ecx` is pointing to the first argument of the function `main`. The address just above \(i.e. -0x4\) is actually the _return address_ of the main's callee function, i.e. `__libc_stat_main`.

#### Line 16: ret

Finally, the last line: the instruction `ret`! That last instruction is the equivalent of `pop eip`: it redirects the execution flow to the address stored at the top of the stack. This is meant to return to the callee, right after the `call` instruction.

{% hint style="info" %}
The instruction pointer `eip` cannot be used as operand. So we cannot use `lea`, `mov`, `pop`, `push` with that register. That's why we have special instructions meant to manipulate eip \(and thus the execution flow\), like `call`, `jmp` or `ret`.
{% endhint %}

Now you understand why the `call` instruction is pushing to the stack the address of the next instruction \(also known as **return address**\): so that at the end of the called function, `ret` can be used to come back where it was initially. But that means that before we execute `ret`, we need to clean the stack and make sure `esp` is pointing to that **return address** previously pushed by `call`.

```text
(gdb) x/3i $eip
=> 0x8048434 <main+41>:	ret    
   0x8048435:	xchg   ax,ax
   0x8048437:	xchg   ax,ax
(gdb) x/3x $esp
0xbfffef4c:	0xb7e21637	0x00000001	0xbfffefe4
(gdb) nexti
0xb7e21637 in __libc_start_main [...]
(gdb) x/3i $eip
=> 0xb7e21637 <__libc_start_main+247>:	add    esp,0x10
   0xb7e2163a <__libc_start_main+250>:	sub    esp,0xc
   0xb7e2163d <__libc_start_main+253>:	push   eax
(gdb) x/3x $esp
0xbfffef50:	0x00000001	0xbfffefe4	0xbfffefec
```

In the four first lines, we list the current and next instructions. The two instructions after `ret` are `xchg ax,ax`. The function is usually finished after ret, what follows in memory is usually another fucntion or some filler between functions. 

{% hint style="info" %}
The `xchg` instruction exchange the content of the two operands. Here in this case, exchanging the content of the register `ax` with the regsiter `ax` does nothing. Sometime, the compiler/linker uses `chxg ax,ax`, instead of the`nop` instruction  because it takes 2 bytes in memory \(i.e. 0x66 0x90\) and not one byte like the `nop` instruction, which makes it easier for the memory alingment \[[2](https://stackoverflow.com/a/2136065)\]. `xchg ax,ax` is often used as filler.
{% endhint %}

So the current instruction is `ret` and the top of the stack contains the value `0xb7e21637` as seen in lines 5-6. Once the instruction `ret` executed, the execution flow got redirected back in the funciton `__libc_stat_main` \[[3](http://refspecs.linuxbase.org/LSB_3.1.0/LSB-generic/LSB-generic/baselib---libc-start-main-.html)\] at the address `0xb7e21637`. And when we look at the stack in lines 13-14, the address has been popped out and the top of the stack is now pointing at the next variable.

{% hint style="info" %}
The function `__libc_stat_main` is the initialization routine that starts the `main` function.
{% endhint %}

And voila, this was was the assembly translation for the `main` function of _hello-world.c_ by gcc 5.4.0! I hope this wasn't too painful for you to follow.

#### Wrap up

Reading line by line assembly is rather easy. The main complexity lies in building a comprehensive understanding of what the function \(or pieces of function\) does: putting all the lines together to make sense out of it and understanding what the developer intended to it.

Most of lines we've seen doesn't make too much sense or seems useless. To be honest, the function `main` could have been simplified with the following 4-lines:

```text
push   0x80484c0
call   0x80482e0
add    esp, 0x4
ret
```

But instead, we have all those noisy useless lines. Why? The main reason is optimisation, e.g. reduce compilation time, execution time, output size, etc. Compilers are incredibly complex and well written. So what seems to be stupid and/or useless lines are usually meant to be like that.

Those extra lines are actually what you will struggle a lot when reversing applications. Through experience, you will learn to find naturally the appropriate depth of understanding in the assembly code, i.e. ignoring line\(s\) because considered as useless to understand \(in the context of your task\) or on the contrary taking it/them into consideration.

## Comments

Comments in assembly start with a semi-colon \(`;`\). Comments are not multi-lines, so if you want to have comments on consecutive lines, you will simply need to use a semi-colon at each line. Comments can be placed on empty lines or behind an instruction:

```text
push   0x80484c0  ; Pointer to the string "Hello World!"
call   0x80482e0  ; Call printf
add    esp,0x4    ; Remove the pushed argument (i.e. "Hello World!")
; The return address is now at the top of the stack
ret
```

## Variables

There are three main areas where variables are located. Static/Global variables are located in a fix memory location \(usually in the _.data_ or _BSS_ segment\), arguments and local variables of functions are located in the stack, and finally, dynamically allocated variables are located in the heap.

Unlike in C, compiled code \(without debug information\) do not have alias for variables. If you want to read or manipulate a variable, you need to provide the address where the variable is located \(or an offset that points to that memory location\).

### Type verification

Variables in assembly don't really have a type. Address point to binary data. The way you want to manipulate that data is entirely up to you. You could for instance execute a multiplication operation between two characters from a string. 

One thing the CPU needs to know though is on how many bytes the instruction operates: **BYTE**, **WORD** or **D**ouble **WORD**.

```text
mov al,  BYTE PTR [esp] ; Copies the  4-bit value at the top of the stack in  AL
mov ax,  WORD PTR [esp] ; Copies the 16-bit value at the top of the stack in  AX
mov eax,DWORD PTR [esp] ; Copies the 32-bit value at the top of the stack in EAX
```

### Allocating memory

Function variables are usually located in the stack. The size in memory depends on the type of the variable when declared in C. Whenever we enter a function, we usually have the _prolog_, which is preparing the stack frame, then we have the memory allocation, which basically subtract the total size of all function memory to the stack pointer.

{% code-tabs %}
{% code-tabs-item title="add.c" %}
```c
#include <stdio.h>

void add(int, int); // Function prototype

void main(int argc, char** argv)
{
    add(16, 26);
}

void add(int a, int b)
{
    int x, y, sum;
    char sign;
    
    x = a;
    y = b;
    
    sum = x + y;
    
    if(sum > 0)
        sign = '+';
    else
        sign = ' ';
    
    printf("%d + %d = %c%d", x, y, sign, sum);
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

If we look at the function add, we have 3 integers and 1 char. Integers are 4 bytes long and sign are 1 byte longs. We should therefore have \(3 x 4\) + 1 = 13 bytes allocated at the beginning of the function.

```text
$ gcc add.c -o add
lab@lab-vm:~$ gdb -q add
Reading symbols from add...(no debugging symbols found)...done.
(gdb) set disassembly-flavor intel
(gdb) disassemble add
Dump of assembler code for function add:
   0x08048434 <+0>:	push   ebp
   0x08048435 <+1>:	mov    ebp,esp
   0x08048437 <+3>:	sub    esp,0x18
   ...
```

As you can see after the _prolog_, we subtract 0x18 to the the stack pointer... 0x18, not 0xd... why? The two main reasons why the allocated memory space doesn't match the total size of local variable are:

1. Sometimes, variable doesn't need to be stored in the stack and registers are sufficient. In this case we would see less memory allocated. But this could also be the other way round, during the translation from C to assembly, the compiler might notice that extra memory should be allocated for intermediate variables that was not needed to declare in C.
2. In order to keep the stack aligned, the compiler will allocate extra space in the stack so that stack pointer is a multiple of 16.

In our case, the 0x18 bytes allocated instead of the 0xd bytes needed are most likely meant to keep the stack aligned:

```text
(gdb) break *0x08048439
Breakpoint 1 at 0x8048439
(gdb) run
Starting program: /home/lab/add 

Breakpoint 1, 0x08048439 in add ()
(gdb) info registers esp 
esp            0xbfffef18	0xbfffef18
(gdb) nexti
0x0804843a in add ()
(gdb) info registers esp
esp            0xbfffef00	0xbfffef00
```

As we can see, after the memory allocation, the stack is a multiple of 16.

## Math/Logic operations

## Branching

Comparison Test Un/Signed integer

## Loop

## Functions

## References

* \[[1](https://stackoverflow.com/a/28475252)\] [https://stackoverflow.com/a/28475252](https://stackoverflow.com/a/28475252)
* \[[2](https://en.wikipedia.org/wiki/Data_segment)\] [https://en.wikipedia.org/wiki/Data\_segment](https://en.wikipedia.org/wiki/Data_segment)
* \[[3](https://stackoverflow.com/a/2136065)\] [https://stackoverflow.com/a/2136065](https://stackoverflow.com/a/2136065)
* [ASCII](https://www.asciitable.xyz/) [https://www.asciitable.xyz/](https://www.asciitable.xyz/)

