# 内存管理

## 普通内存管理

### 内存管理基础知识

#### 内存概念

+ 内存是用于存放数据的硬件。程序执行前需要先放到内存中才能被$CPU$处理用于缓冲速度。
+ 内存地址从$0$开始，每个地址对应一个存储单元。
+ 存储单元的大小不定：
  + 按字节编址，则每个存储单元大小为$1B$，八个二进制位。
  + 按字编址，每个存储单元大小为一个字，根据计算机的字长确定大小，若字长为$16$位，则一个字大小为$16$个二进制位。

#### 内存管理功能

1. 操作系统负责内存空间的分配与回收。
2. 操作系统需要提供某种技术从逻辑上对内存空间进行扩充。
3. 操作系统需要提供地址转换功能，负责程序的逻辑地址与物理地址的转换。
   + 相对地址（逻辑地址）：从$0$号开始，程序员只用知道逻辑地址，可有相同逻辑地址。
   + 绝对地址（物理地址）：内存物理单元的集合，是地址转换的最终地址。从逻辑地址到物理地址就是**地址重定位**。
4. 操作系统需要提供内存保护功能。保证各进程在各自存储空间内运行，互不干扰：
   + 在$CPU$中设置一对上、下限寄存器，存放进程的上、下限地址。进程的指令要访问某个地址时，$CPU$检查是否越界。
   + 采用重定位寄存器（又称基址寄存器，用于地址相加）和界地址寄存器（又称限长寄存器，用于地址比较）进行越界检查。重定位寄存器中存放的是进程的起始物理地址。界地址寄存器中存放的是进程的最大逻辑地址。

#### 程序载入过程

+ 程序运行过程：
  1. 编译：由编译程序将用户源代码编译成若干个目标模块（编译就是把高级语言翻译为机器语言）。
  2. 链接：由链接程序将编译后形成的一组目标模块，以及所需库函数链接在一起，形成一个完整的装入模块。
  3. 装入（装载）：由装入程序将装入模块装入内存运行。
+ 链接的三种方式（将独立的逻辑地址合并为完整的逻辑地址。）：
  1. 静态链接：在程序运行之前，先将各目标模块及它们所需的库函数连接成一个完整的可执行文件（装入模块），之后不再拆开。
  2. 装入时动态链接：将各目标模块装入内存时，边装入边链接的链接方式。
  3. 运行时动态链接：在程序执行中需要该目标模块时，才对它进行链接。其优点是便于修改和更新，便于实现对目标模块的共享。
+ 装入的三种方式（将逻辑地址转换为物理地址）：
  1. 绝对装入（单道程序阶段、未产生操作系统）：在编译时，如果知道程序将放到内存中的哪个位置，编译程序将产生绝对地址的目标代码。装入程序按照装入模块中的地址，将程序和数据装入内存。
     + 绝对装入只适用于单道程序环境。
     + 程序中使用的绝对地址，可在编译或汇编时给出，也可由程序员直接赋予。
     + 通常情况下都是编译或汇编时再转换为绝对地址。
  2. 静态重定位（可重定位装入）（多道批处理操作系统）：编译、链接后的装入模块的地址都是从$0$开始的，指令中使用的地址、数据存放的地址都是相对于起始地址而言的逻辑地址。可根据内存的当前情况，使用单独的装入程序将装入模块装入到内存的适当位置。装入时对地址进行“重定位”，将逻辑地址变换为物理地址（地址变换是在装入时一次完成的）。
     + 静态重定位的特点是在一个作业装入内存时，必须分配其要求的全部内存空间，如果没有足够的内存，就不能装入该作业。
     + 作业一旦进入内存后，在运行期间就不能再移动，也不能再申请内存空间。
  3. 动态重定位（动态运行时装入）（现代操作系统）：编译、链接后的装入模块的地址都是从$0$开始的。装入程序把装入模块装入内存后，并不会立即把逻辑地址转换为物理地址，而是把地址转换推迟到程序真正要执行时才进行。因此装入内存后所有的地址依然是逻辑地址。这种方式需要一个重定位寄存器的支持。
     + 采用动态重定位时允许程序在内存中发生移动。
     + 可将程序分配到不连续的存储区中：在程序运行前只需裂入它的部分代码即可投入运行，然后在程序运行期间，根据需要动态申请分配内存。
     + 便于程序段的共享，可以向用户提供一个比存储空间大得多的地址空间。

### 内存空间分配回收

分为：

+ 连续分配管理：
  + 单一连续分配。
  + 固定分区分配。
  + 动态分区分配。
+ 非连续分配管理（离散分配方式）：
  + 基本分页存储管理。
  + 基本分段存储管理。
  + 段页式存储管理。

连续分配是指为用户进程分配的必须是一个连续的内存空间。而非连续分配反之。

+ 内部碎片，是已经被分配出去（能明确指出属于哪个进程）却不能被利用的内存空间。
+ 外部碎片，是还没有被分配出去（不属于任何进程），但由于太小了无法分配给申请内存空间的新进程的内存空闲区域。

#### 单一连续分配

+ 内存被分为系统区和用户区。系统区通常位于内存的低地址部分，用于存放操作系统相关数据；用户区用于存放用户进程相关数据。
+ 内存中只能有一道用户程序，用户程序独占整个用户区空间。
+ 优点：
  + 实现简单。
  + 无外部碎片。
  + 可以采用覆盖技术扩充内存。
  + 因为只有一道程序，所以肯定不会访问越界，不一定需要采取内存保护（如早期的$PC$操作系统$Ms-Dos$）。
+ 缺点：
  + 只能用于单用户、单任务的操作系统中。
  + 有内部碎片。即分配给某进程的内存区域中，如果有些区域没有用上的部分
  + 存储器利用率极低。

#### 固定分区分配

+ 将整个用户空间划分为若干个固定大小的分区，在每个分区中只装入一道作业。
+ 分区的方式：
  + 分区大小相等：缺乏灵活性，但是很适合用于用一台计算机控制多个相同对象的场合。
  + 分区大小不等：增加了灵活性，可以满足不同大小的进程需求。根据常在系统中运行的作业大小情况进行划分（比如划分多个小分区、适量中等分区、少量大分区）。
