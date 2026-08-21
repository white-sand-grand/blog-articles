---
title: 一切皆插件的背后我选择开发一个管理插件
description: 终于起了个大早赶上 DSH 开源做第一批贡献
pubDate: 2026-08-21
category: opensource
tags: ['开源', 'GitHub', 'deepseek']
draft: false
---

# 未来已来

## 背景

早就想赶上第一批开源贡献，拿到自己第一个 awesome。我本来是填写了 harness 内测问卷的，但是没选我上，看见他们进群了天天装作自己没进去，到最后大合照才发现这么多熟人。。。

终于也是开放了，我就开始试用，找着卡兹克的推荐下了一堆插件，工作区选在了我放在 D盘 的一个垃圾文件夹，结果直接暴了。因为我下的一个UI合集插件。

我用 glm5.3 修了半天，最终在插件的 issue 里找到了答案，于是我做这个插件的欲望愈发强烈，开始了我的插件设计

## 具体介绍

这里放一个地址先：
[github链接: dsh-plugin-doctor](https://github.com/white-sand-grand/dsh-plugin-doctor)

### dsh-plugin-doctor(以下为删减版README)


[![Awesome DSH Plugin](https://awesome-dsh-plugin.com/badge.svg)](https://awesome-dsh-plugin.com)

`dsh-plugin-doctor` 是 DSH 插件生态的诊断与决策工具。它可以搜索社区插件、比较功能重复、识别聚合 bundle 提供的子插件、检查批量安装冲突、审计实际使用量，并生成插件全景关系图。
灵感来源自 claude code 的命令 "/doctor" 以及无数次下载插件造成的崩溃和冲突。

当前版本：`1.0.0`。插件要求 Node.js `>=22.19`，默认只读，不会自行安装、卸载或启动 Web UI。

#### 安装

在运行 DSH 的同一环境中执行：

dsh plugin --profile web add github:white-sand-grand/dsh-plugin-doctor
dsh web

#### 能做什么

| 用户需求 | 工具行为 |
| --- | --- |
| 找一个插件 | 搜索 GitHub `dsh-plugin` 主题和降级数据源，返回匹配度、功能标签、stars、更新时间和安装引用 |
| 判断功能是否重复 | 比较说明文本、功能标签和依赖，并给出去重或自研建议 |
| 询问插件有什么联系 | 生成相似度关系图；节点带中文功能解释，连线说明相似度及重叠来源 |
| 安装多个仓库前检查风险 | 预检包名、工具名、Cordis patch 行和 peer 依赖；冲突或无法检查时返回 `INSTALL BLOCKED` |
| 检查聚合 bundle | 展开已安装的 DSH 插件形依赖；子插件标记 `providedBy`，不会被建议单独卸载 |
| 查看实际使用情况 | 扫描本地已落盘 session 日志，按 `dsh.tools` 统计调用量、会话数和最近使用时间 |
| 查看插件总体格局 | 生成 Mermaid 关系图，并按 `core`、`active`、`idle`、`review` 分层 |
| 安装前了解官方已知问题 | 在搜索、推荐或收到仓库链接准备安装时，询问是否检查官方仓库的未关闭 Issue，并显示相关链接和风险 |

#### 六个工具

- `plugin_community_search`：社区插件搜索与过滤。
- `plugin_similarity_analyze`：相似度、重复组和不可替代性分析。
- `plugin_recommend`：综合搜索和本地库存做安装、去重或自研决策。
- `plugin_install_guard`：批量安装预检，只读不安装。
- `plugin_usage_audit`：本地 session 用量审计，不上传日志。
- `plugin_landscape`：插件全景与关系图，可把社区候选加入图中。

Agent 会根据问题自动选择工具。

#### 安装前官方 Issue 检查

当用户通过社区搜索寻找可安装插件、请求推荐插件，或直接发送多个仓库链接要求安装时，插件会先询问是否检查这些官方仓库的未关闭 Issue。检查内容包括 Issue 标题和正文与用户需求或故障现象的匹配度，并提供官方链接，例如递归文件监视导致 Web UI 卡顿的未解决报告。

用户明确选择“不检查本会话”后，本会话不会再次弹出这个提醒；下一次新会话会重新询问。用户取消弹窗或当前环境没有交互能力时，插件会说明检查未完成，不会把它误报成“没有已知问题”。Issue API 暂时不可用只影响提醒信息，不会绕过 `plugin_install_guard` 的失败关闭规则。


#### 直接提问与使用 doctor 的区别

直接对 DSH 说“帮我找个插件”时，模型通常只能根据已有上下文给出一般性建议，不能稳定地搜索社区、读取本地包元数据、量化相似度或检查 Cordis 注册冲突。安装多个仓库时也可能跳过预检。

#### 差异

| 直接提问 DSH | 使用 `dsh-plugin-doctor` |
| --- | --- |
| 依赖模型记忆或临时搜索 | 使用 GitHub、缓存和 registry 降级链，并注明数据来源 |
| “看起来重复”但没有数值依据 | 返回文本、功能标签、依赖三类相似度和重复组 |
| 可能漏掉聚合 bundle 内的子插件 | 读取已安装 bundle 依赖并标记 `providedBy` |
| 多个仓库直接尝试安装 | 先检查包名、工具名、patch 行和 peer 依赖；无法验证时失败关闭 |
| 很难知道插件是否真正被使用 | 从本地 session 日志统计调用量、会话数和最近使用时间 |
| 只能用文字描述插件关系 | 输出 Mermaid 关系图和逐对中文解释 |

## 准备做成什么及感悟

以后准备给这篇 blog 再补一补，准备写点技术上的思路，我对这个插件的想法就是——还不够敏捷方便，输入指令还是需要自然语言描述，准备探寻一下一些问题能否采取类似 skills 调用的方式，比如生成关系图的部分。目前我遇见的插件问题还是很多，尤其像新版本视觉发布造成一堆视觉插件的荒废，这说明版本变动同样适合加入其中进行考量

以上
26.8.21