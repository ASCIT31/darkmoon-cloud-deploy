# AI modes — Connected, Private, Local

The AI backend is chosen **per node** at install time. It decides which outbound flow
applies (see [Network](network.md)) and whether a GPU is ever useful.

## Connected

The core calls a **hosted LLM provider** over TCP/443, **always through the Privacy
Gateway**. Simplest to run; requires outbound egress to the provider.

```bash
--provider anthropic --model claude-opus-4-6 --api-key sk-ant-...
```

> **Connected is data-minimized, not offline.** The Privacy Gateway strips and tokenizes
> sensitive data before it reaches the model, but Connected mode still sends (minimized)
> traffic to the provider. If you need *no* model egress, use Local.

## Private

The core calls a **customer-operated LLM endpoint** (self-hosted or a private tenancy)
on 443 or a custom port. Model traffic stays inside infrastructure you control.

```bash
--anthropic-url https://llm.internal.example.com --anthropic-model claude-opus-4-6 --anthropic-key <KEY>
```

## Local

Inference runs **on the node itself** over loopback (`127.0.0.1:<port>`). There is **no
LLM egress at all** — nothing about the model calls leaves the machine.

```bash
--local --local-engine ollama --local-url http://127.0.0.1:11434 --local-model <model-id>
```

Local mode needs extra memory per model, on top of your base [profile](sizing.md):

| Model size | Extra RAM / VRAM |
| --- | --- |
| 7B | +8 GB |
| 13B | +16 GB |
| 33B | +32 GB |

## GPU

A **GPU is only for Local inference**. Connected and Private modes do their inference off
the node, so they never need one. The DarkMoon engine itself does not require a GPU.

## Two things people conflate

- **Privacy Gateway ≠ offline.** It is data-minimization. Connected mode still talks to
  the provider (minimized). Only Local mode removes LLM egress.
- **GPU ≠ required.** It only helps Local inference. If you are Connected or Private, an
  instance without a GPU is correct.

## Air-gapped

A fully air-gapped deployment (no egress whatsoever, ever) is **not a validated
configuration today**. Local mode removes *LLM* egress, but install/update still needs
Docker Hub, and Pro still needs periodic license validation (cache-tolerant — see
[Doctor & lifecycle](doctor-and-lifecycle.md#license--slots)).