+ 记录分区的方法：操作系统需要建立一个数据结构——**分区说明表**，来实现各个分区的分配与回收。每个表项对应一个分区，通常按分区大小排列。每个表项包括对应分区的天小、起始地址、状态（是否已分配）。
+ 分区分配过程：当某用户程序要装入内存时，由操作系统内核程序根据用户程序大小检索该表，从中找到一个能满足大小的、未分配的分区，将之分配给该程序，然后修改状态为“己分配”。
+ 优点：
  + 实现简单。
  + 无外部碎片。
+ 缺点：
  + 当用户程序太大时，可能所有的分区都不能满足需求，此时不得不采用覆盖技术来解决，但这又会降低性能。
  + 会产生内部碎片，内存利用率低。

#### 动态分区分配

+ 动态分区分配又称为可变分区分配。这种分配方式不会预先划分内存分区，而是在进程装入内存时根据进程的大小动态地建立分区，并使分区的大小正好适合进程的需要。因此系统分区的大小和数目是可变的。
+ 记录分区的方法：
  + 空闲分区表：每空闲分区对应表项。表项中包含分区号、分区大小、分区起始地址、分区状态等信息。
  + 空闲分区链：每个分区的起始部分和末尾部分分别设置前向指针和后向指针。起始部分处还可记录分区大小等信息。
+ 分配分区：
  + 当选择的分区分区大小大于分配空间，则分区大小相减，并修改起始地址。
  + 当选择的分区分区大小等于分配空间，则删除该表项。
+ 回收分区：
  + 若回收区的后面或前面有一个相邻的空闲分区则合并为一个。
  + 若回收区的后面和前面都有一个相邻的空闲分区则合并为一个。
  + 若回收区的后面或前面都没有一个相邻的空闲分区，则增加一个表项。
+ 动态分区分配会导致外部碎片，可用通过**紧凑**（拼凑）技术来移动进程位置合并空闲空间。

动态分区分配算法，为了解决动态分区分配方式中如何从多个空闲分区中选择一个分区分配：

1. 首次适应算法（$First\,Fit$）：
   + 算法思想：每次都从低地址开始查找，找到第一个能满足大小的空闲分区。
   + 如何实现：空闲分区以**地址**递增的次序排列。每次分配内存时顺序查找空闲分区链（或空闲分区表），找到大小能满足要求的第一个空闲分区。
   + 优点：
     + 首次适应算法每次都要从头查找，每次都需要检索低地址的小分区。但是这种规则也决定了当低地址部分有更小的分区可以满足需求时，会更有可能用到低地址部分的小分区，也会更有可能把高地址部分的大分区保留下来。
     + 算法开销小，每次分区后不需要对分区队列重新排序。
   + 缺点：
     + 造成低分区大量内存碎片。
     + 都要重复经过已经分配的底层区间，增加查找开销。
2. 最佳适应算法（$Best\,Fit$）：
   + 算法思想：由于动态分区分配是一种连续分配方式，为各进程分配的空间必须是连续的一整片区域。因此为了保证当“大进程”到来时能有连续的大片空间，可以尽可能多地留下大片的空闲区，即，优先使用更小的空闲区。
   + 如何实现：空闲分区按**容量**递增次序链接。每次分配内存时顺序查找空闲分区链（或空闲分区表），找到大小能满足要求的第一个空闲分区。当分配完后需要重新调整空闲分区链（或空闲分区表）。
   + 优点：容易保存大分区。
   + 缺点：
     + 每次都选最小的分区进行分配，会留下越来越多的、很小的、难以利用的内存块。因此这种方法会产生很多的外部碎片且很难查找回收。
     + 算法开销大，每次分区外需要对分区队列进程重新排序。
3. 最坏适应算法（$Worst\,Fit$）或最大适应算法（$Largest\,Fit$）：
   + 算法思想：为了解决最佳适应算法的问题――即留下太多难以利用的小碎片，可以在每次分配时优先使用最大的连续空闲区，这样分配后剩余的空闲区就不会太小，更方便使用。
   + 如何实现：空闲分区按**容量**递减次序链接。每次分配内存时顺序查找空闲分区链（或空闲分区表），找到大小能满足要求的第一个空闲分区。当分配完后需要重新调整空闲分区链（或空闲分区表）。
   + 优点：可用减少难以利用的小碎片。
   + 缺点：
     + 每次都选最大的分区进行分配，虽然可以让分配后留下的空闲区更大，更可用，但是这种方式会导致较大的连续空闲区被迅速用完。如果之后有“大进程”到达，就没有内存分区可用了。
     + 算法开销大，每次分区外需要对分区队列进程重新排序。
4. 临近适应算法（$Next\,Fit$）：
   + 算法思想:首次适应算法每次都从链头开始查找的。这可能会导致低地址部分出现很多小的空闲分区，而每次分配查找时，都要经过这些分区，因此也增加了查找的开销。如果每次都从上次查找结束的位置开始检索，就能解决上述问题。
   + 如何实现:空闲分区以地址递增的顺序排列（可排成一个循环链表）。每次分配内存时从上次查找结束的位置开始查找空闲分区链（或空闲分区表），找到大小能满足要求的第一个空闲分区。
   + 优点：减少了检索空闲分区的次数，提高了效率。
   + 缺点：邻近适应算法的规则可能会导致无论低地址、高地址部分的空闲分区都有相同的概率被使用，也就导致了高地址部分的大分区更可能被使用，划分为小分区，最后导致无大分区可用。

所以综合效果来看，首次适应算法>最佳适应法?临近适应法>最大适应法。

首次适应法和临近适应法只用简单查找，最佳适应法和最大适应法都需要对可用块进行排序和遍历。

#### 基本分页存储管理

分页：

