# qtcloud-work CLI 重构路线图

把 `data/report/evaluation/qtcloud-work-cli.md`（意图审查）与 `data/report/plan/qtcloud-work-cli.md`（重构方案）点出的依赖环，按站排成路线。每站行为不变：命令面、`--json` 四字段、报错文字都不改；做完跑 `cargo fmt --check`、`clippy --all-targets --locked -- -D warnings`、`cargo test --locked` 三绿。

## 环一 order ↔ workspace

现状：`order` 引 `crate::workspace::LocalWorkspace`（`src/order/mod.rs:29`），`workspace` 引 `crate::order::WorkOrder`（`src/workspace/local.rs:23`）。`5aeaf54` 把装载搬进 `locate/`，专为消这条环；`4457a2e`、`79d0204` 又把装载并回 `workspace/`。

与意图的关系：意图要求「装载独立、聚合之间不转圈」；`CONTRIBUTING.md`、`docs/dev-guide`、`STATUS.md` 仍按 `locate/` 叙述，与此冲突。

拆法：二选一，见重构方案·待确认的决策一。留 `workspace/` 就把 `local` 明文标为「装载、不属于聚合，与 `order` 是互认」并改文档；立回独立层就恢复「`workspace` 不引 `order`」。命名归用户。

完成判据：两个目录之间只剩一个方向，或规范明文承认互认；上述三份文档里不再出现 `locate/`。

## 环二 order ↔ prompts

现状：`order/ai.rs` 引 `crate::prompts` 的 `Facts` / `prompt_for` / `judge_prompt` / `previous_records`（`:5`、`:12`、`:43`、`:157`）；`src/prompts.rs:94` 又引 `crate::order::WorkRecord`。

与意图的关系：意图要求「聚合件不与适配件同层」。

拆法：把 `previous_records` 搬进 `order/ai.rs`，`prompts` 只收纯数据 `Facts` 与 `&[Criterion]`，见重构方案·模块拆分四与待确认的决策三。

完成判据：`prompts.rs` 里不再出现 `crate::order`。

## 相邻一条 order → audit

不是环，是单向越层：`src/order/execute.rs:47`、`:48`、`:136`、`:137` 调 `crate::audit::items_of` 与 `crate::audit::run`，与「聚合不得依赖服务」冲突。与环一批处理，拆法见重构方案·待确认的决策二。

## 顺序

先定环一的落位与环二的单向，再动代码；两环拆完，`order` 的依赖面只剩 `criterion`、`workflow` 与装载。相邻一条随环二一起改。清理项（两处 `short`、事件负载、失效注释、`criterion` 名与类、契约测试清单）见重构方案·模块拆分五至七，不单列站。
