# Introduction

This course will try to explain to you the most common vulnerabilities identified in binary applications, i.e. buffer overflow, _use-after-free_, _format string_ and _integer under/overflow_.

## Content

What will this course teach you?

* Understanding the structure and purpose of the CPU and the memory
* Basics of C programming
* Basics of assembly
* Understanding and exploiting basic buffer overflow
* ~~Understanding and exploiting basic use-after-free vulnerability~~
* ~~Understanding and exploiting basic format string vulnerability~~
* ~~Understanding and exploiting basic integer overflow and underflow vulnerabilities~~

What this course won’t teach you?

* Reverse engineering
* Coding in C or Assembly \(although we will briefly cover both\)
* Explain, use and create fuzzers
* Create your own shellcode

Minimum requirements:

* Being familiar with computer
* Being familiar with Linux
* Being able to read C code \(or similar language\)

Before we start with the course, we will need to clarify and set the limit of what is a binary application and what are security vulnerabilities in the context of this course.

A **binary application**, also known as **software**, is a collection of instructions and data that tells a computer how to work. Instructions are machine language supported by an individual processor – typically a **C**​entral **P**​rocessor **U**​nit. Machine language consists of groups of binary values which are read and understood by the processor and correspond to an operation. After each instruction \(i.e. operation\) executed by the processor, the state of the computer change from its preceding state. For example, an instruction may change the value stored in a particular memory location in the computer. 

_Softwares_ are usually written in high-level programming languages, such as C, C++ or .NET. Those languages are easier and more efficient for programmers because they are closer to natural languages than machine languages. High-level languages are translated into machine language using a _compiler_ \[[1](https://en.wikipedia.org/wiki/Web_application)\].

A **security vulnerability** is a weakness that allows an attacker to reduce a system’s information assurance \[[2](https://en.wikipedia.org/wiki/Vulnerability_%28computing%29)\]. Typically, a vulnerability will allow an attacker to compromise the **C**​onfidentiality and/or the **I**​ntegrity and/or the **A**​vailability of a system, hence the [CIA](https://en.wikipedia.org/wiki/Information_security#Key_concepts) concept in IT security.

For the sake of simplicity, this course will only cover applications running on Linux [i386](https://en.wikipedia.org/wiki/Intel_80386) \(32-bit architecture\). The high-level language used for demonstrations will be **C** since it is a widely known language that produces binary application relatively easy to reverse. As for the [lab](lab.md) environment, we will rune the exercises on an Ubuntu virtual environment.

## Licence

This course is licensed under the Apache Licence, Version 2.0.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)

## About the author

My name is Antonin Beaujeant \([@beaujeant](https://twitter.com/beaujeant)\), I’m a security professional doing mostly penetration tests. I don’t have extensive knowledge of binary security. Most of my experience in that area is coming from CTF and security challenges.

There are already tons of information available for free on the internet, but I decided to create my own course nonetheless. The topic can be complex and overwhelming, so I wanted to create a course the way I wish I would have learned it.

Everybody has its own preference for learning, e.g. reading, listening, visualizing, etc. This means the structure of this course might be odd, and the way I explain things might not make sense to you. This is why I tried to add references as much as possible when suited so that you can get more information or read the same content with a different approach and style.

## How to contribute

I’d be more than happy to hear your feedback for improvement in both _substance_ and _form_. If you see any, feel free to report it via the GitHub [issue page](https://github.com/beaujeant/appsec101/issues) or via [pull requests](https://github.com/beaujeant/appsec101/pulls). I’m not a native English speaker, so this means there are probably lots of typos, repetitions and weird turns of phrase. Furthermore, as mentioned previously, I’m not an expert in binary security, so the content of this course might be inaccurate or maybe even incorrect. While sometimes, I’m inaccurate on purpose for the sake of simplicity, I did my best to avoid any incorrect statement and/or explanation. I created that course for educational purpose and would feel terrible if it actually misinformed its audience instead. So I would really appreciate if you could help me identifying erroneous content and improve this course.

## References

* \[[1](https://en.wikipedia.org/wiki/Web_application)\] https://en.wikipedia.org/wiki/Web\_application
* \[[2](https://en.wikipedia.org/wiki/Vulnerability_%28computing%29)\] https://en.wikipedia.org/wiki/Vulnerability\_\(computing\)
* [CIA](https://en.wikipedia.org/wiki/Information_security#Key_concepts): https://en.wikipedia.org/wiki/Information\_security\#Key\_concepts
* [i386](https://en.wikipedia.org/wiki/Intel_80386): https://en.wikipedia.org/wiki/Intel\_80386
* [@beaujeant](https://twitter.com/beaujeant): https://twitter.com/beaujeant
* [issue page](https://github.com/beaujeant/beaujeant.github.io/issues): https://github.com/beaujeant/beaujeant.github.io/issues
* [pull requests](https://github.com/beaujeant/beaujeant.github.io/pulls): https://github.com/beaujeant/beaujeant.github.io/pulls

