# CLAUDE.md

> AI 助手入口文档 - 学期项目规范导航

## 文档职责

本文档是 AI 助手的**首要阅读文件**，负责：
1. 说明文档体系和职责分工
2. 提供规范文档导航
3. 指导渐进式获取所需规范

**本文档不包含具体规范内容**，具体要求请查阅对应的规范文档。

---

## 文档体系

```
    ├─→ AGENTS.md          # Codex 配置
    ├─→ CLAUDE.md          # Claude Code 配置
    └─→ PROJECT/
        ├─→ PROJECT.md     # 全项目通用规范（必读）
        ├─→ BACKEND.md     # 后端开发规范
        ├─→ FRONTEND.md    # 前端开发规范
        ├─→ TESTING.md     # 测试框架规范
        └─→ SPARK.md       # Spark 流水线开发规范
```

---

## 职责分工

### AGENTS.md（本文档）
- **职责：** 文档导航和使用说明
- **读者：** 所有 AI 助手
- **何时读：** 首次进入项目

### CLAUDE.md
- **职责：** Claude Code/Codex 特定配置
- **读者：** Claude Code、Codex
- **何时读：** 首次进入项目

### PROJECT/PROJECT.md
- **职责：** 全项目通用规范（技术栈、目录结构、文档要求、Git 规范）
- **读者：** 所有开发者和 AI 助手
- **何时读：** 所有任务开始前（**必读**）

### PROJECT/BACKEND.md
- **职责：** 后端开发规范（Java、Spark、测试、性能优化）
- **读者：** 后端开发者和 AI 助手
- **何时读：** 后端开发任务

### PROJECT/FRONTEND.md
- **职责：** 前端开发规范（技术栈白名单、目录结构、交互设计）
- **读者：** 前端开发者和 AI 助手
- **何时读：** 前端开发任务

### PROJECT/TESTING.md
- **职责：** 测试框架规范（Submodule 管理、CI 触发、测试标准）
- **读者：** 测试框架开发者和 AI 助手
- **何时读：** 测试框架开发任务

### PROJECT/SPARK.md
- **职责：** Spark 流水线开发规范（目录结构、脚本要求、Hive on Spark 配置）
- **读者：** Spark 开发者和 AI 助手
- **何时读：** Spark 流水线开发任务

---

## 使用流程

### 第一步：理解文档体系
阅读本文档（AGENTS.md）和 CLAUDE.md，了解文档组织方式。

### 第二步：读取通用规范
**必读：** `PROJECT/PROJECT.md`  
包含所有项目必须遵守的硬性要求。

### 第三步：渐进式获取专项规范
根据当前任务类型，按需读取：
- 后端任务 → `PROJECT/BACKEND.md`
- 前端任务 → `PROJECT/FRONTEND.md`
- 测试框架任务 → `PROJECT/TESTING.md`
- Spark 流水线任务 → `PROJECT/SPARK.md`

### 第四步：执行任务
遵循规范进行开发，保持文档同步。

---

## 快速索引

| 我需要... | 查阅文档 |
|----------|---------|
| 了解技术栈要求 | PROJECT/PROJECT.md |
| 了解目录结构规范 | PROJECT/PROJECT.md |
| 了解文档要求 | PROJECT/PROJECT.md |
| Java 代码规范 | PROJECT/BACKEND.md |
| Spark 开发规范 | PROJECT/BACKEND.md |
| 测试规范 | PROJECT/BACKEND.md |
| Git 提交规范 | PROJECT/PROJECT.md |
| 前端技术栈白名单 | PROJECT/FRONTEND.md |
| 前端目录结构 | PROJECT/FRONTEND.md |
| 测试框架开发 | PROJECT/TESTING.md |
| Submodule 管理 | PROJECT/TESTING.md |
| CI 触发机制 | PROJECT/TESTING.md |
| Spark 流水线开发 | PROJECT/SPARK.md |
| 流水线目录结构 | PROJECT/SPARK.md |
| Hive on Spark 配置 | PROJECT/SPARK.md |

---

## 重要原则

1. **渐进式阅读：** 不要一次性读完所有文档，按需获取
2. **必读优先：** PROJECT.md 是所有任务的必读文档
3. **遵守规范：** 规范文档中的要求是强制性的
4. **保持同步：** 代码变更时必须同步更新文档

---

**文档版本：** 2.0  
**更新日期：** 2026-06-16  
**维护者：** xty
