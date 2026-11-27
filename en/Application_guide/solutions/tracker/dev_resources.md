# Development Resources Summary

## Development board documentation

- model：EC200UEUAA QuecPython standard borad
- Purchase link：[click](https://www.waveshare.com/ec200u-au-c4-p01.htm)
- docs:
  - [specification](https://developer.quectel.com/en/wp-content/uploads/sites/2/2024/11/EC200U_A_C4-P01-Series-EVB_Specification_and_User_Guide.pdf)
  - [schematic](https://developer.quectel.com/en/wp-content/uploads/sites/2/2024/11/EC200U_A_C4-P01-Series-EVB_SCH.pdf)
  - [Screen printing](https://developer.quectel.com/en/wp-content/uploads/sites/2/2024/11/EC200U_A_C4-P01-Series-EVB_Silkscreen.pdf)
  - [User guidance](https://developer.quectel.com/doc/quecpython/Dev_board_guide/en/ec200x-evb.html)

## module information

- model：[EC200UEUAA](https://developer.quectel.com/en/modules-cat/ec200u-series) 
- [specification](https://developer.quectel.com/en/wp-content/uploads/sites/2/2024/11/Quectel_EC200U_Series_LTE_Standard_Specification_V1.4.pdf)
- [driver download](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverU_V1.0.19.zip)

## Accessories information

- GNSS antenna+adapter cable
  - model：YLY001CA + YM0003AA
  - buy link：[click](https://e.tb.cn/h.TpAFyEz02BnCHRD?tk=fznae6ITVEX)

## Development tool

- QuecPython debug tool - QPYcom
  - version：V4.1.0
  - download [QPYcom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip)
  - [User Guidance](../../dev-tools/QPYcom/README.md)
- Editor - VSCode
  - download [VSCode](https://code.visualstudio.com/)

## Firmware

- version：EC200UEUAAR03A04M08_OCPU_QPY_BETA0418
- download [Firmware](https://raw.githubusercontent.com/QuecPython/solution-tracker/refs/heads/master/QPY_OCPU_BETA0001_EC200U_EUAA_FW/QPY_OCPU_BETA0001_EC200U_EUAA_FW.pac)

## Source code

- Version: V2.0.0

- The steps for downloading source code from the Github repository are as follows: 

  ```bash
  # 1.pull source code
  git clone https://github.com/QuecPython/solution-tracker.git
  
  # 2.enter root dir
  cd solution-tracker/
  
  # 3.check branch
  git checkout master
  
  # 4.init sub module
  git submodule init
  
  # 5.update sub module
  git submodule update
  
  # 6.enter sub module dir
  cd code/modules/
  
  # 7.check branch
  git checkout master
  ```

- [Download Zip package](https://codeload.github.com/QuecPython/solution-tracker/zip/refs/heads/master)