+ 将内存空间分为一个个大小相等的分区（比如每个分区$4KB$），每个分区就是一个“页框”（$Page\,Frame$），或称“页帧”、“内存块”、“物理块”。每个页框有一个编号，即“页框号”（或者“内存块号”、“页帧号”、“物理块号”）页框号从$0$开始。
+ 将用户进程的地址空间也分为与页框大小相等的一个个区域，称为“页”（$Page$）或“页面”。每个页面也有一个编号，即“页号”，页号也是从$0$开始。所以页面不同于页框，是进程的逻辑概念。（进程的最后一个页面可能没有一个页框那么大。）
+ 页框不能太大，否则可能产生过大的内部碎片，也不能太小，否则回页面数过大，页表过长占用内存，同时增加硬件地址转换的开销，降低页面换入换出的效率。
+ 外存中也同样的单位进行划分，直接称为“块”（$Block$）。
+ 操作系统以页框为单位为各个进程分配内存空间。进程的每个页面分别放入一个页框中。也就是说，进程的页面与内存的页框有一一对应的关系。
+ 各个页面不必连续存放，也不必按先后顺序来，可以放到不相邻的各个页框中。
+ 为了方便计算页号、页内偏移量、页面大小一般设为$2$的整数幂。
+ 如果每个页面大小为$2^kB$，用二进制数表示逻辑地址，则末尾$k$位即为页内偏移量，其余部外就是页号。

地址结构：

+ 分为页面偏移量$W$和页号$P$两个部分。
+ 长度为$32$位，$0\sim11$是页内地址，即页内偏移量，每页大小为$4KB$。
+ $12\sim31$为页号，代表每个进程内的页的顺序，地址空间最多允许$2^{20}$页。

页表：

+ 因为允许将进程的各个页离散地存储在内存不同的物理块中，但系统应能保证进程的正确运行，即能在内存中找到每个页面所对应的物理块，所以为了能知道进程的每个页面在内存中存放的位置，操作系统要为每个进程建立一张页表，其中页表大小也与页面一样被页框约束。
+ 一个进程对应一张页表。
+ 进程的每一页对应一个页表项。
+ 每个页表项由“页号”和“块号”组成。完成从逻辑的页号向物理的块号的映射。
+ 页表记录进程页面和实际存放的内存块之间的对应关系。

假设某系统物理内存大小为$4GB$，页面大小为$4KB$，则每个页表项至少应该为多少字节？

+ 首先$4GB=2^{32}B$，$4KB=2^{12}B$。
+ 因此$4GB$的内存总共会被分为$2^{32}\div2^{12}= 2^{20}$个内存块，因此内存块号的范围应该是$0\cdots2^{20}-1$，因此至少要$20$个二进制位才能表示这么多的内存块号，因此至少要$3$个字节才够，因为每个字节$8$位，$3\times8=24>20$。
+ 所以最少为三个字节。
+ 深入来看，因为每页表项会顺序连续存储在内存中，若该页表在内存中存放的起始地址是$X$，则$M$号页对应的页表项存放在内存地址$X+3\times M$。
+ 同时因为页面大小为$4KB$，所以每个页框（即可用存放的最大值）大小为$4\times1024\div3=1365$个页表项，但是此时每页会余下$4\times1024\mod3=1$页内碎片，所以一定会在中间空出内存，所以$1365$号页在页框约束下会在新的下一个页框存储，表项会存放在$X+3\times1365+1$处。这时候地址公式就不管用了。
+ 而如果每个页表项占$4$字节，则每个页框刚好能放下$1024$个页表项，从而没有余数，能减少查找的麻烦。
+ 所以理论上$3B$就能表示内存块的范围，但是为了方便页表查找（对齐），实际上会多一些字节，使得每个页面能装下整数个页表项。

页式基本地址变换机构：

+ 可用借助页表进行转换，通常会在系统中设置一个页表寄存器（$PTR$），存放页表在内存中的起始地址$F$和页表长度$M$（即这个进程里有多少页）。进程未执行时，页表的始址和页表长度放在进程控制块（$PCB$）中，当进程被调度时，操作系统内核会把它们放到页表寄存器中。
+ 在页式存储管理的系统中时，只用确定页面大小和逻辑结构就能得到物理地址。

基本地址变换机构需要先查询页表，再查询内存两次操作：

1. 要算出逻辑地址$A$对应的页号$P$与页内偏移量$W$。页号$P$=逻辑地址$A$÷页面长度$L$（取除法的整数部分）。页内偏移量$W$=$A$逻辑地址%页面长度$L$（取除法的余数部分）。
2. 检测页号$P$是否越界。如果页号$P$大于等于页表长度$M$，则内中断（因为页号从$0$开始，页表长度至少为$1$，从而$P=M$页会越界）。
3. 根据页表寄存器中的页表项地址$PA$=页表起始地址$F$+页号$P$×页表项长度$PL$，得到页表中对应的页表项，从而确定页面存放的内存块号$B$。
   + 页表长度指的是这个页表中总共有几个页表项，即总共有几个页。
   + 页表项长度指的是每个页表顶占多大的存储空间。
   + 页面大小指的是一个页面占多大的存储空间。
4. 最后物理地址$E$=内存块号$B$×页面大小$L$+页内偏移量$W$（如果内存块号和业内偏移量用二进制表示，则直接拼接起来就是最终物理地址了）。

#### 具有快表的地址变换机构

+ 时间局部性：如果执行了程序中的某条指令，那么不久后这条指令很有可能再次执行;如果某个数据被访问过，不久之后该数据很可能再次被访问（因为程序中存在大量的循环）。
+ 空间局部性：一旦程序访问了某个存储单元，在不久之后，其附近的存储单元也很有可能被访问（因为很多数据在内存中都是连续存放的）。
+ 快表，又称相连寄存器（$TLB$），是一种访问速度比内存快很多的高速缓冲存储器，用来存放当前访问的若干页表项，以加速地址变换的过程。与此对应，内存中的页表常称为慢表。
+ 由于查询快表的速度比查询页表的速度快很多，因此只要快表命中，就可以节省很多时间。
+ 因为局部性原理，一般来说快表的命中率可以达到$90\%$以上。
+ 快表的地址变换过程：
  1. $CPU$给出逻辑地址，由某个硬件算得页号、页内偏移量，将页号与快表中的所有页号进行比较。
  2. 如果找到匹配的页号（即命中），说明要访问的页表项在快表中有副本，则直接从中取出该页对应的内存块号，再将内存块号与页内偏移量拼接形成物理地址，最后，访问该物理地址对应的内存单元。因此，若快表命中，则访问某个逻辑地址仅需一次访存即可。
  3. 如果没有找到匹配的页号，则需要访问内存中的页表，找到对应页表项，得到页面存放的内存块号，再将内存块号与页内偏移量拼接形成物理地址，最后，访问该物理地址对应的内存单元。同时将其存入快表，以便后面可能的再次访问。但若快表已满，则必须按照一定的页面置换算法对旧的页表项进行替换。因此，若快表未命中，则访问某个逻辑地址需要两次访存。

