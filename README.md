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
