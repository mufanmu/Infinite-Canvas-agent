# 思维模式多图确认流程重构计划

## 项目背景

项目路径：`/Users/yangfan/Desktop/Infinite-Canvas`
核心文件：
- `static/js/smart-canvas.js`（约18863行）—— Agent 面板全部逻辑
- `static/css/smart-canvas.css`（约1560行）—— Agent 面板样式
- `static/smart-canvas.html`—— Agent 面板 HTML 结构
- `main.py`（约17677行）—— 后端服务

当前版本：v1.3，已有思维模式开关功能，但多图确认流程有严重 bug。

---

## 用户反馈的核心问题

### 问题1：选项流丢失数量信息
用户说"生成4张龙"，LLM 返回4个龙的风格选项（options）。用户选了一个后，代码把选项的 value 填入输入框并发送 sendAgentMessage()。但这个 value 是类似"龙，风格：水墨"的描述，不包含"4张"。LLM 收到后只生成1张。

代码位置：`static/js/smart-canvas.js` 第17268行
```js
agentInput.value = value;  // value 只有风格描述，没有数量
sendAgentMessage();
```

### 问题2：确认一张就开始生图，不等全部确认
思维模式下，confirmAgentPrompt 每次确认后立即调用 runAgentGenerations。用户确认第1个后画布就开始出占位/生图了，还没确认完后面3个。

代码位置：`static/js/smart-canvas.js` 第17888-17911行
```js
async function confirmAgentPrompt(assistantMsg){
    // ...
    assistantMsg.generations.push(gen);
    assistantMsg.promptIdx = idx + 1;
    // 确认后立即生图！应该等全部确认完再生
    runAgentGenerations(assistantMsg, userMsg);
}
```

---

## 所有需要修复的问题（按优先级）

### P0：必须修复（解决核心 bug）

#### 1. prompts 数据结构升级（string[] → object[]）

当前 prompts 是纯字符串数组，确认时无法携带 count、use_last_outputs、use_attachments。

当前代码（第17721行）：
```js
parsed.prompts = parsed.generations.map(g => String(g.prompt || '').trim()).filter(p => p);
```
只提取了文本，count/use_last_outputs/use_attachments 全丢了。

改为对象数组：
```js
prompts: [
    {prompt: "水墨龙...", count: 1, use_last_outputs: false, use_attachments: true, status: "pending"},
    {prompt: "油画龙...", count: 1, use_last_outputs: false, use_attachments: true, status: "pending"},
]
```

每个 prompt 对象携带完整生成属性 + 独立状态。

#### 2. 每个 prompt 独立状态

当前只有一个 `promptIdx` 指向"当前显示第几个"。

改为每个 prompt 有独立 status：
- `pending`：还没轮到
- `current`：当前正在看
- `confirmed`：已确认
- `skipped`：跳过不生成
- `editing`：正在内联编辑

#### 3. 四个操作按钮重新定义

当前操作（第17278-17291行）：确认/重新生成/修改

改为：**确认 / 修改 / 重新生成 / 跳过**

- **确认**：标记当前为 confirmed，自动跳到下一个 pending，如果没有下一个了→触发生图
- **修改**：当前提示词进入内联编辑模式（直接在卡片里改文本，不跳到输入框），用户改完点"保存"→标记为 confirmed。不再设置 agentBypassThinkingNext，不跳出确认流程
- **重新生成**：只重新生成当前这一条，不影响其他已确认的。发给 LLM："请重新生成第N条提示词，要求与之前不同"。LLM 返回新 prompt 文本，替换当前这一条
- **跳过**（新增）：标记当前为 skipped，不生成这张，跳到下一个

#### 4. 全部确认后才触发生图

改 `confirmAgentPrompt`（第17888行）：
- 确认后只标记 status=confirmed，推进到下一个 pending
- 检查是否还有 pending 状态的 prompt
- 如果没有 pending 了（全部 confirmed 或 skipped）→ 才调用 runAgentGenerations
- 生图时只生成 status==="confirmed" 的，跳过 skipped 的

`runAgentGenerations` 一次性接收所有 confirmed prompts，统一计算占位位置，整齐排列。

#### 5. 选项选择时补全上下文

改第17268行，选项被选中后不是只发选项 value，而是带上原始请求：

```js
// 找到触发选项流的原始用户消息
const originalRequest = findOriginalRequest();
agentInput.value = `${originalRequest}，选择：${value}`;
// 例如："生成4张龙，选择：龙，风格：水墨"
```

#### 6. 透传 LLM 的 generation 属性

确认创建 generation 时（第17902行），不再重置 count/use_last_outputs/use_attachments，而是保留 LLM 原始返回的值：

当前代码：
```js
const gen = {prompt, count:1, use_last_outputs:false, use_attachments:false, results:[], status:'running'};
```

改为：
```js
const gen = {
    prompt: confirmedPrompt.prompt,
    count: confirmedPrompt.count || 1,
    use_last_outputs: confirmedPrompt.use_last_outputs || false,
    use_attachments: confirmedPrompt.use_attachments || false,
    results: [],
    status: 'running'
};
```

