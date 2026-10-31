# Kronos：用途、运行条件与效果边界

核查日期：2026-10-06。源码快照：`67b630e67f6a18c9e9be918d9b4337c960db1e9a`。本次核查官方 README、预测代码、Web UI 文档、回测代码与作者论文；未下载模型权重，未实测推理速度或复现收益。

## 它是什么

Kronos 是面向金融 K 线的预训练时序模型。输入历史开盘价、最高价、最低价、收盘价，以及成交量、成交额和时间信息，输出未来若干根 K 线的预测数值。它先把连续行情编码成离散 token，再用自回归 Transformer 生成后续序列。[官方仓库](https://github.com/shiyu-coder/Kronos)

作者论文称模型预训练使用了 45 个交易所、约 120 亿条 K 线记录。这是作者披露的训练规模，本次没有独立审计训练数据。[论文](https://arxiv.org/abs/2508.02739)

它接受结构化行情数据，不能直接把新闻、财报或 K 线截图当成核心预测接口的输入。可以把它理解成“根据前面的 K 线，续写后面的 K 线”。输出是模型生成的估计路径，不能当成真实的未来行情。

## 可以用来做什么

| 场景 | 具体用法 | 需要补上的工作 |
|---|---|---|
| 价格路径研究 | 输入过去 400 根五分钟 K 线，预测未来 120 根五分钟 K 线 | 与随后真实行情比较，并与简单基线比较 |
| 波动研究 | 从预测的高低价、收益序列研究波动信号 | 独立验证误差与稳定性 |
| 量化信号研究 | 把预测收益变成股票排序或择时信号 | 策略规则、风险控制、成本、样本外回测 |
| 模拟行情 | 采样生成 K 线序列，用于实验 | 模拟路径不等于真实事件或压力风险分布 |
| 特定市场适配 | 使用自己的行情数据微调模型 | 数据清洗、时间切分、训练资源与复验 |

官方提供 A 股数据微调和 Qlib 回测示例，明确说明这是简化演示，还需要组合优化、风险因子处理等工作才能形成完善策略。[微调说明](https://github.com/shiyu-coder/Kronos#-finetuning-on-your-own-data-a-share-market-example)

## 模型与运行条件

| 已发布模型 | 参数量 | 最大上下文 |
|---|---:|---:|
| Kronos-mini | 410 万 | 2048 根 |
| Kronos-small | 2470 万 | 512 根 |
| Kronos-base | 1.023 亿 | 512 根 |

README 中还列出约 4.992 亿参数的 large，但标注尚未开放。这里的参数量是预测模型的公布数字，实际运行还要加载匹配的 tokenizer。[模型列表](https://github.com/shiyu-coder/Kronos#-model-zoo)

需要 Python 3.10+、PyTorch、依赖库、对应 Hugging Face 权重和历史行情数据。核心预测器支持 CPU、NVIDIA CUDA、Apple MPS，并自动选择设备；源码没有给出统一的最低显存门槛或可复用的本机延迟保证。[预测器代码](https://github.com/shiyu-coder/Kronos/blob/67b630e67f6a18c9e9be918d9b4337c960db1e9a/model/kronos.py#L484)

行情至少有 `open/high/low/close` 列且不能含 NaN。缺少成交量时补零；已有成交量但缺少成交额时，代码使用成交量乘四个价格的均值估计成交额。实际应用应优先提供正确成交量和成交额。还需要历史时间戳与未来时间戳，股票的未来时间戳应按交易日历生成。

预测长度以“根”为单位：120 根五分钟线与 120 根日线不是同一时间跨度。代码里的 `sample_count` 是多次采样后内部取平均，不会自动输出经过校准的置信区间。[预测代码](https://github.com/shiyu-coder/Kronos/blob/67b630e67f6a18c9e9be918d9b4337c960db1e9a/model/kronos.py#L389)

## 快速体验路径

先看作者的 [BTC/USDT 演示](https://shiyu-coder.github.io/Kronos-demo/)。README 将其描述为未来 24 小时预测展示，本次未确认演示当前刷新状态。

本地网页界面的官方启动路径如下，建议在独立 Python 环境中安装：

```powershell
git clone https://github.com/shiyu-coder/Kronos.git
cd Kronos
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pip install -r webui/requirements.txt
cd webui
..\.venv\Scripts\python.exe run.py
```

按官方文档，启动成功后访问 `http://localhost:7070`，准备自己的 CSV 或 Feather 行情数据，选择模型和设备，再查看预测图表。首次加载需要取得权重。上述命令是依据文档整理的启动路径，本次未在本机执行。[Web UI 文档](https://github.com/shiyu-coder/Kronos/blob/master/webui/README.md)

如果通过 Python 集成，入口是 `KronosPredictor.predict()`，返回包含未来 OHLCV 和成交额的 DataFrame。[预测示例](https://github.com/shiyu-coder/Kronos/blob/master/examples/prediction_example.py)

## 效果能信到哪一步

论文报告价格预测的 RankIC 相对基线提升，以及波动预测和模拟行情方面的结果。RankIC 衡量预测排序与真实排序的相关性；“相对提升 93%”不能读作“预测准确率 93%”，也不代表交易胜率或收益率。[论文](https://arxiv.org/abs/2508.02739)

作者做了历史实验和回测，但本次没有独立复现。当前仓库 Qlib 回测确实配置了开仓成本、平仓成本和最低费用，不能笼统说其完全忽略交易成本；这也不等于完整覆盖了实盘滑点、市场冲击、交易可执行性和各种风控情形。[Qlib 回测代码](https://github.com/shiyu-coder/Kronos/blob/67b630e67f6a18c9e9be918d9b4337c960db1e9a/finetune/qlib_test.py#L120)

实际可用性应该用目标市场的连续样本外数据检验：固定数据频率、复权方式、模型版本和采样设置，按时间滚动预测；与不变价、简单动量等基线比较；再评估含成本收益、最大回撤和不同市场阶段的稳定性。漂亮预测曲线只能证明生成与显示功能，不能证明交易价值。

结论：它适合当作量化研究中的预测组件和实验基线。模型规模和 CPU 支持使本地尝试具备可行路径，具体速度尚未实测；能否形成可交易优势，仍需要自己的数据、策略和样本外验证。
