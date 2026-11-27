# 短信

##  短信概念

短信是指用户通过手机或其他终端设备发送或者接收的文字或者数字信息，是Short Message Service的简称。一般长度最大为140个英文或数字字符、70个中文字符。

短信主要通过蜂窝无线网络进行传输，一般情况下短信不需要互联网连接即可发送，其通过基站将信息传递到SMS中心（即SMSC）。SMSC将收到的短信转发到接收设备附近的基站。最后基站将信息发送到接收设备。从上可以看出短信主要依靠蜂窝无线网络，不依赖互联网。

短信数据流向框架如下图。

![](../media/network-comm/sms/1-1.png)

<center>图1-1</center>

数据流向说明：

* 发送端编辑短信，通过无线信号将消息内容发送到基站。

* 基站收到消息内容经过一系列网元处理将其转发到运营商短信服务中心。

* 运营商短信服务中心经过一系列网元处理将数据转发到接收端附近的基站。

* 接收端附近的基站将短信内容发送到接收端。

## 背景知识

### 短信实现方案演进

2G时代采用比较复杂的CS/PS域架构，SMS(短信)作为上个世纪的产物，一直寄居在2G的CS域架构下。2G/3G时代网络部署的短信网元为SMSC，主要用于短信的存储和转发，在各运营商中数量众多。

到LTE时代，LTE核心网已经没有了短信业务单元，LTE核心网MME单元和2G的MSC服务器单元通过SGs进行建连，进而实现短信功能。

短信功能实现分为三类：

* CS短信，短信通过2G/3G网络的电路域进行收发，模块注册在4G网络下可以通过CSFB形式回落到2G/3G网络实现SMS功能支持。

* SMS Over SGs，也叫NAS短信，需要SIM卡支持2G/3G/4G联合注册。这种短信，从UE到基站是通过LTE的NAS消息传送，到达核心网后，核心网通过SGs接口连接2G/3G网络的MSC网元，由MSC转发到目标设备。

* SMS Over IMS，IMS（VOLTE）短信，SMS消息走LTE网络，需要模组支持VOLTE并且IMS注册成功。

#### 2G/3G 网络架构  

如下图2-1所示，在2G/3G时代，UE设备通过短信收发机将数据最终发送到SMSC，SMSC将收到的信息转发出去，以此完成短信通信。

![](../media/network-comm/sms/2-1.png)

<center>图2-1</center>

#### 4G 短信网络架构

如今4G短信网络框架图，如下图2-2所示，UE将短信数据发送到eNodeB，eNodeB将短信数据发送到SMSC，SMSC再将数据发送到接收端。两者的区别在于2G/3G和4G网元结构不同，4G网元以及网络结构在这方面比2G/3G要分类更多些，其通过新建网元IP-SM-GW实现SMS短信收发功能。

![](../media/network-comm/sms/2-2.png)

<center>图2-2</center>

### 短信相关介绍

目前发送短信消息分为：TEXT模式、块模式和PDU模式。

* TEXT模式即常用的文本模式。
* PDU即数据单元模式。
* 块模式现在很少用。QuecPython目前不支持此种模式。

###  短信编码方式

短信编码方式有：GSM 编码方式，IRA 8bit编码方式，UCS2编码方式。

* GSM 编码7个比特位表示一个短信内容的字符数据。

* UCS2为16个比特位表示一个短信内容的字符数据。
* IRA为8个比特位表示一个短信内容的字符数据。

由于单条短信底层最多支持140个字节，每个字节8bit。由此可知计算单条短信可发送最大字符数据公式如下：

```
	短信内容最大字符数 = 140 * 8 / E(E - 多少个比特位表示一个字符数据)
```

从上述可知：

```
	GSM 编码方式E = 7，短信内容最大字节数 = 140 * 8 /  7 = 160
	IRA 编码方式E = 8，短信内容最大字节数 = 140 * 8 /  8 = 140
	UCS2编码方式E = 16，短信内容最大字节数 = 140 * 8 /  16 = 70
```

由于QuecPython未开放IRA编码方式，目前仅支持GSM，UCS2编码方式，以下只介绍GSM编码短信和UCS2编码短信。

#### GSM短信编码

GSM编码也叫GSM 7-bit编码，利用7个bit来表示一个字符，键盘键入字符为标准的ASCII码，ASCII码是8比特数据。在实际发送数据时，只有GSM编码对应的字符值和ASCII编码对应的字符值是相同的才能正确的接收数据，否则会造成发送和接收的数据不一致的情况。GSM编码对应的数值不是ASCII码对高位简单的舍弃，需要自行进行鉴别。GSM 7 bit编码表参考3GPP TS 03.38。

