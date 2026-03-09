# ⭐ StarAutoManager

> 使用 LLM 智能分析你的 Star 分类习惯，自动整理未分类的 GitHub Star。

[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-automated-blue?logo=githubactions)](https://github.com/features/actions)
[![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-blue?logo=python)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**[English](README.md)**

## 工作原理

StarAutoManager 不是随机分类，而是**先学习你已有的 Star Lists 分类风格**，再用相同的逻辑对未分类仓库进行归类。

```
┌─────────────────────────────────────────────────┐
│  1. 获取你已有的 Star Lists（作为训练数据）        │
│  2. 获取所有已 Star 的仓库                        │
│  3. 找出未分类的仓库                              │
│  4. LLM 分析你的分类模式（粒度、命名、维度）       │
│  5. 按照你的风格对未分类仓库进行分类               │
│  6. 通过 GitHub GraphQL API 直接应用修改          │
│  7. 生成报告（GitHub Issue + STARS.md）           │
└─────────────────────────────────────────────────┘
```

### 学习 → 分类 流水线

LLM 以你已有的列表为上下文：
- 学习你的分类粒度（宽泛 vs 精细）
- 识别你的分类维度（按领域、技术、用途还是混合）
- 学习你的命名习惯
- 然后用相同的逻辑对新仓库分类

**冷启动？** 如果你还没有 Star Lists，它会从零提出一个合理的分类结构。

## 功能特点

### 核心
- 🧠 **学习后分类** — 先分析你的分类习惯，再做决定
- 🔄 **增量处理** — 缓存机制避免重复分类
- 🎯 **置信度评分** — 每个分类结果标注 high/medium/low 置信度
- 📖 **二次分析** — 对低置信度仓库获取 README 重新评估
- 📦 **批量处理** — 并发 LLM 调用，高效处理数百个 Star
- 🆕 **智能建列表** — 仓库不适合现有列表时建议创建新列表
- 🏃 **预览模式** — `dry_run: true` 仅预览不修改

### 智能分析
- 🗑️ **过期检测** — 标记已归档或长期未更新的仓库
- 🔄 **重复检测** — 发现功能相似的仓库
- 🌐 **语言统计** — 按编程语言统计你的 Star
- 🏷️ **主题分析** — 统计最常见的 Topic
- 🏥 **列表健康度** — 警告过大或过小的列表

### 自动化
- ⏰ **定时运行** — 通过 GitHub Actions cron 配置
- 🖱️ **手动触发** — 按需运行，支持自定义参数
- 📊 **GitHub Issue 报告** — 每次运行生成详细报告
- 📄 **STARS.md 生成** — 漂亮的分类 Star 列表

## 快速开始

### 1. Fork 本仓库

Fork 或使用模板创建。

### 2. 配置 Secrets

在 **Settings → Secrets and variables → Actions** 中添加：

| Secret | 必需 | 说明 |
|--------|------|------|
| `STAR_GITHUB_TOKEN` | ✅ | GitHub PAT，需要 `repo`、`read:user`、`user` 权限 |
| `LLM_BASE_URL` | ✅ | OpenAI 兼容的 API 地址 |
| `LLM_API_KEY` | ✅ | LLM API 密钥 |
| `LLM_MODEL` | 可选 | 模型名称 — 设置后覆盖配置文件中的 `model` |

> **⚠️ 注意**：请使用 [Fine-grained PAT](https://github.com/settings/tokens?type=beta) 或经典 PAT，不要使用默认的 `GITHUB_TOKEN`（无法管理 Star Lists）。所需权限：`repo`、`read:user`、`user`。

### 3. 自定义配置（可选）

```bash
cp config.example.zh.yaml config.yaml
```

编辑 `config.yaml` 调整行为：

```yaml
llm:
  model: "gpt-5.4"   # 模型名称（也可通过 LLM_MODEL secret 覆盖）
  temperature: 0.3        # 温度越低分类越稳定
  batch_size: 20          # 每批发送的仓库数
  language: "zh"          # 提示语言: "en" 或 "zh"

categorization:
  max_new_lists: 5        # 每次最多创建的新列表数
  min_confidence: "medium" # 自动执行阈值: high, medium, low
  fetch_readme: true      # 二次分析提高准确率
  dry_run: false          # 设为 true 仅预览
  max_repos_per_run: 100  # 每次最多处理数（0 = 不限）
```

### 4. 运行

- **自动运行**：每周一 09:00 UTC（可在 workflow 中修改）
- **手动运行**：Actions → StarAutoManager → Run workflow

## 手动触发选项

| 输入 | 默认值 | 说明 |
|------|--------|------|
| `dry_run` | `false` | 仅预览，不实际修改 |
| `max_repos` | `100` | 最多处理的仓库数 |
| `force_recategorize` | `false` | 忽略缓存，重新分类所有仓库 |

## 支持的 LLM 提供商

支持任何 OpenAI 兼容的 API：

| 提供商 | `LLM_BASE_URL` | 推荐模型 |
|--------|-----------------|----------|
| OpenAI | `https://api.openai.com/v1` | `gpt-5.4` |
| DeepSeek | `https://api.deepseek.com` | `deepseek-chat` |
| Google Gemini | `https://generativelanguage.googleapis.com/v1beta/openai/` | `gemini-2.5-flash-lite` |
| MiniMax | `https://api.minimaxi.com/v1` | `MiniMax-M2.5` |
| GLM / 智谱 | `https://open.bigmodel.cn/api/paas/v4` | `glm-5` |
| Ollama（本地） | `http://localhost:11434/v1` | `qwen3.5` |
| 其他兼容接口 | 你的地址 | 你的模型 |

## 限制

- **Star Lists 上限**：每个用户最多 32 个列表（GitHub 硬限制）
- **GraphQL API 速率**：5,000 点/小时 — 工具自动监控并等待
- **LLM 准确性**：取决于模型质量，推荐 `gpt-5.4` 或 `deepseek-chat`
- **冷启动**：首次运行（无已有列表）会产生较宽泛的分类

## 项目结构

```
StarAutoManager/
├── .github/workflows/
│   └── star-manager.yml         # GitHub Actions 工作流
├── scripts/
│   ├── __init__.py
│   ├── models.py                # 数据模型
│   ├── github_client.py         # GraphQL API 客户端
│   ├── llm_client.py            # LLM 分类引擎
│   ├── star_manager.py          # 流水线编排
│   ├── reporter.py              # Issue 创建 & STARS.md 生成
│   └── main.py                  # 入口
├── config.example.en.yaml       # 英文配置模板
├── config.example.zh.yaml       # 中文配置模板
├── requirements.txt             # Python 依赖
├── README.md                    # English
└── README.ZH.md                 # 中文
```

## 许可证

MIT
