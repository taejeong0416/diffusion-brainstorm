---
name: diffusion-brainstorm
description: A hierarchical, diffusion-style brainstorming method that deliberately scatters candidate ideas (including intentional "noise") across layers, prunes them on two axes (quality AND diversity), and propagates the *reasons for rejection* sideways so discarded ideas can still shape surviving ones — then crystallizes the survivors into concrete proposals. Use this skill whenever the user wants to brainstorm, generate, or explore ideas for research topics, competition/contest entries, product or business concepts, project directions, names, angles, or any open-ended "give me ideas for X" / "help me come up with approaches to X" request — even if they don't say the word "brainstorm." Especially use it when the user wants breadth, unconventional cross-domain ideas, or wants to escape generic/safe suggestions. Do NOT use it for narrow factual lookups or when the user wants a single direct answer.
---

# Diffusion Brainstorm

A brainstorming method inspired by diffusion models: start coarse and noisy, then iteratively denoise toward concrete ideas. Unlike flat "list 20 ideas" brainstorming (which converges on safe, mainstream suggestions) this method deliberately injects noise to widen exploration, then removes it in a way that lets rejected ideas leave a useful trace.

This skill packages mechanisms studied in the LLM reasoning literature (hierarchical decompose-score-prune as in RDoLT; reuse of discarded sibling nodes as in SIGMA / I-MCTS; diffusion-style propose-and-filter as in "Diffuse Thinking") into a single practical brainstorming workflow. It is a *practical packaging*, not a novel algorithm — see README for citations.

## When to apply

Apply when the request is open-ended idea generation: research/contest topics, product or business ideas, project angles, naming, strategy directions. Skip for narrow factual questions or when a single direct answer is wanted.

## The four core principles (what makes this different)

1. **Coarse-to-fine.** Start with deliberately broad, even vague top-level categories. Get the layout right before committing to detail — this avoids locking into the first detail and getting stuck in a local optimum.
2. **Noise injection.** At every layer, deliberately include 2–3 candidates that look "off" — far-domain transplants, deliberately weird angles. Noise widens the search radius; some of it converts into signal during pruning.
3. **Two-axis pruning.** Never prune on "looks good" alone. Score each candidate on (a) quality/validity AND (b) distance from sibling candidates. Without axis (b), the model quietly keeps only safe mainstream ideas and the noise injection is wasted.
4. **Lateral propagation of rejection reasons.** When you cut a candidate, record *why*. That reason becomes an input to sibling/child nodes — a cut "too far-fetched" idea often tells you how to sharpen a neighbor. This is the denoising step: the predicted noise is subtracted to reveal the real signal.

## Workflow

Run the layers below. Default depth is 3 layers; go deeper (Layer 4+) only if the user wants research-grade specificity. Keep the whole thing visible to the user as a running structure — the layers and the prune decisions are the value, not just the final list.

### Layer 1 — Scatter the top-level categories

- Generate **7–9** broad categories, intentionally coarse.
- Mark **2–3** of them explicitly as `[noise]` — far-domain or deliberately odd.
- Cover at least: the obvious/mainstream space, 1–2 adjacent domains, and 1–2 genuinely distant domains (e.g. for a tech topic: borrow from finance, logistics, biology, distributed systems).

### Prune Layer 1 (two axes)

- Score each on quality × diversity.
- Cut the weak ones. For each cut, **write a one-line rejection reason.**
- **Deliberately keep at least one `[noise]` candidate** if it shows any measurable path to becoming real. Converting noise → signal here is the key event of the method; call it out when it happens.
- Pass surviving rejection reasons forward as hints for the next layer.

### Layer 2 — Expand survivors into children (with fresh noise)

- For each surviving category, generate child candidates, again including a `[noise]` child.
- Use the rejection reasons from Layer 1 as inputs. A reason like "too far / not measurable" often tells a sibling how to become measurable.

### Prune Layer 2

- Same two-axis prune. Record rejection reasons.
- **Lateral propagation:** when you cut a child, check whether its rejection reason should sharpen a *sibling* (e.g. "option-pricing is too far" → "but that's really about evaluating conditional future value → push the dual-process sibling that way"). Make these hand-offs explicit.

### Layer 3 — Crystallize

