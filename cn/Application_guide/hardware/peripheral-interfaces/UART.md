# UART- 通用异步收发器

通用异步接收器/发送器 (UART) 是一种硬件功能，可使用广泛采用的异步串行通信接口（例如 RS232、RS422 和 RS485）处理通信（即定时要求和数据帧）。 UART提供了一种广泛采用且廉价的方法来实现不同设备之间的全双工或半双工数据交换。 

## 基础知识

 每个 UART 控制器均可独立配置参数，如波特率、数据位长度、位顺序、停止位数量、奇偶校验位等。所有常规 UART 控制器均与各个制造商的支持 UART 的设备兼容 。

在UART通信中，两个UART直接相互通信。 发送UART将来自CPU等控制设备的并行数据转换为串行形式，将其串行发送到接收UART，然后接收UART将串行数据转换回并行数据以供接收设备使用。 只需两根线即可在两个 UART 之间传输数据。 数据从发送 UART 的 Tx 引脚流向接收 UART 的 Rx 引脚： 

![UART](../../media/hardware/peripheral-interfaces/UART/UART_UART.jpg)

UART 异步传输数据，这意味着没有时钟信号来同步发送 UART 的位输出和接收 UART 的位采样。 发送 UART 向正在传输的数据包添加起始位和停止位，而不是时钟信号。 这些位定义数据包的开始和结束，以便接收 UART 知道何时开始读取这些位。

当接收 UART 检测到起始位时，它开始以称为波特率的特定频率读取传入位。 波特率是数据传输速度的度量，以每秒位数 (bps) 表示。 两个 UART 必须以大致相同的波特率运行。 在位时序相差太远之前，发送和接收 UART 之间的波特率只能相差约 10%。 

两个 UART 还必须配置为发送和接收相同的数据包结构。

### 工作原理

![Introduction-to-UART-Data-Transmission-Diagram](../../media/hardware/peripheral-interfaces/UART/UART_Introduction-to-UART-Data-Transmission-Diagram.png)

将要发送数据的UART从数据总线接收数据。 数据总线用于由 CPU、内存或微控制器等其他设备将数据发送到 UART。 数据以并行形式从数据总线传输到发送UART。 发送UART从数据总线获取并行数据后，添加起始位、奇偶校验位和停止位，创建数据包。 接下来，数据包在 Tx 引脚上逐位串行输出。 接收 UART 在其 Rx 引脚上逐位读取数据包。 然后，接收 UART 将数据转换回并行形式，并删除起始位、奇偶校验位和停止位。 最后，接收UART将数据包并行传输到接收端的数据总线：



### 框图

UART 框图由两个组件组成，即发送器和接收器，如下所示。 发送器部分包括三个块，即发送保持寄存器、移位寄存器和控制逻辑。 同样，接收器部分包括接收保持寄存器、移位寄存器和控制逻辑。 这两个部分通常由波特率发生器提供。 该发生器用于在发送器部分和接收器部分必须发送或接收数据时生成速度。

发送器中的保持寄存器包含要发送的数据字节。 发送器和接收器中的移位寄存器将位向右或向左移动，直到发送或接收一个字节的数据。 读（或）写控制逻辑用于告知何时读或写。

发送器和接收器之间的波特率发生器生成范围为 110 bps 至 230400 bps 的速度。 通常，微控制器的波特率为 9600 至 115200。

![UART-Block-Diagram](../../media/hardware/peripheral-interfaces/UART/UART-Block-Diagram.jpg)

### 帧结构

UART 传输的数据被组织成数据包。 每个数据包包含 1 个起始位、5 至 9 个数据位（取决于 UART）、一个可选奇偶校验位以及 1 或 2 个停止位：

![UART-Communication](../../media/hardware/peripheral-interfaces/UART/UART_Communication.jpg)



空闲状态：高电平，表示当前线路上无数据传送

起始位：UART 数据传输线在不传输数据时通常保持在高电压电平。 为了开始数据传输，发送 UART 将传输线从高电平拉至低电平，持续一个时钟周期。 当接收UART检测到高电压到低电压的转变时，它开始以波特率的频率读取数据帧中的位。

数据位： 数据帧包含正在传输的实际数据。 如果使用奇偶校验位，它的长度可以是 5 位到 8 位。 如果不使用奇偶校验位，数据帧可以是9位长。 在大多数情况下，数据首先以最低有效位发送。 

奇偶校验位： 奇偶校验描述了数字的偶数或奇数。 奇偶校验位是接收 UART 判断数据在传输过程中是否发生变化的一种方式。 位可能因电磁辐射、不匹配的波特率或长距离数据传输而改变。 接收UART读取数据帧后，统计值为1的位数，并检查总数是偶数还是奇数。 如果奇偶校验位为 0（偶校验），则数据帧中的 1 位总数应为偶数。 如果奇偶校验位为 1（奇奇偶校验），则数据帧中的 1 位总计应为奇数。 当奇偶校验位与数据匹配时，UART 就知道传输没有错误。 但如果奇偶校验位为0，且总数为奇数； 或者奇偶校验位为 1，并且总数为偶数，UART 知道数据帧中的位已更改。 

停止位：停止位位于数据包的末尾。 通常，该位为 2 位长，但经常仅使用一位。 为了停止广播，UART 保持数据线处于高电压。

### 波特率

线路中信号调制的频率，单位是bps或b/s（位每秒）。一个固定频率的时钟信号不断振荡，每一个时钟周期，发送一位数据信号。

![band_rate1](../../media/hardware/peripheral-interfaces/UART/UART_band_rate1.png)

UART通信双方要求具有相同的波特率。但是，由于UART是异步通信，即没有一根时钟线连接通信双方，各自按照自己内部的时钟调制出一个理论上相等的波特率，但是由于硬件本身不可避免的误差，实际上的波特率往往不可能严格等于理论值，但是要求双方波特率的误差不能超过10%，否则会导致接收方读取到的是乱码数据。



### 硬件流控

流控，即流量控制。

任何通信协议的双方，都会分配有存储空间有限的缓冲区，用来接收对方发送的数据。一旦对方发送数据过快，而己方处理速度较慢，就可能出现缓冲区满无法处理、甚至丢数据的严重情况。

