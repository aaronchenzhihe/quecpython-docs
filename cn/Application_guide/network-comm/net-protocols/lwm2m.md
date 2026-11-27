# LwM2M 协议应用指导

本文档介绍如何在 QuecPython 中使用 LwM2M 协议接入设备管理服务器，包括协议原理、服务器参数准备、客户端配置、注册状态判断、心跳更新、注销和常见问题排查。

> 以下型号支持 `lwm2m` 模块：EC200UEU_AB_SANX、BG95M8_SANX、EC200AAU_HA_SANX、EG912NEN_AA。定制固件的功能支持情况以实际固件为准。

## LwM2M 协议概述

LwM2M（Lightweight Machine to Machine）是 OMA SpecWorks 面向物联网设备定义的轻量级设备管理协议。它采用客户端-服务器模型，通常基于 CoAP 和 UDP 通信，可配合 DTLS 实现身份认证与传输加密。

LwM2M 将设备能力组织为对象（Object）、对象实例（Object Instance）和资源（Resource）。服务器通过统一的数据模型识别设备信息、连接状态和固件升级能力，并对设备执行读取、写入、执行和观测等管理操作。

LwM2M 具有以下特点：

- 轻量化：协议开销较小，适用于蜂窝物联网设备和资源受限终端。
- 标准化数据模型：通过对象、实例和资源描述设备能力，便于不同设备接入同一管理平台。
- 设备生命周期管理：支持 Bootstrap、注册、更新和注销流程。
- 安全通信：支持预共享密钥和证书安全模式，也可在调试环境中使用无加密模式。
- 远程管理：可用于设备状态观测、参数配置和固件升级等场景。

## LwM2M 应用场景

LwM2M 常用于需要批量接入、统一监控和远程维护的物联网设备，例如：

1. 智能表计：采集表计状态和计量数据，远程调整上报周期。
2. 资产追踪：观测终端在线状态、信号质量和位置信息。
3. 工业设备：读取运行状态，下发配置并执行远程维护操作。
4. 智慧城市：集中管理路灯、停车检测器和环境传感器。
5. 远程升级：由服务器下发固件地址，设备按配置执行下载和更新。

## LwM2M 通信机制

一个典型的 LwM2M 系统包含以下角色：

- **LwM2M 客户端**：运行在 QuecPython 设备上，使用 Endpoint Name 标识设备，并向服务器发起注册。
- **Bootstrap Server**：可选角色，用于向客户端下发设备管理服务器地址和安全凭据。
- **LwM2M Server**：也称设备管理服务器（DM Server），负责设备注册、状态观测、参数配置和固件升级等管理操作。

### 客户端生命周期

QuecPython 客户端接入 LwM2M 服务器时，主要经历以下流程：

1. 设备完成注网并获得可用的数据网络。
2. 创建并初始化 `lwm2m` 对象。
3. 配置安全参数、服务器参数、Endpoint Name、URC 和 FOTA 策略。
4. 注册回调函数，用于接收连接结果和服务器下发事件。
5. 调用 `register()` 发起注册请求。
6. 通过回调中的 `ready` URC 和 `stat()` 判断最终注册结果。
7. 注册成功后，模块自动执行心跳更新；仅在特殊场景下手动调用 `update()`。
8. 不再使用服务时调用 `unregister()` 注销。

`register()`、`update()` 和 `unregister()` 都是异步操作。接口返回 `0` 仅表示请求已成功提交，不代表服务器已经完成相应操作，最终结果应结合 URC 和 `stat()` 判断。

### 安全模式

`lwm2m.Security` 配置中的 `security_mode` 决定客户端与服务器之间的安全方式。

| `security_mode` | 安全方式 | `config_list` |
| --- | --- | --- |
| `0` | Pre-Shared Key（PSK） | `[serverID, SSID, server_addr, bootstrap, 0, psk_id, psk_key]` |
| `2` | Certificate | `[serverID, SSID, server_addr, bootstrap, 2, ca_cert, client_cert, client_key]` |
| `3` | No Security | `[serverID, SSID, server_addr, bootstrap, 3]` |

生产环境建议使用 PSK 或证书模式。无安全模式不会对通信进行加密或身份认证，仅适合受控网络中的联调。无安全模式应使用 `coap://` 地址；若误用 `coaps://` 地址，模块会按 PSK 模式处理。

### 服务器类型

客户端可以直接连接设备管理服务器，也可以先连接 Bootstrap Server。

