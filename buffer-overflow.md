# Buffer overflow

Finally, we made it: the first chapter actually talking about application security! Hopefully, by now, you should have all the necessary knowledge to understand what is a buffer overflow and how to exploit it, so let's get started.

## Definition

In the name of the vulnerability "buffer overflow", we have **buffer**... and **overflow**. A buffer if an area in memory where pieces of information \(e.g. variables, saved registers, etc\) are stored temporary to be used later. Typically, the **stack** can be considered as a buffer.

{% hint style="info" %}
In this beginner course, we will focus on stack-based buffer overflow only.
{% endhint %}

In a buffer, information is stored contiguously \(one after another\), and we've seen it earlier, when stored in memory, data doesn't have type. This is just a succession of `1` and `0`. What differentiate one piece of information from another is:

* the pointer used to indicate the beginning of the data
* the instruction to be executed
* \(optionally\) the size directive

![mov instruction with size directive](.gitbook/assets/size-directive.png)

But for the CPU itself whether the data processed was initially an integer, a char or float, it doesn't matter, it will execute the instruction regardless of the type of data pointed by the operand\(s\).This means if we execute an instruction at the **wrong address**/**offset**, we might _mistakenly_ alter neighbour piece\(s\) of information in the buffer.

```text
; snipped based on simple-add.c
mov    eax,DWORD PTR [ebp+0x8]
mov    DWORD PTR [ebp-0xc],eax
mov    eax,DWORD PTR [ebp+0xc]
mov    DWORD PTR [ebp-0a],eax  ; wrong offset which will overwrite previous local variable
```

![mov instruction with wrong offset](.gitbook/assets/wrong-offset.gif)

The most common situation whenever a wrong address/offset occurs is when browsing and writing in an array. The iteration counter used for indexing might not be properly verified and end up being higher than the actual size of the memory area allocated for the array, resulting in the array being **overflow** and overwriting the subsequent piece\(s\) of information.

![Loop overflow](.gitbook/assets/loop-overflow.gif)

So now you know what is a **buffer** and what we mean by **overflow** in that context.

## Impacts

Overwriting neighbour memory areas could be leveraged for **malicious** purpose. Here are two examples to illustrate the impact of a buffer overflow.

### Edit sensitive neighbour variables

Let say for instance that you are playing an online video game. Your profile should be stored in the stack on the server side. Your profile contains your name, your location, how much money you have \(wallet\), your status \(allied, enemy or moderator\) and the list of items in your inventory.

![Game server stack](.gitbook/assets/game-stack.png)

Now, let say the function used to update your name doesn't verify if the new name fits the memory area allocated for the name variable in the stack. You could try to update your name, with a string long enough that it overflow the variable and overwrite the content of the next variable, i.e. your balance \(wallet\). For instance with the name "Queen Daenerys of the House Targaryen." \(35 characters\), which will overflow the 32 bytes array.

![Wallet overflowed](.gitbook/assets/game-wallet-overflow.png)

We now have `0x002e6e72` = 3042930 in our wallet!

{% hint style="info" %}
Remember that a string always ends up with a `NULL` character
{% endhint %}

But then, why overwriting only the balance you may ask? Why not also overwriting the status to be a moderator? 

When overwriting the money variable, we didn't have to write a specific value, we just wanted to be more rich, so we just we just typed random text to overwrite the memory area where your balance is stored. If we were clever, we could also look in the ASCII table which printable character represent the higher value. Unfortunately, the highest value is the non-printable character `DEL`, so we could use the `~` instead \(`0x7e` in hexadecimal\). So, we can change our name to "Queen Daenerys of the House Targary~~~" in order to have `0x007f7f7f` = 8355711.

![Optimised waller overflow](.gitbook/assets/game-waller-overflow-opt.png)

But let's get back to our initial question: what if we want to overwrite the status variable. Status can only be `0x00000000` \(moderators\), `0x00000001` \(allies\) or `0x00000002` \(enemies\). What we can do is to overwrite the balance variable with four characters \(the size of an integer\) and stop there. The `NULL` string terminator character of the new name would overwrite the first byte of the status integer variable. Since the integers are stored in little-endian, overwriting the first byte will actually overwrite the least significant bit and thus would change the value to `0x00000000`.

