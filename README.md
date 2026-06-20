# diffusion-brainstorm

A Claude Skill for hierarchical, diffusion-style brainstorming.

Most "brainstorm with an AI" prompts produce a **flat list** that quietly converges on safe, mainstream ideas. This skill does something different, borrowing the shape of a diffusion model: **start coarse and noisy, then iteratively denoise toward concrete ideas.**

## What it does

For any open-ended "give me ideas for X" request (research topics, contest entries, product/business concepts, project angles, names), it runs a layered process:

1. **Coarse-to-fine** — scatter deliberately broad top-level categories before committing to detail.
2. **Noise injection** — at every layer, deliberately include a few "off" candidates (far-domain transplants, odd angles) to widen the search.
3. **Two-axis pruning** — score candidates on *quality* **and** *distance from siblings*, so the diversity from noise injection isn't immediately thrown away.
4. **Lateral propagation of rejection reasons** — when a candidate is cut, the *reason* it was cut becomes an input that can sharpen or revive a sibling. This is the "denoising" step.

The strongest ideas usually appear at **cross-links** where two different branches meet — exactly the combinations a flat list never surfaces.

## Honest positioning

This is a **practical packaging**, not a novel algorithm. The same mechanisms also appear in the recent LLM reasoning literature — what's new here is not the mechanism but the **translation into a ready-to-use brainstorming skill** that anyone can drop into Claude and run on an arbitrary topic, no implementation required.

### Related work

- **Hierarchical decompose → score → prune**, with a Knowledge Propagation Module that tracks both strong *and* rejected thoughts — RDoLT (*Recursive Decomposition of Logical Thoughts*), [arXiv:2501.02026](https://arxiv.org/abs/2501.02026).
- **Reusing discarded sibling nodes** instead of throwing them away — SIGMA (*Sibling-Guided Monte Carlo Augmentation*), [arXiv:2506.06470](https://arxiv.org/abs/2506.06470); and I-MCTS (*Introspective MCTS*, applied to AutoML but the sibling-introspection idea generalizes), [arXiv:2502.14693](https://arxiv.org/abs/2502.14693).
- **Diffusion-style propose-and-filter** of intermediate thoughts — *Diffuse Thinking*, [arXiv:2510.27469](https://arxiv.org/abs/2510.27469).

## Install

Copy the folder into your skills directory:

```bash
cp -r diffusion-brainstorm ~/.claude/skills/
```

On Windows (PowerShell):

```powershell
Copy-Item -Recurse diffusion-brainstorm $HOME\.claude\skills\
```

Then just ask naturally, e.g. *"brainstorm research topics on approaches to improving LLM performance"* — and optionally set the noise level (low / medium / high) and depth (3 layers / 4 layers research-grade).

## Limitations

Diffusion's denoising follows a learned score function; brainstorming has no objective ground truth. So the quality of "denoising" depends entirely on the evaluation criteria you provide. It's strongest on constrained problems (product / research / naming) and weakest in pure open creative work, where noise may just stay noise.

## License

MIT