#### 多级页表

单级页表的缺点：

1. 单级页表的所有表项都必须连续存储，实现起来比较困难。
2. 进程在一段时间内只需要访问少数页面就可用正常运行，无需整个页表都常驻内存。

将页表进行分组离散地放入内存块，并为离散分组的页表再建立一张页表，称为页目录表、外层页表或顶层页表，页目录表页保存序号和内存块号两项。两级页表结构的逻辑地址结构分为一级页号、二级页号和页内偏移量三项。

地址转换过程：

1. 按照地址结构将逻辑地址拆分成三部分。
2. 从$PCB$中读出页目录表始址，再根据一级页号查页目录表，找到下一级页表在内存中的存放位置。
3. 根据二级页号查表，找到最终想访问的内存块号。
4. 结合页内偏移量得到物理地址。

注意点：

+ 若采用多级页表机制，则各级页表的大小不能超过**一个**页面。因为顶级页表只能有一个，否则一个页面放不下页表项。如果一个页面框放不下就需要多个页面框，而如果需要多个页面框就会导致多个页面框有相同的页号，就不能区分出哪个是顶级页表。
+ 两级页表的访存次数分析（假设没有快表机构）：
  1. 第一次访存：访问内存中的页目录表。
  2. 第二次访存：访问内存中的二级页表。
  3. 第三次访存：访问目标内存单元。

#### 基本分段存储管理

+ 进程的地址空间：按照程序自身的逻辑关系划分为若干个不等长的段，每个段都有一个段名（在低级语言中，程序员使用段名来编程），每段从$0$开始编址。
+ 内存分配规则：以段为单位进行分配，每个段在内存中占据连续空间，但各段之间可以不相邻。
+ 分段系统的逻辑地址结构由段号（段名$S$）和段内地址（段内偏移量$W$）所组成。
+ 段号的位数决定了每个进程最多可以分几个段段内地址位数决定了每个段的最大长度是多少。
+ 页式系统中逻辑地址的页号和页内偏移量对用户透明，但是段式系统段号和段内偏移量必须用户显式提供，一般由编译程序完成。
+ 程序分多个段，各段离散地装入内存，为了保证程序能正常运行，就必须能从物理内存中找到各个逻辑段的存放位置。为此，需为每个进程建立一张段映射表，简称“段表”：
  1. 每个段对应一个段表项，其中记录了段号、该段在内存中的起始位置（又称“基址”）和段的长度。
  2. 各个段表项的长度是相同的。

某系统按字节寻址，采用分段存储管理，逻辑地址结构为（段号$16$位，段内地址$16$位），因此用$16$位即可表示最大段长。物理内存大小为$4GB$（可用$32$位表示整个物理内存地址空间)。因此，可以让每个段表项占$16+32=48$位，即$6B$。由于段表项长度相同，因此段号可以是隐含的，不占存储空间。若段表存放的起始地址为$R$，则K号段对应的段表项存放的地址为$R+K\times6$，段号并不占内存空间。

段式存储地址变换过程：

1. 根据逻辑地址得到段号$S$和段内地址$W$。
2. 根据段号$S$是否大于等于段表寄存器的段表长度$M$，判断上是否越界，若是则越界中断。（段表长度至少为$1$）。
3. 查询段表寄存器中的段表，找到对应的段表项，段表项分为段长$C$和段基址$B$，段表项的存放地址$SA$=段表始值$F$+段号$S$×段表项长度$SL$。
4. 检测段内地址$W$是否大于等于段长$C$（这也是段式存储与页式存储的最大不同，段式存储不定长），若是则越界中断。
5. 计算得到物理地址$E$=段基址$B$+段内地址$W$。

分段、分页管理的区别：

&nbsp;|与信息的关系|主要目的|是否对用户可见|长度|用户进程地址空间
:----:|:---------:|:-----:|:-----------:|:--:|:------------:
分页|信息的物理单位|实现离散分配，提高内存利用率|仅仅是系统管理上的需要，完全是系统行为，对用户是不可见的|页的大小固定且由系统决定|是一维的，程序员只需给出一个记忆符即可表示一个地址
分段|信息的逻辑单位|更好地满足用户需求|一个段通常包含着一组属于一个逻辑模块的信息。分段对用户是可见的，用户编程时需要显式地给出段名|不固定，决定于用户编写的程序|二维的，程序员在标识一个地址时，既要给出段名，也要给出段内地址。

+ 分段比分页更容易实现信息的共享和保护。
+ 只需让各进程的段表项指向同一个段即可实现共享。
+ 不能被修改的代码称为纯代码或可重入代码（不属于临界资源），这样的代码是可以共享的。可修改的代码是不能共享的（比如，有一个代码段中有很多变量，各进程并发地同时访问可能造成数据不一致）。
+ 分页时页面不是按逻辑模块划分的。这就很难实现共享。

#### 段页式存储管理

&nbsp;|优点|缺点
:----:|:-:|:--:
分页管理|内存空间利用率高，不会产生外部碎片，只会有少量的页内碎片|不方便按照逻辑模块实现信息的共享和保护
分段管理|很方便按照逻辑模块实现信息的共享和保护|如果段长过大，为其分配很大的连续空间会很不方便。另外,段式管理会产生外部碎片

+ 段页式存储的过程：
  1. 将进程按逻辑模块分段，再将各段分页（如每个页面$4KB$）。
  2. 再将内存空间分为大小相同的内存块/页框/页帧/物理块。
  3. 进程前将各页面分别装入各内存块中。
+ 段页式系统逻辑地址结构：
  + 由段号$S$、页号$P$、页内地址$W$（页内偏移量）组成。段号的位数决定了每个进程最多可以分几个段。页号位数决定了每个段最大有多少页。页内偏移量决定了页面大小、内存块大小是多少。
  + 系统含有一个段表寄存器，指出作业的段表始址和段表长度。
  + “分段”对用户是可见的，程序员编程时需要显式地给出段号、段内地址。而将各段“分页”对用户是不可见的。系统会根据段内地址自动划分页号和页内偏移量。
  + 因此段页式管理的地址结构是二维的。