![Status overflowed](.gitbook/assets/game-status-overflow.png)

### Control the execution flow

In the first example, we overwrite local variables located right after the overflowed array. Those variables were really specific to the program itself in located conveniently after the vulnerable memory area. Now, what if the vulnerable buffer doesn't contains any interesting variable to overflow? How else could I leverage a buffer overflow? You might have guessed by now based on the title: we could also also overwrite the return address! If we overwrite the return address, we could basically redirect the execution flow \(almost\) anywhere we want once the current function reaches the `ret` instruction.

{% hint style="info" %}
This could be done only if the vulnerable buffer is actually located in the current stack frame
{% endhint %}

If we re-use the previous example with the online game, we would need to overflow the name variable, overwrite the status and the inventory list, then we should have right after that the saved `ebp` and finally the return address. Here is the stack if we change our name to "Daenerys of the House Targaryen the First of Her Name".

![Return address overflowed](.gitbook/assets/ret-overflow.png)

At the end of the function, when the epilog is executed, the recovered stack frame from the caller might be completely wrong \(depending how you overwrite the saved `ebp`\), but this shouldn't be a problem, you now have control of the execution flow and can decide what can be executed next.

## Purpose

Some might wonder why is that interesting to redirect the execution flow or overwrite sensitive variable? Why not just writing a program that does exactly what I want instead exploiting a buffer overflow to redirect the execution to another potentially interesting function? Well, exploiting a buffer overflow might be interesting in several scenarios, but here are the three main ones:

#### Escalate privilege

When using a Linux, you are limited to your own user's right. Whenever you run a program, it inherit your access permission. So if you create a program that open and read `/etc/shadow` \(which has read and write access restriction to the user `root` only\), your program won't have the permission to do so. However, if you manage to find and exploit a buffer overflow on an application running with higher privileges \(let say `root`\), you will be able to execute malicious code with those privilege \(e.g. read `/etc/shadow`\).

One interesting feature of Linux is the SUID. SUID, short for **S**et owner **U**ser **ID** up on execution, is a special type of file permissions given to a file. When defined, it gives temporary permissions to a user to run a program/file with the permissions of the file owner rather that the user who runs it. In simple words users will get file owner's permissions as well as owner UID and GID when executing a program \[[1](https://www.linux.com/blog/what-suid-and-how-set-suid-linuxunix)\]. So this means if the `root` user creates a program, set the permission so that anyone can execute it, than set SUID, you will be able to run that program and the program will have root access permission. So, if you find a buffer overflow in that program, you will be able to execute code as `root`.

{% hint style="info" %}
This course didn't cover the Linux system and its permission so that's fine if this part is not clear to you.
{% endhint %}

#### Remote access

You have an application vulnerable to a buffer overflow running on a remote host that I don't have access to. That host has interesting information \(e.g. documents, mails, etc\) and/or resources \(e.g. network access to interesting asset\). The application is a service listening on the network for user inputs, e.g. a web server, DNS server, an online video game server, etc. The data received from the network is used by the function vulnerable to buffer overflow. An attacker could thus send a specifically crafted payload on the service via the network to exploit that vulnerability and execute its own code to gain remote code execution and control the server.

#### Social engineering

For this last example, let's consider a PDF reader vulnerable to buffer overflow. The exploitation is done when parsing the PDF to display the file. An attacker could create a malicious PDF that exploits the vulnerability to install a virus. It would then share the PDF using social engineering technique such as phishing to compromise targeted victims.

## Exploitation

In this chapter, we will see how to identify a buffer overflow and control the execution flow to execute a malicious code. We could also explain how to overwrite neighbour variables, but if you understand how to exploit a buffer overflow to redirect the execution, I'm sure you will know how to reuse the technique to overwrite other variable than the return address.

Here is the source code of the application we will use to demonstrate the exploitation steps:

