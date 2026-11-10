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

## 原 PLAN-0002 AlphaGen 预算试跑 · 2026-11-10

- **目的** 测量单组完整训练的运行成本。项目转向先部署候选方法，再做统一对照，试跑按用户指示提前结束。
- **配置与输入** AlphaGen `923273b`，CPU，种子 0，池容量 10，名义预算 250,000 步，使用 PLAN-0001 冻结的 Tushare 快照。
- **结果位置** `methods/AlphaGen/out/plan2-pilot.log` 和 `methods/AlphaGen/out/table2-csi300/pool-10-seed-0.work-ff7ecdf5/`。
- **结论** 最后一轮完整记录为 45,056 步。最佳中途检查点为 38,912 步，含 10 个因子，2019 年验证 IC 为 0.03053365，Rank IC 为 0.04198371。检查点压缩包、因子及权重已核验。停止前进程峰值工作集约 1.36 GiB。试跑未达到完整预算，测试集未用于评估。

## PLAN-0002 AlphaForge 部署验收 · 2026-11-10

- **AlphaForge 部署提交** `590c88b6be394754dbbe11cf90c73e0b2a52fe7e`，分支 `codex/alphaforge-deployment`。
- **目的** 验证 AlphaForge 的因子挖掘、独立公式池和动态组合能在真实 A 股日频行情上运行并复跑。
- **配置与输入** 官方提交 `d0cfc27df23c60f271bc885fd43027b86b787746`，本地分支 `codex/alphaforge-deployment`，Windows CPU，种子 0。用 PLAN-0001 冻结的 Tushare 日线缓存派生未复权 Qlib 快照，保留沪深 300 历史成分和交易日历。挖掘使用 2018 年的 222 个有效特征日，组合在 2019 年的 223 个特征日验证。标签为 `VWAP[t+21] / VWAP[t+1] - 1`，每段末尾剔除 21 个特征日以隔离标签。数据核验见 `methods/AlphaForge/out/plan2-data-qa.json`。
- **结果位置** `methods/AlphaForge/out/plan2smoke_csi300_2018_0/`、`methods/AlphaForge/out/plan2smoke_repeat_csi300_2018_0/`。命令和环境见 `methods/AlphaForge/REPRODUCING_PLAN2.md`。挖掘耗时约 27.6 秒，组合约 16.6 秒，均含各自的数据加载时间。
- **结论** 挖掘生成 3 个可独立读取的公式。使用默认筛选阈值时，组合每天选出 1 个因子，验证 IC 为 0.03304116，Rank IC 为 0.05379290。复跑的公式 CSV、指标 JSON 和预测张量完全一致。另用放宽阈值的独立诊断验证了三因子回归路径。以上指标用于部署验收，正式对照将使用后续统一口径。
