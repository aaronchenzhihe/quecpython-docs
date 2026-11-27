# 存储设备应用指导

本文档旨在介绍QuecPython存储设备类型、使用方式和常见问题，指导客户使用QuecPython存储设备。

## 概述

QuecPython设备存储系统分为内置存储系统和外置存储系统。内置存储系统一般是NAND flash或者NOR flash，外置存储一般是通过SPI、SDIO等接口外扩存储空间，可以是NOR flash、SD卡、EMMC等。使用存储的方式有两种：一种是挂载文件系统，用于存放数据和参数。常用的文件系统有littleFS、FATFS、EFS、SFFS文件系统等。另一种是不挂文件系统，直接裸分区访问，用于存放代码镜像、配置参数、GUI字库等。其中NOR flash可以直接运行代码，NAND flash不能直接运行代码。

## 内置存储

### 概述 - 内置存储

设备内置的存储设备为模组的正常运作提供了必须的存储空间，用来存放镜像和数据。一般会按照作用划分为多个分区，每个分区相对独立。根据功能需求，决定是挂文件系统还是作为裸分区使用。

### 分区划分

不同芯片平台的模组有着不同的分区结构。总体来说可以分为以下几个部分：

a）程序存储分区：底层和应用层处理程序等镜像存放的区域。
b）出厂参数存储分区：射频参数，模组配置等存放的区域。并且会有个备份区，在数据异常损坏时，会从备份区恢复，保证模块正常运行。
c）文件系统分区：用于存放用户应用脚本和用户参数。

#### 各平台分区结构

##### ECx00N & EGx00N & ECx00M & EGx00M系列

以EC600N型号为例：

| 分区                | 作用                             |
| ------------------- | -------------------------------- |
| bootloader          | 存储bootloader镜像               |
| ptable              | 存储flash分区信息                |
| fwcerts             | 存储固件证书                     |
| rd                  | 存储可信数据                     |
| apn                 | 存储APN镜像                      |
| cp                  | 存储kernel侧镜像                 |
| dsp                 | 存储网络协议栈镜像               |
| rfbin               | 存储射频驱动镜像                 |
| logo                | 存储开机logo显示镜像             |
| reserved            | 预留空间                         |
| fota_pkg            | 存储FOTA差分包                   |
| customer_app        | 存储APP镜像(QuecPython程序实现)  |
| customer_fs         | 用户文件系统(对应磁盘"usr")      |
| customer_backup_fs  | 用户文件备份系统(对应磁盘"bak")  |
| backup_restore_info | 存储备份还原信息                 |
| fota_param          | 存储FOTA升级参数                 |
| updater             | 存储FOTA升级程序镜像             |
| erase_rd            | 存储flash擦除信息记录            |
| nvm                 | 系统参数文件系统                 |
| secdat              | 存储用户非易失数据               |
| quec_cfg            | 移远自定义参数分区               |
| factory_a           | 存储基础配置及射频等出厂参数     |
| factory_b           | 存储基础配置及射频等出厂参数备份 |

##### ECx00U & EGx00U & ECx00G系列

以EC600U型号为例：

| 分区               | 作用                                                     |
| ------------------ | -------------------------------------------------------- |
| bootloader         | 存储bootloader镜像                                       |
| ap                 | 存储kernel侧AP镜像(应用处理)                             |
| customer_app       | 存储APP镜像(QuecPython程序实现)                          |
| fs                 | 底层文件系统分区(存储FOTA差分包等)                       |
| secdat             | 存储用户非易失数据                                       |
| fs_config          | 存储用户文件系统配置信息(用来动态调整"usr"和"bak"的大小) |
| customer_fs        | 用户文件系统(对应磁盘"usr")                              |
| customer_backup_fs | 用户文件备份系统(对应磁盘"bak")                          |
| modem              | 存储kernel侧CP镜像(网络处理)                             |
| factory            | 存储基础配置及射频等出厂参数                             |

