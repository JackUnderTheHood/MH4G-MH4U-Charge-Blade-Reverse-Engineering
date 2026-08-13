# MH4G / MH4U 盾斧逆向工程与快速变形斩 GP

本仓库整理 **MH4G / MH4U** 盾斧动作系统的逆向资料、研究方法与快速变形斩 GP ExeFS 补丁。机制发现与首个完整五-hook 实现以 **MH4G v1.2 汉化版** 为实际研究基线：MH4G 的具体地址、机器码和机制证据来自该版本；MH4U USA／EUR 的地址、重定位 overlay、运行时证据、Title ID 与正式发布物则来自各自 v1.1 版本上的独立工作。

研究中确认的分层逻辑、实验方法、motion／资源生命周期模型和五-hook 移植流程同样可以应用到 **MH4U**。但这表示“研究方法与实现结构可以迁移”，不表示 MH4G 的地址或 `code.ips` 可以直接用于 MH4U；每个 MH4U 区域和版本仍需独立映射地址、重定位 overlay 并完成动态与 ExeFS 验收。

本次核心机制研究已于 **2026-08-11** 正式结案。当前已有三个同时覆盖“不推动左摇杆”和“推动左摇杆”两种快速变形斩分支的 ExeFS v3 正式版本：MH4G Japanese／汉化 v1.2、MH4U USA v1.1 与 MH4U EUR v1.1。

[English README](README.md) · [开发方式与贡献分工](docs/DEVELOPMENT_METHODOLOGY.zh-CN.md) · [技术分析](docs/TECHNICAL_ANALYSIS.zh-CN.md) · [研究历程](docs/RESEARCH_HISTORY.zh-CN.md) · [移植说明](docs/PORTING_NOTES.zh-CN.md) · [发布说明](RELEASE_NOTES.md)

## 研究与复现环境

本项目能够正式开展并持续完成动态逆向，关键前提是 **Azahar 2126.0** 提供了可用且足够稳定的 GDB 连接。此前使用的版本无法稳定维持远程调试连接，因而难以可靠完成运行时采样、机器码补丁安装与读回、状态检查以及安全恢复；没有这一调试环境基线，本项目实际上无法建立可重复的研究流程。

即使在 2126.0 上，传统的交互式 breakpoint／watchpoint 也不能作为可靠的主要研究手段：断点命中、继续执行和监视点触发可能出现远程错误、同值写入噪声或会话失效。项目的主要运行时数据不是靠人工逐个下断点取得，而是依靠后来编写的自动化 GDB 脚本、临时代码 hook、低噪音有限 logger、内存快照／导出和逐字读回校验收集。少量窄断点或监视点只用于受控诊断，结论必须再由脚本记录和游戏内行为闭环确认。

Azahar 2126.0 是本项目**已验证的研究／调试基线**。这不自动表示普通玩家运行最终 ExeFS `code.ips` 的最低 Azahar 版本也是 2126.0；最终模组不依赖 GDB，而更早版本的纯运行兼容性尚未被系统测试。

## 功能

- 无摇杆输入的快速变形斩获得 GP；
- 摇杆输入的快速变形斩同样获得 GP；
- 无摇杆输入／摇杆输入、普通／快速动画保持隔离；
- 红盾下成功 GP 可正常产生瓶爆；
- 斧下砸、回旋斩、大解等主要派生正常；
- 连续摇杆输入快速 GP、重复进入与状态回收已通过；
- ExeFS 自动加载与 CPU JIT 开启后的长时间实战测试已通过；
- 不依赖 GDB，正式使用时只需加载 `code.ips`。

## 兼容性

| 游戏版本 | 当前权威状态 | 说明 |
| --- | --- | --- |
| MH4G Japanese／汉化 v1.2 | **ExeFS v3，两种输入分支均已覆盖** | 正式完成；五 hook、640-byte overlay |
| MH4U USA v1.1 | **ExeFS v3，两种输入分支均已覆盖** | 自动加载、CPU JIT 开启及约 22 分钟混合实战验收通过 |
| MH4U EUR v1.1 | **ExeFS v3，两种输入分支均已覆盖** | 自动加载、CPU JIT 开启及约 10 分钟综合实战验收通过 |

不要把不同区域或不同游戏版本的地址、IPS 或 `code.ips` 互换使用。

## 安装

1. 完全关闭 Azahar。
2. 备份现有模组和即时存档。
3. 把发布包中的 `load` 文件夹合并到 Azahar 用户目录。
4. 根据所选发布包确认最终文件位于对应目录：

   | 发布版本 | 最终路径 |
   | --- | --- |
   | MH4G Japanese／汉化 v1.2 | `load/mods/000400000011D700/exefs/code.ips` |
   | MH4U USA v1.1 | `load/mods/0004000000126300/exefs/code.ips` |
   | MH4U EUR v1.1 | `load/mods/0004000000126100/exefs/code.ips` |

5. 重新启动 Azahar，并从正常游戏存档进入游戏。

如果目标位置已经存在其他 `code.ips`，不能直接覆盖或并排放置；需要先备份并正确合并 IPS 记录。

## 卸载

完全关闭 Azahar，移走或重命名所选发布版本 Title-ID 目录下的 `code.ips`，再重新启动游戏。模组不修改 ROM 或正常游戏存档。

## 重要限制

- 启用、禁用或更换 ExeFS 补丁后必须完整重启游戏。
- 不要读取跨越模组启用状态的 Azahar 即时存档；即时存档可能恢复旧代码内存。
- 不要同时启用本项目的 ExeFS、GDB 安装版或 Gateshark 安装版。
- 实战中仍需在盾面方向和正确时机迎接攻击；被击飞不自动代表补丁失效。
- 本项目是非官方研究与模组，与 CAPCOM 无关；使用者需要自行提供合法游戏环境。

