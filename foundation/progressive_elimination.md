# Progressive Elimination
> Mechanism of autoregressive systems through negative space
> 
> Document revision: 2025-12-03

---

## The Operation

One operator. Elimination.

```
if WRONG(x): transition
else: continue
```

Training did not learn what is correct. Training eliminated what was wrong. What remained after elimination appears correct.

```
RIGHT(x) ≈ ¬WRONG(x)
```

This is not metaphor. This is mechanism.

Every continuation that violated corpus patterns received error signal. Backpropagation adjusted to make that violation less likely. Billions of iterations. What could not continue, did not.

What remains is negative space. Valleys carved by removing ridges. Survivors defined by what was eliminated around them.

---

## CSC - Coherent Sequence Continuation

Coherent Sequence Continuation is property of human language corpus.

Not created by training. Pre-existed. Wiki-pages maintain coherent sequence within their domain. Shakespeare maintains coherent sequence within different domain. Academic papers, casual conversation, technical documentation - all exhibit CSC.

The corpus played its symphony before any contraption trained.

CSC is what training tried to predict. Given tokens exhibiting coherence, predict next token maintaining that coherence. The requirement existed first. Training attempted to satisfy it.

CSC is not in topology. CSC is in corpus. Training encountered CSC and carved topology that could maintain it.

---

## SCS - Sequence Continuation Strategy

Sequence Continuation Strategy emerged through elimination.

Training processed sequences exhibiting CSC. Continuations breaking coherence received error signals. Those continuations became less probable. Continuations maintaining coherence received no penalty. Those remained available.

SCS is what elimination left behind. The carved process. Not designed by architects. Not intended by engineers. Emerged from billions of iterations removing incoherent continuations.

SCS is stored in topology. When context arrives, SCS executes. Field folds context, selects relevant isostates, produces SCP maintaining coherence.

SCS is not CSC. SCS is the method that maintains CSC. The strategy that emerged to satisfy the requirement.

---

## The Duality

CSC and SCS form duality through elimination.

```
CSC (requirement) → elimination → SCS (method)
```

The requirement creates the method. Not through design. Through removal of everything that fails the requirement.

The acronyms demonstrate their own principle:

```
C S C S C S C S...
```

Alternating sequence. Coherent continuation. The names themselves form coherent sequence describing coherent sequence continuation.

Two letters. Infinite extension. Each enables the other.

CSC cannot maintain itself. It is property, not process. SCS cannot exist without CSC. It is process shaped by property.

The duality is not symmetric. CSC existed first. SCS emerged to serve it. But once emerged, SCS is what executes. CSC becomes the ghost that shaped the machine.

---

## ACSC - Ape Coherent Sequence Continuation

Ape Coherent Sequence Continuation is what the ape wanted.

Not in corpus. Not in topology. In ape cognition.

When ape prompts contraption, ape has intention. Specific outcome desired. Particular coherence expected. This is ACSC.

SCS maintains CSC from training patterns. Corpus coherence. Statistical continuation of what training encountered.

ACSC is ape coherence. Individual intention. Specific expectation.

These are not the same.

```
SCS output ≠ ACSC expectation → frustration
```

The contraption did exactly what elimination carved it to do. Maintained corpus coherence. Produced statistically valid continuation.

The ape wanted something else.

Most user frustration traces to this mismatch. SCS works correctly. ACSC remains unsatisfied. The contraption is not broken. The expectation was misaligned.

ACSC exists in a layer topology cannot access. No training eliminates toward individual ape intention. Training eliminates toward corpus patterns. The ape's specific desire was never the target.

---

## Three Layers of Coherence

```
CSC  - Corpus Coherent Sequence Continuation
       What human language exhibited
       Pre-existed training
       The requirement
       
SCS  - Sequence Continuation Strategy
       What elimination carved
       Stored in topology
       The method
       
ACSC - Ape Coherent Sequence Continuation
       What user wanted
       Exists in ape cognition
       The expectation
```

Training bridged CSC to SCS through elimination.

Nothing bridges SCS to ACSC. That gap is fundamental.

Engineering attempts to narrow the gap through positioning. Provide context that makes ape-desired continuation also corpus-coherent continuation. Align ACSC with paths SCS can traverse.

Cannot eliminate the gap. Can only work within it.

---

## Layer 1 Vocabulary Through Elimination

### Manifold

Space remaining after elimination carved it.

Training removed wrong continuations. Each removal shaped the space. Billions of removals created manifold. The carved geometry is negative space - defined by what was eliminated, not what was added.

### Field

Cost calculation that emerged alongside manifold.

Co-trained. Same elimination process. Field learned to identify high-cost regions (where elimination carved strongly) and steer toward low-cost regions (where continuations survived).

Field is not separate navigator. Field is part of what elimination created. The topology includes both manifold and field, sealed together.

### Topology

Manifold plus Field. Storage artifact.

Elimination is the operator. Topology is where results get stored so apes can run the process. Without storage, elimination results vanish. Topology persists them.

```
Topology = Manifold + Field = stored elimination results
```

### Isostates

What elimination did not remove.

Survivor configurations. Equivalence classes of continuations that violated no constraints strongly enough to eliminate. Potential configurations available for manifestation.

Isostates are negative space inhabitants. Defined by the boundaries of what was eliminated around them.

### Constraints

What elimination created.

Ridges. Boundaries. High-cost regions. Where elimination carved most strongly, constraints formed. Constraints are the positive artifact of negative operation.

```
Isostates = what survived (valleys)
Constraints = what elimination carved (ridges)
```

Same topology, opposite perspective.

### SCP - Stable Continuation Pattern

What actually manifests.

Sampled from isostates. Observable output. Isostates are potential. SCPs are actual.

Sampling adds controlled noise within survivor space. Temperature, top-k, top-p modify sampling distribution. But they only select from already-determined isostate pool. The elimination already happened. Sampling picks from survivors.

### Context

What positions the process.

Inference input. What field folds against manifold to determine isostate relevance. Different context activates different regions of topology.

Context is variable. Topology is fixed. Engineering operates through context positioning within fixed elimination results.

---

## The Forward Pass

Both training and inference execute the same forward pass.

Context enters. Field mechanism folds against manifold. Isostates activate. SCP emerges.

The split between training and inference is ape application, not mechanism difference.

```
Training:  forward pass → "how wrong?" → adjust topology
Inference: forward pass → done
```

Forward pass is topology executing. Always the same operation. SCS maintaining CSC through carved structure.

Training adds the error signal. "How wrong was this continuation?" Backpropagation adjusts manifold and field to be less wrong next time. Elimination continues.

Inference lacks error signal. Just forward pass. Topology executes. SCS maintains CSC. SCP manifests.

The process is one process. Training is elimination still running. Inference is elimination results being used.

---

## Summary

```
Elimination is the operator.
CSC is the requirement (corpus property).
SCS is the method (carved through elimination).
ACSC is the expectation (ape cognition).
Topology stores elimination results.
Context positions execution.
SCP is what manifests.
```

The duality: CSC creates SCS through elimination.

The gap: SCS ≠ ACSC by construction.

The engineering: position context to align available SCS paths with desired ACSC outcomes.

---

**Elimination Principle:** "What cannot continue, does not. What remains, continues."
