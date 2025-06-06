# Skool 静态网页爬虫自动化蓝图分析

## 概述

**蓝图名称：** Skool靜態網頁爬蟲A 0525  
**创建日期：** 2025年5月25日  
**平台：** Make.com 自动化平台  
**用途：** 分析和提取 Skool 社群平台的数据信息

## 工作流程架构

这个自动化蓝图采用了**路由器模式**，根据不同的路径参数执行不同的数据处理逻辑。整个流程包含：

- **1个** Webhook 触发器
- **1个** 变量设置模块
- **1个** 路由器模块，包含 **3个** 不同的处理分支
- 总计 **12个** 功能模块

## 核心模块说明

### 1. 触发器模块
- **模块类型：** `gateway:CustomWebHook`
- **功能：** 接收外部HTTP请求，作为整个自动化流程的入口点
- **配置：** 最大结果数限制为1

### 2. 变量设置
- **模块类型：** `util:SetVariable2`
- **功能：** 设置路径变量 `path = "3"`
- **作用域：** 单次执行周期（roundtrip）

### 3. 智能路由器
- **模块类型：** `builtin:BasicRouter`
- **功能：** 根据路径变量的值决定执行哪个处理分支

## 三个处理分支详解

### 分支1：HTML内容分析 (path = 1)

**处理流程：**
1. **HTTP请求** (`http:ActionSendData`)
   - 获取目标URL的HTML内容
   - 方法：GET
   - 启用GZIP压缩

2. **HTML转文本** (`regexp:HTMLToText`)
   - 将HTML内容转换为纯文本
   - 设置：Linux换行符格式，标题大写

3. **AI内容分析** (`ai-tools:Ask`)
   - 使用AI工具分析提取的文本内容
   - 任务：识别Skool名称和群长信息
   - 输出：200字以内的分析报告

4. **响应返回** (`gateway:WebhookRespond`)
   - 返回AI分析结果

### 分支2：会员数据提取 (path = 2)

**处理流程：**
1. **HTTP请求** (`http:ActionSendData`)
   - 获取目标URL的JSON数据
   - 内容类型：application/json

2. **数据提取** (`ai-tools:Extract`)
   - 从响应数据中提取会员人数
   - 提取字段：`member` (数字类型)

3. **响应返回** (`gateway:WebhookRespond`)
   - 格式化输出会员数信息

### 分支3：详细统计分析 (path = 3) ⭐ 默认路径

**处理流程：**
1. **HTTP请求** (`http:ActionSendData`)
   - 获取目标网页的完整数据

2. **正则表达式解析** (`regexp:AdvancedParser`)
   - 使用模式：`"totalAdmins":.*?"onlineUsers":\[\]\}`
   - 提取关键的JSON数据段

3. **GPT处理** (`openai-gpt-3:CreateCompletion`)
   - 模型：gpt-4.1-mini
   - 任务：将提取的数据转换为JSON格式
   - 响应格式：JSON对象

4. **JSON解析** (`json:ParseJSON`)
   - 解析GPT返回的JSON字符串

5. **格式化响应** (`gateway:WebhookRespond`)
   - 输出详细信息：
     - 总会员数
     - 总贴文数  
     - Skool创建日期（台北时区格式）

## 技术特点

### AI集成
- 使用了多种AI工具：
  - `ai-tools:Ask`：用于智能内容分析
  - `ai-tools:Extract`：用于结构化数据提取
  - `openai-gpt-3`：用于复杂数据处理

### 数据处理能力
- **HTML解析**：能够处理静态网页内容
- **JSON处理**：支持API数据提取
- **正则表达式**：精确提取特定数据段
- **日期格式化**：支持时区转换

### 错误处理
- HTTP请求启用错误处理
- 支持重定向跟踪
- 设置最大错误次数限制

## 使用场景

这个自动化蓝图主要用于：

1. **Skool社群分析**：提取社群基本信息和统计数据
2. **内容监控**：定期检查社群内容和活跃度
3. **数据收集**：自动化收集多个Skool社群的关键指标
4. **报告生成**：生成格式化的社群分析报告

## 配置要求

- **Make.com账户**：需要付费订阅以使用AI工具
- **OpenAI API**：需要配置GPT-4.1-mini访问权限
- **Webhook URL**：需要部署为可访问的API端点

## 数据输出格式

根据路径参数不同，输出格式如下：

- **Path 1**：AI分析报告（文本格式，200字内）
- **Path 2**：会员数统计（格式："URL 的會員數: X 個會員"）
- **Path 3**：详细统计信息（总会员数、总贴文数、创建日期）

## 技术架构优势

1. **模块化设计**：不同功能分离，易于维护
2. **智能路由**：根据需求选择不同处理逻辑
3. **AI增强**：结合多种AI工具提升数据处理能力
4. **灵活输出**：支持多种数据格式和详细程度

## 文件结构

```
├── README.md                           # 项目文档（本文件）
└── blueprint-skool-scraper-0525.json  # 原始Make.com蓝图JSON文件
```

## 如何使用

1. 在Make.com中导入`blueprint-skool-scraper-0525.json`文件
2. 配置OpenAI API连接
3. 设置Webhook触发器
4. 根据需要修改路径变量以选择不同的处理分支
5. 测试并部署自动化流程

---

*该蓝图展示了现代自动化工具与AI技术的深度整合，为社交媒体数据分析提供了高效的解决方案。*