| 连接方式 | `serverID` | `SSID` | `bootstrap` | 说明 |
| --- | --- | --- | --- | --- |
| 设备管理服务器 | `1` | `1000` | `0` | 客户端已知服务器和安全参数时直接注册 |
| Bootstrap Server | `0` | `100` | `1` | 先获取设备管理服务器和安全配置 |

服务器地址 `server_addr` 推荐使用完整 URI 格式，如无安全模式 `coap://address:port`、PSK/证书模式 `coaps://address:port`，最长支持 256 个字符；也支持 `address:port` 格式，模块会按安全模式自动补全协议前缀。端口、安全模式、Endpoint Name 和安全凭据必须与服务端配置一致。

## QuecPython 配置项

`config(config_type, config_list)` 用于设置 LwM2M 客户端参数。常用配置项如下：

| 配置类型 | `config_list` | 作用 |
| --- | --- | --- |
| `lwm2m.Reset` | `[]` | 清除已有配置 |
| `lwm2m.Security` | 随安全模式变化 | 配置服务器地址和安全凭据 |
| `lwm2m.Server` | `[serverID, life_time, pmin, pmax, disable_timeout, storing, binding_mode]` | 配置设备管理服务器参数 |
| `lwm2m.Epnamemode` | `[mode]` | 选择 Endpoint Name 的生成方式 |
| `lwm2m.Epname` | `[epname]` | `mode=1` 时设置自定义 Endpoint Name |
| `lwm2m.Urc` | `[URC_onoff]` | 开启或关闭 URC 上报 |
| `lwm2m.Fota` | `[download, update]` | 配置固件自动下载和自动更新策略 |

`lwm2m.Server` 中各参数的含义如下：

- `life_time`：注册生存时间，范围为 1～86400，单位为秒。
- `pmin`：默认最小观测周期，范围为 1～86400，单位为秒。
- `pmax`：默认最大观测周期，范围为 1～86400，单位为秒。
- `disable_timeout`：服务器连接被禁用后的超时时间，范围为 1～86400，单位为秒。
- `storing`：是否保存服务器信息，`0` 表示不保存，`1` 表示保存。
- `binding_mode`：服务器连接方式。当前使用基于 UDP 的 `"U"` 模式。

Endpoint Name 支持以下配置方式：

- `mode=1`：通过 `lwm2m.Epname` 设置自定义 Endpoint Name。
- `mode=3`：在 PSK 模式下使用 `psk_id` 配置 Endpoint Name。

若要通过回调接收 URC，必须将 `lwm2m.Urc` 配置为 `[1]`。若应用需要自行处理固件下载和更新通知，应将 `lwm2m.Fota` 配置为 `[0, 0]`。

## LwM2M 应用

本节以客户端直接连接设备管理服务器为例，介绍从环境检查到注册成功的完整流程。示例采用无安全模式便于展示参数结构，实际项目应根据服务器配置切换为 PSK 或证书模式。

### 准备服务端参数

运行代码前，需要从 LwM2M 平台获取以下信息：

| 参数 | 示例占位值 | 说明 |
| --- | --- | --- |
| 服务器地址 | `coap://lwm2m.example.com:5683` | 替换为实际域名或 IP 地址和端口，无安全模式使用 `coap://` 前缀 |
| Endpoint Name | `quecpython-device-001` | 必须与平台创建设备时的标识一致 |
| 安全模式 | `3` | 示例为无安全模式，生产环境建议使用 `0` 或 `2` |
| PSK ID、PSK Key | 由平台分配 | 仅 PSK 模式需要 |
| CA 证书、客户端证书、客户端私钥 | 由平台提供 | 仅证书模式需要 |

不同平台对 Endpoint Name、PSK Key 编码和证书内容的要求可能不同，应以平台创建设备时生成的参数为准。

### 检查固件和网络

可使用 [QPYcom](../../dev-tools/QPYcom/qpycom-repl.md) 在交互界面导入模块。未抛出异常表示当前固件包含 `lwm2m` 模块。

```python
import lwm2m
```

使用 `checkNet.waitNetworkReady()` 检查蜂窝网络状态：

```python
import checkNet

stage, state = checkNet.waitNetworkReady(30)
print(stage, state)
```

返回 `stage=3`、`state=1` 表示网络已就绪。若网络未就绪，应先检查 SIM 卡、天线、信号和 APN 配置。

### 创建并初始化客户端

创建 `lwm2m` 对象后调用 `init()` 初始化：

```python
import lwm2m

client = lwm2m()
result = client.init()
print("init result:", result)
```

