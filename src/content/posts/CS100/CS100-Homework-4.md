---
title: CS100 Homework [4]
description: This is the first post of my new Astro blog.
published: 2023-03-18 21:43:56
tags:
  - C 语言
  - CS100
  - 上科大
  - 计算机科学
category: 作业
---

**Problems**:
- [String and Character Manipulation](https://cs100.geekpie.club/p/P401?tid=641599c647556d5d171f0718)
- [Memory Leak Checker](https://cs100.geekpie.club/p/23?tid=641599c647556d5d171f0718)
- [Interactive Brainfuck (Part. A)](https://cs100.geekpie.club/p/25?tid=641599c647556d5d171f0718)
- [Interactive Brainfuck (Part. B)](https://cs100.geekpie.club/p/26?tid=641599c647556d5d171f0718)


<!--more-->

---
# CS100 Homework 4 (Spring 2023)
作业要求和所有附带文件请去 piazza 下载。

第二题和第三题我们下发了很多文件，请务必去 piazza 下载，以免遗漏。

请注意整理好你电脑里的文件，正确使用“文件夹”的功能，不要让它们散落满地。
(3.18 20:49) 第一题的描述刚刚更新

(3.18 21:09) 第一题刚刚加上了 sanitizer，请不要出现内存泄漏或者非法内存访问。

(3.19 23:07) 第二题 fhq_treap 测试点已更新，piazza 上也已经更新。原来的程序中有 invalid memory access，无法检测，已经去掉。

(3.20 01:32) 第三题已上传

(3.21 23:53) prob3-B (404) 已将时限调整至 2 秒

请注意第一题对于代码简洁性的要求。尤其是前六个函数如果不能用一条 return 语句完成，将导致这个部分零分。

我们最终会以 code interview 的方式检查简洁性，但是是取你在OJ上的最后一次提交，而非让你自己准备一段代码给TA看。

---

## String and Character Manipulation

It is often a good exercise to implement your own versions of some standard library functions. In this task, you need to implement some standard library functions related to character and string manipulations.

The functions you need to implement are listed below.

```c
int hw4_islower(int ch);
int hw4_isupper(int ch);
int hw4_isalpha(int ch);
int hw4_isdigit(int ch);
int hw4_tolower(int ch);
int hw4_toupper(int ch);

size_t hw4_strlen(const char *str);
char *hw4_strchr(const char *str, int ch);
char *hw4_strcpy(char *dest, const char *src);
char *hw4_strcat(char *dest, const char *src);
int hw4_strcmp(const char *lhs, const char *rhs);
```

Your implementations should conform to the C standard. The standard documentations and examples on these functions can be found at https://en.cppreference.com/w/c/string/byte. **Do not trust materials from any other sources like Wikipedia, Baidu Zhidao, CSDN, Zhihu, etc.**

Note that:

- All functions' return types, parameter declarations and behaviors should meet the requirements in the cppreference pages (accessible via the link above). The function names are prefixed `hw_` to avoid name collisions.
- Be careful: The cppreference website contains both C and C++ documentations. Refer to the C documentation pages instead of the C++ ones. C and C++ have slight differences and are not fully compatible.
- You may ignore the `restrict` qualifier on the pointer parameters of `strlen` , `strchr` , `strcpy` , `strcat` and `strcmp`.
- Due to backward compatibility issues, the C standard requires `strchr` to return `char *` instead of `const char *`. To disable the warning about "discarded qualifiers", just use an explicit cast like `char *pos = (char *)str;` or `return (char *)str;`.
- The behaviors of some functions are subject to locale settings, as is mentioned by the standard documentations. **You only need to work under the default locale (`"C"`).**
- It is not allowed to use any standard library functions. You must implement everything on your own.
- It is guaranteed that the tests do not lead to undefined behaviors.

Moreover, you need to make your implementation **as simplified as possible**. In details:

- For `hw4_islower` , `hw4_isupper` , `hw4_isalpha` , `hw4_isdigit` , `hw4_tolower` and `hw4_toupper` , the function body should contain **only one statement**, which is a `return` statement.
- For `hw4_strlen` , `hw4_strchr` , `hw4_strcpy` , `hw4_strcat` and `hw4_strcmp` , these functions should be no more complex than a single pass of the given strings. You can't scan the strings over and over again, or go back and forth between some positions. **For example, if your `hw4_strlen` traverses the whole string to count the number of characters, you can't call it in `strcmp` to simply obtain the length of the strings, because you still need to traverse the strings to do the comparison.**
- You may define helper functions, but make sure they are necessary and meaningful.

### Submission and grading

On the OJ, submit your code containing the definitions of the functions. Do not include the `main` function in your submission. The OJ will check whether your implementations are correct.

We will have an off-line check after the deadline. You will need to explain your implementations to your TAs. The grade for this problem is the combination of the results on OJ and that of the off-line check.

Violation of certain rules will not be detected on OJ. For example, you may get full score on OJ if you call the standard library versions of these functions directly, but this will result in zero points in the end.

### Samples

No samples will be provided here, because the cppreference pages already have many.


---


## Memory Leak Checker

### Background (You'd better not skip this. There is some useful information here.)

Making a C program memory-safe is too hard! You need to make sure every memory access is valid, and that all the dynamically allocated memory is correctly deallocated (freed). Is there a tool that can check the memory management automatically, so that memory-related bugs can be detected without manual debugging all day long?

Well, we do have some quite effective tools for detecting these annoying bugs. GCC and Clang have [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) ([paper](https://www.usenix.org/conference/atc12/technical-sessions/presentation/serebryany)) and [UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html). Clang also has [MemorySanitizer](https://clang.llvm.org/docs/MemorySanitizer.html). [Valgrind](https://valgrind.org/) is a well-known suite of tools for detecting memory management and threading bugs, but it is only supported on Linux and Mac OS X. But Windows users don't have to be frustrated, because [Visual Studio](https://visualstudio.microsoft.com), **the best IDE in the world**, has all these tools and can handle everything for you.

### Description

In this problem, you will write a very simple tool for detecting memory leaks and invalid calls to `free`. **Memory leak** happens when some blocks of memory have been allocated on heap (by `malloc` , `calloc` , `realloc` or `aligned_alloc`) but are not freed. A call to `free` with argument `ptr` is **valid** if and only if
- `ptr` is a null pointer, or
- `ptr` is a pointer that is returned by a previous call to `malloc` , `calloc` , `realloc` or `aligned_alloc` , and it is the first time it is passed to `free`.

Before starting, make sure you have read the cppreference page on [malloc](en/cppreference.com/w/c/memory/malloc), [calloc](en.cppreference.com/w/c/memory/calloc) and [free](en.cppreference.com/w/c/memory/free). You can ignore the paragraphs about thread-safety and "synchronization-with". **Do not trust materials from any other sources like Wikipedia, Baidu Zhidao, CSDN, Zhihu, etc.**

### Main ideas

The idea of detecting memory leaks is straightforward:
- First, we need to record every call to `malloc` and `calloc`. We may use some data structure (e.g. an array) to store the addresses of allocated memory.
  - Since most of you have little knowledge about other fancy data structures (e.g. hash-tables), using an array is enough for this homework.
  - To simplify the task, you don't need to care about `realloc` or `aligned_alloc`.
- When `free(ptr)` is called, check whether `ptr` exists in the array.
  - If the array contains this address, this is a valid operation. It is safe to deallocate the memory, and we may just remove the corresponding record from the array. (Note: To remove a record from the array, we recommend you simply set the recorded pointer to `NULL`.)
  - If not, this call to `free` is invalid, and you should print some error message.
- When the program is terminating, go through the whole array and check whether anything remains in it. Memory leak happens if the array still contains any non-null pointers, and in this case you should print some information about these pointers.

### Obtaining additional information

It is of great help if you can record not only the address of the allocated memory, but also some other information, like the name of the source file and the line number. For example, you may want to see an error message like

```
100 bytes memory not freed (allocated in file hw3_problem1.c, line 32)
```

or

```
Invalid free in file hw3_problem1.c, line 49
```

To achieve this, our array element should be a `struct`:

```c
#define MAX_RECORDS 1000

struct Record 
{
    void *ptr;              // address of the allocated memory
    size_t size;            // size of the allocated memory
    int line_no;            // line number, at which a call to malloc or calloc happens
    const char *file_name;  // name of the file, in which the call to malloc or calloc happens
};

struct Record records[MAX_RECORDS];
```

How do we obtain the file name and the line number? The C standard provides the macros `__LINE__` and `__FILE__`. You can try on your own (in `line_and_file.c`) to see what they represent:

```c
#include <stdio.h>

int main(void) 
{
    printf("%s\n", __FILE__);
    printf("%d\n", __LINE__);
    printf("%d\n", __LINE__);
    printf("%d\n", __LINE__);
    printf("%d\n", __LINE__);
    return 0;
}

```

See [this webpage](https://en.cppreference.com/w/c/preprocessor/replace#Predefined_macros) for standard definitions of `__LINE__` and `__FILE__`.

Now here is how the magic happens: we use `#define` to replace the calls to `malloc` , `calloc` and `free` , so that these operations can be captured by us:

```c
void *recorded_malloc(size_t size, int line, const char *file) 
{
    void *ptr = malloc(size);
    if (ptr != NULL) 
    {
        // record this allocation
    }
    return ptr;
}

void *recorded_calloc(size_t cnt, size_t each_size, int line, const char *file) {
    void *ptr = calloc(cnt, each_size);
    if (ptr != NULL) 
    {
        // record this allocation
    }
    return ptr;
}

void recorded_free(void *ptr, int line, const char *file) 
{
    // ...
}

#define malloc(SIZE) recorded_malloc(SIZE, __LINE__, __FILE__)
#define calloc(CNT, EACH_SIZE) recorded_calloc(CNT, EACH_SIZE, __LINE__, __FILE__)
#define free(PTR) recorded_free(PTR, __LINE__, __FILE__)
```

### Leak checking

In the end, we need to check whether all allocated blocks of memory have been freed. This should be done when the program terminates. To force a function to be called on termination, we need another "black magic":

```c
void check_leak(void) __attribute__((destructor));

void check_leak(void) 
{
    // Traverse your array to see whether any allocated memory is not freed.
}

```

With an additional declaration marked by `__attribute__((destructor))` , this function will be called automatically when the program terminates. You may try the following code (in `destructor.c`) on your own:

```c
#include <stdio.h>

void fun(void) __attribute__((destructor));

void fun(void) 
{
    printf("Goodbye world!\n");
}

int main(void) {}

```

Even though the `main` function is empty, this program will print `Goodbye world!`.

### Usage

This is what we want to achieve: Place the code you wrote in a header file `memcheck.h`:

```c
// memcheck.h
#ifndef CS100_MEMCHECK_H
#define CS100_MEMCHECK_H 1

#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>

#define MAX_RECORDS 1000

struct Record { /* ... */ };

struct Record records[MAX_RECORDS];

void *recorded_malloc(/* ... */) 
{
    // ...
}

void *recorded_calloc(/* ... */) 
{
    // ...
}

void recorded_free(/* ... */) 
{
    // ...
}

void check_leak(void) __attribute__((destructor));

void check_leak(void) 
{
    // ...
}

#define malloc(SIZE) recorded_malloc(SIZE, __LINE__, __FILE__)
#define calloc(CNT, EACH_SIZE) recorded_calloc(CNT, EACH_SIZE, __LINE__, __FILE__)
#define free(PTR) recorded_free(PTR, __LINE__, __FILE__)

#endif // CS100_MEMCHECK_H
```

Then, `#include "memcheck.h"` in the file you want to check:

```c
// Suppose this is my_code.c

#include "memcheck.h"

// Some code that may have memory problems:
int main(void) 
{
    int **p = malloc(sizeof(int *) * 5);
    for (int i = 0; i != 5; ++i)
        p[i] = malloc(sizeof(int) * 3); 
    // do something
    // ...
    // ...
    free(p[2] + 3); // Suppose this is line 11
    free(p);
    return 0;
}

```

Compile and run `my_code.c`. The output should be:

```
Invalid free in file my_code.c, line 11
Summary:
12 bytes memory not freed (allocated in file my_code.c, line 10)
12 bytes memory not freed (allocated in file my_code.c, line 10)
12 bytes memory not freed (allocated in file my_code.c, line 10)
12 bytes memory not freed (allocated in file my_code.c, line 10)
12 bytes memory not freed (allocated in file my_code.c, line 10)
5 allocation records not freed (60 bytes in total).

```

### Detailed requirements 

The detailed requirements on how you should deal with certain situations and what to print are as follows.

- `free(NULL)` should do nothing.
- If `malloc` or `calloc` returns `NULL` (indicating failure), no memory is allocated and nothing should be recorded.
- When an invalid `free` happens, you should (in the function `recorded_free`)
  - detect it, and output

    ```
    Invalid free in file FILENAME, line N
    ```
    with a newline (`'\n'`) at the end, where `FILENAME` is replaced with the file name, and `N` is replaced with the line number, and
  - **prevent the call to the real `free` function**, so that no invalid `free`s actually happen.
- When the program terminates (i.e., in the function `leak_check`), you should make a summary of the memory usage of this program:
  - First, output
    
    ```
    Summary:
    ```
    with a newline at the end.
  - Traverse your array to see whether any memory is not freed. For each such record, output
    
    ```
    M bytes memory not freed (allocated in file FILENAME, line N)
    ```
    with a newline at the end, where `M` is replaced with the size (in bytes) of that block of memory, `FILENAME` is replaced with the file name, and `N` is replaced with the line number. **These lines can be printed in any order.** You don't need to care about singular/plural forms of "byte". Just print `bytes` even if `M == 1`.
  - As a conclusion, if memory leak is detected, output

    ```
    N allocation records not freed (M bytes in total).
    ```
    with a newline at the end, where `N` is replaced with the number of allocation records that are not freed, and `M` is replaced with the total size (in bytes) of memory leaked. You don't need to care about singular/plural forms of "record". Just print `records` even if `N == 1`.
    
    If there is no memory leak, output

    ```
    All allocations have been freed.
    ```
    with a newline at the end.

### Special things you should know

- The behaviors of `malloc(0)` , `calloc(0, N)` and `calloc(N, 0)` are **implementation-defined**. They may return a null pointer (with no memory allocated), but they may also allocate *some* memory and return a pointer to it. Whenever the returned pointer is non-null, it should be recorded, and **it is also considered as memory leak if that memory is not freed**.
- Due to alignment requirements or some other possible reasons, `malloc` and `calloc` may allocate more memory than you expect. For example, `calloc(N, M)` may allocate more than `N * M` bytes of memory, and `malloc(0)` may allocate some memory, as has been mentioned. But you don't need to consider these complex things - just count `N` for `malloc(N)` and `N * M` for `calloc(N, M)`. In other words, the following is a possible summary output:
  
  ```
  Summary:
  0 bytes memory not freed (allocated in file a.c, line 10)
  1 allocation records not freed (0 bytes in total).
  ```

### Notes

- We have provided you with a template code `memcheck.c` that contains everything mentioned above.
- It is guaranteed that the total number of calls to `malloc` and `calloc` will not exceed `MAX_RECORDS` , which is `1000`.
- Feel free to add other helper functions, if you need. **Contact us if you encounter name collision problems.**
- You may try some fancy data structures, like linked-lists or hash-tables, to optimize the time- and space-complexity. But these will **not** give you additional (bonus) points.

### Samples

#### Sample program 1

`sample1.c`

```c
#include "memcheck.h"

int main(void) 
{
    int **matrix = malloc(sizeof(int *) * 10);
    for (int i = 0; i != 10; ++i)
        matrix[i] = malloc(sizeof(int) * 5);
    // do some calculations
    for (int i = 0; i != 10; ++i)
        free(matrix[i]);
    free(matrix);
    matrix = NULL;
    free(matrix); // free(NULL) should do nothing
    return 0;
}

```

Output:

```
Summary:
All allocations have been freed.
```

#### Sample program 2

`sample2.c`

```c
#include "memcheck.h"

// Use __attribute__((__unused__)) to suppress warning on unused variable 'p'.
void happy(void *p __attribute__((__unused__))) {}

int main(void) 
{
    double *p = malloc(sizeof(double) * 100);
    free(p + 10);
    free(p);
    free(p);
    char *q = calloc(10, 1);
    float *r = malloc(0);
    happy(q);
    happy(r);
    return 0;
}

```

**Possible output: (The line numbers may be different)**

If `malloc(0)` allocates some memory and returns a non-null pointer, the output should be

```
Invalid free in file sample2.c, line 9
Invalid free in file sample2.c, line 11
Summary:
10 bytes memory not freed (allocated in file sample2.c, line 12)
0 bytes memory not freed (allocated in file sample2.c, line 13)
2 allocation records not freed (10 bytes in total).
```

If `malloc(0)` does not allocate any memory and returns a null pointer, the output should be

```
Invalid free in file sample1.c, line 9
Invalid free in file sample1.c, line 11
Summary:
10 bytes memory not freed (allocated in file sample2.c, line 12)
1 allocation records not freed (10 bytes in total).
```

### Submission

Submit your `memcheck.h` to OJ (or copy and paste its contents).

### Going further

So far, so good. You have written something that you can really make use of. Although its function is limited, it can be easily extended using the same or a similar idea. For example:

- Add support for [`realloc`](https://en.cppreference.com/w/c/memory/realloc) and [`aligned_alloc`](https://en.cppreference.com/w/c/memory/aligned_alloc), and maybe also for C23 [`free_sized`](https://en.cppreference.com/w/c/memory/free_sized) and [`free_aligned_sized`](https://en.cppreference.com/w/c/memory/free_aligned_sized).
- Detect double-free among the invalid `free` calls. "Double-free" typically refers to a call `free(ptr)` in which the memory pointed to by `ptr` has already been freed.

Furthermore, the way we replace `malloc` , `calloc` and `free` with our own functions is through the preprocessor directive `#define` , which is somewhat weak and may cause problems. For example, every source code file we want to check must `#include "memcheck.h"` , which may cause redefinition of symbols at link-time. User code may also escape from this by simply `#undef` these macros.

A possibly better solution is to hack the **dynamic linking** of these functions, e.g. using some magic provided by `<dlfcn.h>` , or the [malloc hooks](https://www.gnu.org/software/libc/manual/html_node/Hooks-for-Malloc.html) provided by the GNU C library. STFW (Search the Friendly Web) on your own.

There are some language constructs in C++ that allow user-declared replacements for memory allocation and deallocation functions ( `operator new` and `operator delete` ), so that you don't have to do it in a "hacky" manner. See [my video](https://www.bilibili.com/video/BV17N4y1u7Cn/?spm_id_from=333.999.0.0) for details.

### Attachments

[👉 Go to GitHub](https://github.com/zivmax/CS100-homework-attachments/tree/main/hw4_attachments/prob2)


### Test cases
[👉 Go to GitHub](https://github.com/zivmax/CS100-2023-Spring-homework-resources/tree/main/hw4_attachments/prob2/testcases)


---


## Interactive Brainfuck

[Brainfuck](https://en.wikipedia.org/wiki/Brainfuck) 是一种只由 8 种符号构成的，极小化，深奥 (esoteric) 的程序语言，由 Urban Müller 在 1993 年创造。Müller 的目标是创建一种简单的、可以用最小的编译器来实现的、符合图灵完全思想的编程语言。Brainfuck 这个名字已经暗示了它的代码很难读懂。在这次作业中，你将一步步编写一个能运行任意 Brainfuck 代码的程序，或者说，一个解释器。

### 程序 = 状态机？

“[状态机](https://en.wikipedia.org/wiki/Finite-state_machine)”是一种表示有限个状态以及在这些状态之间的转移和动作等行为的数学计算模型。这个概念远远没有听起来那么抽象：你的电脑就可以是一个具有“开机”“关机”两个状态，可以通过开机按钮来在这两个状态之间转移的状态机；你也可以是一个具有“睡觉”、“吃饭”、“学习”、“娱乐”等状态，可以根据时间、心情等条件在这些状态间转移的状态机。

根据这种思想，任何计算机程序也能被看作是一个状态机：状态是计算机的内存（中的所有变量，这些变量变化的值就对应着不同的状态），而执行任意一条简单语句（任意一行最简单的代码）都会转移到一个新的状态。想象一下，你的 `main` 函数的第一行代码决定了程序的初始状态，之后执行的每一句代码都在转移到下一个状态，直到这个程序运行结束。

### Brainfuck 就是在模拟状态机

根据这个思想，如果能模拟出一个进行运算过程的状态机（[图灵机](https://en.wikipedia.org/wiki/Turing_machine)），就相当于具有了模拟出任何计算机程序的能力。人用纸和笔就能进行任何简单的运算：

- 在纸上写上或擦除某个符号；
- 把注意力从纸的一处移动到另一处；

而 Brainfuck 语言所做的也正是如此：

Brainfuck 为每个程序创建了一个长度为 30000 字节，已初始化为零的数组。此外，它还具有一个初始指向数组第一个字节（即下标为 0 的元素）的指针（假设它叫 `ptr`）。它的 8 种运算符分别对应以下 8 种操作：

| 运算符 |                                           含义                                           |    C语言的等价表达    |
| :----: | :--------------------------------------------------------------------------------------: | :-------------------: |
|  `>`   |                                指针前进到下一格（自增1）                                 |        `ptr++`        |
|  `<`   |                                指针回退到上一格（自减1）                                 |        `ptr--`        |
|  `+`   |                                   指针所指字节的值加一                                   |      `(*ptr)++`       |
|  `-`   |                                   指针所指字节的值减一                                   |      `(*ptr)--`       |
|  `.`   |                                    输出所指字节的内容                                    |    `putchar(*ptr)`    |
|  `,`   |                                 向指针所指的字节输入内容                                 |  `*ptr = getchar()`   |
|  `[`   |  **(Part B内容)** 若指针所指字节的值为零，则向后跳转，跳转到其对应的 `]` 的下一个指令处  | `while (*ptr != 0) {` |
|  `]`   | **(Part B内容)** 若指针所指字节的值不为零，则向前跳转，跳转到其对应的 `[` 的下一个指令处 |          `}`          |

除了这八种运算符，其他的所有字符都会被视作代码的注释。

Brainfuck 这种极端简化的深奥语言 (esolang) 被称为图灵焦油坑 (Turing Tarpit)。虽然理论上可以写出任何程序（图灵完备），但是即使只是写简单的程序也不现实。例如下面是 Brainfuck 的“Hello world”（`tests/A/hello_world.bf`）：

```brainfuck
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.>
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.>
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.>
++++++++++++++++++++++++++++++++.>
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.>
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.>
++++++++++.>

```

### 我们的代码框架，以及，你需要做什么？

我们为你提供了两个文件：`ibf.h` 与 `ibf.c` 。所有你需要理解的部分和需要完成的工作都在 `ibf.c` 文件中。头文件 `ibf.h` 中包含了我们提供的魔法 (magic!)，如交互输入输出、命令行参数读取等，尝试阅读或读懂这个文件都不是必要的。

我们已经为你设计好了代码的框架：用一个结构体 `struct brainfuck_state` 来储存上述的 Brainfuck 状态模型：里面包括一个字节数组 `memory_buffer` 和一个储存当前指针位置的数值 `memory_pointer_offset` 。这个简单的结构体足以完成 part A 中的所有工作：实现除了 `[` 与 `]` 两个循环运算符以外的其余六种运算符操作。也就意味着，这时你就可以运行任何不包含 `[` 与 `]` 的 Brainfuck 程序！

简单地说，在本次作业中，你需要填写我们为你在 `ibf.c` 文件中留出的几个函数，实现相应的功能。在实现的过程中，你可能会需要定义新的函数或为结构体添加新的成员，你都可以自由地去做。

### Part A

在这一部分里，你需要实现：

- Brainfuck 状态 `struct brainfuck_state` 的创建与删除
  - `brainfuck_state_new()`
  - `brainfuck_state_free()`
- 除 `[` 与 `]` 两个循环运算符以外的六种运算符
  - `brainfuck_execute_plus()`
  - `brainfuck_execute_minus()`
  - `brainfuck_execute_previous()`
  - `brainfuck_execute_next()`
  - `brainfuck_execute_input()`
  - `brainfuck_execute_output()`
- 执行一段仅含这六种运算符的 Brainfuck 程序
  - `brainfuck_main()`

关于这些函数的具体说明及注意事项，请阅读代码中对应函数前的注释。

### 如何使用

我们为你提供的代码是一个交互式的程序，也就是它的名字（Interactive BrainFuck, IBF）的由来。你可以如同在命令行使用 `python` 指令交互式运行 Python 一样交互式地运行 Brainfuck 代码。我们提供了样例程序，在终端中进入当前目录并输入 `.\ibf_std` (Windows) 或 `./ibf_std` (MacOS/Linux) 即可运行。建议你在开始写之前先试用以下以大致了解它在做什么。你也同样可以对比你自己写完的程序与样例程序在相同输入时的表现来检查你的程序中可能的问题。

若你直接运行程序而不传入任何命令行参数（例如，由 VSCode 的 code runner），你将会看到两行说明信息，以及一行以 `>>>` 开头的交互式输入框。

试着输入这个简单的程序：`,+++.`

对照上面的运算符含义，可以看出，这段程序先输入一个字符，再将它的值加 3（ASCII 码向前移 3 位），最后输出。输入这个程序后回车确认，此时程序会等待 `,`（输入运算符）的输入。再向它输入 `a` 你就会看到它给出的输出 `d` 。

此时程序的状态仍然保留，你可以通过 `>>>` 交互式输入框输入代码并继续运行。试着再输入 `....` ，将当前字节打印 4 次，你会得到 `dddd` 。

我们还提供了很多（很厉害的）示例程序。IBF 也可以运行文件中的 Brainfuck 程序。你需要在命令行中运行你生成的可执行文件，为它提供 Brainfuck 程序文件的路径作为参数。
- Windows下，指令将类似 `.\ibf .\tests\hello_world.bf` 。
- MacOS/Linux下，指令将类似 `./ibf ./tests/hello_world.bf` 。

相同主名的文件为一组示例。程序、输入（部分程序无）、输出的扩展名分别为 `.bf` ，`.in` 和 `.out` 。

当提交至 OJ 时，你只需要提交 `ibf.c` 文件中的代码。


### Part B

在这一部分里，你将需要实现最后两个运算符 `[` 与 `]` 。

只靠一个字节数组与一个指针是做不到运行循环的。（想想为什么？）因此，你需要给结构体添加额外的成员以实现 `[` 与 `]` 两个循环运算符。

那么，需要添加哪些额外成员，用来做什么呢？我们鼓励你思考写出你自己的方案，但你也可以直接采用我们为你提供的一种方案。它被以注释的形式藏在了代码里，解除注释即可使用。我们添加的额外成员有：
- `char* loop_buffer;`            一个可以存放循环体的字符串。
- `size_t loop_buffer_size;`      一个可以保存循环体长度的变量。
- `size_t unmatched_depth;`       一个可以保存当前未匹配的嵌套循环层数的变量。

基于你的实现，你可能不会用到全部这些新成员，也可能会需要自己定义其他的成员变量或为需要的功能添加新的函数。这都是你的自由。

#### Part B 注意事项

- 循环可能嵌套，请确保你能够处理这样的情况。
- 每个 `[` 与 `]` 判断是否为零的字节与当前指针所指位置有关。
- 我们已经帮你处理了括号不匹配时的报错情况。换言之，你可以相信 `brainfuck_main()` 需要运行的代码中，括号是匹配的。所以你在 `brainfuck_main()` 的最后只需要返回 `true` 。
  - 有意思的小测试：当你运行交互式终端形式的 ibf 时，若你输入的一行代码并不完整而有未被匹配的 `[` ，此时代码并不会被运行，而是等待你继续向终端中输入代码，直到括号匹配。这与命令行运行 Python 时可以将循环体分行输入类似。试着先输入一行 `,>++++[` ，再输入 `<.>-]` ，最后给里面的逗号 `,` 输入一个字符看看它会怎样运行！

### Test cases
For part A [👉 Go to GitHub](https://github.com/zivmax/CS100-2023-Spring-homework-resources/tree/main/hw4_attachments/prob3/tests/A)

For part B [👉 Go to GitHub](https://github.com/zivmax/CS100-2023-Spring-homework-resources/tree/main/hw4_attachments/prob3/tests/B)


### Attachments

[👉 Go to GitHub](https://github.com/zivmax/CS100-homework-attachments/tree/main/hw4_attachments/prob3)


---

## Reference solutions
*Remember, you should always solve all the problems yourself and passed all the testcases first.*

- [String and Character Manipulation](https://github.com/zivmax/CS100-2023-Spring-homework-resources/tree/main/hw4_refsol/prob1)
- [Memory Leak Checker](https://github.com/zivmax/CS100-2023-Spring-homework-resources/tree/main/hw4_refsol/prob2)
- [Interactive Brainfuck (Part. A & B)](https://github.com/zivmax/CS100-2023-Spring-homework-resources/tree/main/hw4_refsol/prob3/stds)
