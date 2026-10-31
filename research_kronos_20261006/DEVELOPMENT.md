# Kronos 研究资料开发说明

这份目录用于保存 Kronos 的可复核研究资料，不是 Kronos 上游源码的替代品。上游源码、模型权重和论文分别以链接及核查提交为准。

## 目录约定

- `kronos_report.md`：面向使用者的用途、运行条件、效果边界和快速体验路径。
- `effects_notes.md`：论文指标、回测设置、数据范围和证据等级的补充核验。
- `sources/`：核查时保存的上游 README、预测器、示例、Web UI 说明、依赖和回测文件；文件名中的 `__` 表示原始路径分隔符。

## 复核流程

1. 记录上游仓库 URL、默认分支、提交 SHA、核查日期和论文版本。
2. 优先从上游 GitHub 内容 API 或固定提交下载文件；不要用未经固定版本的网页截图作为证据。
3. 对 README 的模型规模和论文的实验结果分开记录。论文使用的 large 模型目前未开放时，不能把 large 的结果归到可下载模型。
4. 运行验证时分别记录：依赖安装、源码静态检查、模型权重下载、CPU/CUDA 推理、真实行情回测。没有实际执行的步骤不得写成“已验证”。
5. 任何策略结果都要标明市场、频率、时间切分、复权方式、交易成本、滑点和样本外区间。

## 最小本地验证

```powershell
git clone https://github.com/shiyu-coder/Kronos.git
Set-Location Kronos
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
# 只有准备运行网页界面时才安装 Web UI 依赖
.\.venv\Scripts\python.exe -m pip install -r webui/requirements.txt
```

先做不下载权重的语法检查：

```powershell
.\.venv\Scripts\python.exe -m compileall model examples finetune
```

运行预测前，准备没有 NaN 的连续时间序列，至少包含 `open`、`high`、`low`、`close`；推荐同时提供正确的 `volume` 和 `amount`。未来时间戳必须使用目标市场交易日历，不能把周末或节假日机械填入股票预测。

## 预测实验记录模板

每次实验至少保存以下元数据：

- 数据源、下载时间、复权/合约连续化规则和数据哈希；
- 品种、频率、历史长度、预测长度和交易日历；
- 模型、tokenizer、上游提交 SHA、设备和 dtype；
- `T`、`top_k`、`top_p`、`sample_count` 等采样参数；
- 预测文件、真实后验文件、基线结果和评估脚本版本；
- 含手续费、滑点、最小成交额、涨跌停、停牌和换手限制的结果。

对价格预测，至少同时比较不变价、移动平均或简单动量基线；对交易策略，使用严格的滚动样本外划分，报告收益、最大回撤、换手率、交易成本后结果和不同市场阶段的稳定性。`sample_count` 是多次生成后的内部平均，不应被描述成经过校准的置信区间。

## 变更和发布

更新资料时不要覆盖旧的核查快照。新增目录或文件应注明日期和来源；修改结论时同步更新 `kronos_report.md` 与 `effects_notes.md`。发布前检查 Markdown 链接、代码块中的 Windows 路径、上游提交 SHA 和“未实测”声明，确认没有把作者论文结果写成独立复现结果。