---

### P1：体验提升

#### 7. 全部确认快捷按钮

当 prompts ≥ 2 时，显示"全部确认并生成"按钮。点击后所有 pending 标记为 confirmed，立即触发生图。

#### 8. 修改改为内联编辑

当前 `editAgentPrompt`（第17971行）把 prompt 复制到输入框，设置 bypass 标志，发送后跳出流程。

改为：点"修改"→卡片内提示词文本变成可编辑 textarea→用户改完点"保存"→标记 confirmed→不离开确认流程。

#### 9. 系统提示词动态化

当前 `agentSystemPrompt()`（第17481行）不接受参数，思维模式开/关发给 LLM 的提示词完全相同。

改为根据思维模式开关动态构建：

思维模式 ON 时追加：
```
当前为思维模式。请返回 prompts 数组（不要返回 generations）。
每个 prompt 对象包含：prompt(中文提示词), count, use_last_outputs, use_attachments。
用户会逐个确认后才生图。
如果请求模糊，可以先返回 options 让用户选方向，下一轮再返回 prompts。
如果是修改请求（"换成像素风"），仍返回 prompts，但 use_last_outputs 设为 true。
用户请求N张图时，prompts 必须返回恰好N条。
```

思维模式 OFF 时追加：
```
当前为直接模式。能生成就生成，返回 generations 数组。
```

注意：`agentSystemPrompt(bypassThinking)` 在第17780行已经被调用并传参，但函数定义（第17481行）没接收参数。需要修复函数签名。

#### 10. 确认进度持久化（刷新恢复）

当前 `_setupAgentRecovery`（第16841行）只恢复 LLM task 和生图 task。

需要增加分支：检测到消息有 prompts 且有 pending 状态的 → 恢复确认卡片显示，不触发生图，用户继续从上次中断的地方确认。

需持久化：每个 prompt 的 status、当前 promptIdx、prompts 数组本身（已有）。

#### 11. 状态显示 UI（进度、折叠）

卡片 UI 展示进度：
```
┌─────────────────────────────────┐
│ 📝 提示词确认 (2/4 已确认)       │
│ ✓ #1 水墨龙...        [已确认]  │ ← 折叠，可点击展开反悔
│ ✓ #2 油画龙...        [已确认]  │ ← 折叠，可点击展开反悔
│ ▶ #3 赛博龙...                   │ ← 当前，展开操作按钮
│   [确认] [修改] [重新生成] [跳过] │
│ ○ #4 Q版龙...                   │ ← 待处理，灰色
│ [全部确认并生成]    [全部取消]   │
└─────────────────────────────────┘
```

---

### P2：完善

#### 12. 数量校验

用户说"4张龙"，LLM 返回3个 prompts。
- 卡片顶部显示"用户请求4张，当前3条提示词"
- 提供"补充提示词"按钮 → 让 LLM 再生成几条补齐
- 如果 LLM 返回比请求的多，用户可以跳过多余的

#### 13. 已确认可反悔

已确认的卡片显示为折叠状态 + ✓ 标记。点击展开重新显示操作按钮，用户可以"取消确认"（改回 pending）或"修改"。

#### 14. 确认中发送新消息拦截

检测到有未完成的 prompts（pending 状态存在）时，弹 toast："还有N条提示词未确认，是否放弃当前确认？"用户确认放弃→清除 prompts，发送新消息。

#### 15. 全部取消按钮

用户看到 prompts 后完全不满意，点"全部取消"→清除当前 assistant 消息的 prompts，不触发生图，用户重新输入。

---

## 需要去掉的前端限制（数据驱动改造）

以下前端逻辑替 LLM 做了决定，应该去掉或弱化：

### 去掉：思维模式强制转 prompts（第17716-17728行）

当前思维模式开启时，强制把 generations 转 prompts，不管 LLM 原本返回什么。
改为：系统提示词告诉 LLM 返回 prompts，前端不做强制转换。如果 LLM 仍返回 generations，说明它认为不需要确认，直接生图。

### 去掉：`chatRequestedImageCount`（第16972行）

当前用正则提取"4张"，只认1-4。
改为：完全交给 LLM 理解数量。LLM 在 generations/prompts 中返回对应数量。

### 去掉：修改请求正则识别（第17717行）

当前用正则匹配"改成/换成"等关键词，命中就跳过扩写流程。
改为：LLM 自己判断是不是修改请求，在 prompts 中设 use_last_outputs。

### 去掉：兜底构造 generation（第17665-17690行）

当前 LLM 没返回 generations 时，前端用正则猜意图自己构造。
改为：LLM 没返回就不生，显示 LLM 的 reply/options。

### 保留：JSON 兜底解析（第17502-17548行）

只在 JSON 解析失败时用，合理。

### 保留：占位节点/生图任务/WebSocket（前端职责）

这些是纯前端职责，LLM 管不了。

---

## 呈现规则（纯数据驱动）

LLM 返回的 JSON 字段组合 → 前端呈现方式：

