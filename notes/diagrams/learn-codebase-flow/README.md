# learn-codebase-flow diagrams

[D2](https://d2lang.com/) sources for [learn-codebase-flow.md](../../learn-codebase-flow.md) and [learn-codebase-flow_cn.md](../../learn-codebase-flow_cn.md).

| File | Used in |
| ---- | ------- |
| `lab-workflow.d2` / `.svg` | §1 Lab workflow (EN) |
| `lab-workflow_cn.d2` / `.svg` | §1 Lab 工作流 (中文) |
| `codex-runtime.d2` / `.svg` | §2 Codex runtime (EN) |
| `codex-runtime_cn.d2` / `.svg` | §2 Codex 运行时 (中文) |
| `chains-connect.d2` / `.svg` | §3 How chains connect (EN) |
| `chains-connect_cn.d2` / `.svg` | §3 两条链怎么接上 (中文) |

## Render

```bash
cd notes/diagrams/learn-codebase-flow
for f in *.d2; do d2 --layout dagre "$f" "${f%.d2}.svg"; done
```

Requires [D2](https://d2lang.com/) (`brew install d2` on macOS).

Commit both `.d2` and `.svg` so GitHub renders without local tooling.