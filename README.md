# a2a-context-audit

**Audit A2A context/task correlation from a recorded stream. Dumb math, honest numbers, zero dependencies.**

[![Tests](https://img.shields.io/badge/tests-15%20passed-brightgreen)]() [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)]()

Sixth in the a2a family: [a2a-tax](https://github.com/ZidoCode/a2a-tax) (polling), [a2a-stream-tax](https://github.com/ZidoCode/a2a-stream-tax) (streaming), [a2a-reconnect-tax](https://github.com/ZidoCode/a2a-reconnect-tax) (reconnects), [a2a-artifact-audit](https://github.com/ZidoCode/a2a-artifact-audit) (artifacts), [a2a-task-state-audit](https://github.com/ZidoCode/a2a-task-state-audit) (lifecycle), and now the context/task correlation itself.

The A2A spec leaves `contextId` semantics loose — and clients and servers get it wrong constantly:

- [A2A#1309](https://github.com/a2aproject/A2A/issues/1309) — `contextId` behavior when sent with a `taskId` is ambiguous
- [A2A#2061](https://github.com/a2aproject/A2A/issues/2061) — multi-turn conversation semantics still open
- [A2A discussion #762](https://github.com/a2aproject/A2A/discussions/762) — contextId vs taskId confusion
- [a2a-java#770](https://github.com/a2aproject/a2a-java/issues/770) — client sends messages with empty task + context IDs

a2a-context-audit replays a recorded `message/stream` session (JSONL) and reports every correlation violation.

## Install

```bash
pip install a2a-context-audit   # or: copy the .py file
```

## Usage

```bash
$ a2a-context-audit report stream.jsonl
a2a-context-audit report — 4 task(s), 5 context(s), 5 finding(s)

  [CRITICAL] line 4: task 't2' changed context 'c2' -> 'c2x' (task/context is immutable)
  [CRITICAL] line 6: empty contextId (a2a-java#770 bug class)
  [INFO    ] context 'c3' has 1 event(s) but never produced a task (chat-only or broken correlation)
  [WARNING ] task 't4' never carried a contextId — multi-turn resume will not work (A2A#1309)

  verdict: 2 CRITICAL — context/task correlation is broken; multi-turn and resume will silently misbehave.
$ a2a-context-audit audit stream.jsonl && echo ok   # CI gate: exit 2 on CRITICAL
```

## What it catches

| Finding | Severity | The bug class |
|---|---|---|
| Context mutation mid-task | **CRITICAL** | task/context pair is immutable — clients lose the thread |
| Empty taskId / contextId | **CRITICAL** | [a2a-java#770](https://github.com/a2aproject/a2a-java/issues/770) bug class |
| Task without contextId | WARNING | multi-turn resume will not work ([A2A#1309](https://github.com/a2aproject/A2A/issues/1309)) |
| JSON-RPC request id reused | WARNING | replies become ambiguous |
| Orphan context | INFO | events but never a task — chat-only or broken correlation |

`report` is informational (always exit 0). `audit` is the CI gate: exit 2 on CRITICAL. Deterministic — same stream, same findings, byte for byte. MIT license.
