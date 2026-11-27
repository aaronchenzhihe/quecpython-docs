

# class Record - 录音

该类提供录音功能。

> 目前支持型号：EC600N系列、EC800N系列、EC600M-CN(LA、LE)、EC800M-CN(LA、LE、GA)、EG810MCN_GA、EC600U系列、EC200U系列、EG912U、EG915U、EG915N-EUAG。

## 构造函数

### `audio.Record`

```python
class audio.Record(device)
```

创建Record对象。

> 如果传参，请与audio.Audio(device)的参数保持一致。

**参数描述：**

- `device` - 输出通道，int类型，0表示听筒，1表示耳机，2表示喇叭，缺省值为`0`。具体模块所支持通道详见下表。

**模块输出通道对应表**

| 模块型号              | 听筒   | 耳机   | 喇叭   |
| --------------------- | ------ | ------ | ------ |
| EC200N/EC600N/EC800N  | 支持   | 不支持 | 不支持 |
| EC600M-CN(LA、LE)     | 支持   | 不支持 | 不支持 |
| EC800M-CN(LA、LE、GA) | 支持   | 不支持 | 不支持 |
| EG810M                | 支持   | 不支持 | 不支持 |
| EG915N/EG912N         | 支持   | 不支持 | 不支持 |
| EG912U                | 支持   | 不支持 | 不支持 |
| EC200U                | 不支持 | 不支持 | 支持   |
| EC600U                | 支持   | 支持   | 支持   |
| EG915U                | 支持   | 支持   | 不支持 |
| EC600S                | 支持   | 不支持 | 不支持 |
| EC600K/EC800K         | 支持   | 不支持 | 不支持 |
| EC800G-CN(GA、TT、LA) | 支持   | 支持   | 支持   |
| EC600G-CN(LA)         | 支持   | 支持   | 支持   |

## 方法

### `Record.start`

```python
Record.start(file_name,seconds)
```

该方法用于开始录音，录音数据储存为文件。

> 注：该方法录制AMR、WAV格式的文件时，默认采样率为8000，录制MP3格式时，默认采样率为22050。

**参数描述：**

- `file_name` - 录音文件名，string类型。
- `seconds` - 需要录制时长，单位：秒，int类型。

**返回值描述：**

`0`表示成功。

`-1`表示文件覆盖失败。

`-2`表示文件打开失败。

`-3`表示文件正在使用。

`-4`表示通道设置错误。

`-5`表示定时器资源申请失败。

`-6` 表示音频格式检测错误。

### `Record.stop`

```python
Record.stop()
```

该方法用于停止录音。

**返回值描述：**

`0` 表示播放成功，`-1`表示播放失败。

### `Record.getFilePath`

```python
Record.getFilePath(file_name)
```

该方法用于读取录音文件的路径。

**参数描述：**

- `file_name` - 录音文件名，string类型。

**返回值描述：**

成功返回string类型的录音文件路径。

`-1`表示目标文件不存在。

`-2`表示文件名长度为0。

### `Record.getData`

```python
Record.getData(file_name, offset, size)
```

该方法用于读取录音数据。

**参数描述：**

- `file_name` - 录音文件名，string类型。
- `offset`  - 读取数据的偏移量，int类型。
- `size` - 读取大小，单位字节，int类型。注意 ：需小于10K。

**返回值描述：**

成功返回录音数据，bytearray类型。

失败返回值说明如下：

`-1`表示读取数据错误。

`-2`表示文件打开失败。

`-3`表示偏移量设置错误。

`-4`表示文件正在使用。

`-5`表示设置超出文件大小(offset+size > file_size)。

`-6`表示读取size 大于10K。

`-7`表示内存不足10K。

### `Record.getSize`

```python
Record.getSize(file_name)
```

该方法用于读取录音文件大小。

**参数描述：**

- `file_name` - 录音文件名，string类型。

**返回值描述：**

若获取成功，返回文件大小 ，单位字节。 

失败返回值如下： 

`-1`表示获取文件大小失败。 

`-2`表示文件打开失败。 

`-3`表示文件正在使用 。

`-4`表示文件名长度为0。

### `Record.Delete`

```python
Record.Delete(file_name)
```

该方法用于删除录音文件。

**参数描述：**

- `file_name` - 录音文件名，string类型。

**返回值描述：**

`0`表示成功。

`-1`表示文件不存在。

`-2`表示文件正在使用。

### `Record.exists`

```python
Record.exists(file_name)
```

该方法用于判断录音文件是否存在。

**参数描述：**

- `file_name` - 录音文件名，string类型。

**返回值描述：**

`true`表示文件存在。

`false`表示文件不存在。

### `Record.isBusy`

```python
Record.isBusy()
```

该方法用于判断是否正在录音。

**返回值描述：**

`0`表示不在录音，`1`表示正在录音。

### `Record.end_callback`

```python
Record.end_callback(cb)
```

该方法用于设置录音结束回调。

**参数描述：**

`cb` - 录音结束回调函数，function类型，函数原型：

```python
def cb(audio_msg):
    pass
```

**回调函数参数描述**：

  - `audio_msg` - 录音信息，list类型，其中元素如下：

     ​    `audio_msg[0]`：`file_path` ，文件路径，string类型。

     ​    `audio_msg[1]`：`audio_len` ， 录音长度，int类型。

     ​    `audio_msg[2]`：`audio_state` ，录音状态，int类型，<a href="#label_record_map1">点此查看</a>回调函数参数audio_state说明表。

**返回值描述：**

`0` 表示成功，`-1`表示失败。

<span id="label_record_map1">**audio_state说明表：**</span>