##### xC200A系列

以EC200A型号为例：

| 分区                | 作用                             |
| ------------------- | -------------------------------- |
| NTIM                | 存储分区表信息                   |
| bootloader          | 存储bootloader镜像               |
| MRD                 | 存储底层可靠参数                 |
| MRD_backup          | 存储底层可靠参数备份             |
| logo                | 存储开机logo显示镜像             |
| cp                  | 存储kernel侧镜像                 |
| customer_fs         | 用户文件系统(对应磁盘"usr")      |
| customer_backup_fs  | 用户文件备份系统(对应磁盘"bak")  |
| dsp                 | 存储网络协议栈镜像               |
| rf                  | 存储射频信息                     |
| fota_pkg            | 存储FOTA升级备份                 |
| PSM backup          | 存储PSM备份                      |
| FAT map             | 底层文件系统目录                 |
| FAT sys             | 底层文件系统                     |
| customer_app        | 存储APP镜像(QuecPython程序实现)  |
| block erase record  | 存储flash擦除记录                |
| updater             | 存储FOTA升级程序镜像             |
| FBF                 | 存储tr69参数                     |
| fota_param          | 存储FOTA升级参数                 |
| QuecCfg             | 存储底层重要参数                 |
| PSM                 | 存储底层网络协议参数             |
| VSIM profile        | 保留                             |
| VSIM profile backup | 保留                             |
| factory_a           | 存储基础配置及射频等出厂参数     |
| factory_b           | 存储基础配置及射频等出厂参数备份 |

##### ECx00E系列

以EC800E型号为例：

| 分区                    | 作用                                                |
| ----------------------- | --------------------------------------------------- |
| header1                 | 平台使用                                            |
| header2                 | 平台使用                                            |
| bootloader              | 存储bootloader镜像                                  |
| reserved                | 预留空间                                            |
| fota-backup             | 存储FOTA升级备分                                    |
| default reliable region | 存储工厂参数，和Reliable Region区域互为备份         |
| Reliable Region         | 存储工厂参数，和Default Reliable Region区域互为备份 |
| ap                      | 存储kernel侧镜像                                    |
| customer_app            | 存储APP侧镜像(QuecPython程序实现)                   |
| fs                      | 底层文件系统（存储FOTA差分包等）                    |
| customer_fs             | 用户文件系统(对应磁盘"usr")                         |
| customer_backup_fs      | 用户文件备份系统(对应磁盘"bak")                     |
| secdat                  | 存储用户非易失数据                                  |
| updater                 | 存储FOTA升级程序镜像                                |
| hibernate backup        | 存储休眠时重要参数                                  |
| PlatConfig(16K)         | 存储FOTA升级参数等                                  |

##### BG95 & BG600L系列

以BG95M3型号为例：

| 分区               | 作用                                        |
| ------------------ | ------------------------------------------- |
| reserved           | 预留空间                                    |
| bootloader         | 存储bootloader镜像                          |
| mibib              | 存储分区表信息                              |
| qdsp_swap          | 存储网络处理交换数据                        |
| efs2_factory       | efs2备份区域                                |
| efs2apps_factory   | efs2apps备份区域                            |
| efs2               | CP(网络处理)文件系统                        |
| qdsp_paging        | 存储kernel侧CP相关镜像(网络处理)            |
| qdsp               | 存储kernel侧CP镜像(网络处理)                |
| qdsp_tiny          | 存储kernel侧CP相关镜像(网络处理)            |
| tz                 | 存储Trustzone镜像                           |
| rpm                | 存储RPM镜像                                 |
| tz_devcfg          | 存储Trustzone配置镜像                       |
| ap                 | 存储kernel侧AP镜像(应用处理)                |
| adcb               | 存储audio镜像                               |
| apdp               | 存储应用调试镜像                            |
| loader_sti         | 存储测试镜像                                |
| multiimgoem        | 存储multiimgoem镜像                         |
| multiimgqti        | 存储multiimgqti镜像                         |
| scrub              | 存储FOTA差分包                              |
| secdata            | 存储安全相关的配置                          |
| misc               | 存储备份还原结果标志                        |
| customer_app       | 存储APP侧镜像(QuecPython程序实现)           |
| customer_backup_fs | 用户文件备份系统(对应磁盘"bak")             |
| efs2apps           | AP(应用处理)文件系统(包含用户文件系统"usr") |

