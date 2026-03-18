---
name: xiaohongshu-post
description: 个性化、定制化小红书发布技能。使用子Agent协作流程：researcher搜热点 → writer从飞书拉素材写文案 → analyst优化 → executor发布。
---

# 小红书文案发布 Skill

## 概述

使用 6 个子 Agent 协作流程，人机协作模式。支持：
- 纯文字（长文）
- 图文（图片 + 文字）

## 触发方式

当用户要求：
- "发小红书"
- "写一篇小红书"
- "发布到小红书"
- "帮我发小红书文案"

## 工作流程（必须按顺序执行）

### 步骤 1：调用 researcher 搜热点（可选）
- **Agent**：researcher
- **任务**：搜索今日热点选题，确定写作方向
- **工具**：web_search（Tavily API）
- **关键词**：用户提供的领域 + 热门话题
- **返回**：热点话题摘要
- **如果用户已有明确主题，可跳过此步骤**

### 步骤 2：调用 writer 写文案
- **Agent**：writer
- **任务**：根据热点创作小红书风格文案
- **输入**：researcher 返回的热点信息 + 用户主题
- **输出**：文案初稿
- **飞书拉素材**：从飞书多维表格（AI成长经历库/文案脚本库）拉取素材

### 步骤 3：调用 analyst 优化
- **Agent**：analyst
- **任务**：检查文案质量，优化标题和标签
- **输入**：writer 的文案初稿
- **输出**：优化建议或直接确认

### 步骤 4：发送文案给用户（主 Agent）
- **必须步骤！** 通过飞书把完整文案发给用户，让用户复制粘贴
- **等待用户确认粘贴完成**

### 步骤 5：调用 executor 发布
- **Agent**：executor
- **任务**：打开小红书发布页，辅助发布
- **两种发布方式**：

#### 方式 A：图文发布（推荐）
1. 打开 https://creator.xiaohongshu.com/publish/publish?target=image
2. 点击"上传图文"
3. **关键**：Windows 文件选择弹窗需要用户手动选择图片
   - 用户点击"上传图片"按钮 → 弹出 Windows 文件选择对话框
   - 用户选择图片后点击"打开"
   - 或者用户告诉 Agent 图片路径，Agent 尝试用 browser.upload 上传
4. 用户在左侧输入框粘贴文案
5. 点击发布

#### 方式 B：长文发布
1. 打开 https://creator.xiaohongshu.com/publish/publish?target=text
2. 点击"写长文"
3. 用户粘贴文案
4. 设置标题、话题
5. 点击发布

### 步骤 6：确认发布成功
- 检查页面提示
- 记录到飞书"已发布内容库"

## 发布页面 URL

| 类型 | URL |
|------|-----|
| 图文发布 | https://creator.xiaohongshu.com/publish/publish?target=image |
| 长文发布 | https://creator.xiaohongshu.com/publish/publish?target=text |
| 视频发布 | https://creator.xiaohongshu.com/publish/publish?target=video |

## 已知问题

### Windows 文件选择弹窗
- **问题**：Chrome 扩展无法控制 Windows 系统级文件选择对话框
- **现象**：点击"上传图片"后弹出的 Windows 弹窗，Agent 看不到也无法点击
- **解决方案**：
  1. 用户手动选择图片上传
  2. 用"文字配图"功能（小红书自动生成配图）
  3. 后续探索：AutoHotkey / pywinauto 自动化控制 Windows UI

## 子 Agent 配置

| Agent | 任务 | 工具 |
|-------|------|------|
| researcher | 搜索热点 | web_search, web_fetch |
| writer | 创作文案 + 拉素材 | message, feishu_doc, feishu_bitable_list_records |
| analyst | 优化文案 | （主Agent自带） |
| executor | 浏览器发布 | browser, exec |

## 输出格式

**必须先发给用户确认：**
```
📕 小红书文案（复制粘贴）

---
标题：xxx

正文：xxx

#标签1 #标签2 #标签3
```

**重要提醒：**
1. 必须先把文案发给用户，等用户确认粘贴完成后，再调用 executor 点击发布！
2. 图片上传需要用户配合手动选择文件（当前技术限制）
3. 浏览器在 WSL2 环境中，关闭浏览器会丢失登录状态

## 飞书素材库

| 表格 | 用途 |
|------|------|
| AI成长经历库 | 原始素材 |
| 自媒体选题库 | 待创作选题 |
| 文案脚本库 | 写好的文案 |
| 人设风格库 | 账号风格设定 |
| 已发布内容库 | 已发布内容（标题、平台、链接、日期） |

## Workspace 图片路径

WSL2 中 workspace 图片可复制到上传目录：
```bash
cp /home/success/.openclaw/workspace/xxx.jpg /tmp/openclaw/uploads/
```
然后用 `browser.upload` 尝试上传（不保证成功，主要看网页是否支持）
