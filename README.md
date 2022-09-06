# -进程、线程和协程
1.进程之间的切换开销较大
2.一个标准的线程由线程ID，当前指令指针PC，寄存器和堆栈组成；进程由内存空间：代码、数据、进程空间、打开的文件和一个或多个线程组成
3.通过硬件的计数器中断处理器，让该线程强制暂停并将该线程的寄存器放入内存中。
4.线程是一个进程中代码的不同执行路线。
5.上下文切换-系统调用
## -超线程
利用特殊的硬件指令，把一个物理芯片模拟成两个逻辑处理核心，让单个处理器都能使用线程级并行计算，进而兼容多线程操作系统和软件，减少了CPU的闲置时间，提高的CPU的运行效率。这种超线程技术(如双核四线程)由处理器硬件的决定，同时也需要操作系统的支持才能在计算机中表现出来。
## -协程
每个请求占用一个线程去完成完整的业务逻辑，系统的吞吐能力取决于每个线程的操作耗时，遇到很耗时的I/O行为，整个系统的吞吐立刻下降，如果线程很多时候，会存在很多线程处于空闲状态，造成了资源应用不彻底--*单线程加异步回调*
1.线程的默认stack是1M，协程是1K。  
2.由于在同一个线程上，可以避免竞争关系使用锁  
3.适用于被阻塞的，需要大量并发情况，不适用于大量计算的多线程，遇到这种情况，更好用线程去解决。  
比较项	线程	协程
占用资源	初始单位为1MB,固定不可变	初始一般为 2KB，可随需要而增大  
调度所属	由 OS 的内核完成	由用户完成  
切换开销	涉及模式切换(从用户态切换到内核态)、16个寄存器、PC、SP...等寄存器的刷新，函数运行时使用的寄存器等	只有三个寄存器的值修改 - PC / SP / DX.  
性能问题	资源占用太高，频繁创建销毁会带来严重的性能问题	资源占用小,不会带来严重的性能问题  
数据同步	需要用锁等机制确保数据的一直性和可见性	不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。
线程之间共享代码区，意味着程序中的任何一个函数都可以放到线程中去执行，不存在某个函数只能被特定线程执行的情况。
线程的栈区没有严格的隔离机制来保护，如果严格线程能够拿到另一个线程栈帧上的指针，该线程就可以改变另一个线程的栈区。

## -TLS
存放在该区域中的变量是全局变量，所有线程都可以访问

虽然看上去所有线程访问的都是同一个变量，但该全局变量独属于一个线程，一个线程对此变量的修改对其他线程不可见。
## -thread_local
线程存储期，能够于stati或extern结合，指定内部或外部链接，除了静态成员始终有外部链接，附加的static不影响存储期。
对象的存储在线程开始时分配，在线程结束时解分配。初始化只发生一次：call_once，初始化过程发生在变量声明的位置，并且在本thread中只执行一次。  
所有的非局部thread_local变量初始化是线程启动的一部分，发生在线程函数执行之前。
静态或线程存储期不能通过lambda捕获。thread_local作为类成员变量必须是static。

## -fork
避免写时拷贝额外开销，父进程首先执行的话，可能会开始向地址空间写入。execve(2)负责为进程代码段和数据段建立映射，真正将代码段和数据段内容读入内容是由系统缺页溢出处理程序按需完成的。
fork（）与vfock（）都是创建一个进程，那他们有什么区别呢？总结有以下三点区别：
1.  fork  （）：子进程拷贝父进程的数据段，代码段
    vfork （ ）：子进程与父进程共享数据段
2.  fork （）父子进程的执行次序不确定
    vfork 保证子进程先运行，在调用exec 或exit 之前与父进程数据是共享的,在它调用exec
     或exit 之后父进程才可能被调度运行。
3.  vfork （）保证子进程先运行，在她调用exec 或exit 之后父进程才可能被调度运行。如果在
   调用这两个函数之前子进程依赖于父进程的进一步动作，则会导致死锁。

# -用户态和内核态之间的切换过程
被转换成CPU执行的指令执行的过程，动态执行的指令序列。
对于任何操作系统来说，创建一个新的进程都是属于核心功能，因为它要做很多底层细致地工作，消耗系统的物理资源，比如分配物理内存，从父进程拷贝相关信息，拷贝设置页目录页表等等，这些显然不能随便让哪个程序就能去做，于是就自然引出特权级别的概念，显然，最关键性的权力必须由高特权级的程序来执行，这样才可以做到集中管理，减少有限资源的访问和使用冲突。  
硬件上在执行每条指令时都会对指令所具有的特权级做相应的检查，相关的概念有CPL、DPL和RPL。  

用户态和内核态的转换

1）用户态切换到内核态的3种方式

a. 系统调用

        这是用户态进程主动要求切换到内核态的一种方式，用户态进程通过系统调用申请使用操作系统提供的服务程序完成工作，比如前例中fork()实际上就是执行了一个创建新进程的系统调用。而系统调用的机制其核心还是使用了操作系统为用户特别开放的一个中断来实现，例如Linux的int 80h中断。

b. 异常

        当CPU在执行运行在用户态下的程序时，发生了某些事先不可知的异常，这时会触发由当前运行进程切换到处理此异常的内核相关程序中，也就转到了内核态，比如缺页异常。

c. 外围设备的中断

        当外围设备完成用户请求的操作后，会向CPU发出相应的中断信号，这时CPU会暂停执行下一条即将要执行的指令转而去执行与中断信号对应的处理程序，如果先前执行的指令是用户态下的程序，那么这个转换的过程自然也就发生了由用户态到内核态的切换。比如硬盘读写操作完成，系统会切换到硬盘读写的中断处理程序中执行后续操作等。

相当于执行了严格中断响应的过程，系统调用实际上最终是中断机制实现的，异常和中断的处理机制基本是一致的。
从当前进程的描述符中提取器内核栈的ss0和esp0信息。  
使用ss0和esp0执行的内核栈将当前进程的cs,eip,eflags,ss,esp信息保存起来，完成了由用户帧到内核栈的切换过程，保存了被暂停执行的程序的下一条指令。  
将先前由中断向量检索得到的中断处理程序的cs,eip信息装入相应的寄存器，开始执行中断处理程序，就转到了内核态的程序执行了。
从内核态返回用户态，通过执行指令iret来完成，指令iret会将先前压栈的进入内核态前的cs,eip,eflags,ss,esp信息从栈里弹出，加载到各个对应的寄存器中，重新开始执行用户态的程序。
