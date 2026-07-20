# Infinite-Canvas
Supports comfyui/API calls/modelscope calls

----

## 本分支改动说明（AI Agent 版）

> 本仓库是基于原项目 [hero8152/Infinite-Canvas](https://github.com/hero8152/Infinite-Canvas) 的二次开发 fork，新增了智能画布 AI Agent 面板。
>
> **当前版本：v1.6**（查看 [版本更新大纲](./CHANGELOG.md)）

### 重要提示
- **已关闭自动更新**：导航栏版本号显示为「Agent版(无自动更新)」。若从上游拉取/合并最新代码，可能覆盖本分支的 AI Agent 功能，请谨慎操作并先备份。
- **main 分支与上游分叉**：本 fork 的 `main` 通过强制推送覆盖，历史与上游不一致，切勿直接向上游发起合并。
- **禁止商业用途**：沿用原作者版权声明（见文末），二次开发须保持开源并注明来源作者。
- **不要提交 API Key到仓库**：请在软件自带的「API 设置」界面填写 Key/URL，不要写入代码或提交到仓库（`.gitignore` 已排除敏感文件与运行时数据）。

> This fork adds an AI Agent panel to the canvas. Auto-update is disabled; pulling upstream changes may overwrite the Agent features. The `main` branch was force-pushed and diverges from upstream. Do not commit API keys. Commercial use remains prohibited.

### 版本更新日志

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-07-18 | AI Agent 面板基线：聊天面板、画布联动生图、多对话管理、参数面板、结构化确认流程、生图占位 |
| v1.1 | 2026-07-19 | 系统提示词重写、修改场景智能区分、LLM 任务后端化、刷新恢复、WebSocket 实时通知、占位比例修复、并发多图不叠加、提示词展开收起 |
| v1.3 | 2026-07-19 | 思维模式开关（扩写/确认/重新生成/修改）、全模型模糊需求统一触发选择、发送按钮失效修复、LLM 任务 5 分钟超时保护、恢复逻辑超时清理 |
| v1.4 | 2026-07-19 | 多图确认流程重构（全部确认后统一生图）、prompts 状态机、内联编辑、Skill 完整保留（首因近因双语）、思维模式兜底 bug 修复 |
| v1.5 | 2026-07-19 | 硬软参数分层（数量=软参数输入框>工具栏，比例/分辨率=硬参数工具栏）、Skill=单张图样式不决定数量、统一数量决策函数 |
| v1.5.1 | 2026-07-19 | 单 generation 多图泄漏修复（API 返回多图限制+参考图 URL 过滤）、count=1 强制范围扩大到所有直接模式 |
| v1.6 | 2026-07-20 | LLM 与生图解耦（思维模式 OFF 跳过 LLM 直接生图）、思维模式多轮维度采集（渐进式问答+自定义输入）、LLM 模型选择移入思维模式面板、框选批量发送至 Agent、附件拖拽排序、发送按钮 bug 修复 |

----

配套的chrome采集插件已经上线：https://chromewebstore.google.com/detail/infinite-canvas-%E5%9B%BE%E5%83%8F%E8%A7%86%E9%A2%91%E6%96%87%E5%AD%97%E6%8A%93%E5%8F%96%E5%B7%A5/ajfhnbklbmpfaaookhfakohabnpmlcic?authuser=0&hl=en

详细教程：[https://youtu.be/1y9ShTvgC_w](https://youtu.be/r_y_9ALr7fg)

由于最近很多API网址关停，我找到一个稳定的网址：

https://apib.ai/register?aff=1uyAbb （包含所有生图模型/视频模型/LLM模型）

https://www.fhl.mom/register?aff=86L574B4T2N9  （包含codex和GPT image 2模型）

功能请求/功能更新/视频教程/联系我，都可以在B站评论或私信：https://space.bilibili.com/78652351


----

【新增了version文件，我每次更新都会更新version的版本号，如果你下载version文件，打开项目后，导航栏的GitHub按键就会提示新版本，如果不想查看更新提示，就删除version文件】

【A version file has been added. I update the version number with each update. If you download the version file, the GitHub button in the navigation bar will indicate the new version after opening the project. If you don't want to see update notifications, delete the version file.】

----

支持的功能：
1. 支持几乎所有OpenAI协议的API/异步协议/Gemini协议/方舟协议
2. RunningHub的工作流/AI应用/收费模型调用
3. 火山引擎调用（人脸认证还在修复bug）
4. Modelscope免费LLM模型和图像模型调用
5. 即梦CLI调用，可直接调用即梦高级会员的积分，支持文生图/图生图/文生视频/图生视频
6. 支持调用本地局域网的ComfyUI
7. 扩展图片/360全景图预览截图/视频帧抽取/循环节点等诸多功能
8. tools文件夹中，增加了chrome批量采集到素材库的插件，PS直连画布调用所有功能的插件

--------

已经申请著作权，禁止商业用途

Commercial use is prohibited.


* 可以自己使用和公司使用，禁止用于任何形式的修改封装成商业产品，商用须取得授权。

* 根据代码二次开发的软件必须保持开源并注明来源作者

* This software is for personal and company use only, but is prohibited from being modified or packaged into commercial products in any way. Commercial use requires authorization.

* Software developed based on this code must remain open source and the original author must be credited.

--------


<img width="2079" height="665" alt="image" src="https://github.com/user-attachments/assets/8469923b-f7a2-403c-9c37-e6e789211f28" />

<img width="1865" height="1503" alt="image" src="https://github.com/user-attachments/assets/f4030201-67c6-4845-b08b-b6fdf304afaa" />


<img width="1696" height="1350" alt="b68e144c5b04a322bfd035da4d89aba3" src="https://github.com/user-attachments/assets/0a6090fb-a8dd-4c3d-adee-b1f9233a2d91" />

   
<img width="1525" height="1473" alt="image" src="https://github.com/user-attachments/assets/6f61fcf9-746c-425b-9e36-cfc8d252da7c" />

   <img width="1261" height="864" alt="image" src="https://github.com/user-attachments/assets/57f3e230-3134-488f-8179-d97e7d15383a" />
<img width="1530" height="858" alt="image" src="https://github.com/user-attachments/assets/9990e42d-22d5-4a10-a1e1-ad35a634edd2" />

<img width="1735" height="1400" alt="image" src="https://github.com/user-attachments/assets/d8328ff8-bbe0-4f1c-9ffa-7b56e8a1a51d" />
<img width="2258" height="969" alt="image" src="https://github.com/user-attachments/assets/4a752d99-885d-4ba9-8b86-91b495786b5c" />


<img width="1531" height="1374" alt="image" src="https://github.com/user-attachments/assets/0af79e38-0955-4740-9e65-5c9bb057f58c" />

<img width="2196" height="1040" alt="image" src="https://github.com/user-attachments/assets/6d823668-cde2-4836-8332-1858efe5f520" />
<img width="2214" height="771" alt="image" src="https://github.com/user-attachments/assets/52e10958-753f-45ba-a50e-3bbec27be436" />
