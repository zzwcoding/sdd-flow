# CI 流水线骨架

阶段 6 的产物。原则：**CI 先于业务代码存在**；门禁最小但硬性——lint + 类型 + 测试，全绿才合并。

## 最小骨架（GitHub Actions，按栈替换命令）

```yaml
# .github/workflows/ci.yml
name: ci

on:
  push:
    branches: [main]
  pull_request:

jobs:
  gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # --- spec gate（语言无关，先于一切；specs/ 不存在时自动跳过）---
      - name: spec gate
        run: python3 tools/check_specs.py

      # === 按技术栈替换以下三块 ===

      # --- Node/TypeScript ---
      # - uses: actions/setup-node@v4
      #   with: { node-version: 22, cache: pnpm }
      # - run: pnpm install --frozen-lockfile
      # - run: pnpm lint
      # - run: pnpm typecheck
      # - run: pnpm test

      # --- Python ---
      # - uses: actions/setup-python@v5
      #   with: { python-version: "3.12", cache: pip }
      # - run: pip install -r requirements.txt
      # - run: ruff check .
      # - run: mypy .
      # - run: pytest

      - name: TODO replace me
        run: exit 1
```

## 分支保护（在仓库设置里开，一次性）

- main 分支禁止直接 push，只走 PR
- PR 必须 CI 绿才能合并
- （可选）PR 至少 1 个 review——人和 AI 结对时可以是用户本人点

## Spec 机器校验（spec gate）

`tools/check_specs.py`（来源：sdd-flow `references/spec-validator.py`，阶段 0 就位）在 lint/test 之前跑，把 sdd-flow 的文档约定变成机器门禁：模块依赖无环、spec 触及的模块必须在 modules.md 声明、每条验收标准绑定测试标识、spec 标 `已完成` 时测试必须真的存在于源码。格式契约见 `module-spec-format.md`「机器可校验约定」。

- 脚本零依赖，python3 直接跑；`specs/` 不存在时自动跳过，所以阶段 0 搭 CI 时就可以接上，不会被挡。
- 仓库没有 python3 时，按同一约定移植成 node 脚本即可，规则不变。

## 模块边界检查（有就加，强烈建议）

防止 AI 跨模块顺手 import 内部文件，把"只能走公开接口"变成机器强制：

- **Node/TS**：`dependency-cruiser`——规则示例：禁止 `src/modules/a/**` 引入 `src/modules/b/**` 除 `b/index.ts` 以外的任何文件。
- **Python**：`import-linter`——在 pyproject 里声明分层/模块契约，CI 里跑 `lint-imports`。

边界规则从 `specs/modules.md` 生成；模块划分变了就同步改规则。

## 反馈速度纪律

CI 是 AI 的反馈回路，慢了就没人（没 AI）愿意等：

- 目标：全流程 < 5 分钟。超了先拆慢的测试到 nightly，保主干快。
- 测试在模块 seam 处写，天然可并行；不要为覆盖率数字引入慢测试。

## CD（需要时再加）

 trunk 绿了之后：构建产物 → 打 tag/镜像 → 部署到 staging。一开始手动触发即可，不要第一天就全自动上生产。
