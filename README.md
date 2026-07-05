# SonicNote Sync for Obsidian

![Version](https://img.shields.io/badge/version-1.1.6-blue)
![Obsidian](https://img.shields.io/badge/Obsidian-%3E%3D0.15.0-7c3aed)

将 SonicNote（妙记）云端的录音数据同步到 Obsidian，每条录音生成一个包含元数据、笔记、AI 总结和转录文本的 Markdown 文件。

## 功能

- **一键同步** — 点击 Ribbon 图标或执行命令，自动拉取所有录音
- **智能去重** — 基于 `audio_id` 跳过已同步的录音，支持标题变更后的文件重命名
- **完整内容** — 同步包含笔记、AI 智能总结、学习报告和带说话人标注的转录文本
- **自动同步** — 支持启动时自动同步和定时重同步（每小时/每3/6/24小时）
- **自定义属性** — 可选的 Frontmatter 字段配置，支持添加自定义 YAML 属性

## 安装

### 手动安装

1. 从 [Releases]() 下载 `sonicnote-sync-x.x.x.zip`
2. 解压到 `<vault>/.obsidian/plugins/sonicnote-sync/`
3. 在 Obsidian 设置 → 第三方插件 中启用 "SonicNote Sync"

### 开发者安装

```bash
git clone <repo-url> <vault>/.obsidian/plugins/sonicnote-sync/
cd <vault>/.obsidian/plugins/sonicnote-sync/
npm install
npm run build
```

## 使用

### 登录

1. 打开 SonicNote（妙记）App → 我的 → MCP Key 管理 → 创建 API Key
2. 在 Obsidian 设置 → SonicNote Sync → 账号 → 点击「登录」输入 API Key

### 同步录音

- **手动同步**：点击左侧 Ribbon 的 🎧 图标，或执行命令面板中的 `SonicNote Sync: 同步录音`
- **自动同步**：在设置中开启「启动时自动同步」和/或「定时重同步」

### 同步内容

每条录音生成一个 Markdown 文件，包含：

```markdown
---
audio_id: "xxx"
record_name: "录音文件名"
duration: "1200"
...
---

# 录音标题

## 笔记
用户编辑的笔记内容

## AI 总结
智能总结内容

## 学习总结
学习报告（知识全景图 + 核心收获 + 课后巩固）

## 转录内容
**[00:05] 张三：** 对话内容...
```

## 设置

| 设置项 | 说明 | 默认值 |
|--------|------|--------|
| 同步文件夹 | Markdown 文件存放目录（Vault 相对路径） | `SonicNoteSync` |
| 包含转录内容 | 是否在文件中包含逐字转录 | 开启 |
| 启动时自动同步 | 每次打开 Obsidian 自动执行同步 | 关闭 |
| 定时重同步 | Obsidian 打开期间定时自动同步 | 关闭 |
| 文件属性 | 选择同步到 Frontmatter 的属性字段 | 全选 |
| 自定义属性 | 添加自定义 YAML 属性到所有同步文件 | 无 |

## 开发

```bash
npm install          # 安装依赖
npm run dev          # 开发模式（自动监听文件变化）
npm run build        # 生产构建（类型检查 + 压缩）
```

构建产物：`main.js` → 复制到插件目录即可生效。

## 技术栈

- TypeScript + esbuild
- Obsidian Plugin API
- 妙记 REST API（Bearer Token 认证）

## 文档

- [官方使用文档](https://ainote.easylinkin.com/#/resources/docs)
- [插件下载页面](https://linkaip.easylinkin.com/public/sonicnotesync-obsidian-install.html)

## 许可

MIT
