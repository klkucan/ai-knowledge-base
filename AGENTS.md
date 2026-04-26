# AI Knowledge Base — 版本演进展示仓库

本仓库包含 AI 知识库系统的多个演进版本。

## 版本结构

| 目录 | 版本 | 定位 |
|------|------|------|
| `v1-skeleton/` | 骨架版 | Agent 角色定义 + 基础工作流 |
| `v2-automation/` | 自动化版 | Pipeline + Hooks + CI/CD |
| `v3-multi-agent/` | 多Agent版 | LangGraph 工作流 + Router/Supervisor 模式 |
| `v4-production/` | **生产版** | 多渠道分发 + Docker 部署 + 交互机器人 |

## 快速进入

- **开发/实验** → 查看 `v3-multi-agent/`（设计模式最全）
- **生产部署** → 使用 `v4-production/`（完整功能）
- **简单演示** → 参考 `v1-skeleton/`（最简结构）

## 通用约定（所有版本）

- 数据目录：`knowledge/raw/`（原始）、`knowledge/articles/`（整理后）
- JSON 文件：2空格缩进、UTF-8、ISO 8601 日期
- 标签格式：英文小写、连字符分隔（如 `large-language-model`）
- 核心流水线：Collector → Analyzer → Organizer（单向数据流）
