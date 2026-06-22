# FreeRTOS-IoT-Edge-Node
基于 STM32F407 + FreeRTOS 的物联网边缘节点，实现多传感器采集、MQTT联网、本地缓存与低功耗管理。

## 硬件平台
- **主控**: 星火一号 (STM32F407VGT6, 168MHz, 192KB RAM, 1MB Flash)
- **传感器**: SHT30 (I2C温湿度) / BH1750 (I2C光照)
- **存储**: W25Q128JVSIQ (SPI Flash, 16MB)
- **显示**: SSD1306 0.96" OLED (I2C)
- **联网**: ESP8266-01S (USART2 AT指令)
- **执行器**: 继电器模块 + 有源蜂鸣器

## 软件架构
┌─────────────────────────────────────────┐
│  App Layer: 传感器采集 / MQTT上报 / 报警  │
├─────────────────────────────────────────┤
│  FreeRTOS Kernel: 任务调度 / IPC / 内存  │
├─────────────────────────────────────────┤
│  Driver Layer: I2C / SPI / USART / GPIO │
├─────────────────────────────────────────┤
│  HAL / CMSIS: STM32F4xx Standard Library│
└─────────────────────────────────────────┘
## 任务设计
| 任务 | 优先级 | 周期 | 职责 | 同步机制 |
|------|--------|------|------|---------|
| `vSensorTask` | 3 (High) | 100ms | 采集温湿度+光照 | 写队列 `sensorQueue` |
| `vDisplayTask` | 4 (Mid) | 200ms | OLED刷新显示 | 读队列 |
| `vStorageTask` | 5 (Low) | 事件触发 | Flash本地存储 | 读队列 |
| `vNetworkTask` | 6 (Lower) | 1s | MQTT联网上报 | 读队列 `mqttQueue` |
| `vAlarmTask` | 2 (Highest) | 事件触发 | 异常报警 | 任务通知 |
| `vCLITask` | 7 (Lowest) | 交互式 | 串口命令行 | 无 |

## 核心特性
- **实时性**: 任务通知替代信号量，报警响应延迟 < 1ms
- **可靠性**: 独立看门狗(IWDG) + 断网FatFS缓存 + MQTT补传
- **低功耗**: 空闲任务进入 Sleep 模式，整机功耗降低 60%
- **可维护性**: CLI 终端支持任务状态/内存统计/远程控制

## 快速开始
### 1. 克隆仓库
```bash
git clone https://github.com/saint77777777/FreeRTOS-IoT-Edge-Node.git

### 2. 环境要求
STM32CubeMX 6.10+
Keil MDK-ARM 5.38+ 或 STM32CubeIDE
FreeRTOS V10.4.6 (CMSIS-RTOS V2 wrapper)
### 3. 硬件接线
表格
模块	      接口	    引脚
SHT30	      I2C1	    PB6-SCL, PB7-SDA
BH1750	    I2C2	    PB10-SCL, PB11-SDA
OLED	      I2C1	    与SHT30共总线
W25Q128	    SPI1	    PA5-SCK, PA6-MISO, PA7-MOSI, PA4-CS
ESP8266	    USART     2	PA2-TX, PA3-RX

### 4. 配置WiFi/MQTT
修改 Inc/net_config.h:
c
#define WIFI_SSID     "your_ssid"
#define WIFI_PASSWORD "your_password"
#define MQTT_BROKER   "broker.emqx.io"
#define MQTT_PORT     1883
#define MQTT_TOPIC    "iot/sensor/data"
5. 编译烧录
# STM32CubeIDE
Project -> Build All -> Run -> Debug

# 或 Keil
Rebuild -> Download -> Start Debug
CLI 命令
表格
命令	功能
info	打印任务状态与运行时间统计
heap	查看剩余堆内存与历史最小值
relay on/off	远程控制继电器
reset	系统软复位
help	显示所有命令
性能数据
任务调度: 6任务并发，CPU占用率 < 30%
数据采集: 10Hz采样，延迟 < 500ms
网络上报: MQTT QoS 1，断网缓存 > 1000条
连续运行: 72小时无异常，看门狗零触发
功耗: 全速 120mA → 低功耗 45mA
目录结构
plain
├── Core/
│   ├── Inc/          # 头文件
│   ├── Src/          # 主程序与任务
│   └── Startup/      # 启动文件
├── Drivers/
│   ├── STM32F4xx_HAL_Driver/
│   └── CMSIS/
├── Middlewares/
│   └── FreeRTOS/     # FreeRTOS内核与配置
├── Application/
│   ├── sensor/       # 传感器驱动
│   ├── storage/      # Flash/FatFS
│   ├── network/      # ESP8266/MQTT
│   └── cli/          # 命令行终端
└── README.md

参考
FreeRTOS Official
RT-Thread Documentation
星火一号 BSP
