# ai-novel-screenplay-analyzer

![License](https://img.shields.io/badge/license-MIT-green) ![Python](https://img.shields.io/badge/python-3.10-blue) ![Node](https://img.shields.io/badge/node-20.19%2B-brightgreen) ![Version](https://img.shields.io/badge/version-v2.8-blue)

## 小说剧本智能分析工作台

小说剧本智能分析工作台：面向长篇小说、剧本与改编项目，自动梳理人物关系、章节脉络与关系演化，支持多模型接入、长任务断点恢复及本地私有部署。

项目以本机浏览器访问的 Web 应用形式运行。

它把长篇原文整理为可复核的项目底稿，帮助出版编辑、影视开发、制片和版权评估团队更快形成共同理解。

> 不是替你读小说，而是让团队读同一本小说。

## 功能特性

- **多项目管理**：按原著或剧本创建独立项目，支持重命名、切换、导入和导出。
- **关系全景**：基于 AntV G6 展示人物、势力与关系，支持章节范围、出现次数阈值和关系轨迹筛选。
- **章节脉络**：自动识别章节并按上下文切分长文本，保留章节信息和切片进度。
- **关系演化**：追踪同一对人物在不同章节中的关系变化。
- **人物档案**：查看人物的关系数、出现章节、关键事件和可追溯关系信息。
- **关系行数**：关系描述支持 1/2/3/5 行切换，长文本说明不被截断。
- **多模型接入**：支持 12 个主流/本地 OpenAI 兼容预设，Base URL 与 Model ID 可自定义，任意 OpenAI 兼容端点皆可接入。
- **可靠长任务**：支持暂停、继续、失败切片重试、限流降并发和刷新后恢复。
- **本地优先**：分析数据存储在本机 SQLite，可用于单机或私有部署。

## 目录

- [快速开始](#快速开始)
- [Windows 桌面安装版](#windows-桌面安装版)
- [配置模型](#配置模型)
- [使用说明](#使用说明)
- [开发命令](#开发命令)
- [数据与安全](#数据与安全)
- [项目结构](#项目结构)
- [API 文档](#api-文档)
- [贡献与许可证](#贡献与许可证)
- [社区](#社区)
- [作者](#作者)

## 快速开始

### Windows 桌面安装版

项目提供 Electron + 本地 FastAPI 的 Windows x64 安装包构建流程。安装包会包含应用前端和打包后的 Python 后端；使用者不需要单独安装 Python、Node.js 或 Conda。首次运行后，应用数据与 SQLite 数据库保存在当前 Windows 用户的应用数据目录中，不写入安装目录。使用 AI 分析仍需在应用内配置可用模型及其 API Key。

在 Windows 10/11 x64 的开发机上，准备 Python 3.10+、Node.js 20.19+ 和 npm，然后在仓库根目录运行：

```powershell
npm run desktop:package
```

构建完成后，NSIS 安装程序位于 `release/`。安装包默认是未签名的开发者构建，Windows SmartScreen 可能显示未知发布者提示。CI 也可通过 `.github/workflows/build-desktop.yml` 手动触发并下载构建产物。

若要生成**免安装的单文件 EXE**，在同一 Windows 构建环境运行：

```powershell
npm run desktop:portable
```

便携版输出为 `release/AI-Novel-Analyzer-2.6.0-Portable.exe`，可直接运行或复制到其他目录；首次启动可能需要解包运行时。项目数据库仍保存在 `%APPDATA%\AI Novel Analyzer`，不会自动随 EXE 移动。若 Electron 下载超时，可在构建前设置 `$env:ELECTRON_MIRROR = "https://npmmirror.com/mirrors/electron/"`。

桌面菜单显示为「文件 / 编辑 / 查看 / 窗口 / 帮助」，其中「查看 → 重新加载」（Ctrl+R）仅重新加载页面。**重新加载后请在左侧「项目」中重新选择原项目**，否则页面显示「未选择项目」和 0 人物并不能代表历史分析结果为空。

> 目前桌面安装包只面向 Windows x64；macOS/Linux 仍使用原有 Web 开发/本地部署方式。

### 环境要求

- Python 3.10+
- Node.js 20.19+（或 22.12+；Vite 7 的运行要求）
- npm
- Windows 一键启动可选：Conda/Miniconda 环境 `vevo`

### 安装依赖

在仓库根目录执行：

```bash
# Python 后端依赖
pip install -r requirements.txt

# 前端依赖
npm install
```

使用 Conda 时：

```bash
conda env create -f environment.yml
conda activate vevo
pip install -r requirements.txt
npm install
```

### 启动服务

项目由一个后端服务和一个前端开发服务器组成。

#### Windows 一键启动

```bat
start.cmd
```

Windows 启动脚本会显式将 SQLite 指向当前仓库根目录的 `storymap.db`。

#### 手动启动

终端 1：

```bash
python backend/main.py
```

终端 2：

```bash
cd frontend
npm run dev
```

启动后访问：

- Web 应用：<http://localhost:15173>
- 后端 API：<http://127.0.0.1:28000>
- Swagger 文档：<http://127.0.0.1:28000/docs>

前端 Vite 会把 `/api` 请求代理到后端 `28000` 端口。

## 配置模型

打开应用左侧的模型管理面板，填写模型名称、协议、Model ID、Base URL 和 API Key，然后先测试连接再保存。

支持的协议：

| 协议 | 标识符 | 默认 Base URL | Model ID 示例 |
| --- | --- | --- | --- |
| OpenAI 官方 | `openai` | `https://api.openai.com/v1` | `gpt-4o / gpt-4-turbo` |
| DeepSeek | `deepseek` | `https://api.deepseek.com/v1` | `deepseek-chat / deepseek-reasoner / deepseek-v4-flash` |
| 月之暗面 Kimi | `moonshot` | `https://api.moonshot.cn/v1` | `moonshot-v1-8k / moonshot-v1-32k / moonshot-v1-128k` |
| 智谱 GLM | `zhipu` | `https://open.bigmodel.cn/api/paas/v4` | `glm-4-plus / glm-4-air / glm-4-flash` |
| 通义千问（OpenAI 兼容） | `dashscope` | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `qwen-plus / qwen-turbo / qwen-max` |
| 火山方舟 | `volcano` | `https://ark.cn-beijing.volces.com/api/v3` | `doubao-pro / doubao-lite` |
| 阿里百炼（原生） | `bailian` | `https://dashscope.aliyuncs.com/api/v1` | `qwen-plus / qwen-max` |
| OpenRouter | `openrouter` | `https://openrouter.ai/api/v1` | 任意模型名（Claude/Gemini/Llama 转发） |
| vLLM（自部署） | `vllm` | `http://localhost:8000/v1` | 本地模型名 |
| Ollama（自部署） | `ollama` | `http://localhost:11434/v1` | 本地模型名 |
| LM Studio（自部署） | `lmstudio` | `http://localhost:1234/v1` | 本地模型名 |
| 自定义 OpenAI 兼容 | `custom` | 自定义 | 任意实现 /chat/completions 的端点 |

MiniMax M3、DeepSeek 等兼容 OpenAI 接口的模型，选择对应预设或 `custom` 并填写 Base URL 与 Model ID 即可。

> API Key 只在本地配置。不要把真实密钥写入 README、源码或提交到版本控制。

## 使用说明

### 快速上手

1. 创建一个小说、剧本或 IP 项目。
2. 导入原文或粘贴文本，确认章节识别和切片范围。
3. 选择已配置的模型，提交分析任务。
4. 在关系全景中查看人物、势力、关系和章节分布。
5. 打开人物档案与关系演化，复核关键关系变化。
6. 按章节范围或出现次数筛选重点，并导出项目数据作为备份或报告素材。

### 基础功能

#### 新建项目

为每部原著或剧本创建独立项目，支持重命名、切换、导入和导出，项目数据保存在本地数据库。

![基础功能：新建项目](demo/%E5%9F%BA%E7%A1%80%E5%8A%9F%E8%83%BD%20%E6%96%B0%E5%BB%BA%E9%A1%B9%E7%9B%AE.png)

#### 开始分析

导入原文或粘贴文本后选择已配置的模型提交分析；后端自动识别章节、按上下文切片，支持暂停、继续、失败重试与刷新后恢复。

![基础功能：开始分析](demo/%E5%9F%BA%E7%A1%80%E5%8A%9F%E8%83%BD%20%E5%BC%80%E5%A7%8B%E5%88%86%E6%9E%90.png)

### 高级功能

#### 人物档案

点击图谱中的人物查看档案：关系数、出现章节、关键事件和可追溯关系信息。

![高级功能：人物档案](demo/%E9%AB%98%E7%BA%A7%E5%8A%9F%E8%83%BD%20%E4%BA%BA%E7%89%A9%E6%A1%A3%E6%A1%88.png)

#### 次数和范围

按人物出现次数与章节范围筛选图谱，快速聚焦核心人物与关键关系：

- 视图模式：全部出现 / 多次出现 / 仅出现一次；「多次出现」可设置「最少出现 N 次」（2–10 次）。
- 章节范围：只看指定章节区间（如第 1–5 章）内的人物与关系。
- 图谱范围：可切换「预览」与「全部」，长篇小说默认用精简预览控制渲染开销。

好处：过滤只露一面的路人角色，突出主要人物；范围收窄后图谱更精简、渲染更快，长篇分析不卡顿。

![高级功能：次数和范围](demo/%E9%AB%98%E7%BA%A7%E5%8A%9F%E8%83%BD%20%E6%AC%A1%E6%95%B0%E5%92%8C%E8%8C%83%E5%9B%B4.png)

#### 异名同人合并（多人映射：多别名→同一主名）

不同人名可能是同一人（如「韩老魔」「韩跑跑」→「韩立」）。顶栏「别名管理」可为一个主名登记多个别名，建立别名→主名映射：别名节点在图上变灰虚线连向主名节点，双方连向同一对象的关系边自动合并（次数累加，边上显示「关系（别名）」）。删除映射即可撤销，不改动原始分析数据。

#### 关系行数

关系连线上的说明文字可切换 1 / 2 / 3 / 5 行显示。

好处：关系描述较长（如「结盟后背叛」「名义师徒、实际对手」）时，行数多更完整，不会被截断；行数少图谱更紧凑，适合整体浏览。

### IP 关系全景

基于 AntV G6 展示人物、势力与关系全景，支持章节范围、出现次数和关系轨迹筛选。

![IP 关系全景](demo/IP%20%E5%85%B3%E7%B3%BB%E5%85%A8%E6%99%AF.png)

### 关系演化时间线

追踪同一对人物在不同章节中的关系变化，支持全部、多次与单次模式切换和章节过滤。

![关系演化时间线](demo/%E5%85%B3%E7%B3%BB%E6%BC%94%E5%8C%96%E6%97%B6%E9%97%B4%E7%BA%BF.png)

### 开发费用预估

根据项目规模估算改编开发成本，辅助立项与预算评估。

![开发费用预估](demo/%E5%BC%80%E5%8F%91%E8%B4%B9%E7%94%A8%E9%A2%84%E4%BC%B0.png)

## 开发命令

在仓库根目录：

```bash
npm run dev                       # 启动前端开发服务器
npm run build                     # 构建前端生产包
npm test                          # 运行前端测试
python -m pytest backend -q       # 运行后端测试
python scripts/verify_legacy_baseline.py  # 验证旧契约基线（Task 1）
```

在 `frontend` 目录：

```bash
npm run dev
npm run build
npm test
npm run check:api-endpoints
```

### 平台验证与发布闸门

运行完整的平台闭环验证和发布检查：

```bash
# 导出 OpenAPI v1 快照
python scripts/export_openapi.py --output contracts/openapi-v1.json

# 运行平台闭环验证（schema 迁移、对账、导入导出、别名、任务等）
python scripts/verify_platform.py

# 运行 OpenAPI 破坏性变更检测
python -m pytest backend/test_openapi_snapshot.py -q

# 运行性能门测试
python -m pytest backend/test_platform_performance.py -q

# 完整发布闸门（所有必须通过）
python -m pytest backend -q
npm test --workspace frontend
npm run build --workspace frontend
python scripts/verify_platform.py
git diff --check
```

## 数据与安全

- 默认数据库：仓库根目录 `storymap.db`。
- 可通过 `STORYMAP_DB` 指定数据库路径。
- API Key 和项目数据保存在本地 SQLite 数据库中，不会由 README 或前端源码提供。
- 建议定期使用项目导出功能备份数据，并在公开仓库中忽略数据库文件和密钥文件。

常用后端环境变量：

| 变量 | 作用 | 默认值 |
| --- | --- | --- |
| `STORYMAP_HOST` | 后端监听地址 | `127.0.0.1` |
| `STORYMAP_PORT` | 后端监听端口 | `28000` |
| `STORYMAP_DB` | SQLite 文件路径 | `storymap.db` |
| `STORYMAP_LLM_CONCURRENCY` | LLM 并发数 | `8` |
| `STORYMAP_LLM_REQUEST_TIMEOUT` | 单次分析请求超时（秒） | `120` |
| `STORYMAP_CORS_ORIGINS` | 自定义 CORS 来源列表 | 空 |

## 项目结构

```text
.
├── backend/
│   ├── main.py                         # FastAPI 后端与 SQLite 数据层
│   ├── task_engine.py                  # 后端任务引擎（后台执行、暂停/恢复、降级限流）
│   ├── progress_repository.py          # 旧断点存储（兼容）
│   ├── test_openapi_snapshot.py        # OpenAPI 破坏性变更检测
│   ├── test_platform_performance.py    # 性能门测试
│   └── test_*.py                       # 后端单元测试
├── contracts/
│   └── openapi-v1.json                 # OpenAPI v1 契约快照
├── scripts/
│   ├── export_openapi.py               # 导出 OpenAPI 快照
│   └── verify_platform.py             # 平台闭环验证脚本
├── frontend/
│   ├── src/                            # React 页面、状态与分析流程
│   ├── vite.config.js                  # Vite 端口与 API 代理
│   └── package.json
├── start.cmd                           # Windows 一键启动
├── start_backend.cmd                   # 仅启动后端
├── start_frontend.cmd                  # 仅启动前端
├── requirements.txt
└── package.json
```

## API 文档

后端启动后访问 <http://127.0.0.1:28000/docs> 查看完整 OpenAPI 文档。

主要接口包括：

- `/api/projects`：项目创建、查询、更新、删除、导入和导出
- `/api/projects/{id}/tasks`：创建分析任务（analyze/continue/retry）、查询任务列表与实时进度
- `/api/projects/{id}/tasks/{tid}`：暂停、继续、取消、调整并发（PATCH）与删除（DELETE）
- `/api/projects/{id}/analyze`：提交文本分析任务
- `/api/llm-models`：模型配置管理与连接测试
- `/api/task-progress`：长任务进度保存、恢复和清理
- `/api/projects/{id}/aliases`：异名同人合并（别名→主名映射的增删与图谱修正）

## 贡献与许可证

欢迎通过 Issue 反馈问题，或提交 Pull Request 改进功能、文档和测试。提交代码前请至少运行相关工作区的测试与前端构建。

本项目采用 [MIT License](LICENSE)。

## 社区

本项目在 [LINUX DO](https://linux.do/) 社区进行开源推广，感谢社区佬友的交流、反馈与建议。

## 作者

你们喜爱的老王 — [B 站 @你们喜爱的老王](https://space.bilibili.com/97727630) · [GitHub](https://github.com/ops120/ai-novel-screenplay-analyzer)

---

小说剧本智能分析工作台：让每一次立项，都有原文依据。
