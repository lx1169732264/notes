![nio与io区别](image.assets/nio与io区别.png)

io是单向的输入流不能用于输出

而nio利用缓冲实现了数据在channel中的双向传输



nio为不同的数据类型提供了不同种类的缓冲

allocate()	分配指定大小的缓冲区



## Buffer类4个属性与方法

buffer	标记当前的position

capacity	最大容量

limit	可以操作数据的个数

position	正在被操作数据的位置

​	position<=limit<=capacity



put() get()存取数据

flip()读数据模式	开启之后,当调用get()时, 会将position调为0, limit调为当前最大存储位置,然后再执行get()方法, 不然position并不在起始位置

rewind()重置读数据模式	再次将position调为0, limit调为当前最大存储位置

clear()	**并不会删除数据**,只是将三个属性初始化 ,里面的数据处于"被遗忘"状态 ,position和limit都被初始化,难以读取数据

**mark()	记录当前的position位置**

**reset()	配合mark()的使用,回到mark的位置**



## 直接缓冲区与非直接缓冲区

非直接	调用allocate()方法分配缓冲区 ,缓冲区在**jvm**

直接	allocateDirect(),在**物理内存**

jvm对于直接缓冲区,会尽量避免使用中间缓冲区进行数据的读写,而是直接在缓冲区上进行io操作

分配**直接**缓冲区需要**更大的成本** ,也**不会被gc回收** ,会影响应用程序的内存

![image-20200809134909724](G:\笔记\image.assets\image-20200809134909724.png)

对于非直接缓冲区 ,物理磁盘的数据先读取至内核地址空间,再被copy到jvm内存,最后到应用程序

点开allocate()方法也可以看到返回的是heap堆缓冲

![image-20200809142318554](G:\笔记\image.assets\image-20200809142318554.png)



![image-20200809141028612](G:\笔记\image.assets\image-20200809141028612.png)

对于直接缓冲区 ,应用程序通过物理内存映射文件直接与物理磁盘交换数据 省略了copy的步骤

直到gc释放了应用程序与物理内存映射文件的引用 ,才会销毁链接

直接缓冲区的建立与销毁是成本很高的 ,而gc无法及时回收会导致浪费

所以直接缓冲区适合长时间的连接,大文件的传输



# 通道

最早,cpu需要建立若干io接口来进行io操作,这将导致cpu被占用

后来引入了**DMA**直接存储器访问 ,cpu将io操作交给DMA进行 ,DMA先向cpu申请资源 ,然后形成**DMA总线** ,不过总线的过多也会导致总线冲突,最后影响性能

而channel通道就类似于DMA总线 ,是一个完全独立的处理器 ,专门用于处理io ,不需要向cpu申请资源

![image-20200809142947422](G:\笔记\image.assets\image-20200809142947422.png)



## 主要实现类

* FileChannel	               本地传输
* SocketChannel             TCP
* ServerSocketChannel  TCP
* DatagramChannel        UDP

## 获取通道 getChannel()

本地

* FileInputStream/Output
* RandomAccessFile

Web

* Socket
* ServerSocket
* DatagramSocket

JDK1.7中NIO.2针对各个通道提供open()静态方法

JDK1.7中NIO.2的File工具类提供newByteChannel()方法



