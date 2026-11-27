# LwM2M Application Note

This document describes how to use the LwM2M protocol in QuecPython to connect to a device management server, including protocol fundamentals, server parameter preparation, client configuration, registration status checking, heartbeat update, deregistration, and troubleshooting.

> The following models support the `lwm2m` module: EC200UEU_AB_SANX, BG95M8_SANX, EC200AAU_HA_SANX, EG912NEN_AA. Feature support of customized firmware depends on the actual firmware.

## LwM2M Protocol Overview

LwM2M (Lightweight Machine to Machine) is a lightweight device management protocol defined by OMA SpecWorks for IoT devices. It adopts a client-server model, typically communicating over CoAP and UDP, and can work with DTLS for identity authentication and transport encryption.

LwM2M organizes device capabilities into Objects, Object Instances, and Resources. Through a unified data model, the server identifies device information, connection status, and firmware upgrade capabilities, and performs management operations such as read, write, execute, and observe on the device.

LwM2M has the following features:

- Lightweight: Low protocol overhead, suitable for cellular IoT devices and resource-constrained terminals.
- Standardized data model: Describes device capabilities through objects, instances, and resources, facilitating access of different devices to the same management platform.
- Device lifecycle management: Supports Bootstrap, registration, update, and deregistration flows.
- Secure communication: Supports pre-shared key and certificate security modes, and can use the no-encryption mode in debug environments.
- Remote management: Can be used for device status observation, parameter configuration, firmware upgrade, and more.

## LwM2M Application Scenarios

LwM2M is commonly used for IoT devices that need batch access, unified monitoring, and remote maintenance, for example:

1. Smart meters: Collect meter status and metering data, remotely adjust reporting intervals.
2. Asset tracking: Observe terminal online status, signal quality, and location information.
3. Industrial devices: Read operating status, deliver configurations, and perform remote maintenance operations.
4. Smart city: Centrally manage street lights, parking detectors, and environmental sensors.
5. Remote upgrade: The server delivers the firmware address, and the device downloads and updates according to the configuration.

## LwM2M Communication Mechanism

A typical LwM2M system includes the following roles:

- **LwM2M Client**: Runs on the QuecPython device, identified by the Endpoint Name, and initiates registration to the server.
- **Bootstrap Server**: An optional role that delivers the device management server address and security credentials to the client.
- **LwM2M Server**: Also known as the device management server (DM Server), responsible for device registration, status observation, parameter configuration, firmware upgrade, and other management operations.

### Client Lifecycle

When a QuecPython client connects to an LwM2M server, it mainly goes through the following flow:

1. The device completes network attachment and obtains a usable data network.
2. Create and initialize the `lwm2m` object.
3. Configure security parameters, server parameters, Endpoint Name, URC, and FOTA policy.
4. Register the callback function to receive connection results and server-delivered events.
5. Call `register()` to initiate the registration request.
6. Determine the final registration result through the `ready` URC in the callback and `stat()`.
7. After successful registration, the module automatically performs heartbeat updates; only call `update()` manually in special scenarios.
8. Call `unregister()` to deregister when the service is no longer used.

`register()`, `update()`, and `unregister()` are all asynchronous operations. A return value of `0` only means the request has been successfully submitted, not that the server has completed the operation. The final result should be determined by combining URC and `stat()`.

### Security Modes

The `security_mode` in the `lwm2m.Security` configuration determines the security method between the client and the server.

| `security_mode` | Security Method | `config_list` |
| --- | --- | --- |
| `0` | Pre-Shared Key (PSK) | `[serverID, SSID, server_addr, bootstrap, 0, psk_id, psk_key]` |
| `2` | Certificate | `[serverID, SSID, server_addr, bootstrap, 2, ca_cert, client_cert, client_key]` |
| `3` | No Security | `[serverID, SSID, server_addr, bootstrap, 3]` |

It is recommended to use PSK or certificate mode in production environments. The no-security mode does not encrypt communication or provide identity authentication, and is only suitable for joint debugging in controlled networks. For the no-security mode, use a `coap://` address; if a `coaps://` address is used by mistake, the module will process it as PSK mode.

### Server Types

