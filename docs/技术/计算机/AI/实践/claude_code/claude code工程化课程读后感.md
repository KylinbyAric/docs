# 什么是Claude code 
claude code是一个AI agent框架。

## Claude code 底层技术

### Memory(记忆系统)
核心是CLAUDE.md，存储了当前项目的基本信息。

### 扩展层
Commands、Skills、SubAgents、Hooks

Commands:提供基础的claudecode命令，包括读取、写入、bash。大模型相当于大脑，则命令相当于神经系统，通过神经系统调度手脚(agent、mcp)
Skills：解决的是该不该做，怎么做，做到什么程度。是一个很专门化的技能。
SubAgents：子代理解决的是大型任务如何拆分执行问题。
Hooks：钩子，在特定时间触发的时候自动化执行的脚本。用于限制边界。

### 集成层

Headless和MCP
Headless：？？ 。
MCP：Model Context Protocol。相当于手，大模型通过mcp能使用外部工具。

### 编程接口层
Agent SDK。自定义Agent能力---资深用户必备


### 触发方式
![组件触发方式.png](%E7%BB%84%E4%BB%B6%E8%A7%A6%E5%8F%91%E6%96%B9%E5%BC%8F.png)

### 数据流向
![数据流向.png](%E6%95%B0%E6%8D%AE%E6%B5%81%E5%90%91.png)

# Memory记忆系统
claude code启动时，会扫描以下文件，注入到每次对话中
- ~/.claude/CLAUDE.md
- ./CLAUDE.md
- ./CLAUDE.local.md
- ./.claude/rules/*.md

## 编写高效的CLAUDE.md
### 原则1：less is more
简洁，不仅节约上下文，而且让大模型更专注
### 原则2：具体由于泛泛
bad case
```text
# 项目规范
## 代码质量
请写出高质量的代码。代码应该是可读的。使用有意义的变量名。
保持代码整洁。遵循最佳实践。不要写重复的代码。
```
good case
```text
# 项目规范

## TypeScript
- 使用 `interface` 定义对象结构，`type` 用于联合类型
- 禁止 `any`，使用 `unknown` + 类型守卫
- 函数参数 > 3 个时，使用对象参数

## 错误处理
```typescript
// 业务错误
throw new BusinessError('ORDER_NOT_FOUND', '订单不存在');

// 验证错误（Zod 自动抛出）
const data = orderSchema.parse(input);

// controller 中不要 try-catch
// 由全局错误中间件统一处理
```
### 原则3：渐进式披露
??

# SubAgent
一个问题，当AI在进行多轮对话时，比如
- 跑测试
- 搜索代码
- 分析错误
在多轮之后，在这个过程中，很多上下文是当前执行有效，但对之后的决策是一种噪音。大模型的上下文会越来越臃肿，导致AI越来越健忘。那有没有什么解决办法呢？

  
## 什么是subAgent
子代理就是“专职小助手”，完成任务后将结果摘要待会给主Agent---当前对话。

本质是一个SKill??

## sybAgent的核心价值

### 隔离
隔离上下文，避免主对话的干扰
主对话的上下文：
┌─────────────────────────────────────────┐
│ 用户：帮我分析一下这个 bug              │
│ Claude：好的，让我看看...               │
│ [子代理去执行，产生 500 行日志]         │
│ [子代理返回：发现 3 个相关文件]         │
│ Claude：我发现问题在这三个文件...       │
└─────────────────────────────────────────┘

子代理的上下文（独立的，执行完就释放）：
┌─────────────────────────────────────────┐
│ 任务：查找 bug 相关文件                 │
│ [搜索输出 500 行日志]                   │
│ [分析过程...]                           │
│ 结论：3 个相关文件                      │
└─────────────────────────────────────────┘

### 约束
约束，解决的是子代理的边界问题，希望子代理是一个可控的，有边界的。减少不确定性。如Read，那就只有读取而非写入。

### 复用
当子代理越发专业化时，则可以复用。

### 内置子代理
#### Explore 子代理
专注搜索，将grep、分析过程等包装起来，只告诉结论

#### plan子代理
进入规划模式时，会用plan实施方案

#### General-purpose 子代理
通用子代理

### 什么时候该用子代理
不是自动的？？

第一类是高噪声输出的任务
如分析trace,梳理代码的过程。

第二类是边界非常明确的
比如Pii、比如读取

第三类，并行开展的研究型任务
plan??

### 关键约束：子代理不能生成子代理
这会子代理溢出的。。。因此只能由主代理调度。

### 省token
虽然子代理本身会有额外的prompt、来回通信的诉求，有子代理会增加token的消耗。
但对于特型的子代理，可以省token。因为可以让它选择便宜的模型。


# Skill

## 什么是Skill?
SKill是一个特殊的Agent，里面包含描述、触发时机、如何执行。是一种可操作的结构。
一般会包含说明、工具(各种脚本等等)。



## 触发机制
### 显式触发
用户直接指定skill
### 分层评审
由claude理解用户意图，然后根据上下文决策使用哪个skill.

## 如何设计好的Skill

精准的描述调用时机，非常明确的执行步骤


## 命令型SKill
只有用户指定时才会触发，AI绝不主动触发的skill
实现上，命令型SKill是设置了“disable-model-invocation: true”的skill

### 参数传递
单参数:$ARGUMENTS
```text
---
description: Quick git commit
argument-hint: [commit message]
disable-model-invocation: true
---

Create a git commit with message: $ARGUMENTS
```
多参数：$N
```text
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```






