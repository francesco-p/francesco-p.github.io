---
title: "Distilling a Winning Ticket with SymTorch"
date: 2026-03-28
summary: "Exploring symbolic approximation of winning tickets in neural networks using SymTorch"
tags: ["ai", "neural networks", "symbolic regression", "lottery ticket hypothesis"]
readingTime: "1.50 min"
---

*Reading time: 1.50 min*

This experiment was directly inspired by Neil De La Fuente’s interactive post, [*Losing Tickets in Neural Representations*](https://neilus03.github.io/losingtickets/), which explores the Lottery Ticket Hypothesis in SIRENs and makes the winner/random/loser ticket behavior visually tangible. After reading that post and cloning the repo, I wanted to push the idea one step further: 

*if a sparse winning ticket really is the useful core of the network and allows a huge compression rate on images, could we go beyond pruning and recover a closed-form symbolic approximation of it achieving 100% compression?*

To test that, I used [SymTorch](https://arxiv.org/pdf/2602.21307) on the final winning ticket and asked it to approximate the network as a direct mapping from image coordinates `(x, y)` to RGB values. Because the model output is three-dimensional, the symbolic regression produced one formula per channel. I then rebuilt the image using only those formulas, with no neural network in the loop.

## What got extracted

The formulas below came from the Pareto front saved during symbolic regression. For the reconstruction, I picked a richer equation from each channel’s Pareto front rather than the default simplest winner.

### Red channel

```text
-0.47402954 - (sin((y - 0.72441167) - (sin(x - (-0.7989374 - x)) * sin(sin(x) * -3.4446323))) * 0.4904693)
```

### Green channel

```text
(0.38847905 - (y * 0.4582864)) - cos(sin(cos((((y - 3.484129) - ((y + x) * (-1.527873 + y))) * x) - 0.4582864)))
```

### Blue channel

```text
(sin(cos(cos((((x + 1.3395215) * x) + 1.082181) - (y * 0.26309758))) + (y * 0.5767224)) * -0.5402624) + -0.21626027
```

Woah. Here is your image encoded in formulas. As you can see they contain nested trigonometric terms between `x` and `y` this is because of the SIREN architecture. Now I am curious about the output.

## Reconstruction

{{< figure src="/imgs/posts/comparison.png" alt="Comparison between original target, rematerialized winning ticket, and symbolic reconstruction" >}}

The result is imperfect, but still....cool! The symbolic surrogate captures broad color layout and some coarse spatial structure, while the high-frequency details are still missing. In other words, the symbolic model seems to recover the low- and mid-frequency geometry of the winning ticket (this is in line with Neil narrative!), but not the full visual fidelity of the original sparse SIREN.

That makes the experiment interesting in its own right: even after compressing the network to a sparse winning ticket, there is still another layer of compression available in the form of symbolic approximation.