The client can connect directly to the device management server, or connect to the Bootstrap Server first.

| Connection Method | `serverID` | `SSID` | `bootstrap` | Description |
| --- | --- | --- | --- | --- |
| Device management server | `1` | `1000` | `0` | The client registers directly when it already knows the server and security parameters |
| Bootstrap Server | `0` | `100` | `1` | Obtain the device management server and security configuration first |

For the server address `server_addr`, it is recommended to use the full URI format, such as `coap://address:port` for the no-security mode and `coaps://address:port` for PSK/certificate modes, with a maximum length of 256 characters; the `address:port` format is also supported, and the module automatically prepends the protocol scheme according to the security mode. The port, security mode, Endpoint Name, and security credentials must be consistent with the server configuration.

## QuecPython Configuration Items

`config(config_type, config_list)` is used to set LwM2M client parameters. Common configuration items are as follows:

| Configuration Type | `config_list` | Purpose |
| --- | --- | --- |
| `lwm2m.Reset` | `[]` | Clear existing configurations |
| `lwm2m.Security` | Varies with the security mode | Configure server address and security credentials |
| `lwm2m.Server` | `[serverID, life_time, pmin, pmax, disable_timeout, storing, binding_mode]` | Configure device management server parameters |
| `lwm2m.Epnamemode` | `[mode]` | Select how the Endpoint Name is generated |
| `lwm2m.Epname` | `[epname]` | Set a custom Endpoint Name when `mode=1` |
| `lwm2m.Urc` | `[URC_onoff]` | Enable or disable URC reporting |
| `lwm2m.Fota` | `[download, update]` | Configure firmware automatic download and update policy |

The meaning of each parameter in `lwm2m.Server`:

- `life_time`: Registration lifetime, range 1-86400, in seconds.
- `pmin`: Default minimum observation period, range 1-86400, in seconds.
- `pmax`: Default maximum observation period, range 1-86400, in seconds.
- `disable_timeout`: Timeout after the server connection is disabled, range 1-86400, in seconds.
- `storing`: Whether to save server information, `0` for no, `1` for yes.
- `binding_mode`: Server connection method. Currently uses the UDP-based `"U"` mode.

Endpoint Name supports the following configuration methods:

- `mode=1`: Set a custom Endpoint Name via `lwm2m.Epname`.
- `mode=3`: Use `psk_id` as the Endpoint Name in PSK mode.

To receive URC through the callback, `lwm2m.Urc` must be configured as `[1]`. If the application needs to handle firmware download and update notifications by itself, configure `lwm2m.Fota` as `[0, 0]`.

## LwM2M Application

This section takes the client directly connecting to the device management server as an example, introducing the complete flow from environment check to successful registration. The example uses the no-security mode to make the parameter structure clear; in actual projects, switch to PSK or certificate mode according to the server configuration.

### Prepare Server Parameters

Before running the code, obtain the following information from the LwM2M platform:

| Parameter | Example Placeholder | Description |
| --- | --- | --- |
| Server address | `coap://lwm2m.example.com:5683` | Replace with the actual domain name or IP address and port; use the `coap://` prefix for the no-security mode |
| Endpoint Name | `quecpython-device-001` | Must be consistent with the device identifier created on the platform |
| Security mode | `3` | The example uses the no-security mode; `0` or `2` is recommended in production environments |
| PSK ID, PSK Key | Assigned by the platform | Only required for PSK mode |
| CA certificate, client certificate, client private key | Provided by the platform | Only required for certificate mode |

Different platforms may have different requirements for Endpoint Name, PSK Key encoding, and certificate content; refer to the parameters generated when creating the device on the platform.

### Check Firmware and Network

Use [QPYcom](../../dev-tools/QPYcom/qpycom-repl.md) to import modules in the interactive interface. If no exception is raised, the current firmware includes the `lwm2m` module.

```python
import lwm2m
```

Use `checkNet.waitNetworkReady()` to check the cellular network status:

```python
import checkNet

stage, state = checkNet.waitNetworkReady(30)
print(stage, state)
```

A return value of `stage=3` and `state=1` means the network is ready. If the network is not ready, first check the SIM card, antenna, signal, and APN configuration.

### Create and Initialize the Client

