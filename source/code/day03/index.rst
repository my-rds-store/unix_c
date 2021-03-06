##############
内存管理 - d03    
##############

::

    回顾：
       C语言的程序怎么写？ - 包括：C基本语法、数据结构、C标准函数库、常用的算法
       UC - C程序员的错误处理、环境变量、Unix的静态库和共享库
    今天：
       UC - Unix/Linux系统开发(以C语言为主)
        环境变量收尾
        Unix系统的内存管理(纯UC)
    
       C语言没有直接的字符串(文本)类型，字符串用 char* 或者 char[ ]去代表，因此 字符串 可以看出 字符指针，字符串数组就是 字符指针数组 (或char** / char [][])。
       环境表就是一个字符串数组，char* []，用 外部全局变量environ代表。 extern char** environ;
       操作环境表就有两种方式：
        1 用指针操作的方式(C程序员的基本功)
        2 C提供的标准函数
         getenv()  -  获取一个环境变量，用名字找到值
         putenv()/setenv() - 设置环境变量(区别在于如果存在,putenv必定替换，而setenv可以选择)
         unsetenv() - 删除一个环境变量(按名字)
         clearenv() - 删除所有环境变量
    
       #include <stdio.h>
       #include <my.h>
       export  CPATH=.:/home/tarena/uc/day03/
    
       环境变量PATH(Windows中是Path)是系统路径，默认情况下，所有的可执行文件/命令 都需要带上路径才能运行，但如果把命令所在的路径放入系统路径，系统就能自动识别，命令就不需要带路径而是直接运行。
    
       去掉./a.out中的路径，可以把 export PATH=.:$PATH 写入 /home/tarena/.bashrc 文件中。
       重启机器 或者 source .bashrc 就生效了。
    
      Unix/Linux系统函数 - 内存管理机制
        1 六个内存分配/回收 函数 
          malloc()/free()  sbrk()/brk()  mmap()/munmap()
        2 进程内存空间的划分 
          一个进程启动后，都有 哪些内存区域组成
        3 Unix/Linux系统的内存管理机制(原理)
          虚拟内存地址空间
        
      1 Unix/Linux遵循相同的规范，POSIX规范，因此UC和Linux C在编程上是通用的。细微的差别会讲清楚。
      2 分配/回收内存的方式：
       STL容器、JAVA  -  自动管理(分配和回收)   
           |
         C++    -   new分配内存  delete回收内存
           |
         C语言  -  malloc()分配  free()回收
           |
        UC系统函数 - sbrk() brk() 分配、回收
           |
        UC系统函数 - mmap()分配 munmap()回收
          |                                          (用户层)
    ------------------------------------------------------------
          |                                          (内核层)
        UC系统函数 - kmalloc() vmalloc()  
    
      3 进程内存空间的划分
        程序 是源代码编译连接的产物，是硬盘上的文件。
        进程 运行在内存中的程序，程序一旦运行起来就叫进程。
        严格来说，内存空间是针对进程的，与程序无关。程序加载到内存中就是进程了。在很多的情况下，也把进程叫程序。
        Unix/Linux查看系统进程的命令：
          ps -aux (Linux专用) 或 ps -ef(通用)
        一个进程的内存空间包括以下部分：
        3.1 代码区 - 程序的代码(以函数的形式)存入代码区，函数指针就是函数在代码区的地址，代码区 是 只读区域。
        3.2 全局区 - 存放全局变量和static的局部变量，读写权限。
        3.3 BSS段 - 存放未初始化的全局变量(没有写=的)，读写权限。
         全局区和BSS段的区别就在于：虽然两者都是在main()运行前创建，但BSS段会在main()执行之前自动清一次0，而全局区不会。
        3.4 栈区(stack) - 也叫堆栈区，存放局部变量(没有static的)，函数的形参也是在栈区。系统自动管理。
        3.5 堆区(heap) - 也叫自由区，是程序员完全管理的区域，系统不会管理这个区域。malloc()/sbrk()/mmap()主要针对这个区域。内存泄露的重灾区。
        3.6 只读常量区 - 很多的书上都把只读常量区归入代码区，存放字符串的字面值("abc")和const修饰的全局变量。
       (超重点)
    
      4 Unix/Linux系统的内存管理机制(针对32位系统)
        Unix系统采用虚拟内存地址空间 管理内存。虚拟内存地址其实就是一个整数，可以看成是内存的编号，但不是内存自带的物理地址。每个进程在启动后 都先天具备 0-4G的虚拟内存地址(0-4G编号)，这个编号不代表任何的物理内存，也存储不了任何的数据；只有做了内存映射(用虚拟地址 映射 物理内存或硬盘文件)，虚拟内存地址才能存储数据。程序员接触到的全部都是虚拟内存地址。不同的进程虽然使用相同的虚拟内存地址，但映射的物理内存是不同的，因此互相不会影响。
        其中，0-3G是用户使用的，叫用户层(用户空间)，3G-4G是内核使用的，叫内核层(空间)。用户层的程序只能使用用户层的内存，无法直接访问内核空间，除非使用系统提供的相关函数进入内核空间。
       内存的管理是以 字节为单位的，一个字节 等于 8个 二进制位。内存映射 以 字节作为单位 效率太低，内存映射 以内存页作为基本单位，一次映射N个内存页，一个 内存页 是4096(4K)字节。函数getpagesize()可以获取内存页的大小。
       
        所谓的内存分配其实包括两部分：
         1 分配虚拟内存地址
         2 映射物理内存/硬盘文件
    
       如果使用了未映射的虚拟内存地址，就会引发段错误。
       段错误的常见原因：
        1 使用了未映射的虚拟内存地址操作数据
        2 对内存区域进行没有权限的操作(比如：修改只读区)
    
      Linux系统中，几乎一切都可以看成文件，内存也可以用文件的方式进行操作，也对应着文件。比如：进程对应 /proc/PID 目录。
      如果想查看进程的内存信息，可以用 
       cat   /proc/PID/maps 查看。 (PID是 打印出来的进程号)
