# RULES

跨项目通用规则。

## e2e 定位器必须语义化且精确

- **语义化**：只用 `getByRole` / `getByLabel` / `getByText` / `getByTitle` / `getByPlaceholder` / `getByAltText`，不用 CSS 选择器定位本可语义化的元素。`.first()` / `.nth(i)` 只允许用在「顺序本身有含义」处（层叠弹窗的 DOM 顺序、同文案堆叠的 toast），并在用例内注明依据。
- **先限定作用域**：先 `getByRole("dialog", { name })` 收窄到目标弹窗，再在内部定位；同名元素靠作用域区分，不靠猜顺序。
- **名称一律精确**：`getByRole(role, { name, exact: true })`，`getByText` / `getByTitle` / `getByLabel` / `getByAltText` / `getByPlaceholder` 一律 `{ exact: true }`。Playwright 的名称匹配默认是**子串**，会互相吞并：`收藏` ⊂ `取消收藏`、`关闭` ⊂ `关闭查找`、`编辑` ⊂ `取消编辑`、`提示词内容` ⊂ `以提示词内容搜索相似项`。子串匹配既会**误红**（strict mode 命中多个），也会**误绿**（断言 A 却命中 B），后者更难发现。
- **确需子串 / 动态文案**：用锚定正则（如 `^关联图像（\d+）$`），并在该行注释理由；不要用默认的模糊匹配碰运气。
- **状态断言用属性，不用文案互斥**：切换型按钮给固定 `aria-label` + `aria-pressed`；开关类断 `checkbox` 的 `checked`；不要「断言 A 文案消失、B 文案出现」（文案互为子串时会假通过）。
- **命名侧约束（治本）**：新增 `aria-label` / `title` 前先查它与既有名称的子串关系；同一弹窗内的可访问名不得互为子串。
- **强制手段**：静态脚本扫描 e2e 目录，缺 `exact` 即失败并挂进日常质量门（paim 实现：`scripts/check-e2e-locators.mjs` → `pnpm lint:e2e-locators`，已进 `pnpm quick` / `pnpm check`）。脚本提供 `--fix` 自动补、`--all` 查看页面级历史欠账；确需子串的调用在该行注释 `locator-allow-substring` 豁免。