{% code-tabs %}
{% code-tabs-item title="login.c" %}
```c
#include <stdio.h>

int verify_password(void);

void main()
{
    int valid;

    valid = verify_password();

    if(valid)
        printf("Password correct");
    else
        printf("Invalid password");
}

int verify_password()
{
    char password[16];

    printf("Enter password: ");
    gets(password);

    if( ! strcmp(password, "letmein") )
        return 1;
    else
        return 0;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

When compiling, don't forget to use the option `-fno-stack-protector` to remove the stack protection against buffer overflow:

```text
$ gcc login.c -o login -fno-stack-protector
```

{% hint style="info" %}
`gcc` is not going to be happy and will throw some warning because the function `gets` is dangerous. You can ignore the warning and proceed. Here we want to be vulnerable.
{% endhint %}

Now you can test the application:

```text
$ ./login
Enter password: hello
Invalid password

$ ./login
Enter password: letmein
Password correct
```

### Identifying vulnerability

There are two main ways to identified a buffer overflow vulnerability. One is to look at the source code/assembly \(if we have access\) to find known vulnerable function, the other is to inject different type of payload \(usually long one\) and check if the application crashed \(or throw an error\).

#### Source code analysis

Since this course is not meant to teach you reversing, we will assume that we have access to \(a copy\) of the compiled binary application and its source code \(in C\). The first step is to look at the source code for the following known functions prompt to buffer overflow: `gets`, `scanf`, `strcpy`, `strcat`, `sprintf`.

Each time you find any of these functions, check if you control the input, check if the destination where the result will be stored can hold that final string and finally, check if there is any size restriction to prevent an overflow.

In our case, we can see that the function `gets` is used. `gets` doesn't restrict the input size, which you completely control, so the function is always vulnerable to buffer overflow.

#### Blind test

Another way to identify a buffer overflow is to find area in the application taking user input, and for each of them, trying to send a very long string. This is a rather dummy but easy way to test for buffer overflow. Sometimes, in order to trigger a potential buffer overflow, you might need to send some input with a very specific format or size, so this technique is definitely not 100% accurate, but you might find a few vulnerabilities. You can automate this test in a more clever way by using a fuzzer, but this won't be cover in this course.

So, here in this case, the application doesn't seem to take input from the program arguments \(argc\), however, it expects some user input when you run it after printing the message "Enter password: ".

```text
./login
Enter password: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

As you can see, whenever you send a password too long, the application crash with the following error message:

```text
Segmentation fault (core dumped)
```

The reason is because the **return address** has been overwritten with my long "AAAAA...", and therefore, at the end of the epilog, when executing the instruction `ret`, the CPU tries to jump at the address stored in what used to be the return address but is now 0x41414141 \(AAAA in ASCII\). 0x41414141 is not a valid address, hence the crash. 

Let's validate this assumption with gdb:

```text
$ gdb -q login
Reading symbols from login...(no debugging symbols found)...done.
(gdb) set disassembly-flavor intel
(gdb) disassemble verify_password
Dump of assembler code for function verify_password:
   0x080484b5 <+0>:	push   ebp
   0x080484b6 <+1>:	mov    ebp,esp
   0x080484b8 <+3>:	sub    esp,0x18
   0x080484bb <+6>:	sub    esp,0xc
   0x080484be <+9>:	push   0x80485a2
   0x080484c3 <+14>:	call   0x8048330 <printf@plt>
   0x080484c8 <+19>:	add    esp,0x10
   0x080484cb <+22>:	sub    esp,0xc
   0x080484ce <+25>:	lea    eax,[ebp-0x18]
   0x080484d1 <+28>:	push   eax
   0x080484d2 <+29>:	call   0x8048340 <gets@plt>
   0x080484d7 <+34>:	add    esp,0x10
   0x080484da <+37>:	sub    esp,0x8
   0x080484dd <+40>:	push   0x80485b3
   0x080484e2 <+45>:	lea    eax,[ebp-0x18]
   0x080484e5 <+48>:	push   eax
   0x080484e6 <+49>:	call   0x8048320 <strcmp@plt>
   0x080484eb <+54>:	add    esp,0x10
   0x080484ee <+57>:	test   eax,eax
   0x080484f0 <+59>:	jne    0x80484f9 <verify_password+68>
   0x080484f2 <+61>:	mov    eax,0x1
   0x080484f7 <+66>:	jmp    0x80484fe <verify_password+73>
   0x080484f9 <+68>:	mov    eax,0x0
   0x080484fe <+73>:	leave  
   0x080484ff <+74>:	ret    
End of assembler dump.
(gdb) break *0x080484ff
Breakpoint 1 at 0x80484ff
(gdb) run
Starting program: /home/lab/login 
Enter password: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Breakpoint 1, 0x080484ff in verify_password ()
(gdb) x/x $esp
0xbfffeeec:	0x41414141
(gdb) nexti
0x41414141 in ?? ()
(gdb) info registers eip
eip            0x41414141	0x41414141
(gdb) x/10i $eip
=> 0x41414141:	Cannot access memory at address 0x41414141
```

