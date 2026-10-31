# stm32-project

面向 STM32 传统 CubeMX 工程的识别、盘点、MCU 型号核对和 ST 官方资料管理。重点覆盖 STM32F1/F4/G4；不把 STM32C5 的 CubeMX2、`.ioc2` 和 HAL2 流程套入传统工程。

## 文件结构

~~~text
stm32-project/
├── SKILL.md
├── README.md
└── references/
    ├── project-discovery.md
    ├── mcu-identification.md
    ├── ioc-and-toolchain.md
    ├── documentation-management.md
    ├── hardware-context.md
    └── project-inventory-template.md
~~~

## 安装

将整个 `stm32-project/` 目录放入 Codex 的 `.agents/skills/`（项目级）或 `$HOME/.agents/skills/`（用户级）目录。必须保留 `references/`。

## 三个 Skill 的职责

- `stm32-project`：识别工程、MCU、工具链、配置文件和官方资料。
- `stm32-peripheral`：审核 CubeMX 外设配置，分析时钟、触发、同步和板级信号链。
- `stm32-firmware`：编写和审查 C 固件、HAL 调用、中断回调、DMA 和 App/Control 分层。

本 Skill 默认只读，不修改 `.ioc`、生成代码或 IDE/构建配置；详细外设审核和 C 代码实现分别交给对应 Skill。
