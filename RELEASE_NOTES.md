# MH4G / MH4U Charge Blade Reverse Engineering — Release Notes / 发布说明

## MH4G Japanese/localized v1.2 — ExeFS v3, both no-stick and stick-input branches covered

Release date / 发布日期：2026-08-11  
Status / 状态：Final; both no-stick and stick-input branches covered / 正式版；无摇杆输入与摇杆输入两种分支均已覆盖

### 中文

v3 在已验证的无摇杆输入版本基础上增加摇杆输入快速变形入口第五 hook，并复用原有 640-byte overlay。它解决了旧版推动摇杆时进入未覆盖摇杆输入分支、因此看似“快速 GP 偶发失效”的问题。

本版已验证无摇杆输入与摇杆输入的快速 GP、普通分支隔离、红盾瓶爆、连续摇杆输入快速 GP、主要派生、状态回收、ExeFS 自动加载以及 CPU JIT 开启后的长时间实战。当前观察还表明快速 GP 窗口短于普通变形斩，因此人工裁剪尾端的计划已取消。

安装或卸载后必须完整重启，不得读取跨越模组状态的即时存档。如果已经使用其他 `code.ips`，需要合并补丁记录。

### English

v3 adds a fifth hook for the stick-input fast-morph entry while reusing the already validated 640-byte overlay. This closes the gap where moving the stick selected an uncovered stick-input branch, which previously appeared as intermittent fast-GP failure.

The release has been validated for fast-morph GP with no stick input and with stick input, morph-branch isolation, red-shield phial bursts, consecutive stick-input fast-morph GP, primary follow-ups, state recovery, automatic ExeFS loading, and extended play with CPU JIT enabled. The fast GP window has also been observed to be shorter than the morph window, so the planned manual tail trim has been cancelled.

A full restart is required after installation or removal. Do not load save states across different mod states. Existing `code.ips` patches must be merged.

## Artifact identity / 文件标识

### MH4G v1.2 localized/Japanese — ExeFS v3, both no-stick and stick-input branches covered

- Title ID: `000400000011D700`
- Archive: `MH4G_JPN_v1.2_CB_Fast_Morph_GP_v3_Azahar.zip`
- Archive SHA-256: `3D5B864D160497167D2CBD2A3BB6F33128A20A9A6E57CD3940C83387A5BDA941`
- `code.ips`: 698 bytes / 6 records
- `code.ips` SHA-256: `3EB88248D44A9EFE4A83A372A5EA682779BAB2BE8F3E6E8F9101763B88ACA8F4`
- Overlay: 640 bytes
- Overlay SHA-256: `E82E27E04C7163BFBEACBD5ED5B02115B7DFC814803A4EB326102A7B5DC25D03`

### MH4U USA v1.1 — ExeFS v3, both no-stick and stick-input branches covered

- Title ID: `0004000000126300`
- Archive: `MH4U_USA_v1.1_CB_Fast_Morph_GP_v3_Azahar.zip`
- Archive SHA-256: `B8C9D2B9F48E0F277BBBB5E2449E8EC110F8728A8AA6DF44B58AEF3B72F7B787`
- `code.ips`: 698 bytes / 6 records
- `code.ips` SHA-256: `683B2AD2A378CA404CA7976F6D3E6721397A77FAB3357AB2C019CEFB5ED932FE`
- Overlay: 640 bytes
- Overlay SHA-256: `E529D92B9ECFD8BE21D084A87250EC426DF0C1091C0F488AFF72B145783E1F0A`
- Stick-input fifth hook: `00CC33B4=EA04A574 -> 00DEC98C`
- Clean automatic loading and an approximately 22-minute CPU-JIT-on mixed regression passed; all five final state words were zero.

### MH4U EUR v1.1 — ExeFS v3, both no-stick and stick-input branches covered

- Title ID: `0004000000126100`
- Archive: `MH4U_EUR_v1.1_CB_Fast_Morph_GP_v3_Azahar.zip`
- Archive SHA-256: `5ECF2013568EA64C133DFCA7374FDDD580C67A869C388265719629DCFC4EB39B`
- `code.ips`: 698 bytes / 6 records
- `code.ips` SHA-256: `56B266F5FA86346D79339EE84258FC878B23B49408684B7B6DF3237AB3024AB2`
- Overlay: 640 bytes
- Overlay SHA-256: `FB318D5158E4028C45F5FB173D32D9FC5E46D9E179E0FD521D257FAA13949853`
- The formal IPS is byte-identical to the dynamically tested RC1 candidate.
- Deterministic double-build and in-archive file validation passed.
- Clean automatic loading, CPU-JIT-on operation, and an approximately 10-minute mixed gameplay regression passed.

## Regional status / 区域版本状态

- MH4G Japanese/localized v1.2: finalized v3 covering both no-stick and stick-input branches / 已完成同时覆盖无摇杆输入与摇杆输入两种分支的 v3。
- MH4U USA v1.1: finalized ExeFS v3 covering both no-stick and stick-input branches / 已完成同时覆盖无摇杆输入与摇杆输入两种分支的 ExeFS v3。
- MH4U EUR v1.1: finalized ExeFS v3 covering both no-stick and stick-input branches / 已完成同时覆盖无摇杆输入与摇杆输入两种分支的 ExeFS v3。

The USA and EUR v3 ports are complete. Any later work is release maintenance rather than unfinished MH4G mechanism research.
