# AGENTS.md

## 跨项目通用规则

- 质量门与 bindings（**按场景二选一，禁止两个连跑**）：
  - **日常改完（代码）** → 只跑 `pnpm quick`（只读校验：format:check → lint → lint:e2e-locators → typecheck → check:rs → test:compile）；此时不跑 `pnpm check`。
  - **收尾/提交前（代码）** → 只跑 `pnpm check`（format → build:rs → gen:bindings → lint → lint:e2e-locators → typecheck → test → build）；它已覆盖 quick 的全部检查项（`format` 代替 `format:check`），此时再跑 `pnpm quick` 属于重复执行，不做。
  - **禁止同时执行两者**：不得出现 `pnpm quick && pnpm check`、先 quick 再紧接 check 这类连续调用；需要只读预检就先单独跑完 quick，收尾阶段再单独跑 check，两步分开、各自独立汇报结果。
  - `check` 已含 `gen:bindings`，成功后 `src/bindings.ts` 必然最新，**不必再单独跑**；只有「改了 Rust 命令签名/公共 API、且本阶段要跑 `pnpm quick`」时，才先 `pnpm build:rs && pnpm gen:bindings`（quick 不含 gen，且 gen 依赖最新 debug 二进制；`test:compile` 会暴露测试代码里的旧引用）。
  - **仅文档/配置改动**（如 `*.md`，不涉及 `src/`、`src-tauri/`、`e2e/`）不强制跑质量门，但仍必须输出提交信息。
- **提交信息（有改动就输出）**：只要本轮有文件修改（含文档），结束时必须以代码块的形式输出 git commit 信息，格式遵守 `D:/py-code/pbp/.rules/git提交信息规范.md`，并换行写「输出结束」；不以「跑过 check」为前提，也不因「只是文档」省略。若本轮跑过质量门且失败，先修复再输出，不得在失败结果上输出提交信息。
