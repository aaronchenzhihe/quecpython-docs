# 文件系统

本文档旨在介绍QuecPython文件系统类型、使用方式和常见问题，指导客户使用QuecPython文件系统功能。

## 概述

文件系统是指文件和对文件进行操作和管理的软件的集合。文件系统实现了存储空间管理、构造文件结构、提供访问文件的操作接口。使用文件系统存储方式可以方便进行文件的增、删、查、改。

## 文件系统类型

目前各个平台使用的文件系统类型有EFS、SPIFFS、littleFS、FATFS。

| 文件系统名称 | 特点                           | 适用场景                                        | 适用型号                         | 开源网址                                     |
| ------------ | ------------------------------ | ----------------------------------------------- | -------------------------------- | -------------------------------------------- |
| EFS          | 支持各种NOR和NAND技术          | 嵌入式系统NOR flash、NAND flash、SD卡、EMMC存储 | BG95系列模组底层                 | 高通特有，非开源                             |
| SPIFFS       | 低资源消耗、擦写均衡、掉电保护 | 嵌入式系统SPI NOR flash存储                     | ECx00U&EGx00U&ECx00G系列模组底层 | https://github.com/pellepl/spiffs            |
| littleFS     | 低资源消耗、擦写均衡、掉电保护 | 嵌入式系统SPI NOR flash存储                     | 各型号QuecPython应用层           | https://github.com/littlefs-project/littlefs |
| FATFS        | 兼容Windows FAT32格式          | 嵌入式系统SD卡和EMMC存储                        | 各型号SD卡和EMMC存储场景         | http://elm-chan.org/fsw/ff/00index_e.html    |

### VFS

虚拟文件系统。在上述实体文件系统基础之上抽象的文件系统，提供统一的接口访问不同的实体文件系统。VFS API兼容POSIX标准API。具体操作步骤如下。

#### 初始化实体文件系统

该步主要进行存储介质硬件初始化，然后挂载实体文件系统，最后获取到实体文件系统的句柄和文件操作接口。每个实体文件系统有各自独立的硬件初始化和挂载接口。如果初始化成功，这些接口最终返回实体文件系统的对象，对象中包含有实体文件系统句柄和文件操作接口等信息。