`init()` 成功返回 `0`，失败返回 `-1`。后续配置应在初始化成功后执行。

### 注册回调函数

回调参数为列表，其中 `event[0]` 和 `event[1]` 是事件类型和事件编码，`event[2]` 携带 URC 数据。不同平台上事件类型可能略有差异，连接状态应以 `event[2]` 中的 URC 为准。

```python
def lwm2m_callback(event):
	print("LwM2M event:", event)

	if len(event) < 3:
		return

	urc = str(event[2])
	if '"ready","successfully"' in urc:
		print("LwM2M register success")
	elif '"ready","failed"' in urc:
		print("LwM2M register failed")
	elif '"update","successfully"' in urc:
		print("LwM2M update success")
	elif '"deregister"' in urc:
		print("LwM2M deregister event")

result = client.register_call(lwm2m_callback)
print("register callback result:", result)
```

`register_call()` 成功返回 `0`，失败返回 `-1`。需要删除回调函数时调用：

```python
client.register_call(None)
```

### 配置无安全模式直连

以下配置用于直接连接设备管理服务器。请先替换服务器地址和 Endpoint Name。

```python
server_address = "coap://lwm2m.example.com:5683"
endpoint_name = "quecpython-device-001"

# 清除旧配置。生产环境仅在服务器或安全参数发生变化时执行。
client.config(client.Reset, [])

# [serverID, SSID, server_addr, bootstrap, security_mode]
security_config = [1, 1000, server_address, 0, 3]
client.config(client.Security, security_config)

# [serverID, life_time, pmin, pmax, disable_timeout, storing, binding_mode]
server_config = [1, 300, 1, 60, 86400, 1, "U"]
client.config(client.Server, server_config)

# 使用自定义 Endpoint Name
client.config(client.Epnamemode, [1])
client.config(client.Epname, [endpoint_name])

# 开启 URC；FOTA 下载和更新均由应用控制
client.config(client.Urc, [1])
client.config(client.Fota, [0, 0])
```

`config()` 成功返回 `0`，失败返回 `-1`，当前固件不支持该配置时返回 `-2`。实际项目中应检查每次配置的返回值，配置失败时不要继续注册。

### 配置 PSK 模式

PSK 模式下，安全配置需要增加 `psk_id` 和 `psk_key`。两者必须与服务端保持一致。

```python
server_address = "coaps://lwm2m.example.com:5684"
psk_id = "device-001"
psk_key = "replace-with-server-psk"

security_config = [
	1,
	1000,
	server_address,
	0,
	0,
	psk_id,
	psk_key,
]
client.config(client.Security, security_config)
```

如果平台要求使用 PSK ID 作为 Endpoint Name，可将 Endpoint Name 模式设置为 `3`：

```python
client.config(client.Epnamemode, [3])
```

如果平台使用独立的 Endpoint Name，则使用 `mode=1` 并通过 `lwm2m.Epname` 配置。

### 配置证书模式

证书模式需要配置 CA 证书、客户端证书和客户端私钥：

```python
server_address = "coaps://lwm2m.example.com:5684"
ca_cert = "replace-with-ca-certificate"
client_cert = "replace-with-client-certificate"
client_key = "replace-with-client-private-key"

security_config = [
	1,
	1000,
	server_address,
	0,
	2,
	ca_cert,
	client_cert,
	client_key,
]
client.config(client.Security, security_config)
```

证书字符串的内容和格式应以服务器及当前固件要求为准。连接时可通过 `dtls` URC 判断握手结果。

### 配置 Bootstrap Server

需要通过 Bootstrap Server 获取设备管理服务器配置时，将 `serverID`、`SSID` 和 `bootstrap` 分别设置为 `0`、`100` 和 `1`。下面以 PSK 模式为例：

```python
bootstrap_address = "coaps://bootstrap.example.com:5684"
bootstrap_psk_id = "device-001"
bootstrap_psk_key = "replace-with-bootstrap-psk"

bootstrap_security_config = [
	0,
	100,
	bootstrap_address,
	1,
	0,
	bootstrap_psk_id,
	bootstrap_psk_key,
]
client.config(client.Security, bootstrap_security_config)
```

Bootstrap 开始和完成时，回调中会收到 `bootstraping` 和 `bootstrap` URC。服务端下发配置后，客户端再向设备管理服务器注册。

> 提示：LwM2M 标准约定 Bootstrap Server 专用端口为 CoAP `5685` / CoAPS `5686`（`5683`/`5684` 是设备管理服务器的端口），但不同平台可能自定义端口，实际以平台配置为准。