各平台提供给用户的空间资源参考[QuecPython 硬件平台支持](../background/hardware-platform.md)。

#### 调整分区

由前面描述可知，提供给用户的可用空间分成两个区，分别为用户文件系统usr分区和用户文件备份系统bak分区。如果这2个分区的默认分布无法满足用户的应用需求，那么客户可以使用QPYcom工具调整该分区。可能需要调整分区的场景：需要存储的文件比较多，且部分文件不需要备份，如存储日志的应用。那么可以通过减小bak分区大小，增大usr分区大小来满足需求。

##### 支持的模组：

目前只有ECx00N & EGx00N & ECx00M & EGx00M系列和ECx00U & EGx00U & ECx00G & ECx00Z系列支持调整分区。且只支持用户文件系统usr和用户文件系统bak之间调整大小。

##### 操作：

以EC600U型号为例

1、点击 "Function Box" -> "Partition Management" -> "UNISOC"

![](../media/system/memory-device/REGIN_ADJUST1.png)

2、选择固件文件如QPY_OCPU_V0002_EC600U_CNLB_FW\QPY_OCPU_V0002_EC600U_CNLB_FW.pac，显示usr和bak分区各自的大小及两者之和的大小。

![](../media/system/memory-device/REGIN_ADJUST2.png)

3、拖拽进度条或者文本框输入修改usr分区的大小，bak分区大小自动更新为新的值。

4、点击"Confirm"按钮后开始生成新固件，等待生成新固件完成。

![](../media/system/memory-device/REGIN_ADJUST3.png)


烧录新生成的固件，并查询新的分区大小分布：

![](../media/system/memory-device/REGIN_ADJUST4.png)


#### 新增分区

##### 使用场景：

在固件释放出去之后，用户需要在原有分区表上新增分区满足应用需求，如新增GUI字库分区用来存放字库文件。

##### 支持的模组：

目前仅ECx00N & EGx00N & ECx00M & EGx00M系列支持新增分区。且只支持新增特定分区如GUI字库分区（需要搭配底层的GUI代码使用）。新增分区起始地址和最大size由QPYcom工具定死（ECx00N & EGx00N系列是将app镜像copy到RAM中运行，app的起始地址可以变动，新增空间时可以从reserved和customer_app分区挪空间。ECx00M & EGx00M系列是flash中直接运行app，app的起始地址不能变动，需要预先将多余空间分给customer_app分区，新增空间时可以从customer_app分区挪空间。）。

##### 操作：

待补充

#### 注意事项

文件系统分区最小size不能小于32KB。每个分区的起始地址和size需要是4KB的倍数。使用QPYcom工具调整分区时已经遵循了这个规则。

### 内置Flash烧录

#### 烧录原理

烧录功能设计每个平台不尽相同，总的烧录机制设计如下：

a）进入下载模式，一般为配置特定引脚的电平或者下发特定的指令后重启设备，即可进入下载模式。此时执行ROM中固化的程序。由于是固化在芯片中的，所以存储空间和运行空间都有限，因此逻辑上比较简单。

b）在下载模式下，初始化UART 或 USB硬件通信，然后通过 UART 或 USB下载扩展烧录程序到芯片的 RAM 中。

c）交出控制权，跳转到 RAM中的扩展烧录程序，执行进一步的烧录功能。这个步骤的主要用途是下载整个固件到 flash 中。它的功能包括初始化Flash，分区，读写 Flash 等。其中上位机和设备的通讯机制一般采用“命令－应答”方式。上位机应用程序发起命令，模块收到后执行相应的指令，再将应答送回应用端。交互流程主要分为：连接、下载开始、下载中、下载完成。命令和数据都会进行CRC校验确保数据的可靠性。

