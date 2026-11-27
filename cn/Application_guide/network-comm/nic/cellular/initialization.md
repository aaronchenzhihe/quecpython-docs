# 蜂窝网卡初始化流程

蜂窝无线网卡的初始化，需要经历如下几个阶段：

![无线网卡初始化流程](../../../media/network-comm/nic/cellular/网卡初始化流程.png)

## 硬件初始化

这个阶段会先进行SIM卡的检测，判断是否插入了SIM卡。如果检测到插入SIM卡，会进行SIM卡相关初始化操作。对应上述流程图中的`Initialize SIM Card`。如果没有检测到SIM卡，则不会再继续进行后面的步骤。



## 网络搜索

当SIM卡初始化完成后，UE会搜索周边可能存在的无线信号，并进行小区搜索。我们将这个过程称之为“网络搜索”，对应上述流程图中的`Search Network`。



## 网络注册

UE在搜网过程中，找到合适的小区后，就会在该小区上发起附着流程（Attach procedure），我们将这个过程称之为“网络注册”，对应上述流程图中的`Network Register`。模组只有网络注册成功了，才能进行无线网卡的激活操作。

UE的网络搜索和网络注册过程，都是由协议栈自动进行的，用户无法介入该过程。但是用户可以通过一些方法来确定UE网络注册是否成功。下面介绍两种方式来判断UE网络注册是否成功。

**方法1：使用AT命令查询**

用户可以发送如下AT命令来查询UE的网络注册状态：

```
AT+CREG?
# 如果网络注册成功，返回值如下，即第二个参数值为1或者5；如果是其他值则说明网络注册失败。
+CREG: 0,1
OK
```

步骤1：将终端设备通过USB数据线连接到电脑的USB端口上。

步骤2：在电脑上打开QPYcom工具，选择AT端口，按照图中所示选择配置参数并打开端口。

![AT查询网络注册状态](../../../media/network-comm/nic/cellular/QPYcom打开AT口.png)

步骤3：发送AT查询指令，确认网络注册状态。

![网络注册成功](../../../media/network-comm/nic/cellular/AT查询网络注册成功.png)



**方法2：使用QuecPython的API查询**

用户可以使用QuecPython的`net.getState`方法来查询UE的网络注册状态。有关该方法的详细说明，请参考QuecPython官方网站API参考手册内[获取网络注册信息](../../../../api_reference/iotlib/net.md#获取网络注册信息)部分。

步骤1：将终端设备通过USB数据线连接到电脑的USB端口上。

步骤2：在电脑上打开QPYcom工具，打开QuecPython的REPL命令端口（图示的 NMEA Port），进入到交互模式。

![打开QPYcom](../../../media/network-comm/nic/cellular/QPYcom打开REPL.png)

步骤3：在命令交互行，按照如下方式调用API查询注网状态。

```python
import net
net.getState()
```

![注网状态查询](../../../media/network-comm/nic/cellular/API查询网络注册成功.png)

当`net.getState`的返回值中，图中红框标注的参数值为`1`或者`5`的时候，表示网络注册成功。`1`表示注册的是归属地网络，`5`表示的注册的是漫游网络。

> 如果模组网络注册失败，请参考《网络异常处理》章节中如下部分进行排查：
>
> * [模组网络注册失败](exception-handling.md#模组网络注册失败)



## 激活网卡

当UE网络注册成功后，就会进行蜂窝无线网卡的激活操作。只有蜂窝无线网卡激活成功，UE才能获取到由核心网分配的IP地址等信息，并为系统的TCP/IP协议栈创建一路虚拟网卡。只有网卡创建并激活成功才能进行socket、http、mqtt等网络业务。该步骤对应上述流程图中的`Activate NIC`。

用户可以通过`dataCall`模块的`dataCall.getInfo`方法来确定网卡是否激活成功，有关该方法的详细说明，请参考QuecPython官方网站API参考手册内[获取拨号信息功能](../../../../api_reference/iotlib/dataCall.md#获取拨号信息功能)部分。

查询方法：

```python
dataCall.getInfo(profileID, ipType)
```

第一个参数`profileID`表示哪一路网卡，第二个参数`ipType`表示IP协议类型。由于QuecPython默认开机自动激活第一路蜂窝无线网卡，并且使用的IP协议类型为IPv4，所以查询时，`profileID`取值为1，`ipType`取值为0。

![拨号信息查询](../../../media/network-comm/nic/cellular/API查询拨号结果.png)

当`dataCall.getInfo`方法的返回值中，网卡激活状态为`1`，并且IP不为`0.0.0.0`，说明网卡激活成功。



> 如果模组无线网卡激活失败，请先请参考《网络异常处理》章节中如下部分进行排查：
>
> * [网络异常处理](exception-handling.md#网络异常处理)