#### UCS2短信编码

UCS2编码用于发送Unicode字符，一条短信最多可发送70字符。UCS2编码可用于发送中文、英文等各种国家的字符。对于字符数值和UCS2编码对应值有标准的编码规定，可自行查询资料。

### 短信的分类

短信可以分为TEXT短信，PDU短信和块短信。TEXT简单易实现，但在某些场景中并不能满足需求。PDU短信，实现比较复杂，需要转码，但基本能满足绝大部分场景需求 。块短信已经很少用，QuecPython目前不支持此种模式，以下不再介绍此模式。

#### TEXT 短信

QuecPython接口发送TEXT短信不需要将TEXT短信内容再进行拆解组合进行发送，只需直接写入发送内容即可，短信内容超过140个字节，QuecPython模组将自动将其转为长短信分为多条短信进行发送（表3-1标注最大字节数超过140个字节的型号）。

在实际发送短信时，键入的数据均为ASCII码，而ASCII码是8比特位字符数据，GSM编码是7比特位字符数据，GSM编码将ASCII码中的一些控制字符表示成了有实际意义的字符数据。在TEXT模式下，GSM编码数据需要保证7比特位的数据数值和ASCII码表中的数值完全一致，才可通过TEXT方式发送短信数据，否则将导致接收端和发送端数据不一致。

#### PDU 短信

QuecPython接口发送PDU短信不再需要将短信内容转成PDU码，底层已完成字段转码，只需直接写入发送内容即可，短信内容超过140个字节，QuecPython模组将自动将其转为长短信分为多条短信进行发送（表3-1标注支持最大短信条数大于1的情况下）。

PDU(Protocol Data Uint，协议数据单元)模式，需将短信内容转换成PDU数据才能进行发送。QuecPython接口已进行了封装，在实际的使用过程中，只需将短信内容直接写入。当调用接口时会自动将短信内容转换成PDU码进行发送，并自动添加短信头部设置。

### PDU串分类及结构

对于短信功能，PDU类型分为以下几类：

* SMS-DELIVER

  短信中心发送到移动终端设备

* SMS-DELIVER-REPORT

  移动终端设备发送到短信中心

* SMS-SUBMIT

  移动终端设备发送到短信中心

* SMS-SUBMITE-REPORT

  短信中心发送到移动终端设备

* SMS-STATUS-REPORT

  短信中心发送到移动终端设备

* SMS-COMMOAND

  移动终端设备发送到短信中心

* 其他种类

  保留使用

对于移动终端设备，着重介绍SMS-SUBMIT和SMS-DELIVER。其结构如下：

![](../media/network-comm/sms/2-6.png)

<center>图2-3</center>

SMS-SUBMIT和SMS-DELIVER类型PDU串格式公有部分介绍，参考3GPP TS 03.40 第9章。比较重要的DCS涉及到短信压缩、短信类型（共四种类型，规定ME对各类型短信采取的策略），短信编码类型（GSM/UCS2）等设置请参考GSM 03.38 第4章。

以下介绍简单介绍SMS-SUBMIT，SMS-DELIVER两种类型，详细内容参考3GPP TS 03.40 协议文档。

#### SUBMIT-PDU

下图为发送PDU数据格式，可以按照如下格式组装数据，各个字段数值含义均可参考3GPP TS 03.40 协议文档，对于长短信UD包含用户数据包问题，可以参考图2-7中Long SMS （UD）,QuecPython底层已经处理，无需关注。

![](../media/network-comm/sms/2-3.png)

<center>图2-4</center>

##### PDU TYPE

PDU类型对于SMS-SUBMIT，结构图如下：

![](../media/network-comm/sms/2-11.png)

<center>图2-5</center>

 * TP-MTI，2bit
    * 表示短信PDU类型，如SMS-SUBMIT、SMS-DELIVER等。
 * TP-RD，1bit
    * 指示短信中心在处理SMS-SUBMIT类型数据时TP-MR和TP-DA相同的情况下，是接收还是拒绝。
* TP-VPF，2bit
  * 有效期格式
* TP-RP，1bit
  * 标记是否设置回复路径
* TP-UDHI，1bit
  * 标记UD是否包含报文头，对于UD，长短信包含报文头，单条短信只包含用户数据。
* TP-SRR，1bit
    * 标记是否需要状态报告

##### VP

VP表示短信在短信中心的有效时间。

![](../media/network-comm/sms/2-4.png)

<center>图2-6</center>

#### DELIVER-PDU