Create the `lwm2m` object and call `init()` to initialize:

```python
import lwm2m

client = lwm2m()
result = client.init()
print("init result:", result)
```

`init()` returns `0` on success and `-1` on failure. Subsequent configuration should be performed after initialization succeeds.

### Register the Callback Function

The callback parameter is a list, where `event[0]` and `event[1]` are the event type and event code, and `event[2]` carries the URC data. Event types may vary slightly across platforms; the connection status should be determined by the URC in `event[2]`.

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

`register_call()` returns `0` on success and `-1` on failure. To remove the callback function, call:

```python
client.register_call(None)
```

### Configure Direct Connection in No-Security Mode

The following configuration is used to connect directly to the device management server. Replace the server address and Endpoint Name first.

```python
server_address = "coap://lwm2m.example.com:5683"
endpoint_name = "quecpython-device-001"

# Clear the old configuration. In production, only execute when the server or security parameters change.
client.config(client.Reset, [])

# [serverID, SSID, server_addr, bootstrap, security_mode]
security_config = [1, 1000, server_address, 0, 3]
client.config(client.Security, security_config)

# [serverID, life_time, pmin, pmax, disable_timeout, storing, binding_mode]
server_config = [1, 300, 1, 60, 86400, 1, "U"]
client.config(client.Server, server_config)

# Use a custom Endpoint Name
client.config(client.Epnamemode, [1])
client.config(client.Epname, [endpoint_name])

# Enable URC; FOTA download and update are both controlled by the application
client.config(client.Urc, [1])
client.config(client.Fota, [0, 0])
```

`config()` returns `0` on success, `-1` on failure, and `-2` if the configuration item is not supported by the current firmware. In actual projects, check the return value of each configuration; do not continue registration if a configuration fails.

### Configure PSK Mode

In PSK mode, the security configuration needs to add `psk_id` and `psk_key`. Both must be consistent with the server.

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

If the platform requires the PSK ID to be used as the Endpoint Name, set the Endpoint Name mode to `3`:

```python
client.config(client.Epnamemode, [3])
```

If the platform uses an independent Endpoint Name, use `mode=1` and configure it via `lwm2m.Epname`.

### Configure Certificate Mode

The certificate mode requires configuring the CA certificate, client certificate, and client private key:

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

The content and format of the certificate strings should follow the requirements of the server and the current firmware. During connection, the `dtls` URC can be used to determine the handshake result.

### Configure the Bootstrap Server

When the device management server configuration needs to be obtained through the Bootstrap Server, set `serverID`, `SSID`, and `bootstrap` to `0`, `100`, and `1` respectively. The following uses PSK mode as an example:

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

When Bootstrap starts and completes, the callback receives the `bootstraping` and `bootstrap` URCs. After the server delivers the configuration, the client registers with the device management server.

> Note: The LwM2M standard defines dedicated ports for the Bootstrap Server as CoAP `5685` / CoAPS `5686` (`5683`/`5684` are the ports of the device management server), but different platforms may customize the ports; follow the actual platform configuration.

### Initiate Registration and Query Status

After completing the configuration and callback registration, call `register()` to initiate the registration request:

```python
result = client.register()
print("register request result:", result)
```

It returns `0` on success and `-1` on failure. After the registration request is successfully submitted, the status can be queried via `stat()`:

| `stat()` Return Value | Status |
| --- | --- |
| `0` | Not registered |
| `1` | Registering |
| `2` | Registered |
| `3` | Deregistering |
| `4` | Registration failed |

For example, wait up to 60 seconds to confirm the registration result:

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

If the status remains `1` for a long time, check the network, server address, and security credentials in combination with the `initial`, `dtls`, and `ready` URCs.

### Send an Update Manually

The module includes an automatic heartbeat flow internally, so calling `update()` manually is usually not required. Call it manually only when debugging or when the business explicitly requires an immediate registration update:

```python
ssid = 1000

if client.stat() == 2:
	result = client.update(ssid)
	print("update request result:", result)
```

`update()` returns `0` on success and `-1` on failure; the final result is notified through the `update` URC.

### Deregister the Client

When the device no longer uses the LwM2M service, call `unregister()` to initiate deregistration:

