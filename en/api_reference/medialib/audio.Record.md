

# Record - Audio Record

Class feature: Recording.

> Currently supported models: EC600N Series, EC800N Series, EC600M-CN(LA/LE), EC800M-CN(LA/LE/GA), EC600U Series, EC200U Series, EG912U, EG915U and EG915N-EUAG.

**Example:**

```python
# -*- coding: UTF-8 -*-
import utime
import audio
from machine import Pin


flag = 1
'''
External speaker plays recording file, select 0
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
        tts.play(1, 0, 2, 'The recording is over, play it')
        aud.play(1, 0, audio.getFilePath(path))
        flag = 0
    elif record_sta == -1:
        print('The recording failure.')
        tts.play(1, 0, 2, 'The recording failure.')
        flag = 0

record = audio.Record()
record.end_callback(record_callback)
record.start('recordfile.wav', 10)

while 1:
    if flag:
        pass
    else:
        break
```

## Constructor

### `audio.Record`

```python
class audio.Record(device)
```

Creates a record object.

> If a parameter is passed, it should be consistent with the parameter of `audio.Audio(device)`.

**Parameter:**

- `device` - Integer type. The output channel. `0` indicates earpiece, `1` indicates headphone and `2` indicates speaker. Default value: `0`. See the table below for the specific channels supported by each module.

**Channels Supported by the Module:**

| Module Series       | Earpiece    | Headphone   | Speaker     |
| ------------------- | ----------- | ----------- | ----------- |
| EC600N Series       | Supported   | Unsupported | Unsupported |
| EC800N Series       | Supported   | Unsupported | Unsupported |
| EC600M-CN(LA/LE)    | Supported   | Unsupported | Unsupported |
| EC800M-CN(LA/LE/GA) | Supported   | Unsupported | Unsupported |
| EG915N              | Supported   | Unsupported | Unsupported |
| EG912N              | Supported   | Unsupported | Unsupported |
| EG912U              | Supported   | Unsupported | Unsupported |
| EC200U Series       | Unsupported | Unsupported | Supported   |
| EC600U Series       | Supported   | Supported   | Supported   |
| EG915U              | Supported   | Supported   | Unsupported |



## Method

### Record.start

```python
audio.start(file_name,seconds)
```

This method starts recording, recording data stored as files.

> When recording files in AMR and WAV formats using this method, default sampling rate 8000. When recording files in MP3 format, default sampling rate 22050.

**Parameter:**

- `file_name` - String type. Name of the recording file.
- `seconds` - Integer type. Recording duration. Unit: second. 

**Return Value:**

`0` - Successful execution

`-1` - Failed to overwrite the file

`-2` - Failed to open the file

`-3` - The file is in use.

`-4` - Channel setting error

`-5` - Timer resource request failure

`-6` - Audio format error

### Record.stop

```python
Record.stop()
```

This method stops recording.

**Return Value:**

`0` - Successful execution; `-1` - Failed execution.

### Record.getFilePath

```python
Record.getFilePath(file_name)
```

This method reads the path of the recording file.

**Parameter:**

- `file_name` - String type. Name of the recording file.

**Return Value:**

Returns the recording file path for successful execution.

`-1` - The target file does not exist. 

`-2`- The file name length is 0.

### Record.getData

```python
Record.getData(file_name, offset, size)
```

This method reads the recording data.

**Parameter:**

- `file_name` - String type. The name of the recording file.
- `offset`  - Integer type. The offset of the data to be read.
- `size` - Integer type. Data size. Unit: byte. The size should be less than 10 KB.

**Return Value:**

Returns the recording data (bytearray type) for successful execution.

The following are return values for failed execution: 

`-1` - Failed to read the data.

`-2` - Failed to open the file.

`-3` - Failed to set the offset.

`-4` - The file is in use.

`-5` - The setting is beyond the file size (offset+size > file_size).

`-6` - The data size to be read is greater than 10 KB.

`-7` - The memory is less than 10 K.

### Record.getSize

```python
Record.getSize(file_name)
```

This method reads the size of the recording file.

**Parameter:**

- `file_name` - String type. The name of the recording file.

**Return Value:**

Returns the file size for successful execution. Unit: byte.

The return values for failed execution are as follows:

`-1` - Failed to get the file size.

`-2` - Failed to open the file.

`-3` - The file is in use.

`-4` - The file name length is 0.

### Record.Delete

```python
Record.Delete(file_name)
```

This method deletes the recording file.

**Parameter:**

- `file_name` - String type. The name of the recording file.

**Return Value:**

`0` - Successful execution

`-1` - The file does not exist.

`-2` - The file is in use.

### Record.exists

```python
Record.exists(file_name)
```

This method determines whether the recording file exists.

**Parameter:**

- `file_name` - String type. The name of  the recording file.

**Return Value:**

`true` - The file exists.

`false` - The file does not exist.

### Record.isBusy

