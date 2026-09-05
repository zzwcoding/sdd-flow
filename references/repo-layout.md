# 仓库目录约定

目标：文件系统结构 = 模块结构。AI 进入仓库时，目录树就是模块地图（渐进式披露：先看接口，需要时再深入实现）。

## 顶层布局

```
<repo>/
├── CONTEXT.md              # 共享术语表（阶段 1 产物，持续更新）
├── AGENTS.md               # 仓库规约：指向本流程与关键文档
├── specs/
│   ├── modules.md          # 模块划分图：每模块一张接口卡（阶段 2 产物）
│   └── <功能名>.md         # 功能 spec（阶段 3 产物）
├── docs/
│   ├── adr/                # 架构决策记录：NNNN-<标题>.md（含架构图过审记录）
│   ├── agents/             # setup-matt-pocock-skills 的配置输出
│   ├── prd.md              # 产品需求文档：顶部 状态: 粗稿|逼问中|已定稿；阶段 1 定稿，决策回写
│   └── architecture-v*.html # 架构图（archify 产物，阶段 2 过审物）
├── lessons/                # 编码窗口的阶段讲解（learn-by-rebuild 纪律，NNNN-阶段名.md）
├── tools/
│   └── check_specs.py      # spec gate 校验脚本（来源 sdd-flow references/spec-validator.py）
├── src/
│   └── modules/
│       └── <模块名>/        # 一个模块一个目录
│           ├── index.*     # 公开接口：唯一允许外部 import 的文件
│           ├── README.md   # 一句话职责 + 使用示例（接口文档的一部分）
│           ├── ...内部实现文件（外部禁止 import）
│           └── *.test.*    # seam 处测试：只测 index 暴露的行为
├── .github/workflows/ci.yml
└── .scratch/               # 本地 markdown issue tracker（无 GitHub 时的 fallback）
```

## 规则

1. **接口文件唯一入口**：`src/modules/a` 只能 import `src/modules/b/index.*`。违反 = CI 边界检查打回。
2. **文件系统即文档**：新模块 = 新目录 + 接口卡进 `specs/modules.md`，两者同时发生；骨架（空目录 + 占位 `index.*`）在阶段 2 过审后立即建齐，不等第一张票。
3. **测试与模块同住**：测试放模块目录内，只通过公开接口调用。跨模块的集成测试放 `tests/`。
4. **命名用 CONTEXT.md 术语**：模块名、文件名、接口名都必须能在术语表里找到。
5. **README 极简**：模块 README 只写"职责一句话 + 怎么调用"，实现细节归代码注释。

## 模块组合（多层结构）

小模块组合成大功能时，包装层本身也是一个模块：

```
src/modules/
├── report/            # 深模块：报表
│   ├── index.ts       # 公开接口：generateReport(input) -> Report
│   └── ...
└── billing/           # 组合模块：计费（内部用 report）
    ├── index.ts       # 外部只看到这个接口，看不到它用了 report
    └── ...
```

外部只关心包装层的输入输出；包装层内部用了哪些模块是实现细节。判断标准不变：包装层的接口也必须明显比它隐藏的东西简单。

## 规模提醒

模块目录超过 ~10 个时，按聚合再分一层目录（如 `src/modules/billing/{invoice,payment,report}/`），并在 `specs/modules.md` 里用同样的层级组织接口卡。