When looking at the different section of the memory \(stack, heap, .text. etc\), we can see that there nothing loaded in memory around the address `0x41414141`:

```text
(gdb) info proc mappings
process 2493
Mapped address spaces:

	Start Addr   End Addr       Size     Offset objfile
	 0x8048000  0x8049000     0x1000        0x0 /home/lab/login
	 0x8049000  0x804a000     0x1000        0x0 /home/lab/login
	 0x804a000  0x804b000     0x1000     0x1000 /home/lab/login
	 0x804b000  0x806c000    0x21000        0x0 [heap]
	0xb7e08000 0xb7e09000     0x1000        0x0 
	0xb7e09000 0xb7fb9000   0x1b0000        0x0 /lib/i386-linux-gnu/libc-2.23.so
	0xb7fb9000 0xb7fbb000     0x2000   0x1af000 /lib/i386-linux-gnu/libc-2.23.so
	0xb7fbb000 0xb7fbc000     0x1000   0x1b1000 /lib/i386-linux-gnu/libc-2.23.so
	0xb7fbc000 0xb7fbf000     0x3000        0x0 
	0xb7fd5000 0xb7fd6000     0x1000        0x0 
	0xb7fd6000 0xb7fd9000     0x3000        0x0 [vvar]
	0xb7fd9000 0xb7fdb000     0x2000        0x0 [vdso]
	0xb7fdb000 0xb7ffe000    0x23000        0x0 /lib/i386-linux-gnu/ld-2.23.so
	0xb7ffe000 0xb7fff000     0x1000    0x22000 /lib/i386-linux-gnu/ld-2.23.so
	0xb7fff000 0xb8000000     0x1000    0x23000 /lib/i386-linux-gnu/ld-2.23.so
	0xbffdf000 0xc0000000    0x21000        0x0 [stack]

```

### Finding offset

If we want leverage a buffer overflow to control the execution flow, we need to know starting from which offset do we start overwriting the return address. One way to know it would be to send the following pattern using gdb: "AAABBBCCCDDDEEEFFFGGGHHH..." \(i.e. the alphabet with each letter repeated three times\).

The **return address** has been overwritten with `0x4b4b4a4a`, which in ASCII is "KKJJ", but since gdb is representing the data in little-endian, we should read the hexadecimal values backward: "JJKK". According to our generated pattern, "JJKK" is at an offset of 28. 

To verify that, we could send a string of 28 "A" followed by 4 "B":

```text
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/lab/login 
Enter password: AAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB               

Breakpoint 1, 0x080484ff in verify_password ()
(gdb) x/x $esp
0xbfffeeec:	0x42424242
```

AAAAAA and counting

Building random string with invalid potential address. Look in crash where it tried to jump and fail. Then find where in the string that "address" is located.

### Limitation

Non-printable character

NULL and stripped characters

### Where to jump

Interesting local function

In the payload in the stack

This is a 101 course so we won't explain more complex exploitation, but just so you know, you could also try to jump in In an imported function such as system or doing ROP.

### Build payload

If you jump to interesting local function, then nothing much to think of. Find the address of that function and overwrite.

If jumping to custom payload in stack, find the payload you want to execute and put it somewhere in your buffer, then calculate the address where that shellcode is located and overwrite return address with the shellcode address. You can use NOP to be inaccurate.

### Send you payload

Depending how your program is expected your user input, send it. You could use echo -ne, but if you want to repeat same character, this could make it tool long. Instead, you could use python instead as follow:

## Mitigation

Lorem

### Canary

Ipsum

### ASLR

Dolor

### NX

Coucou

## References

\[[1](https://www.linux.com/blog/what-suid-and-how-set-suid-linuxunix)\]: [https://www.linux.com/blog/what-suid-and-how-set-suid-linuxunix](https://www.linux.com/blog/what-suid-and-how-set-suid-linuxunix)

