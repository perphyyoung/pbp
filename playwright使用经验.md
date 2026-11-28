# playwright 使用经验

只用于记录不常见的用法。

## 时间类测试

- `page.clock` 是官方时间 mock API（1.45+）：`setFixedTime(t)` 只固定 `Date` / `new Date()`、定时器照常运行，且不需要先 `install()`；要定时器跟随时间跳跃才用 `install()` + `fastForward()`，install 必须发生在页面时钟首次使用前，否则行为未定义。
- `page.clock` 只 mock 页面（渲染进程）上下文的 `Date` / `setTimeout` / `performance` 等，对 Electron 主进程、Node 侧、SQLite 里已写入的时间戳无效。
- 推荐分工：窗口/相对展示的判定逻辑（用 `Date.now()` 算阈值）交给 `setFixedTime` 时间旅行；数据库里的历史时间戳不要靠真实时间流逝造，用 seed（SQL 直插/直改）构造，UI 只断言展示与过滤。
- 死结预警：一旦 clock 快进到未来，主进程后续写入的真实时间戳永远「落后于」虚拟 now——依赖「刚发生的事件落进窗口」的场景（如「最近恢复」）不能用快进验证，必须 seed 历史 first_seen + 真实写入新 status_changed。
- 单测优先：被测函数接受注入时间参数（如 `reconcile(discovered, scanStartedAt)`）时，时间流逝的模拟放在单测里（T0 → T1 多轮调用），e2e 不再重复时间旅行。
