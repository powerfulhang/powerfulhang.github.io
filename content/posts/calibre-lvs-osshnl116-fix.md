---
title: "解决 Calibre LVS 报错 OSSHNL-116：CDL View List 配置指南"
date: 2026-06-24T10:20:00+08:00
draft: false
author: "SiliconBlog"
tags: ["Calibre", "LVS", "OSSHNL-116", "auCdl", "Virtuoso", "PDK", "EDA"]
categories: ["DRC/LVS"]
summary: "Calibre LVS 因 OSSHNL-116 无法导出源网表时，如何检查 PDK Cell View，并正确配置 CDL View List、Stop List 与 Netlist Export Template。"
ShowToc: true
TocOpen: true
---

> **摘要** — Calibre LVS 因 OSSHNL-116 失败，根本原因是 CDL 网表导出器在 PDK 原语 cell 上找不到匹配的 view（通常缺少 `auCdl`）。解决方法是在 **Calibre Netlist Export Setup** 中将 View List / Stop List 与 PDK 实际提供的 view 对齐，而该设置需要从 **Virtuoso → Calibre → Setup → Netlist Export...** 进入，不在 Calibre Interactive 窗口内。

---

## 1. 问题现象

从 Virtuoso Layout Editor 启动 Calibre LVS 后，Source（Schematic）网表导出立即失败，弹窗提示：

```
Exporting source failed
```

查看 Virtuoso CIW 日志，可以看到真正的错误信息：

```
ERROR (OSSHNL-116): Unable to descend into any of the views defined
in the view list, 'cdl schematic', for the instance 'M0' in cell
'test'. Add one of these views to the cell 'dgpfet_33' in the
library '55FSI_3p3', or modify the view list so that it contains
an existing view.

ERROR (OSSHNL-514): Netlist generation failed because of the errors
reported above.
```

## 2. 根因分析

Cadence CDL 网表导出器按设计层次自顶向下遍历。对于每个 instance，它会按 **View List** 中定义的顺序查找对应 cell 的 view。找到后，如果该 view 不在 **Stop List** 中，则继续下降；如果在 Stop List 中，则视为叶子 cell，输出 `.SUBCKT` 调用。

OSSHNL-116 发生在 View List 中列出的**所有 view 都不存在**于某个 cell 上时。以上述报错为例：

- View List = `cdl schematic`
- PDK cell `dgpfet_33`（库 `55FSI_3p3`）既没有 `cdl` view，也没有网表导出器可用的 `schematic` view
- 许多 Foundry PDK 为原语器件提供的是 **`auCdl`** view 而非 `cdl` view

View 名称不匹配 → 网表导出器无法下降 → 网表生成中止 → Calibre LVS 无法继续。

## 3. 解决方案：正确配置 View List 和 Stop List

### 第一步：确认 PDK Cell 实际有哪些 View

在 Virtuoso CIW 中执行：

```scheme
ddGetObj("55FSI_3p3" "dgpfet_33")~>views~>name
```

返回该 cell 的所有 view 列表，例如 `("auCdl" "symbol" "spectre" "layout")`。记下与 CDL 相关的 view 名称，最常见的是 `auCdl`。

### 第二步：打开 Calibre Netlist Export Setup

该设置**不在** Calibre Interactive 窗口内，需要从 Virtuoso 进入：

**Virtuoso 菜单栏 → Calibre → Setup → Netlist Export...**

弹出的 **Calibre Netlist Export Setup** 对话框控制 Virtuoso 如何为 Calibre 导出 Schematic 网表。

### 第三步：修改 View List 和 Stop List

在 Calibre Netlist Export Setup 对话框中：

| 字段 | 修改前（报错） | 修改后（正确） |
|---|---|---|
| **View List** | `cdl schematic` | `auCdl cdl schematic` |
| **Stop List** | `cdl` | `auCdl cdl` |

将 `auCdl` 加入两个列表（并放在最前面），网表导出器就能在 PDK 原语 cell 上找到 `auCdl` view，将其视为叶子 cell，成功生成网表。

### 第四步：重新运行 LVS

回到 Calibre Interactive 窗口，点击 **Run LVS**，网表导出应能成功完成。

## 4. 临时方案：手动导出 CDL 网表

如果无法修改 Netlist Export Setup（例如 PDK 限制），可以绕过自动导出：

1. 在 Virtuoso 中：**File → Export → CDL** — 手动将 Schematic 网表导出为文件
2. 在 Calibre Interactive → Inputs → Source Path 中：
   - **取消勾选** "Export from source viewer"
   - 将 **SPICE File** 指向手动导出的 CDL 网表文件路径
3. 运行 LVS

该方法可行，但每次 Schematic 修改后都需要重新手动导出。

## 5. 总结速查表

| 现象 | 原因 | 修复方法 |
|---|---|---|
| PDK cell 报 OSSHNL-116 | View List 缺少 `auCdl` | 在 View List 和 Stop List 中添加 `auCdl` |
| 找不到设置对话框 | 在 Calibre Interactive 内查找 | 从 Virtuoso 进入：Calibre → Setup → Netlist Export... |

---

**测试环境：** Cadence Virtuoso ICADVM20.1 / IC23.1, Siemens Calibre nmLVS v2024.x, RHEL 7.9 / Rocky Linux 8.10

**标签：** `Calibre` `LVS` `OSSHNL-116` `auCdl` `Virtuoso` `PDK` `EDA`
