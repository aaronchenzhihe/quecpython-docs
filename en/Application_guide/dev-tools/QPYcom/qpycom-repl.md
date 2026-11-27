# REPL Debugging

## REPL Debugging

*REPL* stands for "Read Evaluate Print Loop", the name of an interactive prompt. It allows you to interact with the QuecPython module through a serial tool for REPL interaction.

REPL debugging is required during the debugging process of modules and devices, where it is necessary to communicate with the module to execute commands for debugging. 

Through the REPL debugging serial port, you can input any Python code and view real-time results. This coding method helps you understand and experiment with APIs and libraries, and develop the working code to be included in your project in an interactive way, which improves the efficiency of development and debugging.

The Python REPL debugging window is displayed as follows:

![](../../media/dev-tools/qpycom/repl_py.png)

QuecPython's REPL debugging can only be performed through a serial port, so you need to connect the interactive port of the QuecPython module to your computer.

## Enable REPL Debugging

To enable REPL debugging, connect the module with the QuecPython firmware to your computer through the interactive port using a serial tool.

Different serial tools have different steps to connect the serial port. Take QPYcom as an example.

![](../../media/dev-tools/qpycom/repl_qpycom.png)

1. Open QPYcom and select the interactive port of the module on the "REPL" page, then click "**`Open port`**".
2. On the "`REPL`" page, you can see the "`>>>`'' character. By pressing "**Enter**" on your keyboard to start a new line, you will see another "`>>>`" character.
3. You can input Python code on the page for debugging. After entering a command, press "**Enter**" on your keyboard to execute the command on the module and return the result.

## Disable REPL Debugging

When the project enters the mass production stage, it is necessary to disable the REPL debugging feature of the device by restricting access to the device through the serial port or by disabling communication through the interactive serial port for the sake of stable operation and code security. If there is a need for factory testing after production, the REPL debugging can only be re-enabled through a specific method.

There are two solutions:

- Solution 1: **Using UART Interface**

Initialize the **USB CDC PORT** as a serial port through the `UART` interface.

```python
>>> # Create a UART object
>>> from machine import UART
>>> uart1 = UART(UART.UART3, 115200, 8, 0, 1, 0)
```

After initialization, any data input through the **USB CDC PORT** will no longer be interpreted by the REPL. This method can be used to disable the interactive port. The data input through the interactive port will be interpreted as serial port data. For more information, refer to [UART - Duplex Serial Communication Bus](../../../api_reference/peripherals/machine.UART.md).

If there is a need to disable and re-enable the REPL debugging, call *uart.close()* to disable the UART object created above to restore the REPL debugging through the serial port. This method is usually used in the production testing process, where specific data formats need to be parsed in the serial port data interpretation to meet the requirement of re-enabling REPL debugging through a specific method.

- Solution 2: **Using the system Interface**

Use the `system` interface to enable interaction protection. After enabling interaction protection, all external commands and code input through the serial port will be blocked from execution.

```python
system.replSetEnable(flag, **kw_args)
```

Once enabled, you can use the password set during startup to disable interaction protection.

For more information, refer to [system - System Configuration](../../../api_reference/syslib/system.md).