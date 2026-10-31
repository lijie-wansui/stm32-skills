# ST 官方芯片资料查找与管理

## 1. 资料查找原则

先确认 MCU 完整型号和封装，再查资料。不要只搜索“STM32G4 datasheet”或直接使用搜索结果中的第一份 PDF。

优先入口：
1. ST 官方具体产品页。
2. ST 官方对应系列/子系列 Documentation 页面。
3. ST 官方资源页面中的文档编号和版本。
4. 当前工程所用 STM32Cube 固件包内的 Release Notes、HAL/CMSIS 源码和相关说明。
5. ST 官方应用笔记、用户手册和技术文章。

第三方博客、论坛、网盘和示例仓库可作为线索，不应替代 ST 官方芯片规格和勘误文档。

## 2. 常见文档类型与用途

| 文档 | 常见编号 | 主要用途 | 不应替代什么 |
| --- | --- | --- | --- |
| Datasheet | DSxxxx | 完整订货型号、封装、引脚、存储容量、电气特性、绝对最大额定值 | 不替代外设寄存器级 Reference Manual |
| Reference Manual | RMxxxx | 外设结构、寄存器、时钟、复位、DMA、触发和中断机制 | 不替代具体订货型号的封装/电气规格 |
| Errata Sheet | ESxxxx | 特定器件/硅修订版的已知限制和规避措施 | 不可忽略，也不能用另一型号的 Errata 代替 |
| Programming Manual | PMxxxx | Cortex-M 编程、Flash 编程或特定编程机制；以文档实际标题和范围为准 | 不作为外设寄存器的通用 Reference Manual |
| Application Note | ANxxxx | 某项功能、设计方法、算法或硬件实现建议 | 不自动保证适用于所有 STM32 型号 |
| User Manual | UMxxxx | 工具、开发板、软件包或特定系统的使用说明 | 不替代芯片 Datasheet/Reference Manual |
| HAL/LL Driver Manual | UMxxxx 等 | HAL/LL API 说明、参数和软件使用方法 | 不替代当前工程实际使用的 HAL 头文件和源文件 |
| Release Notes | 软件包版本说明 | Cube 固件包版本、变更、已知问题和兼容性 | 不替代芯片硬件资料 |

不同系列的文档编号和组合不完全相同。某些系列可能共用一份 Reference Manual，但 Datasheet 和 Errata 仍可能按子系列、密度或具体器件区分。

## 3. 查找顺序

1. 使用完整型号进入 ST 官方产品页。
2. 打开该产品页的 Documentation/Resources。
3. 找到适用的 Datasheet、Reference Manual 和 Errata Sheet。
4. 对照文档标题中的器件型号范围，确认是否包含目标完整型号。
5. 检查版本号、发布日期和修订记录。
6. 若问题涉及 HAL，进一步查找该系列 HAL/LL 用户手册及当前固件包 Release Notes。
7. 若问题涉及具体外设应用，再查找相关 AN；确认 AN 适用的 MCU 和软件版本。

不要只依据网页搜索摘要确认文档适用范围。需要时打开官方 PDF 首页、适用型号章节和 Revision History。

## 4. 不同问题对应的优先资料

| 要回答的问题 | 优先资料 |
| --- | --- |
| 这个完整型号有多少 Flash/SRAM？ | 对应 Datasheet 的订货信息/存储容量表 |
| 这个封装有没有某个引脚？ | 对应 Datasheet 的 Pinout 和引脚表 |
| GPIO 能否承受某电压？ | Datasheet 的电气特性、绝对最大额定值和工作条件 |
| 某定时器/ADC/CAN 如何工作？ | 对应 Reference Manual 的外设章节 |
| DMA 请求映射和触发源是什么？ | 对应 Reference Manual，必要时核对 Datasheet 和 Cube 配置 |
| 芯片某个外设是否有已知限制？ | 对应 Errata Sheet，并核对硅修订版 |
| HAL 函数参数和回调怎么用？ | 当前工程 HAL 头文件/源文件 + 对应 HAL/LL 用户手册 |
| CubeMX 某项配置如何生成？ | 对应 CubeMX User Manual 和当前版本工程配置 |
| 某个设计方法如何实现？ | 相关 ST Application Note，再核对目标芯片资料 |

## 5. 版本管理

同时区分三种版本：
- **芯片文档版本：** Datasheet/RM/Errata 的文档修订版。
- **软件包版本：** STM32Cube MCU Package、HAL、CMSIS Device 等版本。
- **工程生成工具版本：** STM32CubeMX/CubeIDE 等版本（若可确认）。