d）烧录完成，重启设备运行已烧录的固件。

用户实际使用时只需要利用烧录工具加载固件，让设备进入下载模式，使用USB或UART连接线连接设备即可进行烧录。具体烧录操作细节参见本地升级指导文档。

#### 烧录的分区概述

烧录固件时，针对不同分区处理方式不同。总体策略为镜像分区、用户文件系统usr分区、用户文件备份系统bak分区需要擦除并写入新的内容。安全数据区和底层参数区不擦写，保留之前的内容。

#### 量产固件

之所以要制作量产固件，是因为QuecPython释放的官方固件只包含了平台层和QuecPython适配层的程序。用户需要基于QuecPython Modules API，使用python语言编写程序实现完整的应用的功能。而这些python程序需要以文件的形式存放在用户文件系统usr分区和用户文件备份系统bak分区中。调试阶段可以使用QPYcom下载脚本的功能将用户文件烧录到设备用户文件系统分区中。而量产时为了提高效率，需要将用户调试好的文件事先打包进QuecPython官方固件中，以实现QuecPython官方固件和用户文件一起烧录到设备中。总结大概步骤为：设备出厂时会带有默认固件，该固件不包含QuecPython相关功能。用户需要烧录QuecPython官方释放的固件，基于QuecPython平台开发调试好用户应用功能后，将用户的脚本文件和QuecPython固件打包成一个固件。将该固件作为最终量产固件烧录到所有设备中。制作量产固件细节可参考[QuecPython 固件脚本合并](../dev-tools/QPYcom/qpycom-merge.md)。

### 内置存储常见应用

#### 程序镜像

存放设备运行需要的程序镜像，一般包括bootloader、kernel、app几个部分。

#### 文件系统

将指定分区挂载成文件系统，用以存放数据。这样可以使用文件的形式对数据进行管理，包括创建、读、写、删除等操作。适合数据量大，经常修改且非关键数据的存储。文件系统使用细节参考[文件系统应用指导](./fs.md)。

#### GUI 字库

将指定分区用来存储GUI显示所需的字库内容。该字库内容包含了不同的字体信息，且有固定的结构。所以需要在烧录固件时一并将字库文件烧录到设备的指定分区。这样GUI显示程序可以通过flash操作接口直接访问字库内容，相比于文件系统的方式速率更快，可以提高GUI显示的实时性。

#### 裸分区操作

如安全数据分区，将指定分区用来存储用户安全数据，以达到数据掉电保存的目的。数据来源为设备运行过程中用户实时产生的内容。所以，该分区不需要事先烧录特定内容到指定分区，且烧录时不擦除该分区。这种方式适合数据量小，不经常修改且关键数据的存储。接口用法参考[SecureData - 安全数据区](../../api_reference/syslib/securedata.md)。

## 外置存储

### 概述 - 外置存储

模组内置存储空间主要用于存放代码镜像和参数，剩余给客户可用的空间有限。如果需要额外的存储空间，就需要外挂存储设备。比如NOR flash、SD卡、NAND flash等。硬件通信接口有SDIO 接口和SPI 接口，目前支持的方式有：SPI NOR flash、SDIO SD卡、SDIO EMMC、SPI SD卡。

### SPI NOR flash应用

#### 原理概述 - SPI NOR flash应用

该种使用方式是在模组的外面通过SPI硬件引脚连接一个NOR flash存储芯片。然后通过应用程序初始化该flash驱动之后，就可以对其进行读写等操作。具体选用哪一款flash芯片，需要用户根据自己的空间需求决定。根据通信的SPI引脚个数可分为4线外挂flash和6线外挂flash方案。其中6线flash方案比4线的方案通信速率更高，所以6线外挂flash方案可以直接运行程序。