下图为接收到PDU数据格式，可以按照如下格式解析数据，各个字段数值表示可以参考3GPP TS 03.40协议文档。

![](../media/network-comm/sms/2-5.png)

<center>图2-7</center>

##### PDU TYPE

PDU类型对于SMS-DELIVER，结构图如下:

![](../media/network-comm/sms/2-10.png)

<center>图2-8</center>

* TP-MTI，2bit
  * 同上
* TP-MMS，1bit
  * 是否等待更多的短信消息
* TP-RP，1bit
  * 同上
* TP-UDHI，1bit
  * 同上
* TP-SRI，1bit
  * 设置是否回复短信状态。

##### SCTS

* 短信中心时间戳，其内容格式参考 3GPP TS 03.40协议文档。

## 短信网络协议框架

短信网络协议框架如下图：

![](../media/network-comm/sms/2-7.png)

<center>图2-9</center>

| SM- AL               | Short Message Application Layer                              |
| -------------------- | ------------------------------------------------------------ |
| SM- TL               | Short Message Transfer Layer                                 |
| SM- RL               | Short Message Relay Layer                                    |
| SM- RP  SMR  CM- sub | Short Message Relay Protocol Short Message Relay (entity) Connection Management sublayer |
| SM- CP  SMC  MM- sub | Short Message Control Protocol Short Message Control (entity) Mobility Management sublayer |
| RR- sub              | Radio Resource Management sublayer                           |
| LLC-sub              | Logical Link Control sublayer                                |
| GRR-sub              | GPRS Radio Resource sublayer                                 |



### 短信发送流程

用户编辑短信完成后，通过移动终端设备主动发送短信数据，网络层回复CP-ACK，随后回复RP-ACK，最后移动终端设备回复CP-ACK。

如下图：

![](../media/network-comm/sms/2-9.png)

<center>图2-10</center>



### 短信接收流程

移动设备先从网络侧接收到短信，之后移动设备终端回复CP-ACK报文到网络，当短信解析完成，移动设备终端再次发送RP-ACK，网络层回复CP-ACK。

如下图：

![](../media/network-comm/sms/2-8.png)

<center>图2-11</center>



## 短信支持平台简介

短信存储位置一般分为SM、ME和MT存储。

* SM存储指接收短信后，将短信存储在SIM卡上，不占用设备储存空间。

* ME存储指接收短信后，将短信存储在设备存储空间，不占用SIM卡空间。

* MT存储指存储在SIM卡和设备空间中，如果SIM卡和设备存储空间其中某一个存储空间已满，则将之后的短信存储在另一个未存满的存储空间中。

目前QuecPython模组仅支持SM或ME存储，二者选一。对于短信数量统计以及功能支持，默认储存情况见下表3-1。

<center>表3-1</center>
<table style="text-align: center">
        <tr>
            <td  style="background-color: red" rowspan="2">型号</td>
            <td  style="background-color: red" rowspan="2">具体型号</td>
            <td  style="background-color: red" rowspan="2">默认存储位置</td>
			<td  style="background-color: red" colspan="2">存储最大数</td>
            <td  style="background-color: red" colspan="2">最大短信内容长度（字节）</td>
            <td  style="background-color: red">备注</td>
        </tr>
    	<tr>
            <td style="background-color: red">SM</td>
            <td style="background-color: red">ME</td>
            <td style="background-color: red">GSM</td>
            <td style="background-color: red">UCS2</td>
            <td style="background-color: red"></td>
        </tr>
        <tr >
            <td rowspan="3">EC600N<br>EG915N<br>EG912N</td>
            <td >EC600N</td>
            <td >SM</td>
            <td >50</td>
            <td >180</td>
            <td >960</td>
            <td >420</td>
            <td ></td>
        </tr>
    	<tr>
            <td >EG915N</td>
            <td >SM</td>
            <td >50</td>
            <td >180</td>
            <td >960</td>
            <td >420</td>
            <td ></td>
        </tr>
    	<tr>
            <td >EG912NEN_AA</td>
            <td >SM</td>
            <td >50</td>
            <td >180</td>
            <td >960</td>
            <td >420</td>
            <td ></td>
        </tr>
   		<tr >
            <td rowspan="1">EC200A</td>
            <td >EC200A</td>
            <td >ME</td>
            <td >50</td>
            <td >180</td>
            <td >960</td>
            <td >420</td>
            <td></td>
        </tr>
    <tr>
    	<td rowspan="1">EC600E</td>
            <td >EC600ECN_LC</td>
            <td >ME</td>
            <td >50</td>
            <td >10</td>
            <td >160</td>
            <td >70</td>
            <td></td>
    </tr>
    <tr>
            <td rowspan="6">BG95<br>BG600</td>
	        <td >BG600LM3</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td >BG95M1</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td >BG95M2</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td >BG95M3</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td >BG95M8</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td >BG95M9</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td rowspan="4">EC200U<br>EC600U<br>EG912U<br>EG915U</td>
        	<td >EC200U</td>
            <td >SM</td>
            <td >50</td>
            <td >100</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
        	<td >EC600U</td>
            <td >SM</td>
            <td >50</td>
            <td >100</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
        	<td >EG912U</td>
            <td >SM</td>
            <td >50</td>
            <td >100</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
        	<td >EG915U</td>
            <td >SM</td>
            <td >50</td>
            <td >100</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
        	<td rowspan="2" >EC600G<br>EC800G</td>
        	<td >EC600GCN_LA</td>
            <td >SM</td>
            <td >50</td>
            <td >100</td>
            <td >160</td>
            <td >70</td>
            <td></td>
    </tr>
    <tr>
        	<td >EC800GCN_GA</td>
            <td >SM</td>
            <td >50</td>
            <td >100</td>
            <td >160</td>
            <td >70</td>
            <td></td>
    </tr>
