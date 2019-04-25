---
layout: post
title: Write a simple aligned malloc and free using 'sbrk'
tags: [c, operating-system]
---

A process always has a memory layout combined of:

-   **Text Segment** holds the binary source of the process, the actual executable program.
-   **Data Segment** consists of the initialized global variables of the process.
-   **BSS Segment** consists of the uninitialized global variables of the process.
-   **Stack** contains all the local variables.
-   **Heap** is used to store the dynamically allocated variables (variables that are allocated on runtime);

The heap is a memory region that have its end marked by a _break_ pointer. Theoretically, if we want to allocate a new memory space, we just need to increase this _break_ pointer and then, move it back to release the allocated memory space by calling `sbrk` and `brk`. Following codes are the naive implementations of `malloc` and `free` based on the theory that we just said:

```c
void *malloc(unsigned int size)
{
    void *ptr;
    ptr = sbrk(0);

    if (sbrk(size) == (void *)-1) return NULL;

    return ptr;
}

void *free(void *ptr)
{
    brk(ptr);
}
```

As you can see, this implementation is very simple. First, you call `sbrk(0)` to get the current address of _break_ pointer. Then, you increase it by `size` bytes. And when you want to release the requested memory, just call `brk` to move the _break_ pointer to the provided pointer address. Too easy, right?

Um, actually, it is not easy as that. For the above implementations, `free` must be called in regard to the order of the called `malloc`. That mean, if you call `malloc` with the order of `p1->p2->p3` then, when you free, it must be `p3->p2->p1`. It is not good at all!

We must find a way to store the information of each allocated memory space and then we can use it to free the correspondent memory space. In additional, because the information of the allocated memory is stored, we can just mark the memory free instead of releasing and returning it back to OS. So that we can just reuse the allocated memories instead of asking the OS for a new one, because these system calls are so expensive.

The way that we are going to use in this post, and it is a widely used way, is to store the information of the allocated memories in a linked list. The structure of each node is as following:

```c
struct block_meta {
    size_t size;
    struct block_meta * next;
    uint8_t free;
}
```