+ 分为段表和页表：
  + 每个段对应一个段表项，每个段表项由段号、页表长度、页表存放块号（页表起始地址）组成。每个段表项长度相等，段号是隐含的。
  + 每个页面对应一个页表项，每个页表项由页号、页面存放的内存块号组成。每个页表项长度相等，页号是隐含的。
  + 一个进程对应一个段表，对应一个或多个页表项，一个段表项就相当于只存一个的页表寄存器。

段页式存储地址变换过程：

1. 根据逻辑地址得到段号$S$、页号$P$和页内偏移量$W$。
2. 根据段号$S$是否大于等于段表寄存器的段表长度$M$，判断上是否越界，若是则越界中断。（段表长度至少为$1$）。
3. 查询段表寄存器中的段表，找到对应的段表项。段表项分为段号$S$和页表长度$L$、段基址$F$和页表存放块号$D$。段表项的存放地址$SA$=段表始值$F$+段号$S$×段表项长度$SL$。
4. 检测页号$P$是否大于等于页表项长度$PL$，若是则越界中断。
5. 根据页表存放块号$D$和页号$P$查询页表，找到对应页表项。页表项分为页号$P$和内存块号$B$。
6. 计算得到物理地址$E$=内存块号$B$+页内偏移量$W$。
7. 也可引入快表机构，用段号和页号作为查询快表的关键字。若快表命中则仅两次访存。

### 内存空间扩充

内存空间的扩充（用容量小的内存运行大的程序）有三种技术：

1. 覆盖技术。
2. 交换技术。
3. 虚拟存储技术。使用覆盖和交换可以实现虚拟存储。

前两种技术不是重点。

#### 覆盖技术

覆盖技术在同一个程序或进程中执行。

+ 覆盖技术的思想：将程序分为多个段（多个模块）。常用的段常驻内存，不常用的段在需要时调入内存。
+ 内存中分为一个“固定区”和若干个“覆盖区”。
+ 需要常驻内存的段放在“固定区”中，调入后就不再调出（除非运行结束）。
+ 不常用的段放在“覆盖区”，需要用到时调入内存，用不到时调出内存。
+ 按照自身代码逻辑结构，让那些不可能同时被访问的程序段共享同一个覆盖区。
+ 缺点：
  + 必须由程序员声明覆盖结构，操作系统完成自动覆盖。
  + 对用户不透明，增加了用户编程负担。

#### 交换技术

交换技术在不同进程或作业之间进行的。

+ 交换（对换）技术的设计思想：内存空间紧张时，系统将内存中某些进程暂时换出外存，把外存中某些已具备运行条件的进程换入内存（进程在内存与磁盘间动态调度）。
+ 交换的位置：具有对换功能的操作系统中，通常把磁盘空间分为文件区和对换区两部分。文件区主要用于存放文件，主要追求存储空间的利用率，因此对文件区空间的管理采用离散分配方式;对换区空间只占磁盘空间的小部分，被换出的进程数据就存放在对换区。由于对换的速度直接影响到系统的整体速度，因此对换区空间的管理主要追求换入换出速度，因此通常对换区采用连续分配方式（学过文件管理章节后即可理解）。总之，对换区的$I/O$速度比文件区的更快。
+ 交换的时机：交换通常在许多进程运行且内存吃紧时进行，而系统负荷降低就暂停。例如在发现许多进程运行时经常发生缺页，就说明内存紧张，此时可以换出一些进程；如果缺页率明显下降、就可以暂停换出。
+ 交换进程的选择：
  + 可优先换出阻塞进程。
  + 可换出优先级低的进程。
  + 为了防止优先级低的进程在被调入内存后很快又被换出，考虑进程在内存的驻留时间。
+ 暂时换出外存等待的进程状态是挂起状态。
+ 处理机调度的中级调度（内存调度）就是交换技术的实现。进程的$PCB$常驻内存。

覆盖用于同一个进程或程序中，二交换用于不同进程或作业之间。

## 虚拟内存管理

虚拟存储技术使用局部性原理，用于对内存空间进行扩充。

传统存储管理特性：

+ 一次性：作业必须一次性全部装入内存后才能开始运行。这会造成两个问题：
  1. 作业很大时，不能全部装入内存，导致大作业无法运行。
  2. 当大量作业要求运行时，由于内存无法容纳所有作业，因此只有少量作业能运行，导致多道程序并发度下降。
+ 驻留性：一旦作业被装入内存，就会一直驻留在内存中，直至作业运行结束。事实上，在一个时间段内，只需要访问作业的一小部分数据即可正常运行，这就导致了内存中会驻留大量的、暂时用不到的数据，浪费了宝贵的内存资源。

高速缓冲技术的思想：将近期会频繁访问到的数据放到更高速的存储器中，暂时用不到的数据放在更低速存储器中。

### 虚拟内存的基本概念

+ 基于局部性原理，在程序装入时，可以将程序中很快会用到的部分装入内存，暂时用不到的部分留在外存，就可以让程序开始执行。
+ 在程序执行过程中，当所访问的信息不在内存时，由操作系统负责将所需信息从外存调入内存，然后继续执行程序。
+ 若内存空间不够，由操作系统负责将内存中暂时用不到的信息换出到外存。
+ 在操作系统的管理下，在用户看来似乎有一个比实际内存大得多的内存，这就是虚拟内存。

+ 虚拟内存的最大容量是由计算机的**地址结构**（$CPU$寻址范围）确定的。
+ 虚拟内存的实际容量=$\min$(内存和外存容量之和，$CPU$寻址范围)。
+ 某计算机地址结构为$32$位，按字节编址，内存大小为$512MB$，外存大小为$2GB$。则虚拟内存的最大容量为$2^{32}B=4GB$，而实际内存是$2GB+512MB$。

虚拟内存特征：

+ 多次性：无需在作业运行时一次性全部装入内存，而是允许被分成多次调入内存。
+ 对换性：在作业运行时无需一直常驻内存，而是允许在作业运行过程中，将作业换入、换出。
+ 虚拟性：从逻辑上扩充可内存的容量，使用户看到的内存容量，远大于实际的容量。