| event | 说明               |
| ----- | ------------------ |
| -1    | 发生错误           |
| 0     | 录音开始           |
| 1     | 录音流数据更新（仅`stream_start`录音时有效） |
| 3     | 录音结束           |

### `Record.gain_set`

```python
Record.gain_set(code_gain,dsp_gain)
```

该方法用于设置录音增益。

> 目前仅EC200N/EC600N/EC800N/EC600S/EG915N/EC600M/EC800M/EG810M/EG800P系列模组支持该功能。

**参数描述：**

- `code_gain` - 上行编解码器增益，int型，[0,4]。
- `dsp_gain` - 上行数字增益，int型，`-36~12` 。

**返回值描述：**

`0` 表示成功，`-1`表示失败。

### `Record.gain_get`

```python
Record.gain_get()
```

该方法获取设置的录音增益。

> 目前仅EC200N/EC600N/EC800N/EC600S/EG915N/EC600M/EC800M/EG810M/EG800P系列模组支持该功能。

**参数描述：**

无参数

**返回值描述：**

返回值是一个元组，返回值形式为：`(code_gain, dsp_gain)`

- `code_gain` - 上行编解码器增益。
- `dsp_gain` - 上行数字增益。

### `Record.amrEncDtx_enable`

```python
Record.amrEncDtx_enable(on_off)
```

该方法用于配置amr录音DTX功能开关。

> 目前仅EC600N/EC800N系列模组支持该功能。

**参数描述：**

- `on_off` - 开关，int型，`1`：开启，`0`：关闭 。
- 不传参数 - 获取当前配置

**返回值描述：**

不传参：返回当前配置。

传参：参数正确无返回，参数错误抛异常。

### `Record.stream_start`

```python
Record.stream_start(format, samplerate, time)
```

该方法用于开始录音音频流。注意：录制音频流的同时，应及时读取音频流。目前是采用循环buf，最大缓存8K字节数据，不及时读取，会导致数据丢失。

**参数描述：**

- `format` - 音频格式，int型，[见常量](audio.Record.md#常量)。不同的模组项目所能支持录制的音频格式不同，支持情况需咨询移远技术人员。
- `samplerate` - 采样率，int型，除MP3格式外，目前支持8000 和 16000，MP3格式下只支持22050 。
- `time` - 录音时长，int型，单位 S (秒)，设置为0表示持续录音，直到使用`Record.stream_stop`接口停止。

**返回值描述：**

`0` 表示成功，`-1`表示失败。

### `Record.stream_read`

```python
Record.stream_read(read_buf, len)
```

该方法用于读取录音流。

**参数描述**

- `read_buf` - 录音流buf，bytearray型 。
- `len` - 读取的长度，int类型。

**返回值描述：**

成功返回实际读取的字节数，失败返回整型-1。

### `Record.stream_stop`

```python
Record.stream_stop()
```

该方法用于停止录音流录制。

**参数描述**

无参数。

**返回值描述：**

成功返回实际读取的字节数，失败返回整型-1。

## 常量

| 常量             | 值   | 说明                                                        |
| ---------------- | ---- | ----------------------------------------------------------- |
| `Record.PCM`     | 1    | 录制PCM格式的音频数据                                       |
| `Record.WAV`     | 2    | 录制WAV格式的音频数据，录制该格式时录制时间需设置为`0`      |
| `Record.MP3`     | 3    | 录制MP3格式的音频数据                                       |
| `Record.AMRNB`   | 4    | 录制AMR-NB格式的音频数据，录制该格式时采样率需设置为`8000`  |
| `Record.AMRWB`   | 5    | 录制AMR-WB格式的音频数据，录制该格式时采样率需设置为`16000` |
| `Record.OGGOPUS` | 11   | 录制OggOpus格式的音频数据                                   |

**示例：**

```python
# -*- coding: UTF-8 -*-
#文件录音
import utime
import audio
from machine import Pin


flag = 1
'''
外接喇叭播放录音文件，参数选择0
'''
aud = audio.Audio(0)
tts = audio.TTS(0)

aud.setVolume(11)

def record_callback(args):
    global flag
    print('file_name:{}'.format(args[0]))
    print('file_size:{}'.format(args[1]))
    print('record_sta:{}'.format(args[2]))

    record_sta = args[2]
    if record_sta == 3:
        print('The recording is over, play it')
        tts.play(1, 0, 2, '录音结束,准备播放录音文件')
        aud.play(1, 0, record.getFilePath('recordfile.wav'))
        flag = 0
    elif record_sta == -1:
        print('The recording failure.')
        tts.play(1, 0, 2, '录音失败')
        flag = 0

record = audio.Record()
record.end_callback(record_callback)
record.start('recordfile.wav', 10)

while 1:
    if flag:
        utime.sleep_ms(200)
    else:
        break
```

```python
# -*- coding: UTF-8 -*-
#录音流
import audio
'''
外接喇叭播放录音文件，参数选择0
'''
rec = audio.Record(0)
aud = audio.Audio(0)

record_time = 10
buf = bytearray(0) #空buff用来缓存音频
aud_type = rec.AMRNB

aud.setVolume(10)

def stream_rec_cb(para):
    global buf
    print("para:", para)
    if(para[0] == 'stream'):
        if(para[2] == 1):
            read_buf = bytearray(para[1])
            rec.stream_read(read_buf,para[1])
            buf += read_buf
            del read_buf
        elif (para[2] == 3):
            pass
            aud.stopPlayStream()
            aud.playStream(aud_type, buf)

rec.end_callback(stream_rec_cb)
rec.stream_start(aud_type, 8000, record_time)
```

