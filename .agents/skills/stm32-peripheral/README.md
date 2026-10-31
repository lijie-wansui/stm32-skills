# stm32-peripheral

面向传统 STM32CubeMX `.ioc` 工程的外设机制、配置审核和时序分析，重点覆盖 STM32F1/F4/G4。用户手动配置 CubeMX，Skill 负责检查和解释，不擅自修改 `.ioc`。

## 文件结构

~~~text
stm32-peripheral/
├── SKILL.md
├── README.md
└── references/
    ├── cubemx-review.md
    ├── timer-pwm.md
    ├── adc-dma.md
    ├── can.md
    ├── uart-spi-i2c.md
    ├── interrupts-triggers.md
    ├── power-electronics-safety.md
    └── peripheral-review-template.md
~~~

## 安装

将整个 `stm32-peripheral/` 目录放入 Codex 的 `.agents/skills/`（项目级）或 `$HOME/.agents/skills/`（用户级）目录。必须保留 `references/`。

## 与另外两个 Skill 的职责边界

- `stm32-project`：识别工程、核对 MCU、管理芯片资料和配置基线。
- `stm32-peripheral`：审核外设参数，分析时钟、PWM、ADC/DMA、通信、触发、同步和硬件信号链。
- `stm32-firmware`：实现具体 C 代码、HAL 调用、中断回调、DMA 缓冲区和 App/Control 逻辑。

本 Skill 不替用户操作 CubeMX，不负责完整 C 驱动实现、编译、烧录或实板测试。也不将 C5 的 CubeMX2、`.ioc2` 和 HAL2 专用流程套入传统系列。