虚拟内存实现需要基于离散分配的内存管理方式基础上。所以根据传统的非连续分配存储管理，可以将虚拟存储的实现分为：

+ 请求分页存储管理。
+ 请求分段存储管理。
+ 请求段页式存储管理。

其主要区别是：在程序执行过程中，当所访问的信息不在内存时，由操作系统负责将所需信息从外存调入内存，然后继续执行程序；若内存空间不够，由操作系统负责将内存中暂时用不到的信息换出到外存。所以操作系统需要提供请求调页或请求调段的功能与页面置换或段置换的功能。

不管哪种方式，都需要有一定的硬件支持。一般需要的支持有以下几个方面：

+ 一定容量的内存和外存。
+ 页表机制（或段表机制），作为主要的数据结构。
+ 中断机构，当用户程序要访问的部分尚未调入内存时，则产生中断。
+ 地址变换机构，逻辑地址到物理地址的变换。

### 请求分页管理方式

是最常用的实现虚拟存储器的方式。基于基本分页系统，增加了请求调页功能和页面置换功能。

#### 页表机制

+ 与基本分页管理相比，请求分页管理中，为了实现“请求调页”，操作系统需要知道每个页面是否已经调入内存：如果还没调入，那么也需要知道该页面在外存中存放的位置。
+ 当内存空间不够时，要实现“页面置换”，操作系统需要通过某些指标来决定到底换出哪个页面：有的页面没有被修改过，就不用再浪费时间写回外存。有的页面修改过，就需要将外存中的旧数据覆盖，因此，操作系统也需要记录各个页面是否被修改的信息。

其中基本分页存储管理的页表项分为隐藏的页号与内存块号，所以请求分页管理存储的页表项分为：

+ 页号（隐藏）。
+ 内存块号（物理块号）。
+ 状态位$P$：表示是否已调入内存，$0$表示未调入，$1$表示已调入，供访问时参考。
+ 访问字段$A$：可记录最近被访问过几次，或记录上次访问的时间，供置换算法选择换出页面时参考。
+ 修改位$M$：页面调入内存后是否被修改过，$0$表示没有，$1$表示修改过。
+ 外存地址：页面在外存中的存放地址，一般是物理块号，供调入参考。

#### 缺页中断机构

缺页中断是因为当前执行的指令想要访问的目标页面未调入内存而产生的，因此属于内中断的故障。一条指令在执行期间可能出现多次缺页中断。

1. 在请求分页系统中，每当要访问的页面不在内存时（状态位为$0$），便产生一个缺页中断，然后由操作系统的缺页中断处理程序处理中断。
2. 此时缺页的进程阻塞，放入阻塞队列。
3. 如果内存中有空闲块，则为进程分配一个空闲块，将所缺页面装入该块，并修改页表中相应的页表项，如内存块号。
4. 如果内存中没有空闲块，则由页面置换算法选择一个页面淘汰，若该页面在内存期间被修改过，则要将其写回外存。未修改过的页面不用写回外存。
5. 调页完成后再将其唤醒，放回就绪队列。

#### 地址变换机构

与普通页表的地址变换机构不同的是，增加了请求调页、页面置换和请求修改内容三个部分。

1. 要算出逻辑地址$A$对应的页号$P$与页内偏移量$W$。页号$P$=逻辑地址$A$÷页面长度$L$（取除法的整数部分）。页内偏移量$W$=$A$逻辑地址%页面长度$L$（取除法的余数部分）。
2. 检测页号$P$是否越界。如果页号$P$大于等于页表长度$M$，则内中断（因为页号从$0$开始，页表长度至少为$1$，从而$P=M$页会越界）。
3. 在快表中查找对应页号，如果找到匹配的页号（即命中），说明要访问的页表项在快表中有副本，则直接从中取出该页对应的内存块号，再将内存块号与页内偏移量拼接形成物理地址，最后，访问该物理地址对应的内存单元。因此，若快表命中，则访问某个逻辑地址仅需一次访存即可。
4. 快表中有的页面一定是在内存中的。若某个页面被换出外存，则快表中的相应表项也要删除，否则可能访问错误的页面。
5. 如果没有找到匹配的页号，则需要访问内存中的页表。
6. 找到对应页表项后，若对应页面未调入内存，则产生缺页中断，之后由操作系统的缺页中断处理程序进行处理，包括调页与页面置换。
7. 根据页表寄存器中的页表项地址$PA$=页表起始地址$F$+页号$P$×页表项长度$PL$，得到页表中对应的页表项，从而确定页面存放的内存块号$B$。
8. 最后物理地址$E$=内存块号$B$×页面大小$L$+页内偏移量$W$（如果内存块号和业内偏移量用二进制表示，则直接拼接起来就是最终物理地址了）。

简单而言，在具有快表机构的请求分页系统中，访问一个逻辑地址时，若发生缺页，则地址变换步骤是：查快表（未命中）――查慢表（发现未调入内存）—―调页（调入的页面对应的表项会直接加入快表）―—查快表（命中）――访问目标内存单元。

### 页面置换算法

假设一个例子，系统为某进程分配了三个内存块，并考虑到有一下页面号引用串（会依次访问这些页面）：$7,0,1,2,0,3,0,4,2,3,0,3,2,1,2,0,1,7,0,1$。使用算法如何置换？

#### 最佳置换算法

即$OPT$算法。

每次选择淘汰的页面将是以后永不使用，或者在最长时间内不再被访问的页面，这样可以保证最低的缺页率。这是不可能实际实现的，因为不可能预测本进程所有页面请求，一般用来评价其他算法效果。

访问页面|7|0|1|2|0|3|0|4|2|3|0|3|2|1|2|0|1|7|0|1
:------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:
内存块1|7|7|7|2| |2| |2| | |2| | |2| | | |7| |
内存块2| |0|0|0| |0| |4| | |0| | |0| | | |0| |
内存块3| | |1|1| |3| |3| | |3| | |1| | | |1| |
是否缺页|√|√|√|√| |√| |√| | |√| | |√| | | |√| |

所以缺页中断发生了$9$次，页面置换发生了$6$次，缺页率$=9\div20=45\%$。

#### 先进先出置换算法

