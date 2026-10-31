<!-- STM32 Agent Skills -->

<div align="center">

# STM32 Agent Skills

**面向传统 STM32CubeMX 工程的 AI 辅助开发技能集**

工程识别 · 外设配置审核 · C 固件开发

![MCU](https://img.shields.io/badge/MCU-STM32F1%20%7C%20F4%20%7C%20G4-03234B?style=flat-square)
![Skills](https://img.shields.io/badge/Skills-3-238636?style=flat-square)
![Version](https://img.shields.io/badge/Version-v1.1-blue?style=flat-square)
![Format](https://img.shields.io/badge/Format-SKILL.md-orange?style=flat-square)

</div>

> 先理解工程，再审核外设，最后实现固件。  
> 本项目面向传统 STM32CubeMX `.ioc` 工程，重点适用于 STM32F1、F4、G4 系列。设计上借鉴了 [STM32C5DevSkills](https://github.com/keysking/STM32C5DevSkills) 的职责拆分和按需加载方式，但不移植其 C5 专用 CubeMX2、`.ioc2`、HAL2 或 `cube mx` 工作流。

## 为什么做这套 Skills

STM32 开发不是简单地生成几段 HAL 代码。工程目录、完整 MCU 型号、CubeMX 配置、HAL 版本、外设触发时序和板级电路相互关联。若只根据系列名称或通用模板回答，很容易把不同芯片的 API、DMA 映射、定时器资源和硬件连接混为一谈。

因此，本项目将任务拆分为三个职责明确、可独立使用也可组合调用的 Skill：

**工程事实 → 外设与时序 → 固件实现**

每个 Skill 以简洁的 `SKILL.md` 作为入口，将详细知识放在 `references/` 中，按任务需要加载，避免一次性读取全部资料。

## 三项核心能力

| Skill | 核心职责 | 典型任务 | 职责边界 |
|---|---|---|---|
| [**stm32-project**](.agents/skills/stm32-project/SKILL.md) | 工程盘点、完整 MCU 型号与封装识别、`.ioc`/生成代码/构建目标交叉核对、HAL/CMSIS 版本识别、ST 官方资料定位 | “先完整阅读这个 G474 工程，确认芯片型号、HAL 版本和工程结构。” | 不负责详细外设参数设计、C 代码实现、CubeMX 修改和构建烧录 |
| [**stm32-peripheral**](.agents/skills/stm32-peripheral/SKILL.md) | CubeMX 配置审核、时钟树、TIM/PWM、ADC/DMA、CAN/UART/SPI/I2C、触发同步和板级信号链分析 | “检查 TIM1 互补 PWM、ADC 触发与 DMA 配置是否满足采样时序。” | 用户手动配置 CubeMX；Skill 负责审核，不直接修改 `.ioc`，不承担完整驱动实现 |
| [**stm32-firmware**](.agents/skills/stm32-firmware/SKILL.md) | C 固件编写与审查、App/Control/Core/main.c 分层、当前工程 HAL API、中断回调、DMA 缓冲区和模块接口 | “按现有工程结构实现 CAN 收发和 Control 层逻辑。” | 不代替 CubeMX 配置和硬件触发时序分析，不猜测 HAL API |

### 推荐工作流

```text
接手工程 / 定位问题
        │
        ▼
  stm32-project
  建立工程事实基线
        │
        ▼
 stm32-peripheral
 审核配置、时钟与时序
        │
        ▼
 stm32-firmware
 实现或修改 C 固件
        │
        ▼
 编译检查 / 板级验证
```

这是一条推荐路径，而不是强制流水线。简单代码任务可以直接使用 `stm32-firmware`；只有当任务跨越多个职责时，才组合调用相关 Skill。

## 快速安装

### 方式一：克隆仓库（推荐）

将仓库中的 `.agents/skills/` 复制到目标 STM32 工程根目录：

```bash
git clone https://github.com/lijie-wansui/stm32-skills.git
```

Windows PowerShell 示例：

```powershell
Copy-Item .\stm32-skills\.agents\skills .\.agents\skills -Recurse -Force
```

Linux / macOS 示例：

```bash
mkdir -p .agents
cp -R stm32-skills/.agents/skills .agents/
```

### 方式二：下载 ZIP

在 GitHub 仓库页面点击 **Code → Download ZIP**，解压后将其中的 `.agents/skills/` 复制到目标工程根目录。

### 用户级安装

如果希望多个工程共用，将三个 Skill 目录复制到用户级目录：

- Windows：`%USERPROFILE%\.agents\skills\`
- Linux / macOS：`$HOME/.agents/skills/`

安装后重新打开 Codex 或开始新会话，并确认三个 Skill 已被发现。若未刷新，可重启 Codex。

## 在 Codex 中使用

Codex 可以根据任务自动选择 Skill，也可以在提示中显式指定：

```text
$stm32-project
先阅读当前工程，确认完整 MCU 型号、HAL 版本和工程结构。

$stm32-peripheral
审核 TIM、ADC、DMA 的配置和触发时序，先不要修改 .ioc。

$stm32-firmware
按照当前工程的代码分层，实现所需功能并核对 HAL API。
```

## 工程代码分层约定

| 目录 / 文件 | 职责 |
|---|---|
| `Core/` | CubeMX 生成的底层初始化、外设句柄和 IRQ 分发 |
| `App/` | 可复用算法与功能模块，例如 SOGI、PID、通用 CAN 应用 |
| `Control/` | 当前项目的控制策略、时序、模块调用、协同逻辑和测试逻辑 |
| `main.c` | 系统初始化、外设初始化调用和控制入口 |

对于 FCS-MPC 等控制项目，通用候选预测和代价计算只有在接口确实可复用时才放入 `App/`；具体项目的轮值时序、总电平协同和节点状态等控制策略放入 `Control/`。不强制建立独立的 `Test/` 目录。

## 设计原则与安全边界

- **以工程证据为准：** 完整 MCU 型号、封装、HAL API、回调签名、DMA 请求和定时器触发关系，必须结合当前工程和对应芯片资料核实。
- **按需加载：** `SKILL.md` 是入口，详细说明放在 `references/`；只读取当前任务需要的章节。
- **最小改动：** 优先沿用已有工程结构、句柄和初始化流程，不为了套用模板而重构正常工作的代码。
- **配置与实现分离：** 默认由开发者手动配置 CubeMX；AI 读取 `.ioc` 和生成代码进行审核。未经明确授权，不修改 `.ioc`、生成初始化代码或重新生成工程。
- **尊重硬件保护：** 不擅自改变 PWM 极性、死区、Break、MOE、门极驱动使能或故障清除逻辑，不绕过硬件保护链。
- **如实报告验证状态：** 配置正确、代码完成、编译通过、已烧录和板级功能验证是不同状态，不得混为一谈。

## 适用范围与非目标

**适用范围**
- 传统 STM32CubeMX `.ioc` 工程。
- 重点面向 STM32F1、F4、G4；具体外设能力和 HAL 行为以完整型号及当前工程版本为准。
- 裸机 C 固件，以及需要分析外设与控制时序的电力电子控制项目。

**当前不覆盖**
- STM32C5 专用 CubeMX2、`.ioc2`、HAL2 和 `cube mx` 命令流程。
- IDE / 工具链安装、独立构建系统、烧录和在线调试专用工作流。
- 自动修改 CubeMX 配置、自动重新生成工程或自动使能功率级。
- 可执行脚本、MCP 服务端或额外运行时依赖。

不将 STM32C5 的 API、命令或外设能力直接套用到 F1/F4/G4。只有在形成稳定、重复的实际需求时，才考虑增加独立的 Build 或 Debug Skill。

## 版本与审核

| 项目 | 内容 |
|---|---|
| 当前版本 | **v1.1** |
| Skill 数量 | 3 |
| 组合包文件数 | 28 |
| 主要适用系列 | STM32F1 / F4 / G4 |
| 静态审核 | [AUDIT_REPORT.md](AUDIT_REPORT.md) |

v1.1 已完成文件结构、YAML frontmatter、Skill 名称与目录一致性、相对引用、职责边界和 ZIP 完整性等静态检查。Codex 实际加载与触发测试、真实工程编译、固件烧录和板级功能验证尚未完成。

## 参考项目

本项目在 Skill 单一职责、入口与参考资料分离、按需加载和安装说明方面，参考了 [keysking/STM32C5DevSkills](https://github.com/keysking/STM32C5DevSkills) 的组织思路。技术内容和适用边界针对传统 STM32CubeMX 工程独立整理，不复制其 C5 专用命令、HAL2 API 或 CubeMX2 工作流。

## 许可

当前仓库尚未指定开源许可证。若计划允许他人复制、修改或再分发，请先补充明确的 LICENSE 文件。
