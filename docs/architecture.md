# Architecture

## Loop

```
host agent (Claude Code / Codex)
   │  task / sub-goal
   ▼
deskjudge ── L0 replay? ──hit──► agent-desktop act ──► snapshot verify ──► done / next
   │ miss                            ▲
   ▼                                 │ element id
L1 decision model ──confident───────┘
   │ abstain / low confidence / repeated failure
   ▼
L2 host LLM decides (and the result is distilled back into L0/L1 context)
```

## Element filtering (why L1 is cheap)

The raw accessibility snapshot is never sent wholesale. Before L1 sees it:

- interactive elements only (buttons, inputs, menu items, links);
- hard cap on element count, prioritised by proximity to the current goal/focus;
- diff-only against the previous snapshot when the screen barely changed;
- each element carries a stable numeric id; the model returns **id + confidence**, never coordinates.

## Verification

After every act, deskjudge re-snapshots and checks the expected state change (element appeared/disappeared, text changed, focus moved). A failed check is a first-class event: retry once at the same tier, then escalate. This snapshot-after-act re-read is the system's cheap, reliable verifier — it is what makes trusting a small decision model safe.

## Safety

- Action classes: `auto` (navigation, typing into focused fields), `preview` (sending messages, changing settings), `confirm` (payments, privilege escalation, file deletion, anything irreversible) — `confirm` always stops for a human.
- Per-app blocklist (password managers, banking by default).
- Read-only mode: snapshot and judge, never act.
- All snapshot data is processed locally or sent only to the provider the user configured with their own key. No telemetry of screen content.

## Providers (L1)

Interface: `decide(elements, goal) -> {element_id, confidence} | abstain`.

- Jev (TypeSafe API) — structured decisions with confidence and abstention.
- Bivia (Articul8 API) — drop-in equivalent, same price class.
- Ollama / local — any small local model wrapped to the same contract; confidence may be self-reported and is trusted less (lower escalation threshold).

## Traces

Each step appends a JSONL record: timestamp, tier used, elements sent, decision, confidence, latency, token/cost estimate, verify result. Traces power failure replay and the public benchmark — no hand-written performance claims.