即$FIFO$算法。

每次选择淘汰的页面是最早进入内存的页面。只考虑进入时间而不考虑访问次数。

实现方法：把调入内存的页面根据调入的先后顺序排成一个队列，需要换出页面时选择队头页面即可。队列的最大长度取决于系统为进程分配了多少个内存块。

访问页面|7|0|1|2|0|3|0|4|2|3|0|3|2|1|2|0|1|7|0|1
:------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:
内存块1|7|7|7|2| |2|2|4|4|4|0| | |0|0| | |7|7|7
内存块2| |0|0|0| |3|3|3|2|2|2| | |1|1| | |1|0|0
内存块3| | |1|1| |1|0|0|0|3|3| | |3|2| | |2|2|1
是否缺页|√|√|√|√| |√|√|√|√|√|√| | |√|√| | |√|√|√

缺页了$15$次，置换了$12$次，所以缺页率为$15\div20=75\%$。

如给定一串页面号：$3,2,1,0,3,2,4,3,2,1,0,4$，使用$FIFO$算法进行置换，会发现分配三个内存块缺页次数为$9$次，而分配四个内存块缺页次数为$10$次。这种当为进程分配的物理块数增大时，缺页次数不减反增的异常现象就是**Belady异常**。

只有$FIFO$算法会产生$Belady$异常，使用队列实现，是队列类算法。另外，$FIFO$算法虽然实现简单，但是该算法与进程实际运行时的规律不适应，因为先进入的页面也有可能最经常被访问。因此该算法性能差。

#### 最近最久未使用置换算法

即$LRU$算法。

每次淘汰的页面是最近最久未使用的页面。（是检查过去的，而$OPT$是检查未来的）

所以这里就需要使用页表中访问字段这一项，用来记录该页面自上次被访问以来所经历的时间$t$，当需要淘汰时，就选择$t$值最大的，即最近最久未使用的页面。

访问页面|7|0|1|2|0|3|0|4|2|3|0|3|2|1|2|0|1|7|0|1
:------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:
内存块1|7|7|7|2| |2| |4|4|4|0| | |1| |1| |1| |
内存块2| |0|0|0| |0| |0|0|3|3| | |3| |0| |0| |
内存块3| | |1|1| |3| |3|2|2|2| | |2| |2| |7| |
是否缺页|√|√|√|√| |√| |√|√|√|√| | |√| |√| |√| |

所以缺页中断发生了$12$次，页面置换发生了$9$次，缺页率$=12\div20=60\%$。

在手动做题时，若需要淘汰页面，可以逆向检查此时在内存中的几个页面号。在逆向扫描过程中最后一个出现的页号就是要淘汰的页面。

该算法的实现需要专门的硬件支持，如寄存器和栈，虽然算法性能好，最接近$OPT$算法，但是实现困难，开销大。

#### 时钟置换算法

即$CLOCK$算法。是一种性能和开销较均衡的算法，又称最近未用算法$NRU$。

实现方法：

1. 为每个页面设置一个访问位：访问位为$1$，表示最近访问过；访问位为$0$，表示最近没访问过。
2. 再将内存中的页面都通过链接指针链接成一个循环队列。
3. 当某页被访问时，其访问位置为$1$。当需要淘汰一个页面时，只需检查页的访问位。如果是$0$，就选择该页换出。
4. 如果是$1$，则将它置为$0$，暂不换出，继续检查下一个页面，若第一轮扫描中所有页面都是$1$，则将这些页面的访问位依次置为$0$后，再进行第二轮扫描。
5. 第二轮扫描中一定会有访问位为$0$的页面，因此简单的$CLOCK$算法选择一个淘汰页面，最多会经过两轮扫描。

值得注意的是：访问和置换是不同的，扫描指针用于置换，只有缺页中断才会发生指向的变化，而访问和修改数据是另一个指针，会随着访问而不断移动，且是访问指针会影响访问位而不是扫描指针。

假设系统为某进程分配了五个内存块，并考虑到有以下页面号引用串：$1,3,4,2,5,6,3,4,7$。

首先第一步将$1,3,4,2,5$链接成为循环队列，代表置入内存的内存块，此时访问位全部为$1$，表示全部被访问并置入内存。

然后访问到$6$，需要置换出一个页面，所以循环访问$1,3,4,2,5$，将访问位全部改为$0$。

第二轮循环开始，第一个的$1$访问位为$0$，则将$1$换出，$6$换入，访问位置为$1$，变为$6,3,4,2,5$，扫描指针指向$3$。

接着访问$3$和$4$，将访问位都置为$1$，然后是$7$，因为$7$不在内存中，从$3$开始扫描，$3$、$4$访问位为$1$，而$2$访问位为$0$，所以$7$换入$2$换出，$7$的访问位置为$1$，变为$6,3,4,7,5$，扫描指针指向$5$。

#### 改进型时钟置换算法

简单的时钟置换算法仅考虑到一个页面最近是否被访问过。事实上，如果被淘汰的页面没有被修改过，就不需要执行$I/O$操作写回外存。只有被淘汰的页面被修改过时，才需要写回外存。

因此，除了考虑一个页面最近有没有被访问过之外，操作系统还应考虑页面有没有被修改过。在其他条件都相同时，应优先淘汰没有修改过的页面，避免$I/O$操作。这就是改进型的时钟置换算法的思想。

需要利用到修改位，修改位$=0$，表示页面没有被修改过；修改位$=1$，表示页面被修改过。

为方便讨论，用（访问位，修改位）的形式表示各页面状态。如$(1,1)$表示一个页面近期被访问过，且被修改过。

算法规则：

1. 将所有可能被置换的页面排成一个循环队列。初始的(访问位，修改位)可能是任何的$10$组合，这和简单时钟算法初始为全$0$不同。
2. 第一轮（没访问没修改）：从当前位置开始扫描到第一个$(0,0)$的帧用于替换。本轮扫描不修改任何标志位。
3. 第二轮（没访问有修改）：若第一轮扫描失败，则重新扫描，查找第一个$(0,1)$的帧用于替换。本轮将所有扫描过的帧访问位设为$0$。
4. 第三轮（有访问没修改）：若第二轮扫描失败，则重新扫描，查找第一个$(0,0)$的帧用于替换。本轮扫描不修改任何标志位。
5. 第四轮（有访问有修改）：若第三轮扫描失败，则重新扫描，查找第一个$(0,1)$的帧用于替换。需要第四轮扫描只有全部页面都被访问都被修改过这一种情况。
6. 由于第二轮已将所有帧的访问位设为$0$，因此经过第三轮、第四轮扫描一定会有一个帧被选中，因此改进型$CLOCK$置换算法选择一个淘汰页面最多会进行四轮扫描。

