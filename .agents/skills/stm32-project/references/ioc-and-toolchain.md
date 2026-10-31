# CubeMX 配置、生成代码与工具链识别

## 1. 三层配置模型

分析 STM32 工程时，区分：

1. **配置意图：** `.ioc` 或其他配置文件表达的设计设置。
2. **实际编译内容：** IDE/CMake/Make 当前纳入构建的源文件、宏、头文件、链接脚本和编译选项。
3. **实际硬件：** 芯片、封装、板级连线、外部时钟和外围器件。

这三层可能不一致。报告时分别描述，不把 `.ioc` 的设置直接当成运行时事实。

## 2. `.ioc` 文件

若工程包含 `.ioc`，可从中梳理：
- MCU/封装选择。
- GPIO 和外设信号分配。
- RCC 和时钟树配置。
- DMA、NVIC 和外设参数。
- Middleware 和软件组件。
- 工程名称、代码生成结构和目标工具链等项目设置。

具体配置项名称和可用内容可能随 CubeMX 版本、MCU 系列和项目类型变化。读取实际文件，不假设每个 `.ioc` 都有完全相同的键。

`.ioc` 是文本配置文件，不等于当前所有 C 文件都与其同步。用户可能手工改过生成代码，或项目未重新生成。

## 3. 与生成代码对照

按需查看：
- `SystemClock_Config()` 或对应系统时钟函数。
- `MX_GPIO_Init()`、`MX_TIMx_Init()`、`MX_ADCx_Init()`、`MX_DMA_Init()` 等外设初始化。
- `HAL_NVIC_SetPriority()`、IRQ Handler 和中断使能。
- GPIO Alternate Function、上下拉、输出速度和初始电平。
- DMA 句柄绑定、方向、宽度和模式。
- 外设启动函数与运行时调用位置。

注意区分：
- 初始化了外设，不等于已经启动外设。
- 生成了 PWM 初始化，不等于 PWM 已经输出。
- 配置了 ADC，不等于 ADC 已按预期触发和采样。
- 使能了 NVIC，不等于对应中断逻辑已经正确实现。
- `.ioc` 中有外设配置，不等于其源文件实际参与构建。

## 4. 判断哪些源文件真正参与构建

检查当前激活的构建目标：
- IDE 工程中的文件组、排除设置、配置条件和构建目标。
- CMake 的 `target_sources()`、源文件 glob 和编译条件。
- Makefile 中的源文件列表、对象文件规则和宏定义。
- 编译数据库（若工程已有 `compile_commands.json`）。
- 实际 include 路径、预处理宏和链接脚本。

磁盘上存在某个 `.c` 文件，不代表编译器一定会编译它。工程可能存在多个 Debug/Release 配置或多个目标。

## 5. 时钟配置的交叉检查

分别记录：
- CubeMX `.ioc` 的时钟树设置。
- 生成代码中的时钟初始化参数。
- 其他运行时改频、低功耗唤醒或时钟重配置代码。
- 原理图/BOM 中实际外部晶振或时钟源。

区分 MCU 最大允许频率与当前工程配置频率。外设内核时钟、总线时钟和定时器时钟按对应系列参考手册计算，不使用跨系列通用假设。

如果 `.ioc` 与 `SystemClock_Config()` 不一致，报告差异，不擅自认定哪一个是最终正确值。

## 6. 工具链与工程文件识别

常见线索：

| 工具/构建系统 | 常见文件 |
| --- | --- |
| STM32CubeIDE / Eclipse CDT | `.project`、`.cproject`、`.settings/` |
| Keil MDK-ARM | `.uvprojx`、`.uvoptx` |
| IAR Embedded Workbench | `.ewp`、`.eww` |
| CMake | `CMakeLists.txt`、`CMakePresets.json` |
| GNU Make | `Makefile`、`.mk` |
| STM32CubeMX | `.ioc` |

这些文件可能同时存在，或有旧工程副本。通过 README、活动目标、源文件列表和构建配置确认当前使用哪套工程。

必要时识别：
- IDE 名称和版本（仅在项目元数据或用户信息有证据时记录）。
- 编译器及版本。
- CPU/FPU/浮点 ABI 选项。
- 预处理宏和 include 路径。
- Debug/Release 等构建配置。
- 链接脚本和启动文件。
- STM32Cube 固件包与 HAL/CMSIS 版本。

不根据电脑上可能安装了某个 IDE 就认定当前工程使用它；不根据 `.ioc` 选择的生成工具链就认定该工具链当前已安装或实际用于构建。

## 7. 只读边界

本 Skill 只识别和报告，不修改：
- `.ioc`。
- `.project`、`.cproject`、`.uvprojx`、`.ewp`。
- `CMakeLists.txt`、Makefile。
- 编译宏、链接脚本和启动文件。
- CubeMX 生成的外设初始化代码。

用户明确要求修改时，应先说明修改对象、影响和可能的代码再生成风险，并取得授权。若只是代码实现问题，优先由 `stm32-firmware` 在既定工程结构内处理。
