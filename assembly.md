# Assembly

Assembly is a vast topic. We could spend hours, days, month and we still would have plenty to say about. Nevertheless, this is not a reverse engineering course and I’m myself not particularly fluent in assembly, so this course will cover the bare minimum to understand and exploit the vulnerabilities listed for this course, i.e. buffer overflow, format string, use after free and integer over/underflow.

As seen in the [CPU](central-processing-unit.md) chapter, instructions are sent to the CPU in the form of binary values. Each binary instruction correspond to one assembly instruction. It is therefore possible to translate binary instructions directly into assembly instructions and the other way round. There is a one-to-one relation between binary instructions sent to CPU and assembly instructions, which is not the case for the relation between C and binary instruction. For instance, the line `c = 4 + 2;` in C can be translated in several ways and it will be composed of more than one binary instruction \(CPU operation\).

{% hint style="info" %}
The translation from binary instructions to its assembly representation is called *to disassemble*, while the other way, i.e. the translation from assembly instructions to their binary instructions is called *to assemble*.
{% endhint %}

Programs can be developped entirely in assembly. However, if you simply translate assembly instructions to their corresponding binary instructions, the final binary file won't be executable. For this, you need to build a [PE file](memory.md), embed the translated instruction together with *initialized data* and *global variables*, map the data, build the import table, etc. This is the role of the *compiler* and *linker* to do this, but this is another story. In this course we don't want to learn assembly to build application, instead, we want to learn how to read and understand compiled code, which is different exercice and requires slightly different knowledge and skills in assembly.

## Hello world!

Let see how our `hello-world.c` application has been translated by the compiler:

{% code-tabs %}
{% code-tabs-item title="hello-world.c" %}
```c
#include <stdio.h>

void main ()
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
   0x0804840b <+0>:	    lea    ecx,[esp+0x4]
   0x0804840f <+4>:	    and    esp,0xfffffff0
   0x08048412 <+7>:	    push   DWORD PTR [ecx-0x4]
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

Here, we only disassembled the instructions for the function `main` and we already ended up with 16 assembly instructions. For now, no need to understand why the compiler translated the code like this, just try to understand what each line does. Let's dive into it!

```text
0x0804840b <+0>:	    lea    ecx,[esp+0x4]
```

Here is how to read this first line of assembly:

* `0x0804840b` is the address in memory where this binary instruction is located.
* `<+0>` is the offset in byte from the beginning of the function `main`. Since we are at the first instruction of `main`, the offset is `0`. Note that on the second line, the offset is `4`, which means the first instruction took 4 bytes in memory, while on the thrid line, the offset is `7`, which means the second instruction is taking 3 bytes in memory \(`7`-`4`\).
* `lea ecx,[esp+0x4]` is the assembly instruction. The intel assembly syntax start with the instruction followed by zero, one or two operands. Operands are seperated with a coma \(`,`\).

The `lea` is the *load in effective address* instruction. What it does is basically calculating the what is inside the bracket of the second operand and write the result in the first operand.


## Comments

## Variables

## Math/Logic opertations

## Branching

Comparison Test Un/Signed integer

## Loop

## Functions

