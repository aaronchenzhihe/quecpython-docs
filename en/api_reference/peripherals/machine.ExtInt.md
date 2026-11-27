# ExtInt - External Interrupt

This class configures I/O pins to interrupt when external events occur.

## Constructor

### `machine.ExtInt`

```python
class machine.ExtInt(GPIOn, mode, pull, callback, [filter_time])
```

**Parameter:**

- `GPIOn` - Integer type. GPIO pin number to be controlled. See [Pin Modules](machine.Pin.md#machine.Pin) for pin definitions (excluding BG95M3). <a href="#BG95M3_label_pinmap">Click here to view</a> pin correspondences of BG95M3 platform.

- `mode` - Integer type. Trigger mode. <br /> `IRQ_RISING` – Trigger rising edge<br />`IRQ_FALLING` – Trigger falling edge <br />`IRQ_RISING_FALLING` – Trigger rising and falling edge

- `pull` - Integer type. Pull selection mode. <br />`PULL_PU` – Pull-up mode <br />`PULL_PD`  – Pull-down mode<br />`PULL_DISABLE` – Floating mode

- `callback` - Integer type. The interrupt triggers the callback function. <br />A tuple with the length of 2 bytes<br />args[0]: GPIO number<br />args[1]: Trigger edge (0: rising edge 1: falling edge)

- `[filter_time]` - Integer type. Filtering time,uint: ms, default value is 0.Allowed input nonzero value is multiple of 10.If the value is not 0, the filtering is performed at the specified time.

<details>
  <summary><span id="BG95M3_label_pinmap"></span>Pin Correspondences of BG95M1,BG95M2,BG95M3,BG95M6,BG95M8,BG95M9 Module<br /></summary>
GPIO2 – Pin5<br />GPIO3 – Pin6<br />GPIO6 – Pin19<br />GPIO7 – Pin22<br />GPIO8 – Pin23<br />GPIO9 – Pin25<br />GPIO11 – Pin27<br />GPIO12 – Pin28<br />GPIO14 – Pin41<br />GPIO16 – Pin65<br/>GPIO17 – Pin66<br />GPIO18 – Pin85<br />GPIO19 – Pin86
</details>



**Example:**

```python
>>> # Creates an ExtInt object 
>>> from machine import ExtInt
>>> def fun(args):
        print('### interrupt  {} ###'.format(args)) # args[0]: GPIO number args[1]: rising edge or falling edge
>>> extint = ExtInt(ExtInt.GPIO1, ExtInt.IRQ_FALLING, ExtInt.PULL_PU, fun)
```

## Methods

### `extint.enable`

```
extint.enable()
```

This method enables interrupts that is to enable external interrupt of an extint object. When the interrupt pin receives the rising edge signal or falling edge signal, it will call a callback function to execute the interrupt.

**Return Value:**

`0` - Successful execution

`-1` - Failed execution

### `extint.disable`

```
extint.disable()
```

This method disables interrupts associated with extint objects.

**Return Value:**

`0` - Successful execution

`-1` - Failed execution

### `extint.line`

```
extint.line()
```

This method reads the line number mapped by the pin.

**Return Value:**

The line number mapped by the pin.

**Example:**

```python
>>> extint = ExtInt(ExtInt.GPIO1, ExtInt.IRQ_FALLING, ExtInt.PULL_PU, fun)
>>> extint.line()
1
```

### `extint.read_count`

```
extint.read_count(is_reset)
```

This method returns number of times an interrupt was triggered. 

**Parameter:**

- `is_reset` - Integer type. Whether to reset the count after reading. `0` indicates that the count is not resetted and `1` indicates a count resetting.

**Return Value:**

The list `[rising_count, falling_count]`<br />`rising_count`: Number of times that the rising edge triggers an interrupt<br />`falling_count`: Number of times that the falling edge triggers an interrupt

### `extint.count_reset`

```
extint.count_reset()
```

This method clears number of times an interrupt is triggered.

**Return Value:**

`0` - Successful execution

`-1` - Failed execution

### `extint.read_level`

```
extint.read_level()
```

This method reads the current pin level.

**Return Value:**

Pin level. 

`0` - low level. 

`1` - high level.

**Example:**

```python
# Creates an ExtInt object
from machine import ExtInt
import utime

def fun(args):
    print('### interrupt  {} ###'.format(args)) # args[0]: GPIO number args[1]: rising edge or falling edge

# Creates an external interrupt object: GPIO1, falling edge trigger, pull-up mode, enable filtering with a filtering time of 50ms
extint = ExtInt(ExtInt.GPIO1, ExtInt.IRQ_FALLING, ExtInt.PULL_PU, fun, filter_time=50)

print('line: {}'.format(extint.line()))         # Reads the line number mapped by the pin
print('level: {}'.format(extint.read_level()))  # Reads the current pin level

extint.enable()        # Enables the interrupt, after which the interrupt can be triggered normally
utime.sleep(5)         # Waits 5s, during which a falling edge on the external pin triggers the interrupt

print('count: {}'.format(extint.read_count(0))) # Reads the interrupt trigger count without resetting it
extint.count_reset()   # Clears the interrupt trigger count
extint.disable()       # Disables the interrupt
```