#### 应用场景

##### 程序镜像

使用外置flash指定分区存放QuecPython程序镜像，以便可以打开底层更多功能特性，满足用户丰富的应用场景需求。如前所述，该种方案必须使用6线SPI NOR flash。该种方式程序运行和放置在内置存储中的程序运行效果一致。目前仅ECx00U&EGx00U&ECx00G系列特定型号EC600U_CNLB_EXTFS8M、EC600U_CNLC_EXTFS8M支持该特性。

##### 文件系统

类似于内部存储，可以将外置flash指定分区挂载成文件系统，用以存放数据。这样可以使用文件的形式对数据进行管理，包括创建、读、写、删除等操作。适合数据量大，经常修改且非关键数据的存储。外置flash文件系统使用细节参考[文件系统应用指导](./fs.md)。

##### GUI 字库

类似于内部存储，可以将外置flash指定分区用来存储GUI显示所需的字库内容。该字库内容包含了不同的字体信息，且有固定的结构。所以需要在烧录固件时一并将字库文件烧录到设备的指定分区。这样GUI显示程序可以通过flash操作接口直接访问字库内容，相比于文件系统的方式速率更快，可以提高GUI显示的实时性。

#### 支持的模组

当前仅ECx00N & EGx00N & ECx00M & EGx00M系列和ECx00U & EGx00U & ECx00G系列支持外挂SPI NOR flash。

#### 应用时的注意事项 - SPI NOR flash应用

##### 硬件连接

采用的是SPI通信总线，具体选用哪一路SPI引脚需要根据硬件资源支持和用户使用情况决定。

针对4线SPI NOR flash方案，引脚细节可以参考[class SPI – SPI通信](../../api_reference/peripherals/machine.SPI.md)中关于SPI引脚对应关系的介绍。由于可能会有其他外设如LCD也需要使用SPI通信，这时需要用户提前分配好硬件资源，各自使用不同的SPI通道，避免和NOR flash冲突的情况。

针对6线SPI NOR flash方案，当前仅ECx00U & EGx00U & ECx00G系列支持，引脚使用情况，需要参考如下表格：

| 用途 | 引脚功能       | EC200U引脚号 | EC600U引脚号 | EG915U引脚号 | EC800G引脚号 | EC600G引脚号 |
| ---- | -------------- | ------------ | ------------ | ------------ | ------------ | ------------ |
| SPI1 | SPI_FLASH_CS   | 26           | 58           | 5            | 79           | 63           |
|      | SPI_FLASH_CLK  | 27           | 61           | 4            | 101          | 66           |
|      | SPI_FLASH_SIO0 | 24           | 59           | 6            | 100          | 67           |
|      | SPI_FLASH_SIO1 | 25           | 60           | 7            | 108          | 65           |
|      | SPI_FLASH_SIO2 | 13           | 69           | 1            | 109          | 137          |
|      | SPI_FLASH_SIO3 | 135          | 70           | 20           | 107          | 62           |
|      |                |              |              |              |              |              |
| SPI2 | SPI_FLASH_CS   | 124          | 63           | 106          | 无           | 无           |
|      | SPI_FLASH_CLK  | 125          | 66           | 116          | 无           | 无           |
|      | SPI_FLASH_SIO0 | 123          | 67           | 105          | 无           | 无           |
|      | SPI_FLASH_SIO1 | 122          | 65           | 16           | 无           | 无           |
|      | SPI_FLASH_SIO2 | 121          | 137          | 108          | 无           | 无           |
|      | SPI_FLASH_SIO3 | 119          | 62           | 92           | 无           | 无           |

如表格所示，针对ECx00U & EGx00U系列，如果用户不使用MIPI LCD功能，仅外接6线SPI NOR flash。那么可选择上面两路SPI中任意一路作为flash SPI通信接口。如果要同时支持外置6线SPI NOR flash和MIPI LCD功能，则flash SPI通信接口必须选用SPI2这路引脚。针对ECx00G系列，当前只有一路引脚可作为flash SPI通信接口。

