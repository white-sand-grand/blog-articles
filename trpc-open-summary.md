---
title: 腾讯犀牛鸟开源计划——trpc-agent-go
description: 在 feat/eval-promptiter-regression-loop 分支上完成了一个完整的 PromptIter 回归循环示例，共 28 个文件、约 3600 行 Go 代码。
pubDate: 2026-08-06
category: opensource
tags: ['开源', 'GitHub', 'Go']
draft: true
---

# 我的第一个大型开源项目参与经历

## 一、背景介绍

有一天我想日常刷了一会微信，发现了一个学长发的推荐，是腾讯犀牛鸟开源人才培养计划。怀着好奇心我点了进去，发现项目意外的和我心意。于是经过我不断分析，选择了名为trpc-agent-go的一个issue —— 构建 Evaluation + Optimization 的自动回归与提示词优化闭环。

这个项目是构建在 PromptIter 引擎 + Evaluation 服务 之上的全闭环回归验证示例：
```
  baseline 评估 → 失败归因 → PromptIter 优化 →
  候选 prompt 评估（hold-out 验证集）→ 多标准验收门控 → 审计报告（JSON + Markdown）
```
具体地址在https://github.com/trpc-group/trpc-agent-go/issues/2003

## 二、一些总结和思考

### 1.任务要求

```
设计并实现一个可复现的 Evaluation + Optimization pipeline。
输入 baseline prompt、训练评测集、验证评测集和优化配置，
系统自动运行 baseline 评测、定位失败 case、执行若干轮优化、对候选 prompt 做验证集回归，
并输出结构化优化报告和是否接受候选的决策。
```

首先工作需求和项目结构我肯定是用agent去看一遍的，自己看时间也不够用，还是要利用好工具。我只在关键处做技术选型和取舍改造就行了，先让模型帮我可视化了项目结构和缺少的部分，其实回到一个问题：真正的工作不是写代码，而是边界应该划在哪。

### 2.各个取舍