### 发起注册并查询状态

完成配置和回调注册后，调用 `register()` 发起注册请求：

```python
result = client.register()
print("register request result:", result)
```

执行成功返回 `0`，执行失败返回 `-1`。注册请求成功提交后，可通过 `stat()` 查询状态：

| `stat()` 返回值 | 状态 |
| --- | --- |
| `0` | 未注册 |
| `1` | 注册中 |
| `2` | 已注册 |
| `3` | 注销中 |
| `4` | 注册失败 |

例如，等待最长 60 秒确认注册结果：

```python
import utime

for elapsed in range(60):
	register_state = client.stat()
	print("register state:", register_state)

	if register_state == 2:
		print("LwM2M client is online")
		break
	if register_state == 4:
		print("LwM2M registration failed")
		break

	utime.sleep(1)
```

如果状态长时间停留在 `1`，应结合 `initial`、`dtls` 和 `ready` URC 排查网络、服务器地址及安全凭据。

### 手动发送更新

模块内部包含自动心跳流程，通常不需要主动调用 `update()`。只有在调试或业务明确要求立即更新注册信息时才手动调用：

```python
ssid = 1000

if client.stat() == 2:
	result = client.update(ssid)
	print("update request result:", result)
```

`update()` 成功返回 `0`，失败返回 `-1`，最终结果通过 `update` URC 通知。

### 注销客户端

设备不再使用 LwM2M 服务时，调用 `unregister()` 发起注销：

```python
result = client.unregister()
print("unregister request result:", result)
```

接口返回 `0` 表示注销请求已提交。最终结果通过 `deregister` URC 通知，注销完成后 `stat()` 返回 `0`。

## 完整示例代码

下面的示例封装了网络检查、对象初始化、参数配置、回调注册和注册状态等待流程。示例使用无安全模式，运行前必须替换 `SERVER_ADDRESS` 和 `ENDPOINT_NAME`。

```python
import utime
import checkNet
import lwm2m


SERVER_ADDRESS = "coap://lwm2m.example.com:5683"
ENDPOINT_NAME = "quecpython-device-001"
SERVER_ID = 1
SERVER_SSID = 1000


class Lwm2mClient(object):
	def __init__(self, server_address, endpoint_name):
		self.server_address = server_address
		self.endpoint_name = endpoint_name
		self.client = lwm2m()

	def _require_success(self, operation, result):
		if result != 0:
			raise RuntimeError(
				"{} failed, result={}".format(operation, result)
			)

	def _callback(self, event):
		print("LwM2M event:", event)

		if len(event) < 3:
			return

		urc = str(event[2])
		if '"ready","successfully"' in urc:
			print("LwM2M register success")
		elif '"ready","failed"' in urc:
			print("LwM2M register failed")
		elif '"update","successfully"' in urc:
			print("LwM2M update success")
		elif '"deregister"' in urc:
			print("LwM2M deregister event")
		elif '"fota/pkgurl"' in urc:
			print("LwM2M FOTA package URL received")

	def configure(self):
		self._require_success("init", self.client.init())

		# 清除旧参数，确保本次演示使用完整的新配置。
		self._require_success(
			"reset config",
			self.client.config(self.client.Reset, []),
		)

		security_config = [
			SERVER_ID,
			SERVER_SSID,
			self.server_address,
			0,
			3,
		]
		self._require_success(
			"security config",
			self.client.config(self.client.Security, security_config),
		)

		server_config = [
			SERVER_ID,
			300,
			1,
			60,
			86400,
			1,
			"U",
		]
		self._require_success(
			"server config",
			self.client.config(self.client.Server, server_config),
		)

		self._require_success(
			"endpoint mode config",
			self.client.config(self.client.Epnamemode, [1]),
		)
		self._require_success(
			"endpoint name config",
			self.client.config(self.client.Epname, [self.endpoint_name]),
		)
		self._require_success(
			"URC config",
			self.client.config(self.client.Urc, [1]),
		)
		self._require_success(
			"FOTA config",
			self.client.config(self.client.Fota, [0, 0]),
		)
		self._require_success(
			"callback config",
			self.client.register_call(self._callback),
		)

	def register(self):
		self._require_success("register", self.client.register())

	def wait_registered(self, timeout):
		for elapsed in range(timeout):
			register_state = self.client.stat()
			print("register state:", register_state)

			if register_state == 2:
				return True
			if register_state == 4:
				return False

			utime.sleep(1)

		return False

	def update(self):
		if self.client.stat() != 2:
			print("client is not registered")
			return -1
		return self.client.update(SERVER_SSID)

	def unregister(self):
		return self.client.unregister()


stage, state = checkNet.waitNetworkReady(30)
if stage == 3 and state == 1:
	lwm2m_client = Lwm2mClient(SERVER_ADDRESS, ENDPOINT_NAME)
	lwm2m_client.configure()
	lwm2m_client.register()

	if lwm2m_client.wait_registered(60):
		print("LwM2M client is ready")
	else:
		print("LwM2M client registration timeout or failed")
else:
	print(
		"Network connection failed, stage={}, state={}".format(
			stage,
			state,
		)
	)
```

