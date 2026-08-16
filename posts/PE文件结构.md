---
title: PE文件结构
date: 2024-01-21
tag: 免杀
summary: PE 文件结构学习笔记：DOS 头、NT 头、节表、地址转换、dump，以及节表注入和导入表注入。
cover: ./assets/cat-angry.png
---

# PE文件结构

# PE头

```
IMAGE_DOS_HEADER
IMAGE_NT_HEADERS
	IMAGE_FILE_HEADER
	IMAGE_OPTIONAL_HEADER
	IMAGE_DATA_DIRECTORY[1]
	
IMAGE_SECTION_HEADER[1]

[1]代表可变
```



## IMAGE_DOS_HEADER

```
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
    WORD   e_magic;                     // Magic number
    WORD   e_cblp;                      // Bytes on last page of file
    WORD   e_cp;                        // Pages in file
    WORD   e_crlc;                      // Relocations
    WORD   e_cparhdr;                   // Size of header in paragraphs
    WORD   e_minalloc;                  // Minimum extra paragraphs needed
    WORD   e_maxalloc;                  // Maximum extra paragraphs needed
    WORD   e_ss;                        // Initial (relative) SS value
    WORD   e_sp;                        // Initial SP value
    WORD   e_csum;                      // Checksum
    WORD   e_ip;                        // Initial IP value
    WORD   e_cs;                        // Initial (relative) CS value
    WORD   e_lfarlc;                    // File address of relocation table
    WORD   e_ovno;                      // Overlay number
    WORD   e_res[4];                    // Reserved words
    WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
    WORD   e_oeminfo;                   // OEM information; e_oemid specific
    WORD   e_res2[10];                  // Reserved words
    LONG   e_lfanew;                    // File address of new exe header
  } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

重点关注



```
WORD   e_magic;
LONG   e_lfanew;
```

e_magic文件标识，其他的都是16位时代的东西，现在都能随便改



![image-20240120061826380](./assets/posts/PE文件结构/image-20240120061826380.png)

这个就是e_magic，word是两个字节，long是四个字节，前18占60个字节



![image-20240120062431079](./assets/posts/PE文件结构/image-20240120062431079.png)

这个是e_lfanew，所存储的地址是nt头IMAGE_NT_HEADERS



## IMAGE_NT_HEADERS

![image-20240120063638967](./assets/posts/PE文件结构/image-20240120063638967.png)

这里分为64位和32位的，结构是一样的但是长度不一样



![image-20240120063840927](./assets/posts/PE文件结构/image-20240120063840927.png)

主要有三个成员



![image-20240120063923520](./assets/posts/PE文件结构/image-20240120063923520.png)

真正pe的标识



### IMAGE_FILE_HEADER

![image-20240120064449974](./assets/posts/PE文件结构/image-20240120064449974.png)

nt头的第二个成员，文件头



![image-20240120064529284](./assets/posts/PE文件结构/image-20240120064529284.png)

一共20个字节



![image-20240120064723363](./assets/posts/PE文件结构/image-20240120064723363.png)

这一部分就是FileHeader

#### machine

![image-20240120064823788](./assets/posts/PE文件结构/image-20240120064823788.png)

第一个是可执行文件的运行平台取值有



![image-20240120064938968](./assets/posts/PE文件结构/image-20240120064938968.png)

取值有这些



#### NumberOfSections

有多少节区

![image-20240120065343781](./assets/posts/PE文件结构/image-20240120065343781.png)

当前有三个节区



#### TimeDateStamp、PointerToSymbolTable、NumberOfSymbols

![image-20240120065536176](./assets/posts/PE文件结构/image-20240120065536176.png)

这三个没啥用



![image-20240120065616514](./assets/posts/PE文件结构/image-20240120065616514.png)



#### SizeOfOptionalHeader

选项头的大小



![image-20240120065810681](./assets/posts/PE文件结构/image-20240120065810681.png)

选项头的位置在IMAGE_FILE_HEADER的，用于定位节表的位置



![image-20240120065958429](./assets/posts/PE文件结构/image-20240120065958429.png)

在IMAGE_FILE_HEADER结束后，就是可选头，可选头往后E0个字节，选项头的地址加上选项头的大小就是节表的位置



#### Characteristics

属性

![image-20240120070741190](./assets/posts/PE文件结构/image-20240120070741190.png)

上图就是属性所在的位置



![image-20240120070732165](./assets/posts/PE文件结构/image-20240120070732165.png)

对应的值



### IMAGE_OPTIONAL_HEADER32

![image-20240120070914854](./assets/posts/PE文件结构/image-20240120070914854.png)

选项头

#### Magic

![image-20240120071010138](./assets/posts/PE文件结构/image-20240120071010138.png)

取值有三个



![image-20240120071033620](./assets/posts/PE文件结构/image-20240120071033620.png)

这里是32位的选项头，第三个是家电



#### MajorLinkerVersion、MinorLinkerVersion

![image-20240120071156327](./assets/posts/PE文件结构/image-20240120071156327.png)

链接器主版本号和副版本号，基本上没啥用，可以随便改



#### SizeOfCode、SizeOfInitializedData、SizeOfUninitializedData



![image-20240120071353542](./assets/posts/PE文件结构/image-20240120071353542.png)

分别是代码的大小，初始化数据的大小，为初始化数据的大小，可以随便改



#### AddressOfEntryPoint

![image-20240120071703079](./assets/posts/PE文件结构/image-20240120071703079.png)

程序的入口点，模块首地址的偏移



![image-20240120071751377](./assets/posts/PE文件结构/image-20240120071751377.png)

00001000，再od中打开



![image-20240120071928264](./assets/posts/PE文件结构/image-20240120071928264.png)

开始执行的地方就会偏移1000，入口点简写为EP



#### BaseOfCode、BaseOfData

![image-20240120073120802](./assets/posts/PE文件结构/image-20240120073120802.png)

代码的地址和数据的地址，这两个东西也可以随便改



#### ImageBase

模块基址，pe映射进内存的首地址



![image-20240120073339940](./assets/posts/PE文件结构/image-20240120073339940.png)

加上刚刚的入口点就是内存中执行的00400000首地址，一般是建议加载到这个地址，但是不一定都能加载到这个地址



#### SectionAlignment、FileAlignment

![image-20240120075215516](./assets/posts/PE文件结构/image-20240120075215516.png)

内存偏移和文件偏移，涉及到对齐



#### MajorOperatingSystemVersion、MinorOperatingSystemVersion、MajorImageVersion、MinorImageVersion、MajorSubsystemVersion、MinorSubsystemVersion、Win32VersionValue

![image-20240120075833676](./assets/posts/PE文件结构/image-20240120075833676.png)

这几个没啥大用，MinorOperatingSystemVersion不能修改



#### SizeOfImage

可执行文件在内存中的总大小

![image-20240120081250194](./assets/posts/PE文件结构/image-20240120081250194.png)

这里大小是00004000，由各个节的大小计算而来，是内存对齐的值



#### SizeOfHeaders

![image-20240120082018164](./assets/posts/PE文件结构/image-20240120082018164.png)

pe头的大小，跟文件对齐值对齐后的值



#### CheckSum

![image-20240120082131870](./assets/posts/PE文件结构/image-20240120082131870.png)

ring3值较为宽松，驱动会严格检测这个值



#### Subsystem

![image-20240120082521831](./assets/posts/PE文件结构/image-20240120082521831.png)

这玩意不能随便改



![image-20240120082506769](./assets/posts/PE文件结构/image-20240120082506769.png)



#### DllCharacteristics

![image-20240120082745206](./assets/posts/PE文件结构/image-20240120082745206.png)

用于描述可执行文件，也不能随便改



![image-20240120082802836](./assets/posts/PE文件结构/image-20240120082802836.png)

如上图所示



#### SizeOfStackReserve、SizeOfStackCommit、SizeOfHeapReserve、SizeOfHeapCommit

![image-20240120084507673](./assets/posts/PE文件结构/image-20240120084507673.png)

可以改，但是不能太离谱，分别对应的是，保留的栈空间，提交的栈空间、保留的堆空间，提交的堆空间



![image-20240120084639512](./assets/posts/PE文件结构/image-20240120084639512-1705758399792-1.png)



#### LoaderFlags

![image-20240120084803712](./assets/posts/PE文件结构/image-20240120084803712.png)

没啥用，都可以随便改



#### DataDirectory、NumberOfRvaAndSizes

![image-20240120085713449](./assets/posts/PE文件结构/image-20240120085713449.png)

各种表的位置，他的个数由NumberOfRvaAndSizes决定



![image-20240120090235209](./assets/posts/PE文件结构/image-20240120090235209.png)

IMAGE_DATA_DIRECTORY表示表在哪里，有多大

![image-20240120090816813](./assets/posts/PE文件结构/image-20240120090816813.png)

这些是下标对应的表



# 节

## IMAGE_SECTION_HEADER





![image-20240120095317131](./assets/posts/PE文件结构/image-20240120095317131.png)

一共40个字节



    union {
            DWORD   PhysicalAddress;
            DWORD   VirtualSize;
    } Misc;

这部分共享内存占4个字节

### Name

![image-20240120095333164](./assets/posts/PE文件结构/image-20240120095333164.png)

8个字节



### PhysicalAddress、VirtualSize、VirtualAddress、SizeOfRawData、PointerToRawData

倒着看



![image-20240120100047499](./assets/posts/PE文件结构/image-20240120100047499.png)

如上图所示



![image-20240120100617809](./assets/posts/PE文件结构/image-20240120100617809.png)

如上图所示



### 举例

![image-20240120100931454](./assets/posts/PE文件结构/image-20240120100931454.png)

文件偏移400，大小200个字节



![image-20240120101057645](./assets/posts/PE文件结构/image-20240120101057645.png)

然后映射到内存偏移为1000的地址，这里要加上ImageBase的00400000，得到00401000



### 字段解析

| VirtualSize | VirtualAddress | SizeOfRawData | PointerToRawData |
| ----------- | -------------- | ------------- | ---------------- |
| 0x00000011  | 0x00001000     | 0x00000200    | 0x00000400       |
| 0x00000136  | 0x00002000     | 0x00000200    | 0x00000600       |
| 0x0000000c  | 0x00003000     | 0x00000200    | 0x00000800       |

PointerToRawData每次递增200，与SizeOfRawData一致

VirtualAddress每次递增1000，因为1000为一个内存分页

文件中节与节直接连续不断，内存中节与节也是连续不断的



![image-20240120104801175](./assets/posts/PE文件结构/image-20240120104801175.png)

文件中的计算过程如上



![image-20240120105205241](./assets/posts/PE文件结构/image-20240120105205241.png)

内存中的计算，对应改为对齐，意思就是在1011后填充一个内存页达到2000



![image-20240120104146293](./assets/posts/PE文件结构/image-20240120104146293.png)

规律：PointerToRawData和SizeOfRawData都是跟FileAlign对齐，VirtualAddress和VirtualSize是根sectionAlign对齐

上图分别是sectionAlign和FileAlign



SizeOfImage的值就是各节的和，然后加上dos头的分页



### PointerToRelocations、PointerToLinenumbers、NumberOfRelocations、NumberOfLinenumbers

![image-20240120105422786](./assets/posts/PE文件结构/image-20240120105422786.png)

系统不依赖这12个字节，可以不管



### Characteristics

![image-20240120105556444](./assets/posts/PE文件结构/image-20240120105556444.png)

初始化内存属性，节的内存属性



![image-20240120105648355](./assets/posts/PE文件结构/image-20240120105648355.png)

取值有这些，例如可读可写可执行



![image-20240120105901775](./assets/posts/PE文件结构/image-20240120105901775-1705766342027-3.png)

text段是60000020，低字节是20高字节是60



![image-20240120105953362](./assets/posts/PE文件结构/image-20240120105953362.png)

代表就是可读加可执行



![image-20240120110028087](./assets/posts/PE文件结构/image-20240120110028087.png)

低字节就是这个



## 地址转换

VA - virtual Address 虚拟地址 绝对地址

RVA -  relative Virtual address 相对模块基址的偏移

FA -  file address 文件偏移



例如VA:402080,我们减去imageBase得到RVA:2080，2080是落在VirtualAddress的2000到3000中，计算出这个内成页中的偏移得到80，2000到3000对应的PointerToRawData，是600，那么最终FA就是680



## dump

把内存中的文件还原成文件形式



### nt头

![image-20240120234903173](./assets/posts/PE文件结构/image-20240120234903173.png)

pe往下五行半就是nt头的大小，然后从头开始复制400个字节



![image-20240120235145669](./assets/posts/PE文件结构/image-20240120235145669.png)

注意是从零开始



### 节

![image-20240120235345481](./assets/posts/PE文件结构/image-20240120235345481.png)

如上图所示，重复这个操作把三个节都拷完，然后保存



![image-20240121000026883](./assets/posts/PE文件结构/image-20240121000026883.png)

可以正常执行，如何程序中涉及到全局变量，在程序运行时全局变量就已经赋值，在dump后就可能导致无法运行，所以一般在入口点处dump，也就是代码没有执行之前



## 节表注入

### 节间隙



### 添加节

1.节表添加一项

2.添加节数据

3.节表个数加一（NumBerOfSections）

4.修改SizeOfImage，文件在内存中的总大小



![image-20240121033536407](./assets/posts/PE文件结构/image-20240121033536407.png)

这里我们尝试把这个写入到新的节里面



![image-20240121032140520](./assets/posts/PE文件结构/image-20240121032140520.png)

一个节占40个字节，也就是两行半



![image-20240121032250991](./assets/posts/PE文件结构/image-20240121032250991.png)

这就是新节的位置



![image-20240121032513533](./assets/posts/PE文件结构/image-20240121032513533.png)

这里我我们将上一个节的位置和上一个节的大小相加，00000800+00000200=00000A00



![image-20240121040342282](./assets/posts/PE文件结构/image-20240121040342282.png)

小端序的方式写入



![image-20240121032918854](./assets/posts/PE文件结构/image-20240121032918854.png)

文件大小，这里要对齐



![image-20240121040435792](./assets/posts/PE文件结构/image-20240121040435792.png)

这里我们要写入的文件大小为A00，对齐后的值为A00，不需要修改



![image-20240121040531459](./assets/posts/PE文件结构/image-20240121040531459.png)

填入文件大小



![image-20240121033210366](./assets/posts/PE文件结构/image-20240121033210366.png)

00003000+0000000c=0000300c，对齐后的值为00004000，这个值就是内存中的位置



![image-20240121033413844](./assets/posts/PE文件结构/image-20240121033413844.png)

这里填入要执行的代码的大小



![image-20240121040600924](./assets/posts/PE文件结构/image-20240121040600924.png)

这里我们写入文件的大小，也可以手动对齐



![image-20240121034211363](./assets/posts/PE文件结构/image-20240121034211363.png)

剩下的数据之间复制上面的



![image-20240121040001983](./assets/posts/PE文件结构/image-20240121040001983.png)

在文件末尾复制上去节数据



![image-20240121040718069](./assets/posts/PE文件结构/image-20240121040718069.png)

这个是NumberOfSections，把它加一



![image-20240121040752594](./assets/posts/PE文件结构/image-20240121040752594.png)

他就在pe同一行对半后的倒数两个



再修改sizeofimage，这里计算有两种，每一个节在内存中的地址相加，然后再加上nt头再内存中的1000即可。或者最后一个节的地址，加上最后一个节的大小



![image-20240121041044302](./assets/posts/PE文件结构/image-20240121041044302.png)

4000+0A00=4A00，这里需要对齐，最后的值就是5000



![image-20240121041245106](./assets/posts/PE文件结构/image-20240121041245106.png)

这个值就是sizeofimage的大小，修改为



![image-20240121041312091](./assets/posts/PE文件结构/image-20240121041312091.png)

这样即可



![image-20240121041347474](./assets/posts/PE文件结构/image-20240121041347474.png)

这样改完以后就能运行了



### 拓展节

1.将最后一个节扩大

2.添加节数据

3.修改sizeofimage



## 导入表

![image-20240121045551387](./assets/posts/PE文件结构/image-20240121045551387.png)

逻辑关系



![image-20240121050516493](./assets/posts/PE文件结构/image-20240121050516493.png)

找到INT后会把对应函数的地址填入IAT表中



### IMAGE_DATA_DIRECTORY

表在选项头中的IMAGE_DATA_DIRECTORY



![image-20240121051505209](./assets/posts/PE文件结构/image-20240121051505209.png)

对应的下标就是表的位置，IMAGE_DIRECTORY_ENTRY_IMPORT就是导入表



### IMAGE_IMPORT_DESCRIPTOR

![image-20240121051640902](./assets/posts/PE文件结构/image-20240121051640902.png)

导入表的结构体



#### DCharacteristics、OriginalFirstThunk;

![image-20240121052151987](./assets/posts/PE文件结构/image-20240121052151987.png)

导入名称表的地址



![image-20240121052309179](./assets/posts/PE文件结构/image-20240121052309179.png)

对应地址的结构体是PIMAGE_THUNK_DATA，最高位如果是1那么是序号导入，如果最高位是0就是名称导入，则最终指向IMAGE_IMPORT_BY_NAME



![image-20240121052619550](./assets/posts/PE文件结构/image-20240121052619550.png)

第一个没啥用，第二个是字符串



#### TimeDateStamp、ForwarderChain

![image-20240121051818639](./assets/posts/PE文件结构/image-20240121051818639.png)

这里玩意没啥用



#### Name

![image-20240121051908100](./assets/posts/PE文件结构/image-20240121051908100.png)

导入函数的名称



#### FirstThunk

![image-20240121052029535](./assets/posts/PE文件结构/image-20240121052029535.png)

导入地址表的地址



#### hex

![image-20240121055102215](./assets/posts/PE文件结构/image-20240121055102215.png)

从节开始倒数128个字符，也就是8行，就是所有的表，下标为1的是导入表



![image-20240121055241960](./assets/posts/PE文件结构/image-20240121055241960.png)

不是所有都能随意修改，这里的地址是20EC是RVA



![image-20240121055448739](./assets/posts/PE文件结构/image-20240121055448739.png)

落在第二个节里面，文件偏移是600，换算为文件偏移就是6EC



![image-20240121055733811](./assets/posts/PE文件结构/image-20240121055733811.png)

这个就是导入表的第一项，20个字节为一项，导入表以全零项结尾，也就是20个0字符



##### name

![image-20240121060041218](./assets/posts/PE文件结构/image-20240121060041218.png)

name在第12个字节开始



![image-20240121060113578](./assets/posts/PE文件结构/image-20240121060113578.png)

这个就是name,是一个地址212A，这里同样是RVA，对应FA就是72A



![image-20240121060433747](./assets/posts/PE文件结构/image-20240121060433747.png)

这个就是对应的name



##### int导入名称表

![image-20240121060648896](./assets/posts/PE文件结构/image-20240121060648896.png)

地址为2114，对应的FA就是714，指向的是PIMAGE_THUNK_DATA



###### PIMAGE_THUNK_DATA

![image-20240121061001449](./assets/posts/PE文件结构/image-20240121061001449.png)

这是一个数组，以全零结尾



![image-20240121061105094](./assets/posts/PE文件结构/image-20240121061105094.png)

这里最高位是0，是名称导入，指向IMAGE_IMPORT_BY_NAME，RVA位211c,FA为71c



###### IMAGE_IMPORT_BY_NAME

![image-20240121061311819](./assets/posts/PE文件结构/image-20240121061311819.png)

两个字节，加一个指针



![image-20240121061408764](./assets/posts/PE文件结构/image-20240121061408764.png)

这里导入的就是messageboxA

##### 导入地址表

![image-20240121052029535](./assets/posts/PE文件结构/image-20240121052029535.png)

这里会填api的地址



![image-20240121061759911](./assets/posts/PE文件结构/image-20240121061759911.png)

这里的地址是2000，对应的FA为600



![image-20240121061848552](./assets/posts/PE文件结构/image-20240121061848552.png)

这里最终会写入MessageBoxA的地址，而这个1c21就是导入名称表的地址



![image-20240121062206146](./assets/posts/PE文件结构/image-20240121062206146.png)

原先是21c1运行后变为7DCBFDE1



### 导入表注入

在导入表中加入自己编写的dll，当程序运行时就会运行自己的dll



![image-20240121221135431](./assets/posts/PE文件结构/image-20240121221135431.png)

这个就是导入表的位置，我们需要添加新的导入表，但是明显下面有数据所以不能往下添加



![image-20240121221346269](./assets/posts/PE文件结构/image-20240121221346269.png)

把它往下搬



![image-20240121221708148](./assets/posts/PE文件结构/image-20240121221708148.png)

我们需要重新修改导入表的位置，2000+750=2150



![image-20240121051640902](./assets/posts/PE文件结构/image-20240121051640902.png)

接下来需要完善这个结构体，先是导入名称表



![image-20240121222220896](./assets/posts/PE文件结构/image-20240121222220896.png)

这里放在FA:7B0的位置



![image-20240121222322829](./assets/posts/PE文件结构/image-20240121222322829.png)

在7B0的位置放入一个word和导出函数的名称



![image-20240121222453795](./assets/posts/PE文件结构/image-20240121222453795.png)

这里要放入导入的名称



![image-20240121222656460](./assets/posts/PE文件结构/image-20240121222656460.png)

这里就是name的位置，指向导入函数的名称



![image-20240121223444222](./assets/posts/PE文件结构/image-20240121223444222.png)

在最后4个字节写入导入地址表的地址



![image-20240121234419488](./assets/posts/PE文件结构/image-20240121234419488.png)

关系如上，这个是修改后的，与上面的有一定出入，最开始的4个字节，要指向一个地址，然后这个地址再指向字符串才行





![image-20240121235215092](./assets/posts/PE文件结构/image-20240121235215092.png)

这样子就成功注入了，已经成功加载dll了