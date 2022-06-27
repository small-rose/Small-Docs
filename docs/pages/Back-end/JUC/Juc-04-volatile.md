## volatile

> volatile不保证原子性，只保证可见性和禁止指令重排

```java
public class Demo(){

	private static volatile Demo instance = null;

    private Demo() {
        System.out.println(Thread.currentThread().getName() + "\t 执行单例构造函数");
    }

    public static Demo getInstance(){

        if(instance == null){
            synchronized (Demo.class){
                if(instance == null){
                    instance = new Demo(); //pos_0
                }
            }
        }
        return instance;
    }
}

```

pos_0 处的代码转换成汇编代码：

```
0x01a3de1d: movb $0×0,0×1104800(%esi);
0x01a3de24: lock addl $0×0,(%esp);
```

**volatile保证可见性原理**

有 **volatile**变量修饰的共享变量进行写操作的时候会多出第二行汇编代码，通过查IA-32架 构软件开发者手册可知，**Lock**前缀的指令在多核处理器下会引发了两件事情。

（1）将当前处理器缓存行的数据写回到系统内存。

（2）这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效。



>为了提高处理速度，处理器不直接和主内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，但操作完不知道何时会写到内存。如果对声明了volatile的 变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现MESI缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。 

 

JVM**通过内存屏障来实现的**限制处理器的重排序。

什么是内存屏障？硬件层面，内存屏障分两种：读屏障（Load Barrier）和写屏障（Store Barrier）。

内存屏障有两个作用：

（1）阻止屏障两侧的指令重排序；

（2）强制把写缓冲区/高速缓存中的脏数据等写回主内存，或者让缓存（CPU缓存，如L1，L2）中相应的数据失效。

编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。

编译器选择了一个比较保守的JMM内存屏障插入策略，但它可以保证在任意处理器平台，任意的程序中都能 得到正确的volatile内存语义。 

保守策略下四种内存屏障如下：【Load代表读操作，Store代表写操作】

- 在每个volatile写操作的前面插入一个`StoreStore`屏障。

- 在每个volatile写操作的后面插入一个`StoreLoad`屏障。

- 在每个volatile读操作的前面插入一个`LoadLoad`屏障。

- 在每个volatile读操作的后面插入一个`LoadStore`屏障。



> `StoreStore`屏障：对于这样的语句`Store1; StoreStore; Store2`，在`Store2`及后续写入操作执行前，这个屏障会吧`Store1`强制刷新到内存，保证`Store1`的写入操作对其它处理器可见。
>
> `StoreLoad`屏障：对于这样的语句`Store1; StoreLoad; Load2`，在`Load2`及后续所有读取操作执行前，保证`Store1`的写入对所有处理器可见。它的开销是四种屏障中最大的（冲刷写缓冲器，清空无效化队列）。在大多数处理器的实现中，这个屏障是个万能屏障，兼具其它三种内存屏障的功能.
>
> `LoadLoad`屏障：对于这样的语句`Load1; LoadLoad; Load2`，在`Load2`及后续读取操作要读取的数据被访问前，保证`Load1`要读取的数据被读取完毕。
>
> `LoadStore`屏障：对于这样的语句`Load1; LoadStore; Store2`，在`Store2`及后续写入操作被刷出前，保证`Load1`要读取的数据被读取完毕。



对于连续多个`volatile`变量读或者连续多个`volatile`变量写，编译器做了一定的优化来提高性能。



```
class VolatileBarrierExample {
        int a;
        volatile int v1 = 1;
        volatile int v2 = 2;

        void readAndWrite() {
            int i = v1; // 第一个volatile读
            int j = v2; // 第二个volatile读
            a = i + j; // 普通写
            v1 = i + 1; // 第一个volatile写
            v2 = j * 2; // 第二个 volatile写
        }
        // 其他方法 
}
```

针对`readAndWrite()`方法，编译器在生成字节码时可以做如下的优化：


![img](https://cdn.jsdelivr.net/gh/youthlql/lql_img/Java_concurrency/Source_code/Second_stage/0014.png)



**总结：**

（1）`Volatile` 在保证内存可见性这一点上，`volatile`有着与锁相同的内存语义，所以可以作为一个“轻量级”的锁来使用。但由于`Volatile`仅仅保证对单个`volatile`变量的读/写具有原子性，而锁可以保证整个临界区代码的执行具有原子性。所以在功能上，锁比`volatile`更强大；在性能上，`volatile`更有优势。

（2）在禁止重排序这一点上，volatile也是非常有用的。比如我们熟悉的单例模式，其中有一种实现方式是“双重检测锁”

```java
public class DoubleCheckDemo {

    private static DoubleCheckDemo instance; // 不使用volatile关键字
    //private static volatile DoubleCheckDemo instance; // 使用volatile关键字

    // 双重检验锁
    public static DoubleCheckDemo getInstance() {
        if (instance == null) { // 第7行
            synchronized (DoubleCheckDemo.class) {
                if (instance == null) {
                    instance = new DoubleCheckDemo(); // 第10行
                }
            }
        }
        return instance;
    }
} 
```

如果这里的变量声明不使用`volatile`关键字，是可能会发生错误的。它可能会被重排序：

```java
instance = new DoubleCheckDemo(); // 第10行
```

第十行操作，在内存中分分解成三步：

```java
// 可以分解为以下三个步骤
1 memory = allocate();// 分配内存 相当于c的malloc
2 ctorInstanc(memory) //初始化对象
3 s = memory //设置s指向刚分配的地址
```

重排序1-3-2：

```java
// 上述三个步骤可能会被重排序为 1-3-2，也就是：
1 memory = allocate();// 分配内存 相当于c的malloc
3 s = memory //设置s指向刚分配的地址
2 ctorInstanc(memory) //初始化对象
```