注册成功后，应用可继续执行自身业务。需要立即发送注册更新时调用 `lwm2m_client.update()`，退出服务时调用 `lwm2m_client.unregister()`。

## 常用 URC

回调收到的 URC 可以反映客户端所处阶段。不同平台可能缺少部分事件，应以关键结果事件为准。

| URC | 说明 | 重点字段 |
| --- | --- | --- |
| `+QLWURC: "pdp active",result,apn` | PDP 激活结果 | `result`、`apn` |
| `+QLWURC: "initial",result,ssid` | 客户端与服务器的初始化结果 | `result`、`ssid` |
| `+QLWURC: "dtls",result,ssid` | DTLS 握手结果 | `result`、`ssid` |
| `+QLWURC: "bootstraping"` | Bootstrap 流程开始 | 无 |
| `+QLWURC: "bootstrap",result,ssid` | Bootstrap 结果 | `result`、`ssid` |
| `+QLWURC: "registering"` | 正在向服务器注册 | 无 |
| `+QLWURC: "ready",result,ssid` | 注册结果 | `result`、`ssid` |
| `+QLWURC: "update",result,ssid` | 注册更新结果 | `result`、`ssid` |
| `+QLWURC: "deregister",ssid,code` 或 `+QLWURC: "deregister",code` | 注销结果 | 不同平台可能省略 `ssid` |
| `+QLWURC: "fota/pkgurl",url` | 服务器下发固件 URL | `url` |

服务器还可能下发 APN、用户名、密码、鉴权方式、注册生存时间和当前时间等变更事件。完整事件列表请参考 [`lwm2m` API 文档](../../../api_reference/networklib/Lwm2m.md)。

## 常见问题

**Q：导入 `lwm2m` 模块失败怎么办？**

A：确认模组型号在支持列表中，并检查当前 QuecPython 固件是否包含该模块。定制固件的模块支持情况可能不同。

**Q：`register()` 返回 `0`，为什么平台仍显示设备离线？**

A：返回 `0` 只表示注册请求提交成功。应等待 `ready` URC，并确认 `stat()` 最终返回 `2`。同时检查 Endpoint Name 是否与平台配置一致。

**Q：收不到任何回调事件怎么办？**

A：确认已经调用 `register_call()`，并在注册前通过 `config(lwm2m.Urc, [1])` 开启 URC 上报。

**Q：一直收到 `ready,failed` 怎么排查？**

A：依次检查网络状态、服务器地址和端口、`serverID`、`SSID`、Endpoint Name、安全模式及凭据。使用 PSK 或证书模式时，还应先查看 `dtls` URC 是否握手成功。

**Q：PSK 模式下 DTLS 握手失败怎么办？**

A：确认 `psk_id`、`psk_key` 及其编码格式与服务端完全一致，并确认连接端口支持 DTLS。若使用 `mode=3`，还要确认平台的 Endpoint Name 与 `psk_id` 一致。

**Q：是否需要定时调用 `update()`？**

A：通常不需要。模块内部有自动心跳流程，手动频繁调用可能造成无效流量。仅在调试或业务明确要求立即更新注册时调用。

**Q：为什么收不到 FOTA 下载地址？**

A：确认已开启 URC，并将 FOTA 下载和更新策略配置为手动模式 `[0, 0]`，然后检查回调中是否出现 `fota/pkgurl` URC。

**Q：`config()` 返回 `-2` 表示什么？**

A：表示当前固件不支持该配置项。请核对模组型号和固件版本，定制功能以实际固件为准。

## 参考链接

- [`lwm2m` API 文档](../../../api_reference/networklib/Lwm2m.md)
- [QuecPython LwM2M 官方示例](https://github.com/QuecPython/examples/tree/main/network-comm/net-protocols/lwm2m/example_lwm2m.py)
