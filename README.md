# 🌬️ 基于STM32的空气质量监测系统

> 毕业设计项目 | STM32F103ZET6 + FreeRTOS + 多传感器融合 + 物联网云平台

## 📋 项目简介

本项目设计并实现了一套**嵌入式空气质量实时监测系统**，基于STM32F103RCT6微控制器，集成温湿度、PM2.5、CO₂、TVOC、H₂S、NH₃等多种传感器，通过FreeRTOS实现多任务调度，数据经ESP8266 WiFi模块上传至OneNET云平台，同时配备128×160 TFT-LCD实时显示与本地报警功能。

**核心特点：**
- 多传感器数据采集与融合（6种环境参数）
- FreeRTOS实时操作系统多任务架构
- 本地LCD图形化UI + 远程物联网监控
- 动态报警阈值设置（掉电保存）
- 关键代码优化（DMA、滑动平均滤波、DWT精准延时、CRC校验）

---

## 🛠️ 硬件平台

| 组件 | 型号 | 接口/说明 |
|------|------|-----------|
 STM32F103RCT6|ARM Cortex-M3，72MHz|
| **开发板** | STM32F103RCT6最小系统板 | 标准外设接口 |
| **温湿度** | SHT30 | 软件I2C, CRC8校验 |
| **空气质量** | SGP30 | 软件I2C, CO₂+TVOC |
| **PM2.5** | GP2Y1010AU0F | ADC2 + GPIO控制LED |
| **气体传感器** | MQ136(H₂S) + MQ137(NH₃) | ADC1双通道DMA采集 |
| **时钟模块** | DS1302 | GPIO模拟时序, RAM掉电保存 |
| **显示屏** | 1.8寸TFT-LCD 128×160 | SPI1, ST7735驱动 |
| **WiFi模块** | ESP8266 | UART2, MQTT协议 |
| **云平台** | 中移OneNET | MQTT over TCP |

---

## ✨ 功能特性

### 数据采集
- [x] SHT30：温度(-40~125℃)、湿度(0~100%RH)，CRC8查表校验
- [x] SGP30：CO₂(400~60000ppm)、TVOC(0~60000ppb)
- [x] GP2Y1010AU0F：PM2.5浓度，中值滤波抗干扰
- [x] MQ136/MQ137：H₂S、NH₃浓度，滑动平均滤波，ADC1 DMA双通道
- [x] DS1302：实时时钟，支持突发模式高效读写

### 系统功能
- [x] **FreeRTOS多任务**：传感器（1秒周期）、WiFi（10秒上报）、LCD（300毫秒刷新）、系统（20毫秒按键+报警）、心跳
- [x] **图形化UI**：4页界面（主监控/气体详情/系统状态/阈值设置），圆角卡片设计，颜色预警（绿/黄/红）
- [x] **动态阈值**：支持本地修改PM2.5/H₂S/NH₃报警阈值，保存至DS1302 RAM（掉电不丢失）
- [x] **多级报警**：蜂鸣器+报警灯，超标自动触发
- [x] **物联网上传**：MQTT协议连接OneNET，JSON格式数据上报
- [x] **按键交互**：4按键支持页面切换、LED控制、设置编辑（短按/长按区分）

### 代码优化
- [x] LCD批量SPI传输（128字节缓冲），刷新效率提升
- [x] DWT周期计数器实现微秒级精准延时（PM2.5采样时序）
- [x] ADC1 DMA双通道循环采集，CPU零开销
- [x] 滑动平均滤波（MQ传感器，窗口10）
- [x] 中值滤波（PM2.5，5点采样去脉冲干扰）
- [x] SHT30 CRC8查表法（比逐位计算快8倍）

---

## 📁 项目结构
my-project/ ├── Core/ │ ├── Src/ │ │ ├── main.c # 系统初始化，FreeRTOS启动 │ │ ├── freertos.c # 任务创建，互斥锁初始化 │ │ ├── system_data.c # 全局数据结构 + 互斥锁管理 │ │ ├── sensor_task.c # 传感器采集任务（1s周期） │ │ ├── wifi_task.c # WiFi/MQTT连接与数据上报 │ │ ├── display_task.c # LCD显示与UI渲染 │ │ ├── system_task.c # 按键扫描 + 报警逻辑 + 设置编辑 │ │ ├── lcd.c # ST7735驱动（批量SPI优化） │ │ ├── sht30.c # 温湿度驱动（CRC8查表） │ │ ├── sgp30.c # 空气质量驱动（软件I2C） │ │ ├── pm25.c # PM2.5驱动（DWT延时+中值滤波） │ │ ├── mq_senser.c # MQ气体驱动（DMA+滑动平均） │ │ ├── ds1302.c # RTC驱动（突发模式+RAM操作） │ │ ├── esp8266.c # WiFi/MQTT驱动（AT指令+状态机） │ │ ├── key.c # 按键扫描（状态机，消抖） │ │ ├── alarm.c # 蜂鸣器+报警灯驱动 │ │ └── led.c # LED控制 │ └── Inc/ # 头文件 ├── Drivers/ # HAL库驱动 └── README.md

---

## 🔧 环境要求

- **IDE**: Keil MDK 5.38（ARM 编译器 6）
- **固件库**: STM32CubeF1 HAL 库
- **RTOS**: FreeRTOS CMSIS-RTOS v1
- **调试工具**: ST-Link V2，串口助手（115200 波特率）

---

## 🚀 快速开始

1. **克隆仓库**
   ```bash
   git clone https://github.com/zhou-456/my-project.git
┌─────────────────────────────────────────┐
│ FreeRTOS内核 │
├─────────┬─────────┬─────────┬──────────┤
│ 传感器 │ WiFi │ LCD │ 系统 │
│ 任务 │ 任务 │ 任务 │ 任务 │
│ (1秒) │ (10秒) │(300毫秒) │(20毫秒) │
└────┬────┴────┬────┴────┬────┴────┬─────┘
     │         │         │         │
┌────▼────┐ ┌──▼──┐ ┌────▼────┐ ┌──▼──┐
│ SHT30   │ │ESP8266│ │ TFT-LCD │ │按键 │
│ SGP30   │ │MQTT  │ │ 128×160 │ │蜂鸣器│
│ MQ136/7 │ │OneNET│ │  UI     │ │报警灯│
│ GP2Y    │ │      │ │         │ │DS1302│
│ DS1302  │ │      │ │         │ │     │
└─────────┘ └──────┘ └─────────┘ └─────┘
              │
        ┌─────▼─────┐
        │  OneNET   │
        │  云平台   │
        └───────────┘
功能	引脚	模式	
LCD_CS	PA4	SPI1_NSS	
LCD_DC	PB0	GPIO_Output	
LCD_RST	PB1	GPIO_Output	
SHT30_SCL	PB6	GPIO_OD (I2C)	
SHT30_SDA	PB7	GPIO_OD (I2C)	
SGP30_SCL	PB10	GPIO_OD (I2C)	
SGP30_SDA	PB11	GPIO_OD (I2C)	
DS1302_RST PB12 GPIO输出
DS1302_CLK PB13 GPIO输出
DS1302_DAT PB14 GPIO输入/输出
MQ136_ADC PA0 ADC1_IN0（DMA）
MQ137_ADC PA1 ADC1_IN1（DMA）
PM25_LED PA11 GPIO输出
PM25_ADC	PA6	ADC2_IN6	
ESP8266_TX	PA2	UART2_TX	
ESP8266_RX	PA3	UART2_RX	
BEEP	PC13	GPIO_Output	
ALARM_LED	PC14	GPIO_Output	
KEY14	待定	GPIO_Input	