+ $(0,0)$：最近没有使用使用也没有修改，最佳状态。
+ $(0,1)$：修改过但最近没有使用，将会被写。
+ $(1,0)$：使用过但没有被修改，下一轮将再次被用。
+ $(1,1)$：使用过也修改过，下一轮页面置换最后的选择。

### 页面分配策略

#### 页面分配、置换策略

+ 驻留集：指请求分页存储管理中给进程分配的物理块的集合（即允许进入内存运行的最大物理块数量）。
+ 在采用了虚拟存储技术的系统中，驻留集大小一般小于进程的总大小。
+ 若驻留集太小，会导致缺页频繁，系统要花大量的时间来处理缺页，实际用于进程推进的时间很少；驻留集太大，又会导致多道程序并发度下降，资源利用率降低。所以应该选择一个合适的驻留集大小。

页面分配策略：

+ 固定分配：操作系统为每个进程分配一组固定数目的物理块，在进程运行期间不再改变。即，驻留集大小不变。
+ 可变分配：先为每个进程分配一定数目的物理块，在进程运行期间，可根据情况做适当的增加或减少。即，驻留集大小可变。

页面置换策略：

+ 局部置换：发生缺页时只能选进程自己的物理块进行置换。
+ 全局置换：可以将操作系统保留的空闲物理块分配给缺页进程，也可以将别的进程持有的物理块置换到外存，再分配给缺页进程。

所以结合页面分配策略与页面置换策略，可以得到：

+ 固定分配局部置换：
  + 系统为每个进程分配一定数量的物理块，在整个运行期间都不改变。若进程在运行中发生缺页，则只能从该进程在内存中的页面中选出一页换出，然后再调入需要的页面。
  + 这种策略的缺点是：很难在刚开始就确定应为每个进程分配多少个物理块才算合理。
  + 采用这种策略的系统可以根据进程大小、优先级、或是根据程序员给出的参数来确定为一个进程分配的内存块数。
+ 可变分配全局置换：
  + 刚开始会为每个进程分配一定数量的物理块。操作系统会保持一个空闲物理块队列。当某进程发生缺页时，从空闲物理块中取出一块分配给该进程；若已无空闲物理块，则可选择一个未锁定的页面换出外存，再将该物理块分配给缺页的进程。
  + 采用这种策略时，只要某进程发生缺页，都将获得新的物理块，仅当空闲物理块用完时，系统才选择一个未锁定（即可以调出内存）的页面调出。被选择调出的页可能是系统中任何一个进程中的页，因此这个被选中的其他进程拥有的物理块会减少，其他进程缺页率会增加。
+ 可变分配局部置换：
  + 刚开始会为每个进程分配一定数量的物理块。当某进程发生缺页时，只允许从该进程自己的物理块中选出一个进行换出外存。
  + 如果进程在运行中频繁地缺页，系统会为该进程多分配几个物理块，直至该进程缺页率趋势适当程度；反之，如果进程在运行中缺页率特别低，则可适当减少分配给该进程的物理块。
+ 没有固定分配全局置换策略，因为全局置换意味着进程拥有的物理块数量必然会改变，因此不可能是固定分配。

#### 调入页面

调入页面时机：

+ 预调页策略（运行前）：根据局部性原理，一次调入若干个相邻的页面可能比一次调入一个页面更高效。但如果提前调入的页面中大多数都没被访问过，则又是低效的。因此可以预测不久之后可能访问到的页面，将它们预先调入内存，但目前预测成功率只有$50\%$左右。故这种策略主要用于进程的首次调入，由程序员指出应该先调入哪些部分。
+ 请求调页策略（运行时）：进程在运行期间发现缺页时才将所缺页面调入内存。由这种策略调入的页面一定会被访问到，但由于每次只能调入一页，而每次调页都要磁盘$I/O$操作，因此$I/O$开销较大。

调入页面位置：

+ 系统拥有足够的对换区空间：页面的调入、调出都是在内存与对换区之间进行，这样可以保证页面的调入、调出速度很快。在进程运行前，需将进程相关的数据从文件区复制到对换区。
+ 系统缺少足够的对换区空间：凡是不会被修改的数据都直接从文件区调入，由于这些页面不会被修改，因此换出时不必写回磁盘，下次需要时再从文件区调入即可。对于可能被修改的部分，换出时需写回磁盘对换区，下次需要时再从对换区调入。
+ $UNIX$方式：运行之前进程有关的数据全部放在文件区，故未使用过的页面，都可从文件区调入。若被使用过的页面需要换出，则写回对换区，下次需要时从对换区调入。

### 抖动（颠簸现象）

+ 刚刚换出的页面马上又要换入内存，刚刚换入的页面马上又要换出外存，这种频繁的页面调度行为称为抖动，或颠簸。
+ 产生抖动的主要原因是进程频繁访问的页面数目高于可用的物理块数（分配给进程的物理块不够）。所以需要合适的物理块数量。
+ 工作集：指在某段时间间隔里，进程实际访问页面的集合。

### 工作集

+ 窗口尺寸就是驻留集的大小，约束工作集大小，工作集大小小于等于窗口尺寸，实际应用中，操作系统可以统计进程的工作集大小，根据工作集大小给进程分配若干内存块。如窗口尺寸为$5$，经过一段时间的监测发现某进程的工作集最大为$3$，那么说明该进程有很好的局部性，可以给这个进程分配$3$个以上的内存块即可满足进程的运行需要。
+ 驻留集大小不能小于工作集大小，否则会产生抖动现象。
+ 基于局部性原理可知，进程在一段时间内访问的页面与不久之后会访问的页面是有相关性的。因此，可以根据进程近期访问的页面集合（工作集）来设计一种页面置换算法――选择一个不在工作集中的页面进行淘汰。
