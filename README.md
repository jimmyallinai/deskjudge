# deskjudge

**A tiered judgment layer for desktop agents — so computer use stops waiting on a frontier LLM for every single click.**

deskjudge sits on top of [agent-desktop](https://github.com/vercel-labs/agent-desktop) (Vercel Labs, Apache-2.0), a fast CLI that reads the accessibility tree and acts on native apps. agent-desktop is the *hand*: snapshots and clicks are quick. The slow, expensive part is the *judgment* in between — today a frontier model (Claude / Codex class) reads the full element list and decides every step, seconds per step.

deskjudge is the missing judgment layer between the host agent and the hand:

| Tier | What decides | Cost | When |
|---|---|---|---|
| **L0 — Rules & replay** | Recorded flows, element-name rules | zero model calls | known, repeated flows |
| **L1 — Decision model** | A decision model (Jev / Bivia / local small model via Ollama) sees a *filtered* element list (interactive-only, capped, diff-only), returns an element id + confidence | ~$0.042 / M input tokens, output free on Jev/Bivia-class APIs | most steps |
| **L2 — Host LLM** | The host agent itself (Claude Code / Codex) | frontier prices | planning, checkpoint verification, L1 abstains or fails repeatedly |

Every action is verified by re-reading the accessibility tree (snapshot-after-act). Low confidence or abstention escalates one tier. Nothing here is claimed fast until the benchmark below says so, with screenshots and numbers.

## Status

🚧 **Design / pre-alpha.** Architecture and benchmark plan are being built in the open. No performance numbers yet — measuring, not marketing.

## What deskjudge is *not*

- Not a replacement for APIs/CLIs. If a task has a real interface, use it; desktop control is the fallback for apps that don't.
- Not a guarantee for every app. If an app exposes nothing to the accessibility tree, deskjudge is blind there (a vision fallback is on the roadmap, not in the MVP).
- Not affiliated with Vercel. agent-desktop is an independent upstream project; deskjudge only calls its CLI.

## Product surface (planned)

- **MCP server + CLI** — hosts like Claude Code / Codex connect in one line; deskjudge is a tool they call, not another app to babysit.
- **Recipes** — a flow that worked once can be recorded, replayed by L0, and shared as a file. Community recipes are the growth loop.
- **Traces** — every step logs the snapshot diff, the decision, latency and cost. Failures are replayable; the benchmark report is generated from these traces.
- **Safety gates** — actions are tiered: safe actions run, sensitive ones preview, payments / privilege escalation / deletions always require human confirmation. Per-app blocklist, read-only mode, snapshots processed locally.

## Benchmark plan (the receipt)

Three standard tasks (Settings navigation, Notepad editing, file-manager operations) run both ways — host-LLM-decides-every-step vs. deskjudge tiers — reporting per-step latency, total task time, and cost, with screenshots. Results land in `docs/benchmarks/` before any speed claim appears in this README.

## Providers

L1 is provider-pluggable and bring-your-own-key: Jev (TypeSafe), Bivia (Articul8), or a local model via Ollama. No provider is hard-wired; a closed API going down or region-locked must never brick the tool.

## License

Apache-2.0 — same license as upstream agent-desktop, so code can flow both ways. See [LICENSE](LICENSE).