</table>



## 短信的应用

短信在广播，公益信息传播方面有很大的优势，对于一些大型项目是重要的组成部分，以下着重介绍短信的应用。对于esim卡，下载profile一般不会有短信中心码，需要先查询下是否存在短信中心码，若不存在请咨询运营商获取正确的短信中心码进行设置。

### 注册回调函数

此方法用于注册短信回调函数，当用户设备接收到短信时，会通过注册的回调函数来通知当前短信存储位置，以及短信存储位置的索引值。如下示例，如收到一条长短信，分成两条短信到达设备将触发两次回调函数。

注册回调接口如下：

```pyton
sms.setCallback(usrFun)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms

def sms_callback(args):
     print("Get SIM ID :{}".format(args[0]))					
     print("Get Message index :{}".format(args[1]))
     print("Get Message Storage :{}".format(args[2]))

sms.setCallback(sms_callback)

```

### 短信中心码

短信中心码是运营商提供的短信转发中心号码，需从运营商获取。设备先将短信发送到短信服务中心，短信服务中心转发收到的短信到目的设备所在的基站，基站将收到的短信下发到目的设备。短信中心码设置后，将其存入sim卡中，一张sim卡设置一次即可，设置即生效。如需更改可再次设置。

#### 获取短信中心码

短信中心码获取接口如下：

```python
sms.getCenterAddr()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms

center_numer=sms.getCenterAddr()
print("Current center number is :{}".format(center_numer))
```

#### 设置短信中心码

设置短信中心码，一定要保证设置的短信中心码是正确的，否则可能导致短信发送失败。不同国家或地区的短信中心码，请联系当地的运营商进行确认。

短信中心码设置接口如下：

```python
sms.setCenterAddr(addr)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms

# 获取当前短信中心码
center_numer=sms.getCenterAddr()
print("Current center number is :{}".format(center_numer))

#设置短信中心码
center_number="8613010112512"
sms.setCenterAddr(center_number)

```


### 短信存储位置

短信存储位置物理上分为SIM卡、ME。目前仅支持SIM卡存储或ME存储，二者选一，不可同时选择。

#### 获取存储位置

短信存储位置从逻辑上分为读取和删除存储区域、发送和写入存储区域、接收存储区域。需接收存储区域、读取和删除存储区域保持一致，才能对接收到的短信进行读取和删除操作。

获取短信存储位置接口入下：

```python
sms.getSaveLoc()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms

save_info= sms.getSaveLoc()
print("Message read and delete location : {}".format(save_info[0]))
print("Message write and send location  : {}".format(save_info[1]))
print("Message recv and save location   : {}".format(save_info[2]))
```

#### 设置存储位置

短信接收和删除以及读取，均需要从正确的短信存储位置进行操作，需正确设置短信存储位置，才能正常的进行功能应用。

短信存储位置设置接口如下：

```python
sms.setSaveLoc(mem1, mem2, mem3)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms

# 获取当前短信存储位置

save_info=sms.getSaveLoc()

#设置短信读取和删除短信存储位置为SM
save_info[0][0]="SM"
sms.setSaveLoc(save_info[0][0],save_info[1][0],save_info[2][0])

#设置短信接收和读取存储位置为ME
save_info[2][0]="ME"
sms.setSaveLoc(save_info[0][0],save_info[1][0],save_info[2][0])

```


