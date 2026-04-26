# GitHub Actions Workflow YAML 格式说明

> 本文档说明 GitHub Actions 工作流中 YAML 文件的编写格式与固定用法。

---

## 1. 基本结构

```yaml
name: 工作流名称                    # 顶层必填，GitHub Actions 页面显示的名称

on:                                 # 触发条件（必填）
  schedule:                         # 定时触发
    - cron: "0 8 * * *"
  workflow_dispatch:                # 手动触发
  push:                             # push 时触发
    branches: [main]
  pull_request:                     # PR 时触发

permissions:                        # 权限设置
  contents: write

jobs:                               # 任务定义（必填）
  job_name:                         # 任务 ID
    runs-on: ubuntu-latest          # 运行环境
    steps:                          # 步骤列表
      - name: Step 1                # 列表元素用 -
        run: echo "hello"           # 要执行的命令
      - name: Step 2
        uses: actions/checkout@v4  # 使用市场 action
```

---

## 2. 核心语法说明

### 2.1 列表与键值对

YAML 中 `- name:` 前缀的 `-` 表示这是 **列表中的一个元素**：

```yaml
steps:           # steps 是列表
  - name: xxx    # 列表项1（- 前缀表示列表元素）
  - name: yyy    # 列表项2
```

`name` 是该元素的键（key），`"xxx"` 是值（value）。

### 2.2 常用指令

| 语法 | 说明 |
|------|------|
| `${{ secrets.KEY }}` | 引用加密密钥（从仓库 Settings 配置） |
| `${{ env.VAR }}` | 引用环境变量 |
| `${{ github.event_name }}` | 上下文变量 |
| `${{ github.sha }}` | 提交 SHA |
| `${{ github.ref }}` | 分支/标签引用 |
| `with:` | 传参给 action |
| `env:` | 定义环境变量 |
| `if:` | 条件执行 |

### 2.3 环境变量与密钥

```yaml
- name: Run pipeline
  env:
    LLM_PROVIDER: ${{ secrets.LLM_PROVIDER }}
    DEEPSEEK_API_KEY: ${{ secrets.DEEPSEEK_API_KEY }}
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: |
    python pipeline.py --sources github,rss
```

密钥在 GitHub 仓库 **Settings → Secrets and variables → Actions** 中配置。

---

## 3. 触发条件固定写法

### 3.1 定时触发（cron）

```yaml
on:
  schedule:
    - cron: "0 8 * * *"              # 每天 UTC 08:00（北京时间 16:00）
    - cron: "0 0 * * 0"              # 每周日午夜（0 和 7 都表示周日）
    - cron: "0 12 * * *"             # 每天中午
```

### 3.2 手动触发

```yaml
on:
  workflow_dispatch:                 # 手动触发（可带参数）
    inputs:
      logLevel:
        description: '日志级别'
        required: true
        default: 'warning'
      tags:
        description: '构建标签'
        required: false
```

### 3.3 事件触发

```yaml
on:
  push:
    branches: [main, develop]       # 指定分支
    tags: ['v*']                     # 标签模式
  pull_request:
    types: [opened, synchronize]    # PR 事件类型
  release:
    types: [published]               # 发布事件
  issue_comment:                     # Issue 评论
```

### 3.4 多个触发条件

```yaml
on:
  schedule:
    - cron: "0 8 * * *"
  workflow_dispatch:
  push:
    branches: [main]
```

---

## 4. 权限设置

```yaml
permissions:
  contents: write        # 允许写入仓库内容
  pull-requests: write   # 允许操作 PR
  statuses: read         # 只读访问状态
```

默认权限见 GitHub 文档，通常显式声明所需最小权限。

---

## 5. Jobs 与 Steps

### 5.1 Job 基本结构

```yaml
jobs:
  build:                           # Job ID
    runs-on: ubuntu-latest         # 运行环境
    timeout-minutes: 60            # 超时时间（分钟）
    continue-on-error: false       # 是否继续即使失败

    steps:
      - name: Checkout
        uses: actions/checkout@v4
```

### 5.2 常用运行环境

| 环境 | 说明 |
|------|------|
| `ubuntu-latest` | Ubuntu 最新版 |
| `ubuntu-22.04` | Ubuntu 22.04 |
| `windows-latest` | Windows 最新版 |
| `macos-latest` | macOS 最新版 |
| `self-hosted` | 自托管 runner |

### 5.3 多 Job 依赖

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - run: echo "job1"

  job2:
    runs-on: ubuntu-latest
    needs: job1                    # 依赖 job1
    steps:
      - run: echo "job2 after job1"

  job3:
    runs-on: ubuntu-latest
    needs: [job1, job2]            # 依赖多个 job
    steps:
      - run: echo "job3 after both"
```

---

## 6. 常用市场 Action

```yaml
- uses: actions/checkout@v4              # 检出仓库代码
- uses: actions/setup-python@v5          # 配置 Python 环境
    with:
      python-version: "3.11"
      cache: "pip"
- uses: actions/setup-node@v4             # 配置 Node.js 环境
    with:
      node-version: '20'
- uses: actions/cache@v4                  # 缓存依赖
    with:
      path: ~/.npm
      key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
- uses: actions/upload-artifact@v4       # 上传构建产物
    with:
      name: artifacts
      path: dist/
- uses: actions/download-artifact@v4     # 下载构建产物
    with:
      name: artifacts
```

---

## 7. 完整示例

```yaml
name: CI Pipeline

on:
  schedule:
    - cron: "0 8 * * *"
  workflow_dispatch:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  collect:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: "pip"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run collection pipeline
        env:
          LLM_PROVIDER: ${{ secrets.LLM_PROVIDER }}
          DEEPSEEK_API_KEY: ${{ secrets.DEEPSEEK_API_KEY }}
        run: |
          python pipeline/pipeline.py \
            --sources github,rss \
            --limit 20

      - name: Commit and push results
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add knowledge/
          git diff --staged --quiet || git commit -m "chore: daily collection"
          git push
```

---

## 8. 常见错误避免

| 错误写法 | 正确写法 |
|----------|----------|
| `name:xxx`（缺少空格） | `name: xxx` |
| `steps:` 下直接写 `name:`（缺少 `-`） | `steps: - name:` |
| `jobs:` 未缩进 | `jobs:` 顶格，下级缩进 2 空格 |
| `runs-on:` 拼写错误 | `runs-on:`（有连字符） |
| 使用 `${{ env.SECRET }}` | `${{ secrets.SECRET }}` |

---

## 9. 附录：Cron 表达式参考

| 表达式 | 含义 |
|--------|------|
| `0 8 * * *` | 每天 08:00 |
| `0 0 * * *` | 每天午夜 |
| `0 12 * * 0` | 每周日 12:00 |
| `0 */2 * * *` | 每 2 小时 |
| `30 4 1 * *` | 每月 1 日 04:30 |

格式：`分 时 日 月 周`