```python
Record.isBusy()
```

This method determines whether recording is in progress.

**Return Value:**

`0` - Not in progress; `1` - In progress.

### Record.end_callback

```python
Record.end_callback(cb)
```

This method sets the callback function for the end of the recording.

**Parameter:**

- `cb` - Function type. Callback function for the end of the recording. The prototype is as follows:

  ```
  cb(audio_msg)
  ```

  **Parameter of the Callback Function**：

  -  `audio_msg` - List type. The recording information. The elements are as follows:
     
     ​    `audio_msg[0]`：`file_path` . String type. The file path.
     
     ​    `audio_msg[1]`：`audio_len` .  Integer type. The recording length. 
     
     ​    `audio_msg[2]`：`audio_state` . Integer type. The recording status. <a href="#label_record_map1">Click here</a> for the description of `audio_state` of the callback function.

**Return Value:**

`0` - Successful execution; `-1` -Failed execution.

<span id="label_record_map1">**Description of `audio_state`：**</span>

| event | Description                                                   |
| ----- | ------------------------------------------------------------- |
| -1    | Failure                                                       |
| 0     | Start recording                                               |
| 1     | Recording stream data updated (valid only for `stream_start`) |
| 3     | Recording ends                                                |

### Record.gain_set

```python
Record.gain(code_gain,dsp_gain)
```

This method sets the recording gain.

> Currently, only the EC200N/EC600N/EC800N/EC600S/EG915N/EC600M/EC800M/EG810M/EG800P series modules support this function.

**Parameter:**

- `code_gain` - Integer type. Uplink codec gain. Range: [0,4]. 
- `dsp_gain` - Integer type. Uplink digital gain. Range: [-36,12].

**Return Value:**

`0` - Successful execution; `-1`- Failed execution.

### Record.gain_get

```python
Record.gain_get()
```

This method obtains the set recording gain.

> Currently, only the EC200N/EC600N/EC800N/EC600S/EG915N/EC600M/EC800M/EG810M/EG800P series modules support this function.

**Parameter:**

No parameter.

**Return Value:**

The return value is a tuple, with the form of:`(code_gain, dsp_gain)`

-  `code_gain` - Integer type. Uplink codec gain. 
- `dsp_gain` - Integer type. Uplink digital gain.

### Record.amrEncDtx_enable

```python
Record.amrEncDtx_enable(on_off)
```

This method controls the DTX feature of the AMR recording.

> Currently, only the EC600N/EC800N series modules support this feature.

**Parameter:**

- `on_off` - Integer type. Enable or disable the DTX feature.  `1`- Enable; `0`- Disable.
- No parameter passed - get the current configuration

**Return Value:**

No parameter passed: return the current configuration.

The parameter is passed: no return value if the parameter is correct, and an exception is thrown if the parameter is incorrect. 

### Record.stream_start

```python
Record.stream_start(format, samplerate, time)
```

This method starts recording audio streams. Note that when recording an audio stream, max cached 8k bytes,read the audio stream timely. Currently, a loop buffer is used. Failure to read the audio stream in time will result in data loss.

**Parameter:**

- `format` - Integer type. Audio format. [See constant](audio.Record.md#Constant). Different module projects can support different audio formats for recording, and the support situation needs to be consulted with Quectel technical personnel.
- `samplerate` - Integer type. Sample rate. Except for MP3 format, it currently supports 8000 sps and 16000 sps, and only supports 22050 sps in MP3 format
- `time` - Integer type. Recording duration. Unit: second. Setting to 0 means continuous recording until stopped using the `Record.stream_stop` interface.

**Return Value:**

`0` - Successful execution. `-1`- Failed execution.

### Record.stream_read

```python
Record.stream_read(read_buf, len)
```

This method reads the recording stream.

**Parameter:**

- `read_buf` - Bytearray type. Recording stream buffer.
- `len` - Integer type. Length to be read.

**Return Value:**

Returns the number of bytes actually read for successful execution or -1 if failed.

### Record.stream_stop

```python
Record.stream_stop()
```

This method stop the recording stream.

**Parameter:**

No parameter.

**Return Value:**

Returns the number of bytes actually read for successful execution or -1 if failed.

## Constant

| Constant         | Value | Description                                                  |
| ---------------- | ----- | ------------------------------------------------------------ |
| `Record.PCM`     | 1     | Record audio data in PCM format.                             |
| `Record.WAV`     | 2     | Record audio data in WAV format, and set the recording time to 0 when recording this format. |
| `Record.MP3`     | 3     | Record audio data in MP3 format.                             |
| `Record.AMRNB`   | 4     | Record audio data in AMR-NB format, with a sampling rate set to 8000 sps when recording this format. |
| `Record.AMRWB`   | 5     | Record audio data in AMR-WB format, with a sampling rate set to 16000 sps when recording this format. |
| `Record.OGGOPUS` | 11    | Record audio data in OggOpus format.                         |