### 发送短信

QuecPython发送短信接口使用便捷，只需输入接收方电话号码、短信内容，选择编码方式，直接调用接口即可。本功能分为发送TEXT短信、PDU短信两个接口 ，接口风格以及参数是相同的。若短信内容超过140字节，QuecPython底层将自动将其转为长短信，分为多条进行发送（表3-1标注支持最大短信条数大于1的情况下），接收端需要通过PDU方式读取，解析出此条长短信各个子短信的顺序号来重新进行合并才能正常显示为一条短信，否则此条长短信在对端只能拆分为多条短信显示。

#### 发送TEXT短信

发送TEXT短信，发送短信内容超过140字节，则转为长短信自动拆分为多条短信发送（表3-1标注支持最大字节数大于140字节的型号）。

接口如下：

```python
sms.sendTextMsg(phoneNumber, msg, codeMode)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms

sms.sendTextMsg('18158626517', 'first test Message', 'GSM')

# TEXT短信示例 2，通过变量发送数据
data="second test message"
sms.sendTextMsg('18158626517', data , 'GSM')

```

#### 发送PDU短信

发送PDU短信，若短信内容长度超过140字节，则转为长短信自动拆分为多条短信发送（表3-1标注支持最大字节数大于140字节的型号）。

接口如下：

```python
sms.sendPduMsg(phoneNumber, msg, codeMode)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms

# PDU短信示例1，直接填入需要发送的数据
sms.sendPduMsg('18158626517', 'send pdu msg by GSM mode.', 'GSM')

# PDU短信示例 2，通过变量发送数据
data="second test pdu message"
sms.sendPduMsg('18158626517', data, 'GSM')
```

### 获取短信数量

在实际使用过程中，需要查询当前短信数量，来判断当前短信存储空间是否已满。此接口查询需要注意当前设置的短信储存空间，返回的短信数量为当前的短信存储空间的短信数量。

获取短信当前接收条数接口如下：

```python
sms.getMsgNums()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms


#获取当前短信存储信息
save_info=sms.getSaveLoc()

#获取SM存储区域短信数量
sms.setSaveLoc("SM","SM","SM")
print("Current save info : {}".format(save_info))

#获取当前短信存储位置的短信数量
message_count=sms.getMsgNums()
print("Current message count :{}".format(message_count))

#获取ME存储区域短信数量
sms.setSaveLoc("SM","SM","ME")
message_count=sms.getMsgNums()
print("Current message count :{}".format(message_count))
```

### 读取短信

当用户收到短信通知，可以通过以下两个接口来读取信息。分为TEXT读取短信和PDU方式读取短信。当收到长短信时，TEXT读取短信会无法区分顺序，需用PDU方式读取短信，根据短信接收PDU数据格式进行解析，合并，在完整应用示例代码中有对此的演示。

#### 读取TEXT短信

以TEXT方式读取短信内容：

```python
sms.searchTextMsg(index)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms

message_count = sms.getMsgNums()
if message_count > 0:
    message_info = sms.searchTextMsg(0)
    print("Get Messge index 0 info :{}".format(message_info))
```

#### 读取PDU短信

以PDU方式读取短信内容：

```python
sms.searchPduMsg(index)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms

message_count = sms.getMsgNums()
if message_count > 0:
    pdu_message = sms.searchPduMsg(0)
    # 读取到PDU短信，需要解码
    pdu_len = sms.getPduLength(pdu_message)
    message_info = sms.decodePdu(pdu_message, pdu_len)
    print("Get Messge index 0 info :{}".format(message_info))
```

### 删除短信

当短信存储空间已满时，可能会导致设备接收不到短信，用户可通过以下接口来删除一些短信，释放短信存储空间。

#### 删除指定位置短信

删除短信接口如下：

```pyhton
sms.deleteMsg(index [, delmode])
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sms

#-------获取当前短信存储信息，确保删除短信位置不会出错
save_info=sms.getSaveLoc()
print("Will delete message from {} filed!".format(save_info[0][0]))

#删除短信索引为0的短信
sms.deleteMsg(0)

#---------指定删除短信存储区域内指定索引的短信数据,以ME存储区域为例-------
# 步骤 1、设置操作短信存储区域为ME
save_info[0][0]="ME"
sms.setSaveLoc(save_info[0][0],save_info[1][0],save_info[2][0])

# 步骤 2、设置需要操作的短信索引值,以索引值0为例
index  = 0
sms.deleteMsg(index)

```

#### 删除全部短信

接口同上，只做如下示例。