- littleFS对应初始化接口为uos.VfsLfs1()（接口用法参考[注册存储设备 - SPI NOR FLASH](../../api_reference/stdlib/uos.md#注册存储设备---SPI-NOR-FLASH)）
- SPI SD卡FATFS对应初始化接口为uos.VfsFat()（接口用法参考[注册存储设备 - SPI - SD卡](../../api_reference/stdlib/uos.md#注册存储设备---SPI---SD卡)）
- SDIO SD卡FATFS对应初始化接口为uos.VfsSd()（接口用法参考[注册存储设备 - SDIO - SD卡](../../api_reference/stdlib/uos.md#注册存储设备---SDIO---SD卡)）

#### 挂载虚拟文件系统

该步将实体文件系统接口绑定到虚拟文件系统接口。具体接口为uos.mount(vfs_obj, path)。其中参数vfs_obj为上一步初始化实体文件系统返回的对象，参数path为虚拟文件系统的根目录，虚拟文件系统正是以根目录来区分不同的实体文件系统，即每一个实体文件系统绑定一个不同的根目录。根据应用场景，文件系统根目录可以分为：内置NOR flash用户区usr、内置NOR flash备份区bak、外置NOR flash区ext、SD卡区sd、EMMC区emmc。这样不同的存储区域可以使用同一套软件接口传入不同的根目录进行访问。接口用法参考[挂载文件系统](../../api_reference/stdlib/uos.md#挂载文件系统)。

#### 卸载虚拟文件系统

将实体文件系统接口和虚拟文件系统接口解绑，具体接口为uos.umount(path)，其中path同uos.mount()接口传入的path一致。

![](../media/system/fs/FS_1.png)

## 应用

### 基础文件操作

#### POSIX API

当进行基础的文件操作时可直接使用POSIX的接口进行开发。

##### 打开文件

```python
open(file, mode="r")
```

- file: 文件路径(在QuecPython里，用户文件路径在/usr下，所以用户创建读取文件都要在此目录下进行)

- mode: 打开模式，可选，默认为只读

| 模式 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| w    | 以只写方式打开文件。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
| r    | 以只读方式打开文件。文件的指针将会放在文件的开头。           |
| w+   | 以读写方式打开文件。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
| r+   | 以读写方式打开文件。该文件必须存在。文件的指针将会放在文件的开头。 |
| wb   | 以只写方式打开二进制文件。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
| rb   | 以只读方式打开二进制文件。文件的指针将会放在文件的开头。     |
| wb+  | 以读写方式打开二进制文件。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
| rb+  | 以读写方式打开二进制文件。该文件必须存在。文件的指针将会放在文件的开头。 |
| a    | 打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。 |
| a+   | 打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行读写。 |
| ab   | 打开一个二进制文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。 |
| ab+  | 打开一个二进制文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行读写。 |

##### 写入文件

```python
write(str)
```

- str: 要写入的数据

该接口返回成功写入的字符串长度

##### 读取文件

```python
read(size)
```

- size:要读取的长度，单位字节

该接口返回成功读到的数据

读取文件的一行

```python
readline(size)
```

- size:要读取的长度，单位字节

调用此接口会根据结束符自动读取，并返回一个等于size长度的字符串，如果size为-1则返回整行。

读取文件所有行

```python
readlines()
```

调用此接口会根据结束符自动分行读取，并返回一个包含所有分行的列表。

##### 移动文件指针

```python
seek(offset, whence)
```

- offset:  开始的偏移量，也就是代表需要移动偏移的字节数
- whence: 可选，默认值为 0。给offset参数一个定义，表示要从哪个位置开始偏移；0代表从文件开头开始算起，1代表从当前位置开始算起，2代表从文件末尾算起。

##### 关闭文件

```python
close()
```

调用后关闭文件，不再继续操作该文件。

##### 综合示例

```python
# 创建一个test.txt文件,注意路径在/usr下
f = open("/usr/test.txt", "w+")
i = f.write("hello world\n")
i = i + f.write("hello quecpython\n")
print("write length{}".format(i))
f.seek(0)
str = f.read(10)
print("read content:{}".format(str))
f.seek(0)
str = f.readline()
print("read line content:{}".format(str))
f.seek(0)
str = f.readlines()
print("read all lines content:{}".format(str))
f.close()
print("test end")
```

##### 运行结果

```python
import example
>>> example.exec('/usr/example_fs_basic_operation_posix.py')

write length29
read content:hello worl
read line content:hello world

read all lines content:['hello world\n', 'hello quecpython\n']

test end
>>>
```

[点此在github中下载完整代码](https://github.com/QuecPython/examples/blob/main/system/fs/example_fs_basic_operation_posix.py)。

#### uos API

当进行目录等操作的时候，需要调用uos库进行操作，接口用法参考[uos - 基本系统服务](../../api_reference/stdlib/uos.md)。

##### 综合示例

```python
import uos

def main():
    # 创建文件 此处并没有对文件进行写入操作
    f = open("/usr/uos_test","w")
    f.close()
    del f

    # 查看文件是否存在
    t = uos.listdir("/usr")
    print("usr files:{}".format(t))

    if "uos_test" not in t:
        print("file not exist test fail")
        return

    # 查看文件状态
    a = uos.stat("/usr/uos_test")
    print("file size:{}bytes".format(a[6]))

    # 重命名文件
    uos.rename("/usr/uos_test", "/usr/uos_test_new")
    t = uos.listdir("/usr")
    print("test file renamed, usr files:{}".format(t))

    if "uos_test_new" not in t:
        print("renamed file not exist test fail")
        return

    # 删除文件
    uos.remove("/usr/uos_test_new")
    t = uos.listdir("/usr")
    print("remove test file, usr files:{}".format(t))

    if "uos_test_new" in t:
        print("remove file fail, test fail")
        return

    # 目录操作
    t = uos.getcwd()
    print("current path:{}".format(t))
    uos.chdir("/usr")
    t = uos.getcwd()
    print("current path:{}".format(t))
    if "/usr" != t:
        print("dir change fail")
        return

    uos.mkdir("testdir")
    t = uos.listdir("/usr")
    print("make dir, usr files:{}".format(t))

    if "testdir" not in t:
        print("make dir fail")
        return

    uos.rmdir("testdir")
    t = uos.listdir("/usr")
    print("remove test dir, usr files:{}".format(t))

    if "testdir" in t:
        print("remove dir fail")
        return

if __name__ == "__main__":
    main()
```

##### 运行结果

```python
>>> example.exec('/usr/example_fs_basic_operation_uos.py')

usr files:['system_config.json', 'example_fs_basic_operation_uos.py', 'uos_test']
file size:0bytes

test file renamed, usr files:['system_config.json', 'example_fs_basic_operation_uos.py', 'uos_test_new']

remove test file, usr files:['system_config.json', 'example_fs_basic_operation_uos.py']
current path:/
current path:/usr

make dir, usr files:['system_config.json', 'example_fs_basic_operation_uos.py', 'testdir']

remove test dir, usr files:['system_config.json', 'example_fs_basic_operation_uos.py']
>>>
```

[点此在github中下载完整代码](https://github.com/QuecPython/examples/blob/main/system/fs/example_fs_basic_operation_uos.py)。

### 高级文件操作

ql_fs - 高级文件操作，接口用法参考[ql_fs - 高级文件操作](../../api_reference/syslib/ql_fs.md)。

##### 综合示例

```python
import ql_fs

def main():
    # 递归式创建文件夹, 传入文件夹路径
    ql_fs.mkdirs("usr/a/b")

    # 查看文件或文件夹是否存在
    ret = ql_fs.path_exists("/usr/a/b")
    if ret:
        print("make dir success")
    else:
        print("make dir fail")
        return

    # 创建文件或者更新文件数据
    data = {"test":1}
    ql_fs.touch("/usr/a/b/config.json", data)

    # 查看文件或文件夹是否存在
    ret = ql_fs.path_exists("/usr/a/b/config.json")
    if ret:
        print("create file success")
    else:
        print("create file fail")
        return

    # 读取json文件
    data = ql_fs.read_json("/usr/a/b/config.json")
    print("config json read content:{}".format(data))
    data = ql_fs.read_json("/usr/system_config.json")
    len = ql_fs.path_getsize('usr/system_config.json')
    print("system_config json length:{}".format(len))
    print("system_config json read content:{}".format(data))

    # 文件拷贝
    ql_fs.file_copy("/usr/a/b/config.json", "usr/system_config.json")
    data = ql_fs.read_json("/usr/a/b/config.json")
    print("copy json read content:{}".format(data))

    # 获取文件所在文件夹路径
    ret = ql_fs.path_dirname("/usr/a/b/config.json")
    print("path of the file:{}".format(ret))

    # 删除文件夹和其下的文件
    ql_fs.rmdirs("usr/a")

    # 查看文件或文件夹是否存在
    ret = ql_fs.path_exists("/usr/a/b/config.json")
    if ret:
        print("remove file fail, test fail")

    ret = ql_fs.path_exists("/usr/a/b")
    if ret:
        print("remove dir2 fail, test fail")

    ret = ql_fs.path_exists("/usr/a")
    if ret:
        print("remove dir1 fail, test fail")

if __name__ == "__main__":
    main()
```

##### 运行结果

```python
>>>
example.exec('/usr/example_fs_advanced_operation.py')
make dir success

create file success
config json read content:{'test': 1}
system_config json length:15
system_config json read content:{'replFlag': 0}

copy json read content:{'replFlag': 0}
path of the file:/usr/a/b

>>>
```

[点此在github中下载完整代码](https://github.com/QuecPython/examples/blob/main/system/fs/example_fs_advanced_operation.py)。

## 备份还原

### 概述 - 备份还原

QuecPython设备软件由固件和用户应用脚本2部分组成。其中固件存放在系统程序分区包括kernel和QuecPython VM，而用户应用脚本存放在设备文件系统分区。为了确保系统稳定性，设计了文件系统备份还原机制。当前设备划分了2个文件系统分区，分别是：用户文件系统usr和备份文件系统bak。用户文件系统存放用户正常运行需要的py脚本和数据文件，可读写。备份文件系统用来备份用户文件系统中的出厂原始文件，只读。如果开启了备份还原功能，当用户文件系统中的文件被误删除或意外被修改时，会自动从备份文件系统还原原始的文件到用户文件系统。NOR flash空间分布如下图：

![](../media/system/fs/FS_2.png)

### 实现原理

![](../media/system/fs/backup_restore.png)

#### 备份

对用户文件系统的文件进行备份，具体流程是：

1.将用户文件系统下的源文件拷贝一份到备份文件系统。

2.在用户文件系统和备份文件系统各生成一个checksum.json文件。该文件存放的内容是每一个源文件的文件名和其对应的checksum值，checksum值是根据每一个源文件的内容通过CRC32算法计算出来的，所以每一个源文件都有唯一对应的一个checksum值。

3.在备份文件系统生成一个备份还原标志文件backup_restore.json，记录是否开启备份还原功能。

#### 还原

当用户文件系统的文件发生损坏时，从备份文件系统中拷贝对应的文件到用户文件系统。具体流程是：
1.开机检测到备份文件系统中备份还原的标志使能，且用户文件系统下的checksum.json文件存在，这时如果用户文件系统下的某个源文件被删除，则会将备份文件系统中对应源文件拷贝到用户文件系统下，并更新该文件对应的checksum值到用户文件系统下的checksum.json文件中。

2.开机检测到备份文件系统中备份还原的标志使能，且用户文件系统下的checksum.json文件存在，这时如果用户文件系统下的某个源文件被破坏，则会将备份文件系统中对应源文件拷贝到用户文件系统下，并更新该文件对应的checksum值到用户文件系统下的checksum.json文件中。

3.开机检测到备份文件系统中备份还原的标志使能，这时如果用户文件系统下的checksum.json文件不存在，则会将备份文件系统中checksum.json文件拷贝一份到用户文件系统，然后再走上述1、2步的检测流程。

#### OTA更新

当用户文件系统的文件需要OTA升级时，在升级成功后，会将升级后的用户文件的checksum值更新到用户文件系统下的checksum.json文件中。

> 注意：不会更新备份文件系统下的checksum文件和源文件。

### 工具操作

1.选择项目。

2.选择待合并的固件。

3.右键选择usr弹出对话框。

4.点击添加文件选项，添加待合并备份的文件。

5.已添加的待合并的文件。

6.勾选备份按钮。

7.点击合并按钮进行合并固件。合并成功即可生成带备份还原功能的量产固件。

![](../media/system/fs/FS_5.png)

1.右键选择usr弹出对话框。

2.点击添加文件选项，添加待合并备份的文件。

![](../media/system/fs/FS_3.png)

1.已添加的待合并的文件。

2.勾选备份按钮。

3.点击合并按钮进行合并固件。合并成功即可生成带备份还原功能的固件。

![](../media/system/fs/FS_4.png)

## 注意事项

### 掉电保护

为了防止文件操作过程中突然掉电导致的已有数据丢失或文件系统损坏，部分文件系统被设计可以处理随机电源故障。即所有文件操作都有写时复制保证，如果断电，文件系统将恢复到上次已知的完好状态。前述的SPIFFS和littleFS都支持该特性。

### 擦写均衡

由于NOR flash存储的擦写寿命是有限的，为了避免对存储空间某一块频繁擦写导致该块无法使用进而影响到整个flash存储的正常使用的问题，需要设计一套算法将用户的擦写操作平均分散到整个NOR flash存储空间上。而部分文件系统在设计时已经考虑了该特性，如前述的SPIFFS和littleFS文件系统。

### 读写速率

1.读写速率主要取决于硬件通信接口，SPI 6线模式要快于SPI 4线模式，SDIO 4线模式要快于SPI模式。提高速率的方法，减少open、close的频率，文件open一次直到write完所有内容后再close。

2.对于SD卡应用场景，当存储的文件个数超过一定数量如1000个，这时对文件执行open、write、close操作会比较耗时。这时可以减少文件个数，或者增加子目录，在每个子目录中事先创建好每个空文件，以规避这个问题。

3.针对部分应用场景实时性要求比较高的情况，如果可以，使用裸flash接口访问。如GUI字库文件。同时，也可以采用缓存机制提高访问速度，将经常访问的数据缓存到RAM中，下次访问时直接从RAM中读取，如GUI显示用到的图片文件。

### 空间利用效率

由于NOR flash存储空间有限，这里主要考虑NOR flash littleFS文件系统空间使用情况。

#### 小文件存储机制：

当文件大小小于等于一个块（如4KB）时，其消耗恒定为一个块。即使是远小于一个块的文件（如1byte），也要消耗一整个块的空间。这是因为littlefs1.0文件系统缺乏复用多余块空间的机制。而littlefs2.0中引入内联文件机制，则会将小文件存放到所在文件夹所占空间中以减少空间的占用。

#### 大文件存储机制：

当文件大小达到一个块以上时，需要引入ctz逆序链表的机制。相比于传统文件系统的链表，这种链表为逆序，追加数据时不需要额外的开销来重新建立所有的索引。且引入了ctz指针机制，即block N如果是一个能被2^X整除的数，那么他就存在指向N – 2^X的指针。大文件的指针信息和文件本身的内容存储在同一空间，计算文件实际大小时，需要考虑指针占用的空间。所以，大文件的实际使用空间为其本身占用的空间，加上CTZ指针消耗的空间。

#### 文件夹存储机制：

在littlefs文件系统中，新建一个文件夹需要创建一组新的metadata pair来维护此文件夹下的内容，因此会造成两个block的开销。综上，为了高效使用存储空间，避免大量小文件的存储，及避免使用文件夹。

## 常见问题

### 挂载失败

1.如果抛异常”OSError: [Errno1] EPERM“，是因为重复调用uos.mount()。

2.如果外置spi NOR flash挂载抛异常”OSError: [Errno 19] ENODEV“，一般是硬件连接异常，如spi port不对应，硬件连接不可靠。

### 创建文件失败、写文件失败

1.一种情况是，部分型号如果文件没有close重复open则会返回失败。需要确保open、close成对操作。

2.空间还剩余挺多，写文件抛异常error 28报空间不足。如果有f.seek()操作，由于文件系统掉电保护机制，需要保证剩余空间大于seek的目标文件位置到文件末尾的长度。

### with open 和 open的区别

open是python的一个内置接口。with open是使用了with语句的open接口。open()完成后必须调用close()接口关闭文件。因为文件对象会占用系统的资源，同一时间能打开的文件数量也是有限的，另外不成对使用可能会导致其他异常。由于文件读写时有可能产生IO异常，一旦出错，后面的close()就不会调用。with open则可以避免这样的情况，即便在文件读写过程中发生IO异常，也会自动调用close()接口关闭文件。

```python
with open('/usr/test.txt','w+')as f:
    f.write('1234567890')
```