# 实验记录

本文件记录实验索引和简要结论。详细日志、原始输出和结果文件应放到独立结果目录。

## PLAN-0001 环境验收 · 2026-11-10

- **目的** 验证 Windows 环境能通过 Qlib 读取数据，运行 PPO，保存因子池与验证指标。
- **配置与输入** AlphaGen `e3b1526` 派生分支，Python 3.8.20，PyTorch 2.0.1 CPU，种子 0，池容量 10，2048 步。输入为 `methods/AlphaGen/data/synthetic_qlib/` 中 24 只合成证券。
- **结果位置** `methods/AlphaGen/out/synthetic-smoke.log` 和 `methods/AlphaGen/out/synthetic-smoke/checkpoints/`。
- **结论** 生成 4 个因子并保存权重，验证 IC 和 Rank IC 均为有限值。该结果用于部署验收，论文指标对照使用后续真实行情实验。

## PLAN-0001 真实行情短跑 · 2026-11-10

- **AlphaGen 复现提交** `923273be1dae2b5ec736ce17a1fb91320ef0a947`，分支 `codex/kdd-reproduction`，上游基线 `e3b1526`。
- **目的** 验证 Tushare 快照和 AlphaGen 从训练到验证、保存因子池的完整链路。
- **配置与输入** CPU，种子 0，池容量 10，2048 步。输入为 `methods/AlphaGen/data/tushare-csi300/`。快照覆盖 775 只历史成分股和 3558 个交易日。成分按实际调样生效日更新，`SH600849` 通过 `601607.SH` 的更名前行情恢复。
- **结果位置** `methods/AlphaGen/out/tushare-snapshot-qa.json`、`methods/AlphaGen/out/smoke/` 和 `methods/AlphaGen/out/smoke-repeat/`。快照清单 SHA-256 已锁定在 `methods/AlphaGen/reproduction-table2.json`。
- **结论** 核验通过，2019 年验证样本为 224 个交易日。两次短跑均生成 4 个因子，公式、权重和指标 JSON 完全一致。IC 为 -0.08690082，Rank IC 为 -0.08292244。该结果用于链路验收，正式训练与测试安排在下一阶段。