```python
# -*- coding: UTF-8 -*-
#示例
import sms

#-------获取当前短信存储信息，确保删除短信位置不会出错
save_info=sms.getSaveLoc()
print("Will delete all message from {} filed!".format(save_info[0][0]))

#删除全部短信
sms.deleteMsg(1,4)

#---------指定删除短信存储区域内所有的短信数据,以ME存储区域为例-------
# 步骤 1、设置操作短信存储区域为ME
save_info[0][0]="ME"
sms.setSaveLoc(save_info[0][0],save_info[1][0],save_info[2][0])

# 步骤 2、删除ME存储区域所有接收到的短信
index  = 0
sms.deleteMsg(1,4)
```

## 应用示例

长短信合并示例。

```python
# codeing=UTF-8
import net
import checkNet
import sms as SMS

#本示例演示将设备内的两条长短信合并为一条短信
#本例演示要求短信索引值0和短信索引值1的短信为同一条长短信的内容
    
class QuecSMS():

    def __init__(self):
        self.__enable_log = False
        self.sms_set_enable_log()
        self.sms = SMS
        self.sms.setCallback(self.__SMS_Call_back)
        self.message=""

    def sms_deal_phone_number(self,args):
        data=enumerate(args)
        data_str1=""
        data_str2=""
        data_str3=""
        for index,value in data:
            if index % 2 == 0:
                data_str1=data_str1+value
            else:
                data_str2=data_str2+value
        for i in range(0,len(data_str1)):
            data_str3=data_str3+data_str2[i]
            data_str3=data_str3+data_str1[i]
        return data_str3
        
    def sms_display(self):
        self.__log("Get Message Loction : {}".format(self.sms.getSaveLoc()))
        sms.setSaveLoc("ME", "ME", "ME")
        self.__log("Get ME Message Numbers : {}".format(self.sms.getMsgNums()))
        sms.setSaveLoc("SM", "SM", "SM")
        self.__log("Get SM  Message Numbers : {}".format(self.sms.getMsgNums()))

    def sms_set_enable_log(self,flag=True):
        self.__enable_log = flag

    def __log(self,args):
        if self.__enable_log:
            print("QuecSMS_LOG: {}".format(args))


    def __SMS_Call_back(self,args):
        self.__log("Get SIM Id          : {}".format(args[0]))
        self.__log("Get Message index   : {}".format(args[1]))
        self.__log("Get Message Storage : {}".format(args[2]))
        
    def sms_delete_user_data_head(self):
        
        self.message=self.message[:58]+self.message[58+2*6:]
        
    def sms_replace_data_index(self,index,data):
        self.message = self.message[:index] + data + self.message[index+len(data):]
    
    def sms_append_sub_message_data(self,data):
        self.message = self.message + data
        
    def sms_decode_pdu_message(self):
        return self.sms.decodePdu(self.message,self.sms.getPduLength(self.message))
    
    def sms_get_message_info(self,index):
        """
        ：param index    ：短信索引值
        ：tyoe  index    ：整形
        
        ：return         ：元组类型
        元组内容：
        （message_ref，message_total_num，message_seq，sub_message_data，pdu_tye，sub_message_len）
        
        ：message_ref        ：短信参考标识，同一个标识表明为同一条短信
        ：message_total_num  ：此条长短信总条数
        ：message_seq        ：此条短信在长短信中的序号
        ：sub_message_data   ：词条短信的内容
        ：pdu_tye            ：PDU类型，bit 6标记是否包含用户报文头，长短信需要
        ：sub_message_len    ：此条短信内容长度
        
        """
        message0=self.sms.searchPduMsg(index)
        self.message = message0
        sca_num  = int(message0[0:2],16) - 1
        data_len = 2
        addr_type= message0[data_len:data_len+1*2]
        data_len = data_len+1*2
        
        sca      = message0[data_len:data_len+sca_num*2]
        # 电话号码高低位需要转换
        sca = self.sms_deal_phone_number(sca)

        if sca_num % 2 == 1:
             print("Get SCA phone {}".format(sca[:-1]))
        else:
             print("Get SCA phone {}".format(sca))

        data_len = data_len+sca_num*2

        pdu_tye   = int(message0[data_len:data_len+1*2],16)
        data_len = data_len+1*2

        oa_num   = int(message0[data_len:data_len+1*2],16)
        data_len = data_len+1*2
        oa_addr_type   = message0[data_len:data_len+1*2]
        data_len = data_len+1*2

        if oa_num % 2 == 1:
             oa_num_t   = int((oa_num + 1)/2)
        else:
             oa_num_t   = int((oa_num )/2)

        oa       = message0[data_len:data_len+oa_num_t*2]
        # 电话号码高低位需要转换
        oa       = self.sms_deal_phone_number(oa)

        if oa_num % 2 == 1:
             print("Get OA phone {}".format(oa[:-1]))
        else:
             print("get OA phone {}".format(oa))

        data_len = data_len+oa_num_t*2
        pid      = message0[data_len:data_len+1*2]
        
        data_len = data_len+1*2
        dcs      = message0[data_len:data_len+1*2]
        data_len = data_len+1*2
        scts     = message0[data_len:data_len+7*2]
        data_len = data_len+7*2
        sub_message_len = int(message0[data_len:data_len+1*2],16)
        data_len = data_len+1*2
        

        #长短信
        if ((pdu_tye & 0x40 ) >> 6) == 1:
                
                sub_message_head = message0[data_len:data_len+6*2]
                data_len = data_len+6*2

                # 长短信用户数据
                sub_message_data = message0[data_len:data_len+sub_message_len*2]
                #长短信,处理长短信用户报文头，获取IEI判断短信类型，参考3GPP TS 03.40  TP-user-data
                sub_head_iei         = int(sub_message_head[2:4],16)
                if sub_head_iei == 00:
                    # 长短信
                    message_ref = int(sub_message_head[6:8],16)
                    message_total_num = int(sub_message_head[8:10],16)
                    message_seq = int(sub_message_head[10:12],16)
                    return message_ref,message_total_num,message_seq,pdu_tye,sub_message_len,sub_message_data
                else:
                    # 其他类型短信
                    return 0,0,1,pdu_tye,sub_message_len,sub_message_data
        else:
             # 短短信
             sub_message_data = message0[data_len:data_len+sub_message_len*2]
             return 0,1,1,pdu_tye,sub_message_len,sub_message_data
             
if __name__ == '__main__':
    sms0 = QuecSMS()
    sms1 = QuecSMS()
    
    # 读取短信索引值0的短信内容
    message0=list(sms0.sms_get_message_info(0))
    
    # 读取短信索引值1的短信内容
    message1=list(sms1.sms_get_message_info(1))
    print("Get Messge 0 {}".format(message0))
    print("Get Messge 1 {}".format(message1))
    if message0[0] != message1[0]:
        print("Message 0 and Message 1 are not the content data of the same long SMS!")
        exit
    # 解码message0的PDU串
    message0_pdu=list(sms0.sms_decode_pdu_message())
    # 解码message1的PDU串
    message1_pdu=list(sms1.sms_decode_pdu_message())
    
    # 合并短信
    #判断messge 0 和 message 1的短信序列号那个在前
    if message0[2] < message1[2]:
        message_merge=message0_pdu[1]+message1_pdu[1]
    else:
        message_merge=message1_pdu[1]+message0_pdu[1]
    print("Get Merger Message is {}".format(message_merge))
    
```



