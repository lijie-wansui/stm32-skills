# stm32-firmware

面向传统 STM32CubeMX `.ioc` 工程的裸机 C 固件开发与审查，重点覆盖 STM32F1/F4/G4。

## 文件结构

```text
stm32-firmware/
├── SKILL.md
├── README.md
└── references/
    ├── project-structure.md
    ├── coding-style.md
    ├── hal-api-lookup.md
    ├── peripheral-app-modules.md
    ├── interrupts-dma.md
    └── review-and-verification.md
```

## 安装

将整个 `stm32-firmware/` 目录放入 Codex 的 `.agents/skills/`（项目级）或 `$HOME/.agents/skills/`（用户级）目录。不要只复制 `SKILL.md`，否则按需引用的 `references/` 将不可用。

## 职责边界

- `stm32-project`：工程识别、完整 MCU 型号、HAL/工具链版本和官方资料。
- `stm32-peripheral`：CubeMX 外设配置、时钟、触发/同步和硬件信号链。
- `stm32-firmware`：C 代码、App/Control/Core/main.c 分层、HAL API、中断回调、DMA 数据处理和源码审查。

默认不修改 `.ioc` 或 CubeMX 生成初始化文件。具体 HAL API 以当前工程版本为准，不把 C5 CubeMX2/HAL2 流程套入旧系列。
