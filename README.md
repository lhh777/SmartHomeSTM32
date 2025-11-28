# SmartHomeSTM32

面向 STM32F103C8T6 开发板的智能家居示例工程，使用贝壳物联（BIGIOT）云平台上传环境数据并接受远程控制指令。工程基于 MDK-ARM 5.27 和 ST 标准外设库 3.50。

## 硬件与引脚

- 主控：STM32F103C8T6
- 传感器：DHT11（PB7）、烟雾传感器（PB4，外部中断检测）、红外对射/避障传感器（见 `User/Infrare.*`）。
- 网络：ESP8266（PA5、PA6 复位/使能；PB10/PB11 串口）。
- 显示：SPI OLED（PB12–PB15）。
- 执行器：蜂鸣器（PC13）、继电器/LED（PB3、PA11、PA12、PA15）、步进电机（`User/stepmotor` 驱动）。
- USB 转串口：PA9/PA10 与 CH340 等模块相连。

更多详细引脚分配见原始 `README.txt`。

## 目录结构与主要模块

```
SmartHomeSTM32/
├── Libraries/            # ST 标准外设库与 CMSIS 头文件
├── Project/RVMDK（uv5）/  # Keil uVision5 工程配置
├── User/                 # 应用层源码与外设驱动
│   ├── main.c            # 主循环：数据采集、云端交互与指令处理
│   ├── all_init.*        # 系统初始化聚合（SysTick、DHT11、OLED、串口、蜂鸣器等）
│   ├── LED/              # 继电器/LED 控制
│   ├── usart3/           # USART3 配置、收发缓冲
│   ├── cJSON/            # 轻量级 JSON 解析库
│   ├── millis/           # 基于 SysTick 的毫秒计时
│   ├── yun/              # 与贝壳物联云的报文封装（checkin/update/say 等）
│   ├── systick/          # SysTick 定时与延时
│   ├── dht11/            # 温湿度采集驱动
│   ├── OLED.*            # 显示接口与字体表
│   ├── math_display.*    # 数值与状态展示逻辑
│   ├── bsp_beep.*        # 蜂鸣器驱动
│   ├── smoke.*           # 烟雾传感器外部中断配置
│   ├── Infrare.*         # 红外传感器采样
│   └── stepmotor/        # 步进电机控制
├── Listing/              # 编译列表文件（由 Keil 生成）
└── Output/               # 目标文件输出目录
```

## 运行时流程

1. `all_init()` 完成外设初始化、显示欢迎页并配置 SysTick、串口、OLED、DHT11、步进电机与烟雾中断。
2. 主循环定期调用 `checkin()`/`check_status()` 维持与云平台的连接心跳；使用 `DHT11_Read_TempAndHumidity()` 读取环境数据，并通过 `update4()` 上传温湿度、烟雾标志及窗帘状态。
3. 串口接收云端的 JSON 指令后，通过 `processMessage()` 解析：支持控制 LED、蜂鸣器、步进电机，并响应欢迎/登录消息。
4. OLED 屏显示实时温湿度与状态信息；外部烟雾中断更新报警状态，可用 `alert()` 发送告警。

## 开发与编译

1. 使用 Keil MDK-ARM 5.27 打开 `Project/RVMDK（uv5）/` 下的工程文件。
2. 确保安装 ST 标准外设库 v3.5，并在工程设置中包含 `Libraries/` 目录。
3. 连接 STM32F103C8T6 开发板与 ESP8266/WiFi 模块，调整 `User/yun/yun.h` 中的设备 ID 与 API Key 后编译烧录。

## 参考

- 代码示例展示了如何在 STM32 上结合 JSON 解析、云端报文封装与多外设协同，可作为物联网入门项目模板。
