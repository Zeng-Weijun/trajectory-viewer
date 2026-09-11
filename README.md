# Agent Trajectory Viewer

两份轨迹语料、两种读法，GitHub Pages 直接提供服务。
没有构建步骤，没有依赖 —— 仓库里提交的就是线上跑的。

<https://zeng-weijun.github.io/trajectory-viewer/>

| 语料 | 条数 | 事件 | 来源 |
|---|---|---|---|
| Feedback | 45（3 条内联 + 42 条快照懒加载） | 22,076 | EmpiriaAI/EmpiriaAI.github.io |
| SWE 环境 rollout | 83（47 个任务） | 3,178 | `minions_v148_pack_20260911`，codex:glm-5.3-flash |

```
index.html                         落地页
trajectory-dialogue.html           双栏对话视图（+ .css / .js）
trajectory-explorer.html           线性事件流（+ .js，样式在 style.css）
  ?category=swe                    直接打开 SWE 语料
swe-fork.css                       SWE 专用样式（capture gap 事件）
fonts.css  fonts/                  自托管的三个字族
trajectory-data.js                 Feedback：3 条内联轨迹
feedback-snapshot-index.js         Feedback：42 条快照索引（events 懒加载）
data/feedback-snapshots/           Feedback：42 个快照全文
swe-trajectory-data.js             SWE：83 条 rollout 全文（4.8 MB）
schema/swe-trajectory.schema.json  SWE 字段接口，机器可读
SCHEMA.md                          SWE 字段接口：哪些填了、哪些是预留位
FIELD_COVERAGE.md                  SWE 数据从 pack 出来时丢了什么
tools/export_explorer_data.py      pack → 上面的 schema，可重跑
```

两个阅读器读同一批全局变量，按顺序拼接：
`EMPIRIA_RAW_TRAJECTORIES` + `EMPIRIA_SWE_TRAJECTORIES` + `EMPIRIA_FEEDBACK_SNAPSHOTS`。
每条轨迹带 `trajectoryClass`（`feedback` / `swe`），顶栏据此分类切换。

## 双栏对话视图

按**方向**分栏，不是按发言人：

- **左栏** —— 一切喂给模型的：系统提示、注入上下文、真人指令、工具结果
- **右栏** —— 一切模型产出的：推理、助手输出、工具调用

Feedback 语料这个切法是 48.5% / 51.5%，SWE 语料是 46.8% / 53.2%，两栏都不闲着。

布局单位是**回合**而不是事件：一次模型回合是一个块，推理和正文在上，
下面每个工具调用一条子行，答复它的结果就在轴线正对面。一次发出六个
并行调用时是六条对齐子行，而不是六个块 —— Feedback 语料里 14.3% 的调用
属于这种并行批，最宽六个，拆开会把并行报告成串行。

深链 `?run=<shortId>`；加 `&embed=1` 会隐掉自己的顶栏，供线性视图 iframe 嵌入
（线性视图的「Two-lane reader」按钮走的就是这条路）。

### Feedback 数据里两个需要注意的地方

导出的 `user` 和 `system` 两个流名不副实：

- `user` 事件里只有 **69.4%** 是真人在打字，其余是中断标记、后台任务
  回调和斜杠命令展开 —— 只有真人那部分算作「轮次」。
- `system` 事件里只有 **1.7%** 是真正的系统提示，另外 98.3% 是每轮重新
  注入的样板（一个 token 计数器出现了 218 次）。这类事件归入「每轮注入」，
  默认关闭，内容相同的自动折叠成 `×N`。

### SWE 数据里需要注意的地方

**1039 条命令里只有 395 条（38%）在整个 pack 里能找到 stdout。** 这不是导出丢的，
是采集时就丢了：codex transcript 被截到最后 ~59 KB，而 `command_result` 事件
只存命令不存输出。缺输出的调用用 `status: "missing"` 显式标出来，**不算失败**
（`error` / `timeout` / `rejected` 才算），所以「只看失败」的计数是真实的。
完整的字段账见 `FIELD_COVERAGE.md`。

### 其它

- 明暗双主题，默认跟随系统；行距可切紧凑/舒适
- 失败信号（timeout / error / rejected）可筛选，`n` / `N` 在失败之间跳
- 右缘密度条：红=错误、琥珀=超时、紫=拒绝、朱红=用户中断
- 运行来源侧栏：会话/请求 ID、Runtime、服务层级、Token 账、质量流水线
- AI 助手侧栏：OpenAI 兼容端点可配，三级上下文压缩（骨架 / 带摘要 / 仅选中），
  API Key 只存浏览器 `localStorage`；不填 Key 时只组装请求体供后端转发
- 标注侧栏：给任意一步打标签写备注，导出 JSONL

## 线性事件流

按时间顺序铺开的完整消息流，保留原始事件粒度。SWE 语料额外有一个
**Provenance & gate evidence** 面板：commit 来源（作者/父提交/issue）、
任务构造（shape / 题面来源 / 对 agent 隐藏了哪些测试 / 镜像 digest）、
两臂 gate（空 patch 与 gold patch 各自的 rc、reward、原始 pytest 尾巴）、
suite-flip 证据、gold patch 与隐藏测试、运行结果、pack 与预算。

## SWE 字段接口

`SCHEMA.md` + `schema/swe-trajectory.schema.json` 定义 `swe-trajectory/1.0`。
**所有字段可选，`null` 渲染成「这行不显示」而不是留个洞** —— 所以生产方
随时开始填某个预留字段，前端零改动就会出现。分三档：

- **A** 现在就填、现在就被读
- **B** 这次从 pack 文件里补上的（两臂 gate、gold patch、hidden tests、挖矿 provenance…）
- **C** 接口写好了但还没人产出

Tier C 按可视化价值排序：每步 token → 每条命令完整 stdout → 结构化 rollout →
agent 最终 diff → 每个测试的结果 → relay 信封 → 采样参数 → 每个事件的时间戳 →
每次调用的文件改动 → setup 耗时。

每条 SWE 轨迹还带 `capture{available:{…}}`，声明它的生产方到底记了什么，
让阅读器能区分「这次 run 没有推理」和「这个生产方不记录推理」。

## 本地预览

```bash
python3 -m http.server 8000
```

## 来源

阅读器与 Feedback 语料源自 <https://github.com/EmpiriaAI/EmpiriaAI.github.io>；
改动与归属见 `NOTICE.md`。SWE 语料来自 `minions_v148_pack_20260911`。