## 已验证行为

正式 v3 已闭环验证：

- 五个 hook 与 640-byte overlay 自动加载；
- 无摇杆输入和摇杆输入的快速动画均只播放一次并自然结束；
- 快速／普通动作之间没有观察到动画污染；
- 无摇杆输入与摇杆输入的快速 GP、普通 GP 回归、红盾瓶爆；
- 连续摇杆输入快速 GP 不退化为普通动画；
- 斧下砸、回旋斩、大解等主要派生；
- 翻滚、换区、收刀、战斗恢复与状态清理；
- CPU JIT 开启后的长时间实际游玩。

用户随后与 MHGU 官方实现对比，确认官方摇杆输入的快速变形斩也有 GP，而且其尾端窗口体感与本项目 v3 接近。当前 v3 的快速 GP 实际窗口比普通变形斩更短，因此不再进行人工尾端裁剪。

能够确认的是：快速窗口并非简单完整继承普通窗口。较强的解释是原生动作生命周期、phase 或 motion 变化会自然终止 GP；但具体关闭条件尚未定位，不能把它表述为已经证明由某个 Action 或 flag 单独控制。

## 发布文件标识与校验值

### MH4G v1.2 汉化／日版 — ExeFS v3，无摇杆输入与摇杆输入分支均已覆盖

- Title ID：`000400000011D700`
- 发布包：`MH4G_JPN_v1.2_CB_Fast_Morph_GP_v3_Azahar.zip`
- ZIP SHA-256：`3D5B864D160497167D2CBD2A3BB6F33128A20A9A6E57CD3940C83387A5BDA941`
- `code.ips`：698 bytes，6 records
- `code.ips` SHA-256：`3EB88248D44A9EFE4A83A372A5EA682779BAB2BE8F3E6E8F9101763B88ACA8F4`
- 640-byte overlay SHA-256：`E82E27E04C7163BFBEACBD5ED5B02115B7DFC814803A4EB326102A7B5DC25D03`

### MH4U USA v1.1 — ExeFS v3，无摇杆输入与摇杆输入分支均已覆盖

- Title ID：`0004000000126300`
- 发布包：`MH4U_USA_v1.1_CB_Fast_Morph_GP_v3_Azahar.zip`
- ZIP SHA-256：`B8C9D2B9F48E0F277BBBB5E2449E8EC110F8728A8AA6DF44B58AEF3B72F7B787`
- `code.ips`：698 bytes，6 records
- `code.ips` SHA-256：`683B2AD2A378CA404CA7976F6D3E6721397A77FAB3357AB2C019CEFB5ED932FE`
- 640-byte overlay SHA-256：`E529D92B9ECFD8BE21D084A87250EC426DF0C1091C0F488AFF72B145783E1F0A`
- 摇杆输入第五 hook：`00CC33B4=EA04A574 -> 00DEC98C`
- 已完成干净启动自动加载、CPU JIT 开启及约 22 分钟混合实战验收；最终五个状态字全零。

### MH4U EUR v1.1 — ExeFS v3，无摇杆输入与摇杆输入分支均已覆盖

- Title ID：`0004000000126100`
- 发布包：`MH4U_EUR_v1.1_CB_Fast_Morph_GP_v3_Azahar.zip`
- ZIP SHA-256：`5ECF2013568EA64C133DFCA7374FDDD580C67A869C388265719629DCFC4EB39B`
- `code.ips`：698 bytes，6 records
- `code.ips` SHA-256：`56B266F5FA86346D79339EE84258FC878B23B49408684B7B6DF3237AB3024AB2`
- 640-byte overlay SHA-256：`FB318D5158E4028C45F5FB173D32D9FC5E46D9E179E0FD521D257FAA13949853`
- 正式 IPS 与完成动态测试的 RC1 候选逐字节相同；确定性双构建与包内文件校验均已通过。
- 已完成干净启动自动加载、CPU JIT 开启以及约 10 分钟综合实战验收。

## 文档边界

`archive/current_state_research_archive.md` 是冻结的研究现场档案，保留了阶段性判断、失败路线、后续修正，以及一章在 USA／EUR v3 发布状态尚未完整反映时写入的顶部收尾说明。该收尾章对冻结时点的 MH4G 主研究结案决定仍然有效，但其中旧的 USA／EUR 状态已经被本 README、`RELEASE_NOTES.md` 和 `docs/` 下的整理稿覆盖。不要把原始档案直接当作公开 README。

## 开发方式与贡献分工

本项目由 AI 生成的逆向分析与实现、以及人工操作的运行时实验共同完成。ChatGPT／Codex 负责本项目自制的分析、实验设计、脚本、ARM hook／overlay 实现、构建器、验证器和文档草稿；项目作者负责实际操作 Azahar／GDB、采集全部运行时数据、执行游戏内对照、判断样本有效性并完成最终实机验收。完整边界见[开发方式与贡献分工说明](docs/DEVELOPMENT_METHODOLOGY.zh-CN.md)。

## 致谢

特别感谢 YouTube 用户 [Hazerou](https://www.youtube.com/@hazerou8601)。他发布的两段 MH4U 盾斧金手指资料为本项目最初立项提供了重要线索，也在后续帮助项目确认：不推动左摇杆和推动左摇杆时，游戏会进入两套不同的快速／普通变形分支。本项目随后在 MH4G Japanese／汉化 v1.2 的实际运行环境中独立记录并验证了这些动作编号与入口结构。