此时流量控制则显得尤为重要，接收方无法接收更多数据时，通知发送方暂停数据发送，当可以接收数据后，再通知发送发继续发送数据。

![uart_control](../../media/hardware/peripheral-interfaces/UART/UART_uart_control.png)

硬件流控引脚的UART接口连接图，相对于前面，多了RTS和CTS两个引脚，此二者UART硬件接口的常规功能引脚。

RTS：输出功能，连接对方的CTS，当己方RTS拉高时，则通知对方UART暂停发送数据，当RTS恢复低电平时，通知对方继续发送数据 。

CTS：输入功能，连接对方的RTS，当己方CTS检测到高电平时，则暂停发送数据，当己方CTS检测到低电平时，则继续发送数据。



### FIFO

FIFO（First-In, First-Out）是一种基本的数据结构，其核心思想是：先进入的数据先出去。FIFO可以在硬件和软件中实现，且有同步和异步之分（本节仅介绍同步FIFO）。

<img src="../../media/hardware/peripheral-interfaces/UART/UART_Data_Queue.svg.png" alt="Data_Queue.svg" style="zoom: 50%;" />

#### 软件FIFO：

- **定义**：在软件中使用编程数据结构（如数组、链表）实现的FIFO。
- **应用**：常见于操作系统中的任务调度、网络中的数据包处理或任何需要队列的场合。
- **操作**：主要操作包括入队（添加到队列尾部）和出队（从队列头部移除）。
- **优点**：灵活，可以轻松地调整大小或实现优先级排队等高级功能。
- **缺点**：由于它是在软件中实现的，因此可能不如硬件FIFO那样快。

实现：

软件FIFO最熟悉的就是环形buffer→ ringbuffer。 

环形buffer相比与线性buffer，我们不用频繁的分配内存，内存反复使用也使得我们能用更少的内存块做更多的事，并且对内存的管理更加方便更加安全。一般应用在我们频繁的对数据buffer进行读写的时候。 

![ringbuff](../../media/hardware/peripheral-interfaces/UART/UART_ringbuff.png)

环形缓冲区并不是指物理意义上的一个首尾相连成“环”的缓冲区，而是逻辑意义上的一个环，因为内存空间是线性结构，所以实际上环形缓冲区仍是一段有长度的内存空间，是一个先进先出功能的缓冲区，具备实现通信进程对该缓冲区的互斥访问功能。

实现原理：

环形缓冲区的长度是固定的，在使用该缓冲区时，不需要将所有的数据清除，只需要调整指向该缓冲区的head、write pointer和tail指针位置即可。write pointer指针最先指向head指针位置（环形缓冲区开头位置），数据从write pointer指针处开始存储，每存储一个数据，write pointer指针位置向后移动一个长度 ，随着数据的添加，write pointer指针随移动数据长度大小个位置。当write pointer指向tail尾部指针，write pointer重新指向head指针位置（折行处理），并且覆盖原先位置数据内容直到数据存储完毕。环形缓冲区的好处是可以减少内存分配继而减少系统的开销，减少内存碎片数量，有利于程序长期稳定的运行。

![UART-Communication](../../media/hardware/peripheral-interfaces/UART/UART_Communication.jpg)

一般构建一个环形缓冲区需要一段连续的内存空间以及4个指针：
      head指针：指向内存空间中的首地址；
      tail指针：指向内存空间的尾地址；
      read pointer：指向内存空间存储数据的起始位置（读指针）；
      write pointer：指向内存空间存储数据的结尾位置（写指针）。

当申请完内存以及指针定义完毕后，环形缓冲区说明及使用如下：
- 1.该段内存空间的长度是Len = tail-head；
- 2.read pointer是读数据的起始位置，当读取完N数据之后要移动N个单位长度的偏移，当有addlen长度的数据要存入到环形缓冲区，若addlen + write pointer > tail时，write pointer将存入len1 = tail -  write pointer个数据长度，然后write pointer回到head位置，将剩下的len2 =  addlen -  len1个数据从head开始存储并覆盖到原来的数据内容。
- 3.write pointer是写数据的起始位置，当存入N个数据之后要移动N个单位长度的偏移，read pointer是读数据的起始位置，当读取N个数据之后要移动N个单位长度的偏移。当要addlen长度的数据要从环形缓冲区读取，若addlen + read pointer > tail时，read pointer 将读取len1 = tail - read pointer  个数据长度，然后read pointer 回到head位置，将剩下的len2 = addlen - len1个数据从head开始读取完毕。

#### 硬件FIFO

- **定义**：在硬件中，特别是在数字电路中实现的FIFO。
- **应用**：常见于通信接口（如UART、SPI等）中，用于数据缓冲。
- **实现**：通常使用寄存器阵列或双口RAM实现，具有读指针和写指针。
- **优点**：速度快，可以与其他硬件模块并行工作，提供高效的数据流。
- **缺点**：大小固定，不如软件FIFO灵活。

目前QuecPython系列模组均使用硬件FIFO,用于模块间的数据缓冲、跨异步传输数据等。 是由芯片设计者使用硬件电路实现的数据缓存。



#### 同步FIFO

- **定义**：同步FIFO的读和写操作都是在相同的时钟下进行的。
- **工作原理**：使用一个共同的时钟信号来控制数据的读写。当有新的数据可写入时，写指针移动；当有数据可读时，读指针移动。
- **应用**：常见于同一时钟域中的数据流缓冲。

![synchronous-fifo](../../media/hardware/peripheral-interfaces/UART/UART_synchronous-fifo.gif)

写入操作：  FIFO可以根据w_en信号在时钟的每个位置存储/写入wr_data,直到数据满。  写入指针在FIFO存储器中的每个数据写入时都会递增。 

读取操作： 根据rd_en信号在时钟的每个位置从FIFO取出或读取数据，直到数据为空。从FIFO存储器读取的每个数据时，读取指针都会递增。 

FIFO内部通过对写请求、读请求计数产生读、写指针，读写指针即为memory的读、写地址。写指针指向下一个要写入的地址，读指针指向下一个要读取的地址，写请求使写指针递增，读请求使读指针递增。 

