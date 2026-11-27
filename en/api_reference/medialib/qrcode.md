# qrcode- QR Code Display 

Feature introduction: generate a corresponding QR code according to the input content.

## Display QR Code

### `qrcode.show`

```python
qrcode.show(qrcode_str,magnification,start_x,start_y,Background_color,Foreground_color,Rotate)
```

Displays QR codes to LCD.

**Parameter：**

- `qrcode_str` - String type. QR code content. 
- `magnification` - Integer type. Magnification. Range: 1–6. 
- `start_x` - Integer type. The start x coordinate of the displayed QR code.
- `start_y` - Integer type. The start y coordinate of the displayed QR code.
- `Background_color` - Integer type. Background color. Default value: 0xffff.
- `Foreground_color` - Integer type. Foreground color. Default value: 0x0000.
- `Rotate` - Integer type. Rotation angle. Reference [Constant](qrcode.md#Constant). Default value: 0.

**Return Value:**

`0` - Successful execution

`-1` - QR code generation failed

`-2` - Magnification failed

`-3` - Display failed

## Generation QR Code Data

### `qrcode.getQRData`

```python
qrcode.getQRData(qrcode_str,magnification,Background_color,Foreground_color)
```

Generation QR code and retrieve data.

**Parameter：**

- `qrcode_str` - String type. QR code content. 
- `magnification` - Integer type. Magnification. Range: 1–6. 
- `Background_color` - Integer type. Background color. Default value: 0xffff.
- `Foreground_color` - Integer type. Foreground color. Default value: 0x0000.

**Return Value:**

`-1` - QR code generation failed

Return tuple upon success `(side_length, data)`

- `side_length`: Side length of QR code.
- `data`: Data of QR code.

## Constant

| Constant            | Value | Description                                            |
| ------------------- | ----- | ------------------------------------------------------ |
| `qrcode.ROTATE_0`   | 0     | QR code display without rotation                       |
| `qrcode.ROTATE_90`  | 1     | Rotate the QR code clockwise by 90 degrees to display  |
| `qrcode.ROTATE_180` | 2     | Rotate the QR code clockwise by 180 degrees to display |
| `qrcode.ROTATE_270` | 3     | Rotate the QR code clockwise by 270 degrees to display |