# Kronos 研究仓库开发文档

本文档是这个研究仓库的交接说明：它解释仓库里有什么、每部分解决什么问题、怎样复核现有结论，以及下一步怎样把静态研究推进到可重复实验。

本仓库保存的是研究报告和上游源码的**核查摘录**，不是 `shiyu-coder/Kronos` 的完整可运行副本。真正运行预测、网页界面、微调和 Qlib 回测时，应从上游仓库检出固定提交。模型权重、行情文件和本地虚拟环境不进入 Git。

## 1. 项目边界

Kronos 是面向金融 K 线的预训练时序模型：输入历史 OHLCV/成交额与时间戳，生成未来若干根 K 线。本仓库负责：

- 记录它能做什么、需要什么和不能证明什么；
- 固定上游提交、论文版本和核查日期，保存可追溯证据；
- 给出 Windows 本地安装、静态检查和实验交接路径；
- 规划从“能启动”到“样本外验证”的后续开发。

本仓库不负责：

- 发布或托管模型权重、交易所行情或用户密钥；
- 把论文里的 RankIC、回测或演示曲线改写成收益保证；
- 直接提供自动下单、账户管理或投资建议。

## 2. 仓库地图

### 根目录

| 路径 | 作用 |
|---|---|
| `README.md` | 项目首页、结论先看、快速开始和证据状态。适合第一次进入仓库的人。 |
| `.gitignore` | 排除 Python 环境、缓存、模型权重、行情文件和实验输出，避免把大文件或敏感数据提交。 |
| `research_kronos_20261006/` | 按核查日期保存的一组研究资料。日期目录不要覆盖旧快照。 |

### 研究目录

| 路径 | 作用 |
|---|---|
| `kronos_report.md` | 面向使用者的说明：身份、用途、输入、模型规格、启动命令和效果边界。 |
| `effects_notes.md` | 面向复核者的证据笔记：论文指标、数据范围、回测参数、OOD 评测和未独立复现项。 |
| `DEVELOPMENT.md` | 本文档：目录约定、操作流程、实验记录和继续开发路线。 |
| `sources/` | 固定上游提交时保存的少量源码/文档摘录，用于对照报告，不保证单独运行。 |

### `sources/` 摘录

文件名中的 `__` 表示上游路径分隔符，例如 `model__kronos.py` 对应上游的 `model/kronos.py`。

| 文件 | 对应上游内容 | 用途 |
|---|---|---|
| `README.md` | 上游 README | 模型列表、安装、预测、微调和官方声明的来源。 |
| `model__kronos.py` | `model/kronos.py` | `KronosPredictor` 输入契约、设备选择、归一化、采样和输出结构的来源。 |
| `examples__prediction_example.py` | `examples/prediction_example.py` | 最小 Python 预测调用样例。 |
| `finetune__qlib_test.py` | `finetune/qlib_test.py` | Qlib 回测流程和交易成本配置的来源。 |
| `requirements.txt` | 上游根依赖 | 运行核心模型所需的 Python 依赖清单。 |
| `webui__README.md` | `webui/README.md` | Flask 网页界面启动、数据格式和操作步骤。 |
| `webui__requirements.txt` | `webui/requirements.txt` | 只有运行 Web UI 时才需要的额外依赖。 |
| `UPSTREAM_LICENSE` | 上游 `LICENSE` | 摘录源码随研究资料保存时的 MIT 许可声明。 |

这些摘录对应的上游核查提交为：

```text
Repository: https://github.com/shiyu-coder/Kronos
Commit:     67b630e67f6a18c9e9be918d9b4337c960db1e9a
Paper:      arXiv:2508.02739
```

## 3. 四档使用方式

### 3.1 只读研究与静态检查

适合核对结论、源码结构和依赖，不下载权重，也不需要行情数据。阅读顺序建议是：根目录 `README.md`、`kronos_report.md`、`effects_notes.md`、`sources/README.md` 与源码摘录，最后看本文档的继续开发路线。

在本仓库执行：

```powershell
python -m compileall -q research_kronos_20261006/sources
git diff --check
```

### 3.2 上游 Python 推理

在仓库外建立独立目录，避免把上游代码和本研究资料混在一起：

```powershell
git clone https://github.com/shiyu-coder/Kronos.git
Set-Location Kronos
git checkout 67b630e67f6a18c9e9be918d9b4337c960db1e9a
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m compileall model examples finetune
```

首次真正预测还需要从 Hugging Face 获取模型和 tokenizer，需要网络和磁盘空间。上游 Python 入口是 `KronosPredictor.predict()`；输入 DataFrame 至少有 `open`、`high`、`low`、`close`，推荐同时提供正确的 `volume`、`amount` 和交易日历时间戳，输出是未来 OHLCV/成交额 DataFrame。

### 3.3 上游 Web UI

只有需要浏览器操作时才安装额外依赖：

```powershell
.\.venv\Scripts\python.exe -m pip install -r webui/requirements.txt
Set-Location webui
..\.venv\Scripts\python.exe run.py
```

按上游文档访问 `http://localhost:7070`，选择数据文件、模型和设备后查看图表。Web UI 能启动不等于预测质量或交易价值已验证。

### 3.4 微调与 Qlib 回测

