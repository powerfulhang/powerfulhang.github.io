---
title: "SKILL 编程入门：Cadence Virtuoso 自动化的第一步"
date: 2024-01-15T10:00:00+08:00
draft: false
author: "SiliconBlog"
tags: ["SKILL", "Cadence", "Virtuoso", "自动化", "EDA"]
categories: ["SKILL 编程"]
summary: "SKILL 是 Cadence Virtuoso 平台的内置脚本语言。本文介绍 SKILL 的基础语法、常用函数和一个实际的自动化示例。"
ShowToc: true
TocOpen: true
---

SKILL 是 Cadence Virtuoso 平台的内置脚本语言，基于 Lisp 方言设计。掌握 SKILL 编程是半导体 CAD 工程师提升效率的关键技能之一。

## 为什么学 SKILL？

在日常工作中，CAD 工程师经常需要：

- 批量修改版图属性
- 自动生成 Pcell（参数化单元）
- 定制 DRC/LVS 检查流程
- 开发自定义工具菜单

这些任务如果手动完成，效率低且容易出错。SKILL 可以将这些操作自动化。

## 基础语法

### 变量与赋值

```skill
; 定义变量
myVar = 42
myStr = "Hello, Silicon!"

; 列表
myList = '(1 2 3 4 5)
```

### 函数定义

```skill
procedure(addNumbers(a b)
  a + b
)

; 调用
addNumbers(3 4)  ; => 7
```

### 条件判断

```skill
procedure(checkWidth(w)
  if(w < 0.1 then
    printf("Warning: width %.3f below minimum!\n" w)
  else
    printf("Width %.3f is OK.\n" w)
  )
)
```

## 实用示例：批量获取 Cell 信息

```skill
procedure(listAllCells(libName)
  let((lib cellIds)
    lib = ddGetObj(libName)
    when(lib
      cellIds = lib~>cells
      foreach(cell cellIds
        printf("Cell: %s\n" cell~>name)
      )
    )
  )
)

; 使用
listAllCells("myDesignLib")
```

## 小结

SKILL 编程是 CAD 工程师的核心技能。后续文章将深入介绍 Pcell 开发、DRC 规则编写等进阶主题。

---

*本文是 SKILL 编程系列的第一篇，敬请关注后续更新。*
