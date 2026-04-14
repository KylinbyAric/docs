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

# SubAgent的





