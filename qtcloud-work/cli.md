# qtcloud-work CLI 重构路线图

本路线把 `data/report/evaluation/qtcloud-work-cli.md` 的偏移清单与 `data/report/plan/qtcloud-work-cli.md` 的拆法按站排开。

`workspace` 是 `workorder` 的上级容器，容器认内件、内件认容器是常态，两条互认不是环，不列站（报告·偏移 4 是文档没跟上，不是依赖问题）。

每站行为不变：命令面、`--json` 四字段、报错文字都不改；做完跑 `cargo fmt --check`、`clippy --all-targets --locked -- -D warnings`、`cargo test --locked` 三绿。

## 站一 order ↔ prompts（报告·偏移 1）

现状：`src/prompts.rs:94` 的 `previous_records` 收 `&[crate::order::WorkRecord]`；`src/order/ai.rs:5`、`:12`、`:43`、`:157` 又引 `crate::prompts`。两个方向都在。

来路：`2cc2e41`（按聚合/服务/适配立目录、拆长文件）让 `task/ai.rs` 引 `crate::prompts`，是单向的一边；`e689df7`（task 拆工单与工作记录）在 `prompts.rs` 加 `previous_records`、参数写成 `order::WorkRecord`，另一边接上才成环。此前 `prompts.rs` 只引 `criterion`。

拆法：把 `previous_records` 移出 `prompts`，把造话术、调 pi、审从 `order/ai.rs` 移到适配层；`order` 只留走一步的判据与记账。

完成判据：`src/prompts.rs` 与 `src/order/` 之间两个方向都不再出现 `crate::` 互引。

## 相邻一条 order → audit（报告·偏移 3）

不是环，是单向越层：`src/order/execute.rs:47`、`:48`、`:136`、`:137` 调 `crate::audit::{items_of, run}`，与「聚合不得依赖服务」冲突。拆法见重构方案·待确认的决策二。

## 随站一改的文档（报告·偏移 4）

`CONTRIBUTING.md`、`docs/dev-guide/index.md`、`docs/dev-guide/workspace.md`、`STATUS.md` 仍写已不存在的 `locate/`，改成 `workspace` 容器事实。

## 顺序

先拆站一，把 `order` 的依赖面去掉 `prompts`；相邻一条与站一同批；文档随站一改。清理项（两处 `short`、事件负载、失效注释、`criterion` 名与类、契约测试清单）见重构方案·模块拆分五至七，不单列站。