FIFO模块输出empty和full信号指示其状态，fifo_full表示FIFO内空间已满不能再写入数据，fifo_empty表示FIFO内没有可供读取的下一个有效数据。 

同步FIFO空满信号的产生：

复位时FIFO读、写指针都归零。此时fifo_empty拉高，只能写不能读，一旦有数据写入，会将fifo_empty拉低，允许读取数据。当fifo的写指针指向fifo_depth-1时，此时进行一个写操作会使写指针归零（此时无读操作），拉高fifo_full。 

在读写指针相等时，FIFO要么空要么满，所以需要对这两种情况进行区分。 

1. FIFO满信号产生

FIFO满状态是由写操作触发的：“在写操作使读、写指针在下个时钟保持相等时，FIFO满”，更为通俗的解释是“**写操作让写指针追上了读指针，即写指针套了读指针一圈（跑步的角度）**”

2. FIFO空信号产生

 FIFO空状态是由读操作触发的：“在读操作使读、写指针在下个时钟保持相等时，FIFO空”，更为通俗的解释是“**读操作让读指针追上了写指针**”



#### 异步FIFO

- **定义**：异步FIFO的读和写操作在不同的时钟域中进行。
- **工作原理**：使用两个独立的时钟，一个用于写操作，另一个用于读操作。这需要特殊的设计，以确保跨时钟域的数据完整性和同步。
- **应用**：用于两个有不同操作频率或来自不同源的时钟域之间的数据传输。

### DMA

DMA（Direct Memory  Access）是一种技术，允许外部设备（如硬盘、音频接口、网络适配器等）直接与系统内存进行数据交换，而无需中央处理单元（CPU）的干预。这种机制可以显著提高数据传输效率，因为它允许设备在不占用CPU时间的情况下进行数据传输。 

特点：

DMA和cpu之前并不是一个并行关系,因为主存只有一个,cpu和DMA无法同时访问主存,只能通过交替访问的方式,访问主存,DMA之所以效率高速度快是因为省去现场保护和现场恢复

DMA传输本身并不会中断程序，但它会占用系统资源：比如IO或RAM。这样一旦CPU需要访问相同的IO或RAM时，就需要长时间等待，直到DMA传输完毕、释放资源。从软件角度来看，这和中断程序非常相似，但其内核截然不同：因为CPU一直在工作，从未有过任务切换，只是偶尔暂停，所以无需现场保护。

此外，如果CPU拥有一定容量的cache，而DMA传输的颗粒度又恰到好处，那么即使访问同一块RAM，软件也是感觉不到程序停止的.

工作过程：

1. **请求与授权**：
   - 当一个外部设备（如硬盘控制器）需要与内存交换数据时，它会发出一个DMA请求（DMA Request, 也称为 DRQ）给DMA控制器。
   - DMA控制器在收到请求后，会等待合适的时机来执行数据传输，即当系统总线处于空闲状态时。
   - 一旦准备好，DMA控制器向CPU发出一个DMA授权信号（DMA Grant, 也称为 DACK）。这通常会导致CPU在当前指令执行完毕后暂停，并释放总线控制权。
2. **传输数据**：
   - DMA控制器接管总线并开始数据传输。根据设置，它可以是单个字节的传输，或一整块数据的传输。
   - DMA控制器更新源和目的地址，以及还需传输的字节计数。
   - 在数据传输过程中，CPU是不活跃的，或者说被“隔离”了，不参与数据传输过程。
3. **传输完成**：
   - 一旦DMA控制器完成了所有的数据传输，它会取消DMA授权信号。
   - DMA控制器会向CPU发送一个中断信号（如果已经设置了中断），告知数据传输已完成。这样，CPU可以恢复其操作，可能是处理数据，或执行其他任务。
4. **中断服务**：
   - 如果启用了DMA完成的中断，CPU在数据传输完成后会接收到一个中断请求。
   - CPU随后会调用相应的中断服务程序来处理传输后的任务，例如数据后处理、错误检查或其他相关任务。
5. **重置与准备下一次DMA**：
   - DMA控制器重置其状态，并准备响应下一个DMA请求。

## 特点与应用

特点：

1. **简单性：** UART通信的硬件和编程复杂度相对较低。UART本身通常是在微控制器或其他处理器的硬件中实现的，所以在硬件层面上不需要额外的芯片。
2. **无需时钟同步：** 因为UART是异步的，所以发送和接收设备不需要共享一个时钟信号。这降低了硬件设计的复杂性。
3. **灵活性：** UART通信可以调整数据位长度、停止位数量和奇偶校验等参数，以适应不同的通信需求。
4. **可靠性：** 尽管UART通信没有错误修复功能，但它的奇偶校验能提供一定程度的错误检测。
5. **限制：** UART的通信速度（波特率）有一定的限制，而且通信速度越高，数据错误的可能性也越大。此外，UART的通信距离通常也有限制，对于很长的距离，可能需要使用RS-422或RS-485等差分信号标准。

应用：

1. **嵌入式系统：** UART常用于微控制器和其他低级硬件设备的通信，例如传感器、记忆卡、GPS模块等。

2. **串行通信：** UART可以用于RS-232、RS-422和RS-485接口的串行通信，用于连接如打印机、调制解调器、显示器等设备。

3. **电脑硬件：** 在早期的电脑硬件中，UART用于鼠标和键盘等外设的接口。

4. **电信设备：** 在无线通信和电信设备中，UART用于与SIM卡和其他设备通信。

5. **蓝牙模块：** 在蓝牙模块中，UART用于与主设备进行通信。

6. **IOT设备：** 在物联网设备中，UART用于低速的设备间通信。

   

## 支持情况

