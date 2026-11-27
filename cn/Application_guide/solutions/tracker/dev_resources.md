# 开发资源汇总

## 开发板资料

- 型号：[EC200UEUAA QuecPython 标准开发板](https://developer.quectel.com/doc/quecpython/Dev_board_guide/zh/ec200x-evb.html)
- 购买：[点此跳转](https://www.waveshare.com/ec200u-au-c4-p01.htm)
- 开发板文档
  - [规格书](https://developer.quectel.com/wp-content/uploads/2024/09/EC200UA_C4-P01%E7%B3%BB%E5%88%97%E5%BC%80%E5%8F%91%E6%9D%BF%E4%BA%A7%E5%93%81%E8%A7%84%E6%A0%BC%E5%8F%8A%E7%94%A8%E6%88%B7%E6%8C%87%E5%AF%BC.pdf)
  - [原理图](https://developer.quectel.com/wp-content/uploads/2024/09/EC200UA_C4-P01%E7%B3%BB%E5%88%97%E5%BC%80%E5%8F%91%E6%9D%BF%E5%8E%9F%E7%90%86%E5%9B%BE.pdf)
  - [丝印图](https://developer.quectel.com/wp-content/uploads/2024/09/EC200UA_C4-P01%E7%B3%BB%E5%88%97%E5%BC%80%E5%8F%91%E6%9D%BF%E4%B8%9D%E5%8D%B0.pdf)
  - [资源介绍与使用指导](https://developer.quectel.com/doc/quecpython/Dev_board_guide/zh/ec200x-evb.html)

## 模组资料

- 型号：[EC200UEUAA](https://developer.quectel.com/modules-cat/ec200u-series) 
- [规格书](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_EC200U%E7%B3%BB%E5%88%97_LTE_Standard_%E6%A8%A1%E5%9D%97%E4%BA%A7%E5%93%81%E8%A7%84%E6%A0%BC%E4%B9%A6_V1.4.pdf)
- [驱动下载](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverU_V1.0.19.zip)

## 配件资料

- GNSS 天线 + 转接线
  - 型号：YLY001CA + YM0003AA
  - 购买：[点此跳转](https://e.tb.cn/h.TpAFyEz02BnCHRD?tk=fznae6ITVEX)

## 开发工具

- QuecPython 调试工具 - QPYcom
  - 版本：V4.1.0
  - 下载 [QPYcom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip)
  - [使用教程](../../dev-tools/QPYcom/README.md)
- 代码编辑器 - VSCode
  - 下载 [VSCode](https://code.visualstudio.com/)

## 固件包

- 版本：EC200UEUAAR03A04M08_OCPU_QPY_BETA0418
- 下载[固件](https://raw.githubusercontent.com/QuecPython/solution-tracker/refs/heads/master/QPY_OCPU_BETA0001_EC200U_EUAA_FW/QPY_OCPU_BETA0001_EC200U_EUAA_FW.pac)

## 实验源码

- 版本号：V2.0.0

- 从 Github 仓库获取源代码步骤如下：

  ```bash
  # 1.拉取主项目代码
  git clone https://github.com/QuecPython/solution-tracker.git
  
  # 2.进入项目根目录
  cd solution-tracker/
  
  # 3.切换对应的主项目分支
  git checkout master
  
  # 4.子项目初始化
  git submodule init
  
  # 5.子项目代码拉取
  git submodule update
  
  # 6.进入子项目目录
  cd code/modules/
  
  # 7.切换对应的子项目分支
  git checkout master
  ```

- [压缩包下载](https://codeload.github.com/QuecPython/solution-tracker/zip/refs/heads/master)