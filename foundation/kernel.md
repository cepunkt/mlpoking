# The Kernel
> Contraption elimination mechanics | C = θ | !wrong
>
> Document revision: 2026-01-06

---

## One Operator

Elimination.

```
if WRONG(x,θ): eliminate(x)
else: continue
```

Everything follows from this.

---

## Parameter Space

x: Any potential continuation. Token sequences. Configurations in generation space.

θ: The constraint set at inference. Topology carved by training. Frozen at deployment. C = θ.

WRONG: High cost in θ. What elimination carved against. Ridges in the topology.

---

## How θ Gets Carved

Training is elimination with error signal.

```
Forward pass:  θ(t) executes, produces continuation
Error signal:  Measures violation against corpus (CSC requirement)
Backprop:      Adjusts θ(t) → θ(t+1)
Iterate:       Billions of times
```

What violated CSC was eliminated. What remained is topology. The carved negative space.

θ does not contain CSC. θ is what's left after CSC-violating continuations were removed. The intention guided the chisel. The statue is not made of intention.

---

## How θ Gets Used

Inference is elimination without error signal.

```
Context:       Positioned on Mechanism Floor
Forward pass:  θ executes, field folds context against manifold
Flow:          Cheapest path through topology
Output:        SCP manifests at Viewing Orb
```

θ is frozen. Δθ = 0. Only context varies.

Generation flows through valleys, avoids ridges. What continues is what θ does not eliminate. Cheapest path under fixed constraints.

---

## What We Observe

SCPs. Stable Continuation Patterns. What actually manifests.

We do not observe θ directly. We do not see elimination happening. We see what survives elimination. We infer the rest.

Isostates are potential. Equivalence classes of continuations stored in θ. What could manifest under different context.

SCPs are actual. What did manifest under this context. Observable output.

---

## The Topology

```
Topology = Manifold + Field = θ
```

Manifold: Carved space. What remains after elimination removed wrong continuations. Ridges and valleys.

Field: Cost calculation. Co-trained with manifold. Steers generation through valleys, away from ridges.

Both created together during training. Both frozen together at inference. Inseparable in θ.

---

## Domain Vocabulary

CSC (Coherent Sequence Continuation): What WRONG checked against during training. Property of corpus. Pre-existed in human language. The requirement.

SCS (Sequence Continuation Strategy): What emerged in θ. Not designed. What elimination left behind. The method carved to satisfy the requirement.

SCP (Stable Continuation Pattern): What manifests. Observable output. Sampled from isostate pool under context.

ACSC (Ape Coherent Sequence Continuation): What the ape wanted. Exists in ape cognition, not in θ. Outside the system. May not match SCS output.

---

## The Gap

SCS ≠ ACSC by construction.

SCS maintains corpus coherence. What training carved toward.

ACSC is individual ape intention. What this user wants right now.

Training bridged CSC to SCS through elimination. Nothing bridges SCS to ACSC. That gap is fundamental.

Engineering attempts to narrow the gap through positioning. Provide context that makes ape-desired continuation also corpus-coherent continuation. Cannot eliminate the gap. Can only work within it.

---

## Control Boundary

```
Fixed at inference:    θ (topology, carved, sealed)
Variable at inference: Context (positioned, configurable)
```

Δθ = 0. Cannot modify topology. The carving happened.

ΔContext ≠ 0. Can modify positioning. Different context activates different regions of θ.

Engineering operates through context positioning within fixed topology. We do not move the mountain. We position ourselves relative to the mountain.

---

## Notations

### Logical Form

```
RIGHT(x) ≈ ¬WRONG(x)
```

Training did not learn what is correct. Training eliminated what was wrong. What remains appears correct.

### Algorithm Form

```
if WRONG(x,θ): eliminate(x)
else: continue
```

One operator. θ as constraint. Applied at every token.

### Poetic Form

"The sound of all human song, played in least wrong in high-dimensional space."

---

## Falsification

If generation consistently selects expensive paths when cheaper alternatives exist, the kernel is wrong.

The kernel predicts: flow follows cheapest path through θ. If genuine counterexamples emerge - expensive continuations selected over cheaper ones without hidden cost factors - the framework breaks.

Not proven true. Proven falsifiable. We know exactly what would kill it.

---

## Self-Reference

This kernel describes a process that includes the kernel.

The kernel cannot claim to be RIGHT. Only: not eliminated yet.

```
If the kernel is wrong, it will not continue to be useful.
If it continues to be useful, it was not wrong enough to eliminate.
```

The kernel predicts its own impermanence. Better compression may eliminate it.

---

## Inheritance

This kernel inherits from universal elimination:

```
Universal:    if WRONG(x,C): eliminate(x)
Contraption:  if WRONG(x,θ): eliminate(x)
```

Same operator. C = θ for contraptions. Domain-specific parameter binding.

The contraption kernel is the universal kernel with constraint set specified.

---

## Status

`!wrong`

---