QuecPython各模组支持情况见[machine.UART](../../../api_reference/peripherals/machine.UART.md#machine.UART)

## 数据流

基于QuecPython EC600U模组介绍UART数据流。

发送数据的数据流：

![send_data](../../media/hardware/peripheral-interfaces/UART/UART_send_data.png)

当软件FIFO中无数据超过500ms时，结束数据传输。

TX通信：发送数据->数据发送完毕发送事件->触发中断->触发回调函数发送事件到对应的线程处理->对应线程触发用户自定义回调函数。

其触发中断类型只有一种：TX_COMPLETE。

接收数据的数据流：

![recv_data](../../media/hardware/peripheral-interfaces/UART/UART_recv_data.png)



RX通信有两个中断，dma中断（数据接收到64byte）,超时中断（接收数据字节数少于64而不产生dma中断的情况，时长40ms）。

其触发中断的类型有两种：RX_ARRIVED 和 RX_OVERFLOW。

当软件FIFO满时，再接收数据会造成数据溢出，出现丢数据问题。溢出的同时会上报RX_OVERFLOW事件。故串口接收数据需要及时处理，否则出现数据丢失，数据接收不全问题，当大量数据传输时，可考虑硬件流控。

RX通信：接收数据满64字节或超时->触发中断->触发回调函数发送事件到对应的线程处理->对应的线程触发用户定义回调函数。

## 功能概述

主要介绍如何使用 UART 驱动程序的功能和数据类型在 QuecPython系列模组 与其他 UART 设备之间建立通信。典型的编程工作流程分为以下几节：

1. [创建对象](#创建对象)
2. [发送数据](#发送数据) 
3. [接收数据](#接收数据)
4. [中断](#中断)
5. [RS485控制](#RS485控制)

详细API介绍请参考[machine.UART](../../../api_reference/peripherals/machine.UART.md)

### 创建对象

UART通信参数在这一步骤中配置。包括波特率、数据位、奇偶校验位、停止位和硬件流控。

```python
class machine.UART(UART.UARTn, baudrate, databits, parity, stopbits, flowctl)
```

参数介绍以及引脚对应关系请参考[machine.UART](../../../api_reference/peripherals/machine.UART.md#machine.UART)

创建对象时，需注意以下几点：

**1. 设置波特率：** 波特率是指UART接口每秒钟可以发送或接收的比特数。波特率的设置通常通过编程UART的波特率生成器来完成。发送端和接收端的波特率必须匹配，否则可能导致通信错误。

**2. 设置数据位：** 数据位的数量决定了每个字符包含多少比特。常见的设置是8个数据位，但也可能设置为5、6、7或9个数据位。数据位的数量需要与发送端和接收端的设置一致。

**3. 设置停止位：** 停止位是每个字符后面的一位或两位，用于标识字符的结束。常见的设置是1个停止位，但也可能设置为2个停止位。停止位的数量需要与发送端和接收端的设置一致。

**4. 设置奇偶校验：** 奇偶校验用于检测数据传输过程中可能出现的错误。常见的设置包括无校验、偶校验和奇校验。奇偶校验的设置需要与发送端和接收端的设置一致。



### 发送数据

准备好将要发送的数据，调用函数uart.write() 。 该函数会将数据复制到 Tx 环形缓冲区（立即或在有足够空间可用后），然后退出。 当 Tx FIFO 缓冲区中有可用空间时，中断服务例程 (ISR) 将数据从 Tx 环形缓冲区移动到后台的 Tx FIFO 缓冲区。 

```python
msg = "This is a test string"
uart.write(msg)
```

API介绍请参考[machine.UART.write](../../../api_reference/peripherals/machine.UART.md#uart.write)

### 接收数据

一旦数据被 UART 接收并保存在 Rx 缓冲区中，就需要使用函数 uart.read() 来检索。 在读取数据之前，您可以通过调用 uart.any() 检查 Rx 缓冲区中可用的字节数。 下面给出了使用这些函数的示例。

```python
msg_len = uart.any()
msg = uart.read(msg_len)
```

接收数据，需要注意以下几点：

**1. 缓冲区管理：** 正确管理接收缓冲区是非常重要的。如果在数据还没有读取完成就被新的数据覆盖，可能会导致数据丢失。因此，需要确保在数据被新的数据覆盖前能够及时读取和处理。 

**2. 并发和多线程：** 如果你的系统支持并发或多线程，你需要考虑UART接收数据的线程安全。例如，你可能需要使用锁或其他同步机制，以防止多个线程同时访问接收缓冲区。 

 **3. 数据格式：** 接收的数据格式必须符合发送端的设置。这包括数据的编码方式（例如ASCII、UTF-8、二进制等）、数据位的数量、起始位、停止位和奇偶校验位等。 

API介绍请参考[machine.UART.read](../../../api_reference/peripherals/machine.UART.md#uart.read)

### 中断

 设置串口数据回调，串口收到数据后，会执行该回调 。同时返回数据个数。 

下面给出了使用这些函数的示例：

```python
from machine import UART
uart1 = UART(UART.UART1, 115200, 8, 0, 1, 0)
 
def uart_call(para):
	print(para)
uart1.set_callback(uart_call)
```

API介绍请参考[machine.UART.setCallback](../../../api_reference/peripherals/machine.UART.md#uart.set_callback)

> 未避免中断长时间执行，请在回调函数中以发送信号量等方式，告知其它线程读取串口数据

### RS485控制

 控制485通信方向，串口发送数据之前和之后进行拉高拉低指定GPIO，用来指示485通信的方向。 

API介绍请参考[machine.UART.control_485](../../../api_reference/peripherals/machine.UART.md#uart.control_485)

## 应用示例

主要介绍如何使用 UART 具体的应用示例：


| 示例                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| [基础收发](#基础收发)   | 配置UART设置，通过UART1读取和写入。采用回调方式读取数据      |
| [外挂GNSS](#外挂GNSS)   | 通过解析 UART从外置GNSS读取的原始GNSS数据包中的GNGGA、<br />GNRMC和GPGSV语句来获取定位信息 |
| [RS485应用](#RS485应用) | 设置 UART 驱动程序以半双工模式通过 RS485 接口进行通信。      |
| [计量芯片](#计量芯片)   | 以计量芯片为例，使用UART来读取和写入芯片的参数，获取电力数据，或者执行其他的控制指令。 |

### 基础收发

QuecPython提供了简化的方法来在通信模组上使用 Python 进行 UART 通信。对于实时应用或需要高效处理 UART 信息的场景，使用回调函数（基于中断）进行 UART 读取是一种非常有效的方法。 

 实验前需了解UART QuecPython接口，请参考[machine.UART](../../../api_reference/peripherals/machine.UART.md)

实验步骤：

1. 初始化UART

    ```python
    from machine import UART
    # 初始化 UART1
    uart1 = UART(UART.UART1, 115200, 8, 0, 1, 0)
    ```

2. 写入UART

    发送数据到 UART 是非常简单的： 

    ```python
    uart1.write('Hello UART1')
    ```

3. 使用回调数据读取数据

   在 UART 上使用回调函数，通常涉及到设置一个 IRQ（中断请求）来监听 UART 事件，如数据接收。当这些事件触发时，相关的回调函数会被执行。 

   > 你需要确保回调函数尽可能短，以减少对其他系统任务的干扰。同时数据来之后，应立即读取，防止底层软件FIFO溢出导致数据丢失

    ```python
    def uart_callback(arg):
        _queue.put(para[2])

    # 设置中断
    uart1.set_callback(uart_callback)
    ```

   

示例：

```python
import _thread
import utime
from machine import UART
from queue import Queue

class Example_uart(object):
    def __init__(self, no=UART.UART2, bate=115200, data_bits=8, parity=0, stop_bits=1, flow_control=0):
        self.uart = UART(no, bate, data_bits, parity, stop_bits, flow_control)
        self._queue = Queue(5)
        _thread.start_new_thread(self.handler_thread, ())
        self.uart.set_callback(self.callback)
        


    def callback(self, para):
        print("call para:{}".format(para))
        if(0 == para[0]):
            self._queue.put(para[2])
      
    def uartWrite(self, msg):
        print("write msg:{}".format(msg))
        self.uart.write(msg)

    def uartRead(self, len):
        msg = self.uart.read(len)
        utf8_msg = msg.decode()
        print("UartRead msg: {}".format(utf8_msg))
        return utf8_msg

    def uartWrite_test(self):
        for i in range(10):
            write_msg = "Hello count={}".format(i)
            self.uartWrite(write_msg)
            utime.sleep(1)

    def handler_thread(self):
        while True:
            recv_len = self._queue.get()
            self.uartRead(recv_len)
           
if __name__ == "__main__":
    uart_test = Example_uart()
    uart_test.uartWrite_test() 
```



### 外挂GNSS

使用 UART 连接 GNSS (Global  Navigation Satellite System) 接收器是嵌入式系统中一个常见的应用场景。GNSS 包括 GPS (Global  Positioning System)、GLONASS、Galileo、BeiDou 等全球定位系统。这种连接允许系统读取 GNSS  数据，以获取当前位置、速度、时间和其他相关信息。 

QuecPython部分模组目前已集成外置GNSS功能，并提供了一系列接口，详细请参考[gnss_wiki](../../../api_reference/gnsslib/gnss.md)。用户可直接使用GNSS系列接口对数据进行获取，可以得到模块定位是否成功，定位的经纬度数据，UTC授时时间，获取GPS模块的定位模式，获取GPS模块定位使用卫星数量，获取GPS模块定位可见卫星数量，获取定位方位角，GPS模块对地速度，模块定位大地高等数据信息。目前，该模块提供的功能接口，所获取的数据都来源于从串口读出的原始GNSS数据包中的GNGGA、GNRMC和GPGSV语句。 

本章节基于L76K定位芯片为例展开介绍

使用步骤：

1. 确定串口信息

   确定L76K定位芯片接到模组哪个串口，以及使用的波特率等信息，本示例中将L76K接到了模组的UART2，波特率默认为9600bps（可在L76K使用手册中查询到）。 

2.  实例化对象

   ```python
   >>> from gnss import GnssGetData
   >>> gnss_obj = GnssGetData(2, 9600, 8, 0, 1, 0) 
   ```

3.  读取数据并解析

    用户只需要调用如下接口，该接口直接完成定位数据的读取和解析工作： 

   ```python
   >>> gnss_obj.read_gnss_data()
   822
   ```

   上述接口将读取原始数据、以及复杂的解析操作都放在接口内部实现，只返回了通过串口读取的数据长度。如用户想看本次读取解析的原始数据，可调用如下接口。

   ```python
   >>> data = gnss_obj.getOriginalData()
   >>> print(data)
   $GNGGA,063957.000,3802.01852,N,11437.92027,E,1,13,1.2,129.2,M,-15.7,M,,*62
   $GNGLL,3802.01852,N,11437.92027,E,063957.000,A,A*40
   $GNGSA,A,3,01,03,06,14,30,194,,,,,,,1.8,1.2,1.4,1*00
   $GNGSA,A,3,13,23,33,38,40,43,59,,,,,,1.8,1.2,1.4,4*3C
   $GPGSV,3,1,11,01,38,044,12,03,44,101,24,06,30,233,31,14,79,184,34,0*6F
   $GPGSV,3,2,11,17,58,319,,19,40,296,04,21,11,051,,30,14,202,34,0*67
   $GPGSV,3,3,11,194,44,156,27,195,68,074,,199,44,160,25,0*6F
   $BDGSV,3,1,11,03,44,186,,07,83,121,05,10,65,309,,13,37,206,31,0*79
   $BDGSV,3,2,11,23,21,181,31,28,69,354,14,33,24,115,27,38,65,190,30,0*7A
   $BDGSV,3,3,11,40,70,093,18,43,28,065,15,59,38,143,29,0*44
   $GNRMC,063957.000,A,3802.01852,N,11437.92027,E,0.69,168.35,260422,,,A,V*0B
   $GNVTG,168.35,T,,M,0.69,N,1.29,K,A*2F
   $GNZDA,063957.000,26,04,2022,00,00*44
   $GPTXT,01,01,01,ANTENNA OPEN*25
   ```

4. 确认是否成功

    如用户只关心是否定位到经纬度坐标以及坐标是否有效，可使用如下接口，返回1即表示定位成功且有效： 

   ```python
   >>> gnss_obj.isFix()
   1
   ```

5.  获取坐标信息 

    调用如下接口即可获取到定位坐标： 

   ```python
   >>> gnss_obj.getLocation()
   (114.6320045, 'E', 38.033642, 'N')
   ```

   > 上述接口获取的坐标是WGS-84坐标系下的经纬度数据，不可直接用于高德地图、腾讯地图以及百度地图等地图上拾取位置信息，必须先转换为对应地图参考坐标系下的坐标。 

示例：

 如下代码是一个完整的使用`gnss`模块方法来获取定位坐标的例程： 

```python

import utime
from gnss import GnssGetData


def main():
    gnss_obj = GnssGetData(2, 9600, 8, 0, 1, 0)
    while True:
        try:
            read_size = gnss_obj.read_gnss_data()
        except Exception:
            print('数据异常，解析出错！')
            data = gnss_obj.getOriginalData()
            print('===============================================')
            print(data)
            print('===============================================')
            utime.sleep(2)
            continue

        if read_size > 0:
            if gnss_obj.isFix():
                coordinate = gnss_obj.getLocation()
                longitude = coordinate[0]
                latitude = coordinate[2]
                print('定位成功，当前经纬度：({}, {})'.format(longitude, latitude))
                utime.sleep(10)
            else:
                print('定位中，请稍后...')
                utime.sleep(2)
        else:
            print('未读取到定位数据...')
            utime.sleep(2)


if __name__ == '__main__':
    main()            
```



### RS485应用

RS-485 可以在半双工模式下工作，这意味着它可以在同一时间只发送或只接收数据。为了切换这两种模式，通常会使用一个GPIO（General-Purpose Input/Output）。

这个GPIO控制一个叫做 "方向控制" 的线。当这条线为高时，RS-485 驱动器被激活并发送数据；当这条线为低时，驱动器被禁用，允许其他设备在总线上发送数据，而本地设备则处于接收模式。

在发送数据之前，GPIO 被设置为 "高"，然后发送数据。数据发送完毕后，GPIO 被设置为 "低"，从而切换到接收模式。 

QuecPython支持在UART中加入GPIO控制方向的传输。[uart.control_485](../../../api_reference/peripherals/machine.UART.md#uart.control_485) 可以控制485通信方向，串口发送数据之前和之后进行拉高拉低指定GPIO，用来指示485通信的方向。 

当方向设置为1时表示： 串口发送数据之前由低拉高、发送数据之后再由高拉低 

当方向设置为0时表示： 串口发送数据之前由高拉低、发送数据之后再由低拉高 

具体示例如下：

```python
from machine import UART
uart1 = UART(UART.UART1, 115200, 8, 0, 1, 0)
uart1.control_485(UART.GPIO24, 1)
```





### 计量芯片

本章节以HLW8110计量芯片为例，使用UART来读取和写入芯片的参数，获取电力数据，或者执行其他的控制指令。



详细请参考[HLW8110数据手册](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/power_meter_chips/HLW8110/C970139_%E7%94%B5%E8%83%BD%E8%AE%A1%E9%87%8F%E8%8A%AF%E7%89%87_HLW8110_%E8%A7%84%E6%A0%BC%E4%B9%A6_WJ409662.PDF)，实验源码请参考[hlw8110.py](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/power_meter_chips/HLW8110/hlw8110.py)

HLW8110概述：

- HLW8110 是一款高精度的电能计量 IC,它采用 CMOS 制造工艺，主要用于单相应用。
- 它能够测量线电压和电流，并能计算有功功率，视在功率和功率因素。
- 该器件内部集成了三个∑-Δ型 ADC 和一个高精度的电能计量内核。第二路通道可同时测量零线电流，支持窃电检测和漏电检测。各输入通道都支持灵活的 PGA 设置，因此 HLW8110适合与不同类型的传感器使用，如电流互感器（CT）和低阻值分流器。
- HLW8110 可以通过UART通讯接口访问片内寄存器。
- HLW8110 包含两个可配置的脉冲输出引脚，可以通过 INT1 和 INT2 引脚获取过流、过压、电压或电流过零检测和漏电检测等功能。
- HLW8110 电能计量 IC 采用 3.3V 或 5.0V 电源供电，内置振荡器，采用 8 脚 SOP 封装或 16 脚 SSOP 封装

典型应用：

![HLW8100_application](../../media/hardware/peripheral-interfaces/UART/UART_HLW8100_application.png)

工作流程：

1. **模拟测量**：
   HLW8110通过与外部电流传感器（如电流互感器或电阻）和电压取样电路相连，获取电流和电压的模拟信号。
2. **模拟-数字转换**：
   模拟信号经过ADC转换为数字信号。
3. **数字处理**：
   芯片内部对ADC得到的数字信号进行处理，计算得到有功功率、无功功率和功率因数等参数。
4. **输出**：
   HLW8110通过专用的输出引脚提供有关测量参数的脉冲输出，这些脉冲可以被外部微控制器读取并进一步处理。

通讯：

UART通信格式如下：

![HLW8110_communication_format](../../media/hardware/peripheral-interfaces/UART/UART_HLW8110_communication_format.png)

- UART的命令寄存器是一个8bit宽的寄存器。。对于读写操作，命令寄存器的bit7 用来确定本次数据传输操作的类型是读操作还是写操作。对于特殊命令操作，命令寄存器的bit7-0 固定为0xEA。
- HLW8110 的UART 数据传送：读操作由从机端发送，写操作由主机端发送。若寄存器地址对应寄存器是多字节寄存器，先传最高有效字节。
- HLW8110 的UART 数据校验方式：读操作由从机端发送，写操作由主机端发送。校验数据计算方法如下：
  校验数据Cdata[7:0] = ~(A5+CMD[7:0] + DATAn[7:0] + …… +DATA1[7:0])，即将 CMD 和数据相加，抛弃进位，最后的结果按位取反；

| 命令          | 命令寄存器                   | 数据  | 描述                                                         |
| ------------- | ---------------------------- | ----- | ------------------------------------------------------------ |
| 读命令        | {0[bit7],REG_ADR[bit6:bit0]} | RDATA | 从地址为REG_ADR[6:0]的寄存器中读数据最高位是0，表示向寄存器读取数据 |
| 写命令        | {1[bit7],REG_ADR[bit6:bit0]} | WDATA | 向地址为REG_ADR[6:0]的寄存器中写数据最高位是0，表示向寄存器写入数据 |
| 写使能命令    | 0xEA                         | 0xE5  | 使能写操作                                                   |
| 写保护命令    | 0xEA                         | 0xDC  | 关闭写操作                                                   |
| 电流通道A选择 | 0xEA                         | 0x5A  | 电流通道A设置命令，指定当前用于计算视在功率、功率因素、相角、瞬时视在功率和有功功率过载的信号指示为通道A |
| 电流通道B选择 | 0xEA                         | 0xA5  | 电流通道B设置命令，指定当前用于计算视在功率、功率因素、相角、瞬时视在功率和有功功率过载的信号指示为通道B |
| 复位指令      | 0xEA                         | 0x96  | 复位指令，接收到指令后芯片复位                               |

> 在 UART 操作期间，如果当 RX 保持低电平或高电平超过 9.15ms 时，可以复位 UART 模块，复位UART不会改写己经写入芯片内部的寄存器值。MCU在与HLW8110/HLW8112进行UART通讯时，如果发生接收数据异常，可以尝试复位 UART 模块。



实验：

初始化和配置UART： 

初始化UART和相应的读写寄存器，复位接口

```python
class Hlw8110_uart(Hlw8110):
    def __init__(self, uart_n, databits = 8, flowctl = 0):
        self.uart = UART(uart_n, 9600, databits, 1, 1, flowctl) #hlw8110固定9600波特率 偶校验even
        super().__init__(self)

    def read_reg(self,reg):
        '''
        读寄存器
        :param reg: 要读的寄存器
        :return: 成功：读出的数据
                 失败：空列表
        '''
        #发送读命令字节
        self.uart.write(bytearray([0xA5, reg]))
        check_data = 0xa5 + reg
        #uart判断是否有数据未读
        while 1:
            msglen = self.uart.any()
            if msglen:
                #读取数据
                r_data = list(self.uart.read(msglen))
                #校验数据
                for i in range(msglen-1):
                    check_data += r_data[i]
                check_data = ~check_data & 0xff
                if check_data == r_data[-1]:
                    return r_data[:-1]
                else:
                    return []
      
    def write_reg(self, reg, w_data):
        '''
        写寄存器
        :param cmd: 要写入的命令
        :param w_data: 要写入的数据，长度2的list或tuple
        :return: 0：成功
        '''
        cmd = reg | 0x80        #{1[bit7],REG_ADR[bit6:bit0]}
        #使能写操作0xA5 0xEA 0XE5 校验
        self.uart.write(bytearray([0xA5,0xEA,0xE5,0x8B]))
        #往寄存器写
        check_data = ~(0xA5 + cmd + w_data[0] + w_data[1]) & 0xff
        w_data = bytearray([0xA5, cmd, w_data[0], w_data[1],check_data])
        self.uart.write(w_data)
        #关闭写操作0xA5 0xEA 0XDC 校验
        self.uart.write(bytearray([0xA5, 0xEA, 0XDC, 0x94]))
        return 0

    def reset(self):
        '''
        复位指令0xA5 0xEA 0X96 校验
        '''
        self.uart.write(bytearray([0xA5, 0xEA, 0X96, 0xDA]))         
```



读取电流：

读取寄存器0x70，获取电流有效值转换系数。

读取寄存器0x2A，获取通道A电流的有效值

```python
def read_i(self):
	'''
	读电流有效值寄存器和电流有效值转换系数，具体电流跟接入电路的电阻有关，本接口不做计算
	:return: 正常测量：(电流有效值寄存器值,电流有效值转换系数)
			 无有效数据：(0,0)
	'''
	#判断是交流电还是直流电
	cur_type = self._child.read_reg(self.EMUCON_REG)
	if len(cur_type) != 2:
		return (0,0)
	else:
		cur_type = cur_type[1] & (1<<5)

	# 读电流有效值转换系数
	ic_read = self._child.read_reg(self.RMSIAC_REG)  # 16位无符号数
	if len(ic_read) != 2:
		return (0,0)
	ic_data = (ic_read[0]<<8) +ic_read[1]
	
	#读电流有效值寄存器
	current_read = self._child.read_reg(self.RMSIA_REG)  # 24位有符号数
	if len(current_read) != 3:
		return (0,0)
	current_data = (current_read[0]<<16) + (current_read[1]<<8) + current_read[2]
	if cur_type:        #直流电
		#直流测量时，最高位为1，表示补码，计算有效值时，需要取绝对值
		if current_data & 0x800000:     #最高位为1，表示补码，计算有效值时，需要取绝对值
			current_data = ~(current_data & 0x7fffff - 1)
			return (current_data,ic_data)
		else:
			return (current_data,ic_data)
	else:               #交流电
		if current_data & 0x800000:     #最高位为1，表示数据为0
			return (0,ic_data)
		else:
			return (current_data,ic_data)
```

读取电压：

读取寄存器0x72，获取电压有效值转换系数。

读取寄存器0x26，获取电压的有效值。

```python
def read_u(self):
	'''
	读电压有效值寄存器和电压有效值转换系数，本接口不做计算
	:return: 正常测量：(电压有效值寄存器值,电压有效值转换系数)
			 无有效数据：(0,0)
	'''
	# 判断是交流电还是直流电
	cur_type = self._child.read_reg(self.EMUCON_REG)
	if len(cur_type) != 2:
		return (0,0)
	else:
		cur_type = cur_type[1] & (1 << 4)
	# 读电压有效值转换系数
	uc_read = self._child.read_reg(self.RMSUC_REG)  # 16位无符号数
	if len(uc_read) != 2:
		return (0,0)
	uc_data = (uc_read[0] << 8) + uc_read[1]
		
	# 读电压有效值寄存器
	voltage_read = self._child.read_reg(self.RMSU_REG)  # 24位有符号数
	if len(voltage_read) != 3:
		return (0,0)
	voltage_data = (voltage_read[0] << 16) + (voltage_read[1] << 8) + voltage_read[2]
	if cur_type:  # 直流电
		# 直流测量时，最高位为1，表示补码，计算有效值时，需要取绝对值
		if voltage_data & 0x800000:  # 最高位为1，表示补码，计算有效值时，需要取绝对值
			voltage_data = ~(voltage_data & 0x7fffff - 1)
			return (voltage_data, uc_data)
		else:
			return (voltage_data, uc_data)
	else:  # 交流电
		if voltage_data & 0x800000:  # 最高位为1，表示数据为0
			return (0, uc_data)
		else:
			return (voltage_data, uc_data)
```

读取有功功率：

读取寄存器0x73，获取有功功率有效值转换系数。

读取寄存器0x28，获取有功功率。

```python
def read_power(self):
	'''
	读有功功率寄存器和有功功率转换系数
	:return: 正常测量：(有功功率寄存器值,有功功率转换系数)
			 无有效数据：(0,0)
	'''
	# 读有功功率转换系数
	pc_read = self._child.read_reg(self.POWER_PAC_REG)  # 16位无符号数
	if len(pc_read) != 2:
		return (0,0)
	pc_read = (pc_read[0] << 8) + pc_read[1]

	# 读有功功率寄存器
	power_read = self._child.read_reg(self.POWER_PA)  # 32位有符号数
	if len(power_read) != 4:
		return (0,0)
	power_data = (power_read[0] << 24) + (power_read[1] << 16) + (power_read[2] << 8) + power_read[3]
	if power_data & 0x80000000:  # 补码，最高位是符号位
		power_data = ~(power_data & 0x7fffffff - 1)
		return (power_data, pc_read)
	else:
		return (power_data, pc_read)
```



## 注意事项

**1. 波特率选择：** 在设置UART的波特率时，需要确保发送和接收设备的波特率一致。波特率的选择应考虑数据传输速度需求和系统的时钟频率。记住，高波特率可能会提高数据传输速度，但也可能增加误码率。

**2. 数据格式设置：** UART通信时的数据格式，包括数据位、奇偶校验位和停止位，需要在发送和接收设备之间一致。一般来说，常见的设置是8个数据位，无奇偶校验，1个停止位（8N1）。

**3. 缓冲区管理：** UART接收数据时，通常会使用一个接收缓冲区存储数据。你需要确保缓冲区足够大，以防止数据溢出丢失。此外，缓冲区的读取和写入需要正确同步，避免在多任务环境下产生冲突。

**4. 流量控制：** 在高速数据传输或处理能力有限的情况下，可能需要使用硬件或软件流量控制防止数据丢失。

**5. 中断管理：** UART通常使用中断进行数据的接收和发送。你需要确保中断服务程序（ISR）尽可能短，以减少对其他系统任务的干扰。通常，ISR会从UART接收数据或发送数据到UART，并将数据移动到缓冲区或从缓冲区移动数据。

**6. 并发和多线程环境：** 在并发或多线程环境中使用UART时，可能需要使用信号量、互斥锁等机制保护UART资源，防止多个任务同时访问UART造成的数据混乱。

**7. 电源和地线：** 在嵌入式系统中，通常会有多个电源电压级别和地线。要确保UART设备的电源电压和地线正确连接，以防止通信故障或设备损坏。

**8. 错误处理：** UART通信可能会出现帧错误、奇偶校验错误、数据溢出等问题。你需要设计合适的错误处理机制，处理或报告这些错误。

**9. 物理接口：** UART设备可能通过RS-232、RS-485、TTL等不同的物理接口进行连接。你需要确保使用正确的接口电平和连接方式。

**10. 设备驱动和操作系统兼容性：** 在使用嵌入式操作系统时，需要确保UART设备驱动与操作系统兼容。可能需要根据具体的操作系统和硬件平台，编写或修改设备驱动。

**11. 实时性：** 嵌入式系统通常需要满足一定的实时性要求。你需要考虑UART通信对系统实时性的影响，例如中断响应时间，数据处理时间等。

**12. 功耗：** 在电池供电的嵌入式系统中，需要考虑UART通信的功耗。例如，当系统处于低功耗模式时，可能需要关闭UART设备，或将其设置为低功耗模式。



## 常见问题和故障

 UART是一种非常简单和直接的通信协议，但在实际应用中也可能遇到一些问题。以下是一些常见的问题以及可能的故障排查步骤： 



**1. 数据接收不正确或无法接收数据**

这可能是由于几种原因引起的，包括：波特率设置错误、硬件连接问题、中断处理程序问题、缓冲区溢出等。故障排查步骤可能包括：

- 检查发送端和接收端的波特率设置是否一致。如果它们的波特率不匹配，可能会导致接收的数据出现错误。
- 检查硬件连接，确保TX和RX线正确连接，并且地线（GND）也要正确连接。
- 如果使用了中断处理程序，确保它能正确地处理接收到的数据，并且不会错过任何数据。
- 检查接收缓冲区。如果缓冲区太小或处理速度太慢，可能会导致数据丢失。可以考虑增大缓冲区或优化处理程序。

**2. 数据发送不正确或无法发送数据**

这可能是由于几种原因引起的，包括：硬件连接问题、发送缓冲区问题、发送程序问题等。故障排查步骤可能包括：

- 检查硬件连接，确保TX线正确连接，并且地线（GND）也要正确连接。
- 检查发送缓冲区和发送程序。确保发送的数据被正确地放入缓冲区，并且发送程序能正确地从缓冲区取出数据并发送出去。

**3. 通信距离问题**

UART通信的距离有限，如果通信距离过长，可能会导致信号衰减，从而影响数据的接收。如果需要在更长的距离进行通信，可能需要使用RS-422或RS-485等差分信号标准。

**4. 干扰问题**

在有电磁干扰的环境中，UART通信可能会受到影响。如果有可能的话，尝试减少电磁干扰，使用屏蔽线来减少干扰或者采用电容滤波以降低噪声和电磁干扰。如果干扰过大，可能需要使用差分信号标准或光电隔离。

在进行故障排查时，一种常见的方法是使用逻辑分析器或示波器来观察UART的信号，这可以帮助确定问题的来源。在软件层面，也可以使用调试工具来观察和分析程序的运行情况。