# Kronos Financial Forecasting Research

面向 [shiyu-coder/Kronos](https://github.com/shiyu-coder/Kronos) 的证据化研究、运行说明与验证边界。

Kronos 是一个针对金融 K 线的预训练时序模型：输入历史 OHLCV/成交额和时间戳，生成未来若干根 K 线的数值预测。它适合做量化研究中的预测组件、基线和实验原型，不是聊天模型，也不是拿来即用的自动交易系统。

## 文档入口

- [研究报告](research_kronos_20261006/kronos_report.md)：它能做什么、需要什么、如何启动以及效果应如何理解。
- [效果证据与边界](research_kronos_20261006/effects_notes.md)：论文指标、训练数据范围、回测设置和未独立复现项。
- [开发与复核说明](research_kronos_20261006/DEVELOPMENT.md)：目录约定、固定版本、Windows 检查、实验记录模板和发布门槛。
- [上游核查快照](research_kronos_20261006/sources/README.md)：核查时保存的上游 README、预测器、示例、Web UI 说明、依赖与回测文件。

## 结论先看

- 已发布模型：Kronos-mini（410 万参数）、Kronos-small（2470 万）、Kronos-base（1.023 亿）；large（约 4.992 亿）在官方模型表中尚未开放。
- small/base 的最大上下文是 512 根，mini 是 2048 根。预测长度按 K 线根数计算，不能把 120 根五分钟线理解成 120 天。
- 预测器支持 CPU、CUDA 和 Apple MPS。需要 Python 3.10+、PyTorch、对应 Hugging Face 权重和连续的历史行情数据；本仓库没有上传权重或行情数据。
- 作者报告的 RankIC 提升是预测排序相关性的相对指标，不是 93% 准确率、胜率或收益率。本文档没有把作者结果写成独立复现。

## 快速开始

本仓库保存研究资料；运行 Kronos 本体请使用上游仓库：

```powershell
git clone https://github.com/shiyu-coder/Kronos.git
Set-Location Kronos
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
# 只有需要网页界面时才安装这一组
.\.venv\Scripts\python.exe -m pip install -r webui/requirements.txt
Set-Location webui
..\.venv\Scripts\python.exe run.py
```

网页界面按上游文档使用 `http://localhost:7070`。Python 集成入口为 `KronosPredictor.predict()`；输入至少要有 `open`、`high`、`low`、`close`，推荐提供正确的 `volume` 和 `amount`。首次运行还需要从 Hugging Face 获取模型和 tokenizer。

## 证据状态

核查使用的上游提交为 `67b630e67f6a18c9e9be918d9b4337c960db1e9a`，论文为 arXiv `2508.02739`。本次完成了官方 README、预测代码、Web UI 文档、Qlib 回测代码和论文的静态核查；没有下载权重、训练、实测 CPU/CUDA 延迟，也没有独立复现收益。

任何面向真实市场的结论都应补做滚动样本外测试，并与不变价、移动平均或简单动量基线比较，同时纳入手续费、滑点、最小成交额、涨跌停、停牌、换手和回撤约束。

## 来源

- [Kronos 上游仓库](https://github.com/shiyu-coder/Kronos)
- [Kronos 论文（arXiv:2508.02739）](https://arxiv.org/abs/2508.02739)
- [官方 BTC/USDT 演示](https://shiyu-coder.github.io/Kronos-demo/)

研究资料仅用于复核和工程评估，不构成投资建议。