```
{generations: [...]}            → 直接生图（LLM 认为可以生成了）
{options: [...]}                → 显示选项按钮（LLM 认为需要用户选择）
{prompts: [...]}                → 显示确认卡片（LLM 认为需要用户确认）
{reply: "..."}                  → 纯文字回复（LLM 认为需要对话）
{options + generations}         → 先选项，选完直接生图
{options + prompts}             → 先选项，选完进入确认
{reply + generations}           → 一边回复一边生图
```

不再用 thinkingModeOn / isModifyRequest / chatRequestedImageCount 来决定走哪个分支，完全看 LLM 返回了什么。

---

## 关键代码位置索引

| 功能 | 文件 | 行号 | 函数/变量名 |
|------|------|------|------------|
| 系统提示词 | smart-canvas.js | 16750 | AGENT_FORMAT_INSTRUCTION |
| agentSystemPrompt | smart-canvas.js | 17481 | agentSystemPrompt()（需改成接收参数） |
| 数量提取 | smart-canvas.js | 16972 | chatRequestedImageCount()（建议去掉） |
| 修改请求正则 | smart-canvas.js | 17717 | userModifyRe（建议去掉） |
| 兜底构造generation | smart-canvas.js | 17665-17690 | genInProgressRe等（建议去掉） |
| 思维模式强制转prompts | smart-canvas.js | 17716-17728 | thinkingModeOn 块（建议去掉） |
| 批量完整性提示 | smart-canvas.js | 17711 | requestedCount（改为校验） |
| 消息HTML渲染 | smart-canvas.js | 17185-17201 | agentMessageHtml() |
| 选项按钮绑定 | smart-canvas.js | 17243-17277 | data-agent-option |
| 确认卡片绑定 | smart-canvas.js | 17278-17292 | data-agent-prompt-action |
| 选项"确认"处理 | smart-canvas.js | 17248-17267 | value === '确认' 分支 |
| 选项选择后发送 | smart-canvas.js | 17268-17275 | sendAgentMessage() |
| 确认提示词 | smart-canvas.js | 17888-17911 | confirmAgentPrompt() |
| 重新生成提示词 | smart-canvas.js | 17912-17970 | regenerateAgentPrompts() |
| 修改提示词 | smart-canvas.js | 17971-17990 | editAgentPrompt() |
| 执行生图 | smart-canvas.js | 17991+ | runAgentGenerations() |
| 恢复中断操作 | smart-canvas.js | 16841 | _setupAgentRecovery() |
| LLM 任务轮询 | smart-canvas.js | 15228 | pollAgentLlmTask() |
| 发送消息 | smart-canvas.js | 17739 | sendAgentMessage() |
| 处理LLM结果 | smart-canvas.js | ~17600 | processAgentLlmResult() |
| 变量声明区 | smart-canvas.js | 16788-16794 | agentSending/agentThinking/agentBypassThinkingNext |
| 提示词卡片HTML | smart-canvas.js | 17190-17199 | promptCardHtml |
| 提示词卡片CSS | smart-canvas.css | ~1340+ | .agent-prompt-card* |
| 思维模式按钮HTML | smart-canvas.html | ~302 | agentThinkingBtn |
| 思维模式按钮CSS | smart-canvas.css | ~1204 | .agent-thinking-btn |

---

## 改造步骤建议

### 第一步：P0 核心 bug 修复
1. prompts 数据结构升级为 object[]
2. 每个 prompt 独立 status
3. 重写 confirmAgentPrompt（全部确认后才生图）
4. 重写 editAgentPrompt（内联编辑，不跳出流程）
5. 选项选择时补全上下文
6. 透传 generation 属性

### 第二步：P1 体验提升
7. 全部确认快捷按钮
8. 系统提示词动态化
9. 确认进度持久化
10. 状态显示 UI

### 第三步：P2 完善 + 数据驱动改造
11. 去掉前端限制（chatRequestedImageCount/修改正则/兜底构造）
12. 数量校验
13. 已确认可反悔
14. 确认中发送新消息拦截
15. 全部取消按钮

---

## 注意事项

1. **版本号不要动**：main.py 的 sync_static_html_versions() 会在启动时自动改 HTML 版本号，不要手动改版本号文件。用户已关闭自动更新，git 里不要提交 HTML 版本号变更。
2. **不要启动服务测试**：启动服务会触发 sync_static_html_versions 改版本号。改完代码让用户自己重启。
3. **git 提交规范**：用 `git reset --soft` 合并相关提交成一个，不要留一堆零散 commit。
4. **测试要点**：
   - 思维模式 ON + "生成4张龙" → 应返回4个prompts → 逐个确认 → 全部确认后才生图
   - 思维模式 ON + 模糊请求"一只猫" → 应返回options → 选完后再返回prompts
   - 思维模式 OFF + "生成4张龙" → 应直接生4张
   - 确认过程中刷新页面 → 应恢复确认进度
   - 修改单个提示词 → 应内联编辑，不跳出流程
   - 跳过某个提示词 → 应只生成未跳过的
