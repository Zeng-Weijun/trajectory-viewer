# Attribution

## 阅读器

`style.css`、`fonts.css`、`script.js`、`fonts/*.woff2`、`digital-environments.html`
与 `trajectory-explorer.{html,js}` 的上游是 Empiria Labs 的公开站点
<https://empiriaai.github.io/>。字族是 Space Grotesk / Inter Tight /
JetBrains Mono（SIL OFL）。上游站点未声明 license；这些资源在此 vendored
用于阅读本仓库的语料。

`trajectory-dialogue.{html,css,js}` 是本仓库自己的双栏阅读器，在同一套
设计语言上重写。

## 对 `trajectory-explorer.js` 的改动

1. 三份语料拼接：`EMPIRIA_RAW_TRAJECTORIES` + `EMPIRIA_SWE_TRAJECTORIES` +
   `EMPIRIA_FEEDBACK_SNAPSHOTS`；
2. `?category=` 决定打开哪个 tab，默认仍是 `feedback`；
3. 空的侧栏分组不渲染；
4. 新增 `renderSweProvenance()` —— SWE 轨迹复用 pipeline 面板展示挖矿 provenance，
   面板标题与八个小节标题按语料切换；
5. 上游的 SWE 任务/判定面板**写死了 django demo 值**（`'2019-03-24'`、
   `'django/template/engine.py'`、`'test_autoescape_off'`、`Patch exists: 'true'`、
   `Exception: 'None'`），已全部换成真实字段；
6. SWE 轨迹的副标题显示 `repo · task · shape`，不再铺整段题面；
7. 初始选中当前语料里事件最多的那条成功轨迹，而不是第 0 条。

## 对 `trajectory-dialogue.{html,js}` 的改动

1. 同样拼接 SWE 语料；
2. `?embed=1` 隐掉自己的顶栏供 iframe 嵌入，Escape 向父页面
   `postMessage({type:'empiria-exit-timeline-fullscreen'})`。

## SWE 语料

`swe-trajectory-data.js`、`swe-fork.css`、`schema/`、`SCHEMA.md`、
`FIELD_COVERAGE.md`、`tools/export_explorer_data.py` 为本项目原创，
由 `tools/export_explorer_data.py` 从 `minions_v148_pack_20260911` 生成。
