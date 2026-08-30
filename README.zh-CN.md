# vibe-coding-kit

> 给 AI 辅助编程准备的最小起手套件 —— `AGENTS.md` + 流程 Skills + 知识库，从真实项目的踩坑里蒸馏出来。

6 个流程 Skills、1 份协作契约、1 套入门级知识库（只含模板，不带示例条目）。总共约 15 个 Markdown 文件。设计目标：整包复制到你的项目里，再用你自己的真实踩坑把内容长出来。

## 为什么会有这个套件

AI 编程 agent 的默认行为是：

- 每个页面发明新的色值 / 圆角 / 阴影
- 重复写局部组件，而不是先搜项目里有没有现成的
- 小任务也走完整流程（改一句文案 → brainstorm → spec → plan → subagent → test → review）
- 把实现历史写进用户能看到的文案里（「我们已重构 / 新版 / 更智能…」）
- 信模拟器，但模拟器在很多真机行为上说谎

这些坑每一个都花小时级别的成本。它们会反复出现，更好的模型也救不回来。

这个套件是一组 agent 在写代码前会读的 Markdown 文件，外加一个 agent 自己在过程中可以长出来的结构。

## 内容

```text
vibe-coding-kit/
├── AGENTS.md                          ← 协作契约
├── CLAUDE.md                          ← Claude Code 适配器
├── README.md                          ← English version
├── README.zh-CN.md                    ← 本文件
├── LICENSE                            ← MIT
├── .gitignore
├── skills/
│   ├── prototype-to-taro/             ← HTML 原型 → Taro + React 页面
│   ├── refactoring-ui/                ← 视觉层级与设计打磨
│   ├── wechat-tap-gesture/            ← 微信手势 API 的正确流程
│   ├── reusable-component-detect/     ← 何时停下来抽组件
│   ├── product-copy-rewriter/         ← 把实现历史从 UI 文案里拿掉
│   └── task-size-router/              ← 小 / 中 / 大任务的流程选择
└── .knowledge/
    ├── README.md                      ← 何时该新增一条
    └── _templates/                    ← 5 个模板：pattern / anti-pattern / platform-note / decision / ai-coding-note
```

## 三层模型

| 层 | 它回答的问题 | 位置 |
|---|---|---|
| **Rule（规则）** | 什么是永远必须为真的？ | `AGENTS.md` |
| **Skill（流程）** | 这个流程怎么做？ | `skills/<name>/SKILL.md` |
| **Knowledge（事实）** | 我们学到了哪些这个项目特有的事实？ | `.knowledge/<category>/` |

Rule = 不变量。Skill = 操作步骤。Knowledge = 记录。

判断标准：

- 想让 agent 每次都遵守 → Rule。
- 想让 agent 在做某件具体事时才走 → Skill。
- 是一个关于世界（或本仓库）的客观事实，下次写代码时能少踩坑 → Knowledge。

## 安装

```sh
npx skills add <owner>/vibe-coding-kit
```

或者手动复制到你的项目：

```sh
# 从本仓库根目录执行
cp AGENTS.md CLAUDE.md /path/to/your-project/
cp -r skills /path/to/your-project/.claude/skills
cp -r .knowledge /path/to/your-project/.knowledge
```

然后：

1. 改 `AGENTS.md` §Repository map，指向你项目里实际的目录和命令。
2. 如果项目里有 `docs/DESIGN-SYSTEM.md`，更新 `AGENTS.md` §Source of truth。

## 起步

1. 把 `AGENTS.md` 和 `CLAUDE.md` 复制到项目根。
2. 把想用的 skills 复制到项目里的 `.claude/skills/<name>/`。
3. 把 `.knowledge/`（含 `_templates/`）复制到项目根。
4. 每次诊断出一个 bug、验证出一个模式时，新增一条 —— `.knowledge/README.md` 里有判定标准。

## 来源

这套文件蒸馏自真实项目里花了几小时才诊断出来的 bug（Taro + React 微信小程序、Web 应用等）。规则、流程和模板是踩坑出来的产物，不是理论最佳实践。`.knowledge/` 只发模板 —— 因为项目特有事实只有项目自己踩到了才有意义。

## 这个套件不是什么

- **不是框架。** Taro、React、Vue、Svelte、原生 HTML 都适用。
- **不是设计系统。** 你自己的 `docs/DESIGN-SYSTEM.md` 自己带。`prototype-to-taro` 是唯一一个有框架依赖的 skill，其余都和框架无关。
- **不绑定具体 LLM。** 同一份内容 Claude Code、OpenCode、Codex、其他 coding agent 都适用。
- **不重。** 没有默认的 `brainstorm → spec → plan → subagent → review` 流程。默认是 `inspect → act → verify → report`，按任务规模缩放。

## 许可

MIT。