##### 添加型号

用户选好NOR flash芯片型号之后需要查看设备程序是否支持该flash型号。可以通过QPYcom工具加载固件版本查看该固件当前已支持的flash型号（该工具功能待开发）（待补充）。

如果flash型号不在设备支持列表中，需要添加flash型号到设备中。可以在使用QPYcom工具合并固件的时候添加指定的flash型号（该工具功能待开发）（待补充）。

##### 分区调整

在固件释放出去之后，用户需要在原有分区表上新增外置flash分区满足应用需求，如新增GUI字库分区用来存放字库文件。目前仅ECx00N & EGx00N & ECx00M & EGx00M系列支持新增分区。如新增文件系统分区、GUI字库分区等（需要搭配底层的GUI代码使用）。新增分区起始地址和size可由QPYcom工具指定。同理，用户也可以调整外置flash分区，如调整文件系统分区和GUI字库分区分别占用的大小。具体操作待QPYcom工具支持后补充（待补充）。

##### 烧录

带外置flash的固件烧录方式和内置flash的固件一致。需要注意的是外置flash烧录依赖于外置的flash芯片，需要保证外置flash通信是成功的。需要在制作量产固件时指定烧录工具的SPI端口信息，具体操作待QPYcom工具支持后补充。如果下载失败，检查flash芯片是否正确焊接，spi引脚的连接是否正确。

##### 量产固件

制作带外置flash功能的量产固件和内置flash的方式总体一致。特别的是外置flash需要配置flash的型号、指定spi port和分区划分。具体操作待QPYcom工具支持后补充。（待补充）

##### 软件接口

