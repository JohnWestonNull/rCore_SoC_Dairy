# 7月21日，Day 24

## Lab 1 练习题

- 原理：在 `rust_main` 函数中，执行 `ebreak` 命令后至函数结束前，`sp` 寄存器的值是怎样变化的？

- 回答：

  - `__interrupt`:  `sp` 首先减去 -34 * 8 用于存储各种寄存器，然后由 `SAVE` 宏将寄存器中的值存入。
  - `handle_interrupt` & `breakpoint`: 无变化，其中 `breakpoint` 会改变 `sepc` 中的值。
  - `__restore`: `sp` 加回原来的值

  

- 分析：如果去掉 `rust_main` 后的 `panic` 会发生什么，为什么？

- 回答：编译报错，`rust_main` 应当不返回，由 `!` 标记，然而去掉之后则变为返回单元值 `()` 。

  如果没有标记 `!` 则会继续执行指令，但是后面的内容时未知的，因此行为是未定义的。



- 如果程序访问不存在的地址，会得到 `Exception::LoadFault`。模仿捕获 `ebreak` 和时钟中断的方法，捕获 `LoadFault`（之后 `panic` 即可）。
- 回答：直接多加个模式匹配的分支即可。



- 在处理异常的过程中，如果程序想要非法访问的地址是 `0x0`，则打印 `SUCCESS!`。
- 回答：读取 `stval` 的值，判断之后输出。



- 添加或修改少量代码，使得运行时触发这个异常，并且打印出 `SUCCESS!`。
- 回答：可以通过修改页表权限的方式，写入页表，把正常的 RWX 权限改成 WX 之类的即可。

## Lab 2 练习题

- 原理：.bss 字段是什么含义？为什么我们要将动态分配的内存（堆）空间放在 .bss 字段？

- 回答：一般编写的**用户**程序中， .bss 段通常用于存储静态分配的未初始化的变量，在不同的语言中可能有不同的处理方式。

  为什么？不为什么，随便放到一块没人用的内存区域都可以，在 `FrameAllocator` 中将那段避开就好，放 .bss 段可能就是方便，不用特意避开。可以在内存尾部开一块给堆用其实。

  

- 分析：我们在动态内存分配中实现了一个堆，它允许我们在内核代码中使用动态分配的内存，例如 `Vec` `Box` 等。那么，如果我们在实现这个堆的过程中使用 `Vec` 而不是 `[u8]`，会出现什么结果？

- 回答：死循环，`Vec` 尝试调用分配函数，分配函数尝试利用 `Vec` 分配内存。



- 回答：`algorithm/src/allocator` 下有一个 `Allocator` trait，我们之前用它实现了物理页面分配。这个算法的时间和空间复杂度是什么？
- 时间复杂度 O(1)，分配时只需要一个 `if`，空间复杂度 O(n)，我们需要记录分配过的内存。



- 二选一：实现基于线段树的物理页面分配算法（不需要考虑合并分配）；或尝试修改 `FrameAllocator`，令其使用未被分配的页面空间（而不是全局变量）来存放页面使用状态。
- 回答：之前在 `blog_os` 中实现过一个基于链表的实现，参见 [slc_os 分配器实现](https://github.com/JohnWestonNull/slc_os/blob/master/src/allocator/fixed_size_block.rs) 



- 前面说到，堆的实现本身不能完全使用动态内存分配。但有没有可能让堆能够利用动态分配的空间，这样做会带来什么好处？
- 可以利用类似于 Java 中 `ArrayList` 或者 C++ 中 `vector` 的实现方式，每次用尽空间时，翻倍分配空间。这样可以减少空间浪费的同时充分利用内存。