- Among survivors, look for **cross-links** — places where two different branches meet. These intersections are usually the strongest ideas (and the ones flat brainstorming never produces).
- Promote the 3–5 best to concrete proposals. For each, name which branches combined to make it.
- Explicitly flag the two signature outcomes when they occur:
  - **noise → signal**: a candidate that started as `[noise]` became a real proposal.
  - **rejection → revival**: a fully-cut candidate's reason rescued or sharpened a different branch.

### Optional Layer 4 — Research-grade (only if asked)

For each promoted proposal: check prior work (search if tools available), state a concrete hypothesis, and a measurement/eval protocol. This is the step that turns "an interesting angle" into "a topic you could actually submit or research."

## Output format

Show the work as a running structure, not just a final list:

- Use compact tables or short bullet trees for each layer's candidates.
- After each prune, show a short "Cut / Reason / Survives" summary.
- End with a short **"What the method produced"** note: list the noise→signal and rejection→revival events. If neither happened, say so honestly — it means the topic was narrow enough that the noise didn't pay off, and a flatter approach would have done as well.

## Worked example

Topic: **"Research directions for making LLM inference cheaper."** Noise level: high, depth: 3.

### Layer 1 — scatter

| # | Category | Kind |
|---|---|---|
| 1 | Model compression (quantization, distillation) | mainstream |
| 2 | Decoding efficiency (speculative / parallel decoding) | mainstream |
| 3 | Serving systems (batching, KV-cache management) | adjacent |
| 4 | Retrieval to offload parametric memory | adjacent |
| 5 | Hardware-aware kernels | adjacent |
| 6 | **Logistics: tokens as a just-in-time supply chain** | `[noise]` |
| 7 | **Biology: sleep-like memory consolidation** | `[noise]` |
| 8 | **Finance: option pricing on future compute** | `[noise]` |

**Prune L1** (quality × diversity):

- Cut **#5 hardware kernels** — *reason: high quality but overlaps #3 serving; low diversity.*
- Cut **#7 biology** — *reason: too metaphorical to measure* → propagate sideways: "consolidation" really means **caching frequently-recomputed results** → feeds #3.
- **Keep #6 logistics `[noise]`** — measurable path: JIT = "compute only what's needed, when needed" → dynamic depth / early exit.
- **Keep #8 finance `[noise]`** — path: value of continuing computation under uncertainty.
- Survives: #1, #2, #3, #4, #6, #8.

### Layer 2 — expand survivors (fresh noise + L1 hints)

Expand #6 logistics → {early-exit per layer, token-level skipping, **`[noise]` "fast/slow dual-track" routing**}. Expand #8 finance → {adaptive compute budget, confidence-gated halting}.

**Prune L2 — lateral propagation:**

- Cut #8's "option pricing" framing — *reason: too abstract as-is.* But its core — *"the value of continuing"* — is exactly an early-exit halting signal → **sharpens #6 into: early-exit driven by a learned continuation-value head.** (rejection → revival)

### Layer 3 — crystallize (cross-links)

1. **Adaptive-depth decoding** = #6 (early exit) × #8 (value-of-continuing) × #2 (speculative decoding): a per-token head halts computation when expected quality gain < compute cost. *Branches combined: logistics + finance + decoding.*
2. **Distill-plus-retrieve** = #1 × #4: a small distilled model handles the head distribution; retrieval covers the long tail.

### What the method produced

- **noise → signal:** the logistics `[noise]` and finance `[noise]` categories combined into proposal #1 — an angle a flat list rarely reaches.
- **rejection → revival:** cutting finance's "option pricing" wording rescued proposal #1 by reframing it as a continuation-value head; cutting biology's "consolidation" fed a caching insight into the serving branch.

## Honest limitations (state these when relevant)

- Diffusion's denoising follows a learned score function; brainstorming has no such objective ground truth, so the quality of "denoising" depends entirely on the evaluation criteria the user gives. This method is strongest when criteria are clear (constrained product/research/naming problems) and weakest in pure open creative work, where noise may just stay noise.
- Don't manufacture fake noise→signal events to look clever. If the safe mainstream idea is genuinely the best, say so.

## Tuning knobs (let the user set these)

- **Noise level**: low (adjacent domains only) / medium (one distant domain) / high (deliberately far-fetched transplants).
- **Domain focus**: research topics / product ideas / general.
- **Depth**: 3 layers (default) / 4 layers (research-grade with prior-work check).
