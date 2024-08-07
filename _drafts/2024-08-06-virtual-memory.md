---
layout: post
title: 'Into the world of virtual memory'
undescription: 'Exploring the unseen engine of modern computing'
date: 2024-08-06 1:30 -0500
image: /assets/img/craiyon_circuit_board_mushroom.webp
categories: ['Exploration', 'How stuff works']
---

**A brief breakdown of how modern memory management works.**

Today when most programmers deal with very high level programming languages like javascript and python, very little seem to pause to wonder how their code is actually run and how their variables are stored in RAM. 
I'll try to give a high level overview of the sorcerery behind modern memory management.

TODO: Proof Read & Modify

## Physical Memory

In the beginning, there was physical memory. It is the memory that is physically present on the system, like your RAM.
It was limited and every program that ran on a computer had to directly access it.
As the number of programs running on a system increased, it became harder for the individual programs find and allocate memory for themselves.
This lead various engineers to experiment with adding an abstraction layer between the program and the physical memory.
They called it virtual memory.


## Virtual Memory

> In computing, virtual memory, or virtual storage, is a memory management technique that 
> provides an "idealized abstraction of the storage resources that are actually available
> on a given machine" which "creates the illusion to users of a very large (main) memory". - [Wikipedia](https://en.wikipedia.org/wiki/Virtual_memory)


Let me explain.  
In almost every modern Operating System when a program starts up 
it is given a memory space that is sort of like a contiguous array where each cell can hold a single byte. In a 64 bit system, 
this array would be of the size 2<sup>64</sup> bytes. That means about 17,179,869,184 gigabytes if its to be filled.

But where does the OS get that much memory from?

One simple answer. **It doesn't.**

Unlike a regular array, the virtual memory is not a stretch of real continuos memory.
It is a virtual address space given exclusively to that program. 
These addresses are mapped to physical addresses by the OS on demand and a program has no control over how and where 
a particular memory is stored on physical ram. 

A program has the ask the OS first to reserve a part of its virtual memory address for it to use. 
This is the point where the OS maps this chuck of memory onto the actual RAM and allows the prgram to use it.
The OS might also perform additional checks before mapping/allocating that chunk of memory. 
For example, the address 0 is reserved for representing the null pointer, therefore no program is allowed to read/write from it.
There is also reserved read-only space for the program's code.

At first this might look like a pointless system.
Why would any program need to go through a translation layer just to access memory?
There are many reasons, the coolest one being the OS's ability to swap.

> **What is swapping?**    
> Swapping is the process of moving unused memory from the RAM into a storage device like an SSD or a hard disk. 
This is really helpful when the system runs out of memory. 
This process is done entirely by the OS and the programs do not need to care. 
Every OS would have an algorithm to determine what part of memory to swap to disk and if its good you wouldn't notice it at all[^fn1] 
unless whatever program you ran was actively using all that memory (like in the case of games).
{: .prompt-tip }

This entire system of swapping would not be possible if virtual memory was not a thing.

Another reason to use virtual memory is the **ease of use for programmers**. 
Although most people do not deal with memory directly today, it wasn't like that in the 1960's. 
In the absence of a virtual address space, it was really annoying to develop for multi-tasking systems running more than one program at once. 
The program would have to spend a significant amount of time scrubbing through memory to find a space that is just the right size to store a variable.
Without the high level languages with runtimes to abstract all this away, it would have been a nightmare for programmers to code. 
This also causes duplication of effort to solve the same fundamental problem by different program authors, some of which are less efficient than others, 
causing an inefficient use of the available memory. 
By the 1960's the concept of an OS providing a virtual address space started gaining popularity and eventually took over the older system of directly accessing system memory.

There is also the added security that virtual memory provides. It helps prevent malicious programs from tampering with the memory regions of other programs since all they see is its own virtual address space. Malware are still developed to use exploits to break the memory isolation but it is way more complex to do than if there was no virtual memory. Also systems are constantly patched to fix these expoits as soon as they are dicovered.

## Conclusion
In conclusion, virtual memory from a program's perspective is a contiguous address space
that can be addressed using virtual addresses from 0 till the theoretical maximum of the
architecture. The OS can the map these virtual addresses onto physical addresses on demand 
on the RAM or any other media it desires allowing it to isolate process memories from each other 
for security and to swap out less frequently used memory from RAM thereby allowing the the system 
to use more memory than the physically available RAM.

> Please reach out to me via email or open an issue on the github repo if you think there is something factually wrong in this blog post. I tried my best to understand the topic but I'm not an expert in this field.
{: .prompt-info }

## External Resources

Here are some resources to learn more about memory management:
- [ByteLab.Codes](https://www.bytelab.codes/what-is-memory-part-1/)
- [Virtual Memory - Wikipedia](https://en.wikipedia.org/wiki/Virtual_memory)
- [What Every Programmer Should Know About Memory by Ulrich Drepper](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf)


With all that said, time to sign off &#x1FAE1;

![See you later gif](/assets/img/later-imgur.gif)

---

#### Some Foot Notes

[^fn1]: Linux's ability to swap without any noticable interuptions to my workflow at all was one of the main reasons I switched from windows in the first place. I have an old 8GB RAM Asus laptop and I was constantly running out of memory on windows when researching something and having an IDE open. The system would slow down and freeze considerably when I exceeded 8GB. This was not a problem on any distro with swap I tried. To window's credit it did not crash when out of memory but the freezes were a deal breaker for me.