其中文件系统分区使用的场景，文件系统需要执行挂载操作后才能使用，外置4线spi nor flash文件系统初始化接口用法参考[注册littleFS存储设备 - SPI NOR FLASH](../../api_reference/stdlib/uos.md#注册存储设备---SPI-NOR-FLASH)。外置6线spi nor flash仅特定型号特定固件支持，且不支持用户进行挂载操作，仅作内部使用。

##### 读写速率

由于外置flash采用的是4线spi通信，内置flash采用的是6线spi通信。且内置flash spi clock比外置flash spi clock要高。综上因素导致外置flash的速率低于内置flash。大约相差4~8倍。所以在使用外置flash开发应用时，需要考虑速率瓶颈对业务的影响，如实时性要求高的GUI显示功能，涉及到图片文件和字库文件的存储和访问。针对这种应用场景，可以采用缓存机制减少影响，将近期常访问的数据缓存到RAM中，下次访问相同的数据时直接从RAM中读取。该机制一般由固件底层实现，用户可以配置缓存的空间大小和数量。

### SD卡应用

#### 原理概述 - SD卡应用

SD 卡(Secure Digital Memory Card)是一种基于半导体闪存工艺的存储卡。按照外观可以分为SD、mini-SD和Micro-SD。按照SD卡的容量，可以分为SDSC（SD Standard Capacity）（容量不大于2GB）、SDHC（SD High Capacity）（容量大于2GB不大于32GB）、SDXC（SD Extended Capacity）（容量大于32GB不大于2TGB）三种标准。通信接口可以选择SDIO接口或者SPI接口。

#### SDIO接口

SDIO接口方式，使用SDIO总线与SD卡进行通信。SDIO（Secure Digital Input and Output）是安全数字输入输出接口，是从SD内存卡接口的基础上演化出来的一种外设接口。SDIO接口兼容以前的SD内存卡，并且可以连接支持SDIO接口的设备。SDIO信号有三种模式，“单线模式”、“4线模式”、“SPI模式”。这里使用的是“4线模式”。

#### SPI接口

SPI接口方式，使用通用的SPI总线与SD卡进行通信，包括初始化SD卡、对SD卡读、写操作等。

#### 应用时的注意事项 - SD卡应用

##### 硬件连接

SDIO接口的硬件引脚使用参考硬件设计手册SD卡接口相关章节。SPI接口的硬件引脚使用参考[class SPI – SPI通信](../../api_reference/peripherals/machine.SPI.md)中关于SPI引脚对应关系的介绍。

##### 文件系统类型

SD卡功能采用的文件系统是FATFS，支持Windows兼容的FAT32格式。

##### 软件接口

SD卡功能需要执行文件系统挂载操作后才能使用，SPI SD卡具体接口用法参考[注册存储设备 - SPI - SD卡](../../api_reference/stdlib/uos.md#注册存储设备---SPI---SD卡)，SDIO SD卡具体接口用法参考[注册存储设备 - SDIO - SD卡](../../api_reference/stdlib/uos.md#注册存储设备---SDIO---SD卡)。

##### 热插拔

当前仅SDIO SD卡支持卡插拔检测功能，而且需要搭配特定代码使用，具体接口用法参考[设置SD卡检测管脚](../../api_reference/stdlib/uos.md#设置SD卡检测管脚)。

##### 读写速率

由于SDIO接口有4条数据线进行数据传输，而SPI只有一条数据线，所以SDIO模式读写速率会比SPI模式高。SDIO SD卡write速率可以达到5MB/s，SPI SD卡write速率可以达到600KB/s。

##### 应用场景

常用场景有存放音频文件（使用流播放的方式播放音频文件）、存放历史定位信息、存放日志等。

#### 支持的模组

SDIO SD卡功能仅ECx00U & EGx00U系列支持。SPI SD卡功能仅ECx00N & EGx00N系列支持。

#### 常见问题 - SD卡应用

##### 挂载失败

如果出现挂载文件系统失败的情况，说明硬件没有通信成功。需要检查SD卡接口引脚是否连接正确且牢固。同时检查SD卡供电电压是否满足SD卡标准要求。

##### 格式化

如果使用过程中出现文件系统异常，或者文件系统空间存满的情况，可以调用格式化接口对SD卡进行格式化。具体接口用法参考[挂载文件系统](../../api_reference/stdlib/uos.md#挂载文件系统)。

### EMMC应用

#### 原理概述 - EMMC应用

EMMC（Embedded Multimedia Card）嵌入式多媒体卡是一种新的存储技术，由MMC协会订立。主要用于印刷电路板的嵌入式非易失性存储器系统。不可随意移动，是永久性的电路板附件。EMMC和SD的技术本质是一样的，都是从MMC规范演化而来，使用相同的总线规范，SD强调的是安全和数据保护，EMMC强调的是嵌入式的多媒体存储。

#### 通信接口

采用SDIO通信接口。

#### 使用方式

EMMC功能需要执行文件系统挂载操作后才能使用，具体接口用法参考待补充。

#### 支持的模组

仅EC200ACNLA型号支持。

### 常见问题

#### NOR flash寿命

NOR flash最大擦写次数大概是10万次。使用过程中需要避免频繁擦写操作。NOR flash的接口简单，数据操作少，位交换操作少。因此可靠性高，认为不会出现坏块，不需要进行坏块管理。

#### NAND flash寿命

NAND flash最大擦写次数大概是100万次，使用过程中需要避免频繁擦写操作。NAND Flash接口和操作均相对复杂，位交换操作也很多，因此出现问题的几率要大得多，坏块不可避免。需要进行坏块管理，坏块管理机制由底层代码实现。

#### NOR flash和NAND flash特性差异

NOR flash可以直接运行程序，而NAND flash不行。由于对NAND flash闪存的操作都是以块和页为单位的，所以在向NAND flash闪存进行大量数据的读写时，速度要快于NOR flash。NOR flash的容量较小，一般在1~16MB左右。NAND Flash闪存的容量比较大，最大容量己达到8GB。

