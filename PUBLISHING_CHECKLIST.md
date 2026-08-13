# MH4G / MH4U Charge Blade Reverse Engineering — Publishing Checklist / 发布检查清单

## 1. Final regression / 最终回归

- [ ] Confirm the release archive name and SHA-256 against `RELEASE_NOTES.md`.
- [ ] Test a clean Azahar boot from a normal game save, with no GDB/Gateshark installer active.
- [ ] Confirm fast-morph and morph animations with no stick input, plus fast-morph and morph animations with stick input.
- [ ] Confirm red-shield bursts on successful fast-morph and morph GP contacts.
- [ ] Confirm consecutive stick-input fast-morph GP and morph-animation isolation afterward.
- [ ] Confirm axe slam/roundslash and discharge follow-ups.
- [ ] Run a final CPU-JIT-on mixed play session and end in a known safe state.

- [ ] 按 `RELEASE_NOTES.md` 核对发布包名称和 SHA-256。
- [ ] 从正常游戏存档干净启动 Azahar，确保没有 GDB／Gateshark 安装版同时启用。
- [ ] 复核无摇杆输入快速、无摇杆输入普通、摇杆输入快速、摇杆输入普通四种动画。
- [ ] 复核快速和普通 GP 在红盾下的瓶爆。
- [ ] 复核连续摇杆输入快速 GP，以及之后普通动画仍保持隔离。
- [ ] 复核斧下砸／回旋斩和大解派生。
- [ ] 完成一次 CPU JIT 开启的最终混合长测，并在已知安全状态结束。

## 2. Video capture / 视频录制

- [ ] Record an unmodified baseline and patched result under comparable conditions.
- [ ] Show the fast morphs with no stick input and with stick input separately.
- [ ] Include at least one red-shield phial burst and one consecutive stick-input fast GP sequence.
- [ ] Include one morph control to demonstrate no animation contamination.
- [ ] Show the game version, region, Title ID, and mod version in the description rather than overloading the video itself.

- [ ] 在可比条件下录制原版基线与补丁效果。
- [ ] 分别展示无摇杆输入快速和摇杆输入快速变形。
- [ ] 至少展示一次红盾瓶爆和一次连续摇杆输入快速 GP。
- [ ] 展示一次普通变形阳性对照，证明没有动画污染。
- [ ] 在视频说明中写明游戏版本、区域、Title ID 与模组版本。

## 3. Repository / GitHub

- [ ] Use `README.md` as the default page and link `README.zh-CN.md` near the top.
- [ ] Include `RELEASE_NOTES.md`, the edited `docs/` set, and the `archive/README.md` explanation.
- [ ] Decide whether the large chronological archive belongs in the main repository, a release asset, or a separate research archive.
- [ ] Do not upload ROM, decrypted game code, runtime dumps, or copyrighted game assets.
- [ ] Add a license only after choosing terms that actually match the code and documentation ownership.
- [ ] Publish checksums and the exact compatibility matrix.

- [ ] 以 `README.md` 为默认首页，并在顶部链接 `README.zh-CN.md`。
- [ ] 收录 `RELEASE_NOTES.md`、整理后的 `docs/` 与 `archive/README.md` 说明。
- [ ] 决定大型时间线档案放主仓库、Release 附件还是独立 Research Archive。
- [ ] 不上传 ROM、解密游戏代码、运行时 dump 或受版权保护的游戏素材。
- [ ] 确认代码和文档权属后再选择许可证。
- [ ] 公布校验值和精确兼容表。

## 4. YouTube / Nexus Mods / GameBanana

- [ ] Reuse the short summary from `RELEASE_NOTES.md`; do not paste the raw research archive.
- [ ] Put compatibility and installation warnings before the technical backstory.
- [ ] State that the release is unofficial and requires a legally obtained game environment.
- [ ] Link the repository for technical analysis, history, checksums, and porting notes.
- [ ] Link the development-methodology and contribution disclosure; do not describe the project as either AI-only or human-only reverse engineering.
- [ ] Publish both USA and EUR as completed ExeFS v3 ports covering the no-stick and stick-input branches, with their verified checksums.

- [ ] 复用 `RELEASE_NOTES.md` 的短摘要，不直接粘贴研究现场档案。
- [ ] 先写兼容性和安装警告，再写技术背景。
- [ ] 明确这是非官方模组，并要求合法游戏环境。
- [ ] 链接仓库中的技术分析、研究历程、校验值和移植说明。
- [ ] 链接开发方式与贡献分工说明，不把项目写成“AI 单独完成”或“完全没有 AI 参与的个人逆向”。
- [ ] USA 与 EUR 均按“无摇杆输入与摇杆输入两种分支均已覆盖”的 ExeFS v3 发布，并分别附上已验证校验值。

## 5. Archive freeze / 档案冻结

- [ ] Keep `archive/current_state_original.md` unchanged.
- [ ] Keep `archive/current_state_research_archive.md` unchanged after final proofreading.
- [ ] Record new release-engineering work in separate changelogs instead of appending routine publishing steps to the frozen research archive.

- [ ] 保持 `archive/current_state_original.md` 不变。
- [ ] 最终校对后冻结 `archive/current_state_research_archive.md`。
- [ ] 新的发布工程用独立 changelog 记录，不再把日常发布步骤追加进冻结的研究档案。