最新官方文档不一定等于旧项目当初使用的文档版本。为了理解旧工程，应记录当前工程实际使用的软件包版本；为了评估芯片当前已知问题，应另外查看适用的最新官方 Errata。

文档更新时：
- 不静默覆盖旧版 PDF。
- 保留文档编号和修订版信息。
- 记录发布日期和获取日期。
- 记录新版本适用的器件范围。
- 若新版扩大/缩小适用范围或新增勘误，说明对当前芯片是否有影响。
- 不根据“版本号更大”就断言所有芯片问题都已修复。

## 6. 推荐资料目录

可根据项目实际规模采用以下结构，不强制移动已有文件：

~~~text
Docs/
└── Reference/
    ├── INDEX.md
    ├── MCU/
    │   └── <Exact-MCU-or-device-group>/
    │       ├── Datasheet_DSxxxx_RevX.pdf
    │       ├── ReferenceManual_RMxxxx_RevX.pdf
    │       ├── Errata_ESxxxx_RevX.pdf
    │       ├── ProgrammingManual_PMxxxx_RevX.pdf
    │       └── ApplicationNotes/
    ├── STM32Cube/
    │   ├── CubeMX/
    │   └── FirmwarePackage/
    ├── Board/
    │   ├── Schematic/
    │   ├── PCB/
    │   └── BOM/
    └── ExternalComponents/
~~~

如果已有项目使用 `Doc/`、`Hardware/` 或其他结构，优先沿用，不为符合示例而移动文件。

## 7. INDEX.md 建议字段

每份文档至少记录：
- 文档名称和编号。
- 文档类型。
- 适用 MCU 型号范围。
- Revision/版本号。
- 发布日期。
- 本地文件路径。
- ST 官方来源链接。
- 获取日期。
- 对应硅修订版（适用时）。
- 备注：是否已核对、是否存在更新版、与当前工程的关系。

没有下载到本地的文档只记录官方链接和“未归档”，不要伪造本地路径。

## 8. 官方文档组合示例

以下示例用于说明不同系列的文档编号和适用范围并不相同，不是可直接套用的资料清单：

| 系列/子系列示例 | ST 官方页面列出的资料示例 | 识别时的注意事项 |
| --- | --- | --- |
| STM32G4 | RM0440；ES0430（覆盖 G471/G473/G474/G483/G484 等特定器件组）；具体 Datasheet 按子系列/型号选择 | 不能把 ES0430 自动视为所有 G4 型号的 Errata |
| STM32F407/417 | RM0090；ES0182（覆盖 F405/407 与 F415/417 对应器件组）；具体 Datasheet 按型号选择 | 不要将 RM0090 或 ES0182 套用到所有 STM32F4 |
| STM32F103 | RM0008；Datasheet 和 Errata 按低密度、中密度、高密度、XL-density 等器件类别区分 | F103 同一系列内也不能只按“F103”三个字符选资料 |

具体文档版本和适用型号应在 ST 官方页面再次核对。文档编号示例不能代替对完整订货型号、封装和硅修订版的确认。

## 9. 官方入口示例

以下是官方资料入口示例，不代表每个项目都使用相同文档：

- [STM32G4 Documentation](https://www.st.com/en/microcontrollers-microprocessors/stm32g4-series/documentation.html)
- [STM32F407/417 Documentation](https://www.st.com/en/microcontrollers-microprocessors/stm32f407-417/documentation.html)
- [STM32F103 Documentation](https://www.st.com/en/microcontrollers-microprocessors/stm32f103/documentation.html)
- [STM32CubeMX User Manual UM1718](https://www.st.com/resource/en/user_manual/um1718-stm32cubemx-for-stm32-configuration-and-initialization-c-code-generation-stmicroelectronics.pdf)
- [Getting started with STM32Cube MCU Package](https://wiki.st.com/stm32mcu/wiki/Getting_started_with_STM32Cube_MCU_Package)

ST 的 CubeMX 用户手册说明，CubeMX 用于配置 MCU 引脚、时钟、外设并生成初始化代码；实际外设和固件实现仍需要查阅产品技术文档。因此，不能只看 CubeMX 图形配置或生成代码，就替代 Datasheet、Reference Manual 和对应 HAL 资料。

查找时以目标器件的 ST 官方产品页和文档适用范围为准；不要把示例中的文档编号套用到其他系列。
