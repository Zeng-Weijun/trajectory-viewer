# Agent Trajectory Viewer

同一份 agent 轨迹语料的两个静态阅读器，由 GitHub Pages 直接提供服务。
没有构建步骤，没有依赖 —— 仓库里提交的就是线上跑的。

```
index.html                    落地页
trajectory-dialogue.html      双栏对话视图（+ .css / .js）
trajectory-explorer.html      线性事件流（+ .js，样式在 style.css）
fonts.css  fonts/             自托管的三个字族
trajectory-data.js            3 条内联轨迹
feedback-snapshot-index.js    42 条快照的索引（events 懒加载）
data/feedback-snapshots/      42 个快照全文
```

## 双栏对话视图

按**方向**分栏，不是按发言人：

- **左栏** —— 一切喂给模型的：系统提示、注入上下文、真人指令、工具结果
- **右栏** —— 一切模型产出的：推理、助手输出、工具调用

全语料这个切法是 48.5% / 51.5%，两栏都不闲着。

布局单位是**回合**而不是事件：一次模型回合是一个块，推理和正文在上，
下面每个工具调用一条子行，答复它的结果就在轴线正对面。一次发出六个
并行调用时是六条对齐子行，而不是六个块 —— 语料里 14.3% 的调用属于这
种并行批，最宽六个，拆开会把并行报告成串行。

### 数据里两个需要注意的地方

导出的 `user` 和 `system` 两个流名不副实：

- `user` 事件里只有 **69.4%** 是真人在打字，其余是中断标记、后台任务
  回调和斜杠命令展开 —— 只有真人那部分算作「轮次」。
- `system` 事件里只有 **1.7%** 是真正的系统提示，另外 98.3% 是每轮重新
  注入的样板（一个 token 计数器出现了 218 次）。这类事件归入「每轮注入」，
  默认关闭，内容相同的自动折叠成 `×N`。

### 其它

- 明暗双主题，默认跟随系统；行距可切紧凑/舒适
- 失败信号（timeout / error / rejected）可筛选，`n` / `N` 在失败之间跳
- 右缘密度条：红=错误、琥珀=超时、紫=拒绝、朱红=用户中断
- 运行来源侧栏：会话/请求 ID、Runtime、服务层级、Token 账、质量流水线
- AI 助手侧栏：OpenAI 兼容端点可配，三级上下文压缩（骨架 / 带摘要 / 仅选中），
  API Key 只存浏览器 `localStorage`；不填 Key 时只组装请求体供后端转发
- 标注侧栏：给任意一步打标签写备注，导出 JSONL

## 本地预览

```bash
python3 -m http.server 8000
```

## 来源

数据与线性视图来自 <https://github.com/EmpiriaAI/EmpiriaAI.github.io>。