```python
result = client.unregister()
print("unregister request result:", result)
```

A return value of `0` means the deregistration request has been submitted. The final result is notified through the `deregister` URC, and `stat()` returns `0` after deregistration completes.

## Complete Example Code

The following example encapsulates the network check, object initialization, parameter configuration, callback registration, and registration status waiting flow. The example uses the no-security mode; `SERVER_ADDRESS` and `ENDPOINT_NAME` must be replaced before running.

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

		# Clear the old parameters to ensure this demo uses a complete new configuration.
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

After successful registration, the application can continue with its own business. Call `lwm2m_client.update()` when an immediate registration update is required, and call `lwm2m_client.unregister()` when exiting the service.

## Common URCs

The URCs received in the callback reflect the stage the client is in. Different platforms may lack some events; use the key result events as the criterion.

| URC | Description | Key Fields |
| --- | --- | --- |
| `+QLWURC: "pdp active",result,apn` | PDP activation result | `result`, `apn` |
| `+QLWURC: "initial",result,ssid` | Initialization result between the client and the server | `result`, `ssid` |
| `+QLWURC: "dtls",result,ssid` | DTLS handshake result | `result`, `ssid` |
| `+QLWURC: "bootstraping"` | Bootstrap flow starts | None |
| `+QLWURC: "bootstrap",result,ssid` | Bootstrap result | `result`, `ssid` |
| `+QLWURC: "registering"` | Registering with the server | None |
| `+QLWURC: "ready",result,ssid` | Registration result | `result`, `ssid` |
| `+QLWURC: "update",result,ssid` | Registration update result | `result`, `ssid` |
| `+QLWURC: "deregister",ssid,code` or `+QLWURC: "deregister",code` | Deregistration result | Some platforms may omit `ssid` |
| `+QLWURC: "fota/pkgurl",url` | Firmware URL delivered by the server | `url` |

The server may also deliver change events such as APN, user name, password, authentication type, registration lifetime, and current time. For the complete event list, refer to the [`lwm2m` API documentation](../../../api_reference/networklib/Lwm2m.md).

## Frequently Asked Questions

**Q: What should I do if importing the `lwm2m` module fails?**

A: Confirm that the module model is in the supported list, and check whether the current QuecPython firmware contains this module. Module support for customized firmware may vary.

**Q: `register()` returns `0`, but the platform still shows the device as offline?**

A: A return value of `0` only means the registration request was submitted successfully. Wait for the `ready` URC and confirm that `stat()` eventually returns `2`. Also check whether the Endpoint Name is consistent with the platform configuration.

**Q: What should I do if no callback event is received?**

A: Confirm that `register_call()` has been called, and enable URC reporting via `config(lwm2m.Urc, [1])` before registration.

**Q: How to troubleshoot when `ready,failed` is always received?**

A: Check the network status, server address and port, `serverID`, `SSID`, Endpoint Name, security mode, and credentials in order. When using PSK or certificate mode, also check whether the `dtls` URC indicates a successful handshake.

**Q: What if the DTLS handshake fails in PSK mode?**

A: Confirm that `psk_id`, `psk_key`, and their encoding format are exactly consistent with the server, and confirm that the connection port supports DTLS. If `mode=3` is used, also confirm that the platform's Endpoint Name is consistent with `psk_id`.

**Q: Is it necessary to call `update()` periodically?**

A: Usually not. The module has an automatic heartbeat flow internally; frequent manual calls may cause invalid traffic. Call it only when debugging or when the business explicitly requires an immediate registration update.

**Q: Why is the FOTA download address not received?**

A: Confirm that URC is enabled and the FOTA download and update policy is configured as manual mode `[0, 0]`, then check whether the `fota/pkgurl` URC appears in the callback.

**Q: What does `config()` returning `-2` mean?**

A: It means the current firmware does not support this configuration item. Check the module model and firmware version; customized features depend on the actual firmware.

## Reference Links

- [`lwm2m` API documentation](../../../api_reference/networklib/Lwm2m.md)
- [QuecPython LwM2M official example](https://github.com/QuecPython/examples/tree/main/network-comm/net-protocols/lwm2m/example_lwm2m.py)
