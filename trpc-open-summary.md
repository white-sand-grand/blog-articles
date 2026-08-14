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

#### 问题一：引擎的 acceptance loop 要不要关掉？

PromptIter 引擎自带一个 acceptance loop，验证集 mean gain ≥ 0.01 就接受 patch。但这个引擎有个问题：均值上升不代表每个 case 都变好。 val_resolved_freeform_KEY 从 1.0 退化到 0.0 被均值掩盖，并不能很好判断筛选出来。只做了一个均值提升没考虑到其他退化或者超提升的场景。

因此，我选择保留引擎的 acceptance loop，用 wrapper 做第二道独立验收。我们不能用 wrapper 直接代替，先不说让不让，分支维护起来也是一个问题，因此格外加了一层，略微提高了复杂度。
```go
  // main.go中
  // 第一步：让引擎自己跑完整个优化循环
  result, err := runtime.engine.Run(ctx, buildRunRequest(cfg, surfaceID))

  // 第二步：引擎跑完了，提取它接受的 prompt
  accepted := acceptedInstructionText(result, cfg.CandidateInstruction, surfaceID)

  // 第三步：用这个 prompt 重新评估一轮
  candidate, closeCandidate, err := evaluateCandidate(ctx, cfg, fixture, accepted)

  // 第四步：wrapper 层
  decision := pipeline.ApplyGate(
      pipeline.GatePolicy{...},
      baseline.validation,
      candidate.validation,
      pipeline.GateObservations{CandidateModelCalls: cfg.modelCalls.count()},
  )
```
runtime.engine.Run(...) 是引擎部分。pipeline.ApplyGate(...) 是 wrapper 部分。

#### 问题二：gate 的打分

gate 评测，一个新加入的验收标准

我选的5 条独立标准（has_validation_evidence / validation_improves / no_new_hard_fail / key_cases_protected / within_budget）是全部通过才接受。

其实有想过一些其他方法，比如加权得总分，但那个方案就不好写了，尤其是权重设计，对结果影响太大。或许有历史数据来判断权重分配怎么设计，但我没有那么多数据，所以直接一竿子打死了。

#### 问题三：归因分类分几类

我弄了6类

  | 类别 | 优化器的策略 |
  | --- | --- |
  | response_mismatch | 补充分类说明，增加示例 |
  | tool_call_error   | 明确列出可用工具，说明选择标准 |
  | tool_arg_error    | 补充参数说明和格式要求 |
  | route_error       | 修正分发逻辑或意图识别 |
  | format_error      | 明确输出格式（JSON schema / 固定模板） |
  | knowledge_recall  | 改进知识检索或补充知识库 |

就是具体改什么