## 短信常见应用场景

### 功能应用场景

* 应用场景1

新办理SIM卡的情况下，不能确定短信中心码是否已经存在SIM卡中。需要先确定是否存在短信中心码。如不存在，请咨询对应运营商获取短信中心码进行设置，否则短信功能不正常。

* 应用场景２

SIM卡变更运营商，需和原本运营商和新变更运营商确定短信中心码，进行重设，否则短信功能不正常。

* 应用场景３

eSIM卡，在下载profile时，可能运营商不预设短信中心码，此时需要和运营商确定短信中心码。

* 应用场景4

eSIM卡，更换profile时，会导致短信中心码被清除掉，需要重设短信中心码。

* 应用场景5

SM和ME短信存储空间均已满，导致短信接收不到，只有清除短信，释放短信存储空间才能正常接收到短信。

* 应用场景6

接收到短信后未读取短信，此时更换了短信读取和删除存储空间，导致短信读取不到，请更换回接收到短信时的短信读取和删除存储空间，进行短信的读取和删除操作。

* 应用场景7

短信存储空间为SM，SM存储空间已满的情况下，想保留短信同时也保证短信可以正常接收，可以更换短信接收存储空间为ME。如果ME存储空间也已满，只有删除短信，才能正常接收到短信。

* 应用场景8

设备在无信号环境中收不到短信，请移动设备到运营商信号覆盖范围内，此时才能进行短信的正常收发操作。

* 应用场景9

设备长期不在运营商服务范围内，之后又回到运营商服务范围内，但是短信在短信服务中心存储时间到期，导致短信中心不再转发短信，进而导致短信丢失。此种场景，尽量避免，目前QuecPython发送短信有效期为最大，至于运营商短信存储有效期最大为多久，根据各个运营商的设定而定，可以和运营商进行确定。