上游提供 A 股数据准备、tokenizer/predictor 微调和 Qlib 回测示例。这一档需要 Qlib、训练/回测数据和更多计算资源；输出应包括模型 checkpoint、预测结果、基线比较和含成本回测，而不是只保存一张曲线。上游 README 明确它是简化演示，不是生产量化系统。

## 4. 重要输入和输出约束

- 预测长度按 K 线根数计算；120 根五分钟线不是 120 天。
- small/base 最大上下文是 512 根，mini 是 2048 根；超长输入会被截断。
- 输入不能含 NaN；股票时间戳不能机械填入周末或节假日。
- 缺少成交量时源码会补零，缺少成交额时会估算；生产实验应提供正确字段。
- `sample_count` 是多次采样后内部平均，不是经过校准的置信区间。
- 模型不直接读取新闻、财报、订单簿或截图，外生信息需要另建特征和验证链路。

## 5. 验证分层

每次报告结果时都要区分以下状态：

| 层级 | 要回答的问题 | 证据示例 |
|---|---|---|
| 文件核查 | 代码、文档和论文是否对应固定版本？ | 上游 URL、提交 SHA、论文版本 |
| 静态检查 | Python 文件能否编译、依赖声明是否完整？ | `compileall` 输出、依赖安装日志 |
| 启动检查 | Web UI 或脚本能否启动？ | 端口、日志、启动时间和设备 |
| 权重推理 | 指定模型能否返回结果？ | 模型/tokenizer 版本、输入哈希、延迟、内存 |
| 预测评估 | 是否优于简单基线？ | 滚动样本外 MAE/RMSE/RankIC 等 |
| 交易评估 | 含成本后是否稳定？ | 费用、滑点、换手、回撤、成交约束 |

当前仓库完成的是文件核查和源码静态检查；没有完成权重下载、真实推理、延迟测量或独立收益复现。

## 6. 继续开发路线

### 阶段 A：可重复启动

1. 固定 Python、PyTorch、CUDA/CPU 信息和 Hugging Face 模型版本；
2. 在 CPU 上用脱敏 OHLCV fixture 完成一次 `Kronos-small` 推理；
3. 记录权重大小、峰值内存、单次延迟、输出列和设备；
4. 再测 CUDA（如有显卡），把 CPU/CUDA 结果分开；
5. 将日志放入日期化实验目录，不把权重和行情本体提交进 Git。

### 阶段 B：预测基准

1. 明确品种、频率、复权/连续化规则和交易日历；
2. 严格按时间切分训练、验证和测试，避免未来数据泄漏；
3. 与不变价、移动平均、简单动量等基线同时运行；
4. 使用滚动 walk-forward，报告价格误差、收益方向、RankIC 和波动误差；
5. 检查不同市场阶段、品种和预测跨度，不能只挑好看的曲线。

### 阶段 C：回测与风险

1. 明确从预测到持仓的阈值、仓位、调仓频率和持有期；
2. 纳入手续费、滑点、最小成交额、涨跌停、停牌、换手和市场冲击；
3. 报告成本前后收益、最大回撤、换手率、容量和分阶段结果；
4. 检查信号重复使用、幸存者偏差和标签泄漏；
5. 没有稳定样本外证据前，只把 Kronos 当作可替换预测组件。

### 阶段 D：产品化和微调

先把稳定的推理链路封装成 CLI/API，定义输入 schema、配置文件、日志和实验制品目录，再考虑 Qlib/A 股微调：

1. 保存数据来源、下载时间和哈希；
2. 明确微调与测试时间边界；
3. 分开记录 tokenizer/predictor 配置、随机种子、设备和 checkpoint；
4. 用未参与微调的时间段与预训练模型、微调模型、简单基线对照；
5. 增加单元测试、fixture、CI 和失败重试/断点恢复；
6. 接入 Web UI 或真实行情前，先完成成本、风控、订单执行和故障恢复设计。

## 7. 实验记录模板

```text
实验编号：
日期：
代码仓库/提交 SHA：
模型 / tokenizer：
Python / PyTorch / 设备 / dtype：
数据源 / 下载时间 / 数据哈希：
品种 / 频率 / 复权或连续化规则：
历史长度 / 预测长度 / 交易日历：
T / top_k / top_p / sample_count：
训练、验证、测试时间区间：
基线：
预测指标：
交易成本、滑点和成交约束：
输出文件与评估脚本：
结果：
未验证项与风险：
```

## 8. 提交和发布检查

```powershell
python -m compileall -q research_kronos_20261006/sources
git diff --check
git status --short
```

发布前确认：

- Markdown 链接、上游提交 SHA 和论文版本可访问；
- 没有模型权重、行情文件、令牌或本地环境进入提交；
- 新实验写入新的日期目录，没有覆盖旧核查快照；
- “作者报告”“本地复现”“尚未验证”三种表述清楚分开；
- 修改结论时同步更新 `README.md`、`kronos_report.md`、`effects_notes.md`；
- 新增运行结果带有设备、耗时、内存、输入哈希、数据切分和成本假设。

## 9. 当前已知缺口

- 没有完整上游代码、模型权重或真实行情 fixture；
- 没有本机 CPU/CUDA 延迟和显存实测；
- 没有独立复现论文的预测指标或投资回测；
- 没有自动数据下载器、基准评估脚本、CI 或生产推理服务；
- large 模型在官方模型表中未开放，论文中 large 的结果不能直接归因给可下载模型；
- Web UI 的“能启动”与预测质量、交易收益是不同验证问题，必须分别记录。
