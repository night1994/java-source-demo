### volatile 内存可见性与指令重排

#### volatile两大作用

- 保证内存可见性
- 防止指令重排
- 此外需要注意volatile并不保证操作的原子性



#### 内存可见性

- 概念

  jvm内存模型：主内存和线程独立的工作内存

  java内存模型规定，对于多个线程共享的变量，存储在主内存当中，每个线程都有自己独立的工作内存（比如cpu的寄存器）,线程只能访问自己的工作内存，不可以访问其他线程的工作内存。

  工作内存中保存了主内存共享变量的副本，线程要操作这些共享变量，只能通过操作工作内存中的副本来实现，操作完毕之后再同步到主内存当中。

  如何保证多个线程操作主内存的数据完成性是一个难题，java内存模型也规定了工作内存与主内存之间交互的协议，定义了8种原子操作：

  1、lock：将主内存中的变量锁定，为一个线程所独占

  2、unlock：将lock加的锁定解除，此时其他的线程可以有机会访问此变量

  3、read：将主内存中的变量值读到工作内存中

  4、load：将read读取的值保存到工作内存的变量副本中

  5、use：将值传递给线程的代码执行引擎

  6、assign：将执行引擎处理返回的值重新赋值给变量副本

  7、store：将变量副本的值存储到主内存当中

  8、write：将store存储的值写入到主内存的共享变量当中。

  通过上面java内存模型的概述，我们会注意到这么一个问题，每个线程在获取锁之后会在自己的工作内存来操作共享变量，操作完成之后将内存中的副本回写到主内存，并且在其他线程从主内存将变量同步回自己的工作内存之前，共享变量的改变对其是不可见的。即其他线程的本地内存中的变量已经是过时的，并不是更新后的值。

  #### volatile原理

  volatile保证可见性的原理是在每次访问变量时都会进行一次刷新，因此每次访问都是主内存中最新的版本。所以volatile关键字的作用之一就是保证变量修改的实时可见性。

#### 指令重排

指令重排序是jvm为了优化指令，提高程序运行效率，在不影响单线程程序执行结果的前提下，尽可能地提高并行度。编译器、处理器也遵循这样一个目标。注意是单线程，多线程的情况下指令重排序就会造成并发问题。

#### 禁止指令重排

除了前面内存可见性中讲到volatile关键字可以保证变量修改的可见性之外，还有另一个重要的作用：在jdk1.5之后，使用volatile关键字禁止变量重排序。

#### 实现原理

volatile关键字通过提供“内存屏障”的方式来防止指令被重排序，为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。

#### volatile 与synchronized区别

1、volatile不会进行加锁操作。

volatile变量是一种稍弱的同步机制，在访问volatile变量时不会执行加锁操作，隐私也就不会使执行线程阻塞，因此volatile变量是一种比synchronized关键字更轻量级的同步机制

2、volatile变量作用类似于同步变量读写操作：

从内存可见性的角度看，写入volatile变量相当于退出同步代码块，而读取volatile变量相当于进入同步代码块。

3、volatile不如synchronzied安全：
在代码中如果过度依赖volatile变量来控制状态的可见性，通常会比使用锁的代码更脆弱，也更难以理解。仅当volatile变量能简化代码的实现以及对同步策略的验证时，才应该使用它。一般来说，用同步机制会更安全些。

4、volatile无法同时保证内存可见性与原子性

即volatile读是安全的，写不是原子的，非安全的。



#### 使用条件

1、对变量的写入操作不依赖变量的当前值，或者你能确保只有单个线程更新变量的值

2、该变量没有包含在具有其他变量的不变式中。

#### 总结

在需要同步时，第一选择应该是synchronized或者lock，这是最安全的方式，尝试其他任何方式都是有风险的。




