# Infinite-Canvas
Supports comfyui/API calls/modelscope calls

----

## 本分支改动说明（AI Agent 版）

> 本仓库是基于原项目 [hero8152/Infinite-Canvas](https://github.com/hero8152/Infinite-Canvas) 的二次开发 fork，新增了智能画布 AI Agent 面板。

### 新增功能
1. AI Agent 侧边聊天面板（OneBox 风格），可与画布联动生图。
2. `@` 引用当前画布中的图片，最新的图片排在最上面。
3. 统一附件管理：Skill 文档与图片在同一处增删改查，支持一次上传多个。
4. 输入框支持自定义拖拽/自动拉高。
5. 多对话管理（新建 / 删除 / 对话列表，与画布绑定），消息支持复制 / 重试。
6. 生图参数面板：质量（自动/高/中/低）、比例（9 种带图标）、分辨率（1K/2K/4K）、数量（1-8）。
7. LLM 结构化确认流程：提出方案后返回选项按钮，「确认」直接开始生成、「修改」重新生成提示词。
8. LLM 生图能力升级（角色/风格一致性、图片编辑、批量多样性、质量控制、实时反馈、图片组合、动态参数、风格迁移、图片理解等）。
9. 生图占位：生成开始时先在画布放置占位骨架，完成后替换为实际图片；多张并发生成、顶部对齐向右排列。
10. 在画布选中图片的悬浮工具栏最左侧新增「发送至 Agent」按钮。

### 重要警示
- **已关闭自动更新**：导航栏版本号显示为「Agent版(无自动更新)」。若从上游拉取/合并最新代码，可能覆盖本分支的 AI Agent 功能，请谨慎操作并先备份。
- **main 分支与上游分叉**：本 fork 的 `main` 通过强制推送覆盖，历史与上游不一致，切勿直接向上游发起合并。
- **禁止商业用途**：沿用原作者版权声明（见文末），二次开发须保持开源并注明来源作者。
- **不要提交 API Key到仓库**：请在软件自带的「API 设置」界面填写 Key/URL，不要写入代码或提交到仓库（`.gitignore` 已排除敏感文件与运行时数据）。

> This fork adds an AI Agent panel to the canvas. Auto-update is disabled; pulling upstream changes may overwrite the Agent features. The `main` branch was force-pushed and diverges from upstream. Do not commit API keys. Commercial use remains prohibited.

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