### 业务应用场景

由于科技的不断更新进步，短信发送成本越来越低，其在项目中的应用越来越广。常见应用场景如下：

* 用户登录注册账号

  现在越来越多的服务类应用，通过手机号进行用户注册，用户登录，如各种银行手机应用，电商手机应用，以及各种类型的娱乐类手机应用等等。

* 身份验证

  在实际的应用过程中，手机号往往和个人的实名信息紧密相连，这个特性在实际的业务中很容易做到身份验证，各种应用也利用这个特性进行身份的验证。

* 安全验证

  确认验证，在手机应用找回密码，支付确认等场景中，由于手机短信消息不容易窃取，其往往扮演及其重要的作用，在安全支付业务中其特性占据极大优势，也在此类业务中大放光彩。

* 通知告警，风险预警

  在信用卡，借记卡等金融服务类业务中，金融服务商在风控时发现用户账户存在风险性交易或盗刷类交易时，通过短信可以方便的提醒用户进行风险处理，及时挽回损失。

* 应用推广

  在公司新建项目或业务时，为方便推广项目应用，可以通过短信推广到用户。

* 密码找回

  现在无论个人电脑应用还是手机端应用都极其丰富，相应的应用账户越来越多，用户容易忘记密码，为了客户方便以及简化系统安全设计，利用短信进行密码找回是非常方便的方式。

以上常用场景和人民生活息息相关，也很常见。除此之外，公共服务类也经常采用短信的方式通知市民谨防诈骗，进行各种公共类服务的通知预警。

## 常见问题解答

**问题1：QuecPython支持是否支持长短信？如果支持，最大短信长度是多少？**

QuecPython有很多类型的模组，其中有支持长短信，也有仅支持单条短信的模组型号，具体信息详情请见表3-1。



**问题2：QuecPython发送长短信是如何处理的，是分成多包发送还是直接发送一条？**

QuecPython长短信会分解为一条条发送。



**问题3：QuecPython收到长短信是如何处理？是分开存储为多条短信，还是自动合并成一条短信存储？**

QuecPython收到长短信会分开存储。这种情况下需要用PDU方式读取短信，并从PDU编码中用户数据头部信息中抽取短信参考编号、总条数、当前短信序号来进行自行处理。



**问题4：发送端显示短信已经发送成功，但是模组端未接收到短信可能是什么原因？**

可能有如下几种情况，用户可按照下面几种情况依次排查：

* 确定模组端未接收到短信，还是由于解码失败短信未显示。 

* 短信发送成功，表示模组已经正常通过基站将短信数据发送出去，在传播到接收端的过程中出现了异常，这种情况需要从运营商以及基站和短信服务中心处排查问题。

* 对于解码失败导致短信未显示这种情况，需要查看解码设置是否错误。

* 短信中心服务码设置错误，导致短信接收不到。

  此种情况，需要咨询当地运营商短信中心服务码是否正确，如不正确，请调用接口进行重新设置。

* 接收端短信存储空间是否已满。

  此种情况，需用户查询当前短信存储空间是否已经存满 。

* 接收端是否误设了黑名单。

  此种情况，需用户检查设备是否误将对端设置了黑名单。

* 接收端是否SIM卡已停机，欠费。

  此种情况需和运营商沟通查询设备当前状态。


* 接收端是否长时间不在运营商服务区，超出了短信服务中心缓存短信的时限。

  此种情况，需要用户移动到运营商服务区。

* 接收端是否开通了短信业务。

  此种情况请咨询运营商，是否开通了短信业务。

* 接收端短信接收存储位置和短信读取存储位置不同，导致读取短信时从读取位置读取不到短信。

  此种情况请将短信接收存储位置和短信读取存储位置设置为相同存储位置。

**问题5：模组插入SIM卡后，已经可以正常联网，为什么发送短信失败？**

常见短信发送失败的原因可能有下面这些情况：

* 查看是否在运营商开通了短信业务。如果未开通短信业务，请开通短信业务。
* 查看短信服务中心码是否正确。如果未设置或设置错误短信中心码，请咨询运营商正确的短信中心码进行设置。
* 网络运营商短信服务是否正常，可以向运营商进行确认。
* 网络信号是否稳定，需要日志进行判断，请抓取日志，反馈到QuecPython团队进行分析。
* 是否发短信时网络状态正处于激活态到空闲态，或空闲态到激活态的变化过程中，需要日志进行判断，请抓取日志，反馈到QuecPython团队进行分析。
