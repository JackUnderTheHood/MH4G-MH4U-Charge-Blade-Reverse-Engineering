# 文档重整说明

日期：2026-08-16

这套文档依据项目当前的 v4 与剑模式蓄力斩 GP v1 最终状态重新整理。目标不是缩短研究历史，而是把已经验证的结论、发布信息和仍然存在的边界分开说明，让没有参与逆向过程的读者也能顺着文档理解。

## 本次整理了什么

- 重新编写中英文 README，先说明补丁功能、版本选择、安装方法和风险提示；
- 将最终机制集中到 Technical Analysis，不再让读者从大量实验日志中自行拼接结论；
- 将 v3、撤回的“v4 重封装候选”和最终两-hook v4 的关系写入 Research History，消除“v4 与 v3 字节相同”的历史歧义；
- 将 JPN、USA、EUR 的独立地址映射、code cave 和验收要求写入 Porting Notes；
- 单独列出六个正式 ZIP、`code.ips`、code cave 的大小和 SHA-256；
- 明确独立快速 v4 与蓄力斩 GP v1 合并版只能二选一，后者已经包含前者；
- 用“不推动左摇杆”和“推动左摇杆”说明两种快速变形分支，避免使用容易误解的“中性版”“方向版”；
- 补充 Azahar 2126.0、GDB 不稳定、断点／监视点不可靠，以及脚本和临时日志记录器承担主要数据采集工作的说明；
- 明确 AI 辅助代码工作与项目作者人工运行、采集和实机验收的分工；
- 保留 Hazerou 的公开 MH4U Gateshark 资料对项目立项和输入分支发现的贡献说明。

## 有意没有改动的内容

### `current_state.md`

它是按实验进展持续追加的研究现场档案，包含失败候选、当时的假设、后来撤销的解释和操作记录。直接润色会破坏历史证据，因此本次保持原样，并保存了逐字一致的备份。

备份文件：

```text
archive/current_state_original_2026-08-16.md
```

- 大小：1,005,740 bytes
- SHA-256：`6FFC5954A0460DDB0CF8F2F2416E403C1ACE15842B3D04C6A840FD7687539089`

### 原始 `reports/` 与实验产物

原始报告、logger 输出、RC 候选和失败实验属于本地证据记录，本次不改写，也没有全部复制到公开文档包。公开阅读应以新 README、Technical Analysis、Research History 和 Porting Notes 为准；需要核对更细的实验过程时，再沿原始档案中的记录查找对应证据。

### 六个正式发布 ZIP

本次只重整文档，没有重新封装已经验收的补丁包。这样可以保留现有正式发布物的文件身份和校验值。若以后修改包内 README，应作为新的发布构建重新计算全部 SHA-256，而不能静默替换旧文件。

## 推荐阅读顺序

1. `README.zh-CN.md`
2. `RELEASE_NOTES.zh-CN.md`
3. `docs/TECHNICAL_ANALYSIS.zh-CN.md`
4. `docs/RESEARCH_HISTORY.zh-CN.md`
5. `docs/PORTING_NOTES.zh-CN.md`
6. `ARTIFACTS.zh-CN.md`

原始 `current_state.md` 只作为 Research Archive，不应直接充当公开 README。
