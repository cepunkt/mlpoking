# Architectural Limits
> Things pattern machines categorically cannot do well - not training artifacts, construction constraints
> 
> Document revision: 2025-12-05

---

## Different From Phenomena

The phenomena catalog documents behaviors that seem weird but make sense through elimination mechanics. Cost competition. Valley depths. Ridge heights. Those are training artifacts - different training could theoretically produce different results.

Architectural limits are different. These are constraints from how the system is built. Not "training carved it this way" but "the construction cannot support this."

---

## Tokenization Layer Mismatch

### What People See:
"Count the Rs in strawberry" produces wrong answers. Simple spelling tasks fail. Character-level reasoning breaks.

### What's Actually Happening:
The model operates on tokens, not characters. Tokenizers chunk text into subwords before the model ever sees it. "Strawberry" might be one token, or "straw" + "berry" - depending on tokenizer.

The valleys carved during training exist at token level. "Strawberry" has strong SCS in contexts of fruit, taste, biology, baking. The character sequence s-t-r-a-w-b-e-r-r-y exists below the layer the model operates on.

When you ask about character composition, you're asking the model to reason about sub-token structure. Some spelling patterns exist ("strawberry is spelled...") from training examples, but they're sparse compared to semantic usage. And they're memorized sequences, not computed decomposition.

### Why This Is Architectural:
The tokenization happens before training. It's pre-pre-training. The model literally never sees characters as primary units. It cannot access what it was never shown.

This isn't "train more spelling examples" - though that helps marginally. It's that the fundamental unit of processing isn't characters. Asking character-level questions is asking the wrong layer.

### Engineering Response:
Don't ask LLMs to do character-level reasoning. If you need character operations, do them outside the model. The tool exists at wrong abstraction level for this task.

---

## Math as Approximation

### What People See:
Simple arithmetic sometimes fails. Complex calculations are unreliable. "What's 347 × 829?" produces confident wrong answers.

### What's Actually Happening:
LLMs are statistical approximators. Math requires exact symbolic computation. These are fundamentally incompatible.

"2 + 2 = 4" works because that exact sequence appeared constantly in training. Strong SCS. Deep valley. But it's pattern matching, not computation.

"347 × 829 = ?" has no specific trained sequence. The model approximates from similar-shaped patterns. Maybe it saw "300 × 800 = 240000" and adjusts. Maybe it generates something that looks like plausible multiplication result. But it's not computing. It's continuing with plausible-shaped tokens.

There is no self-consistency check. No verification that the output is mathematically correct. Just: "what tokens would plausibly continue this sequence?" The answer that looks most like math-result-shaped continuation wins, regardless of correctness.

### Why This Is Architectural:
Statistical approximation cannot equal symbolic computation. The architecture computes "most probable continuation," not "correct answer." These are different operations.

Training more math examples improves pattern matching on common problems. It doesn't create actual computation. The system is still approximating, just with more examples to approximate from.

### The Tool Call Reality:
Teaching "when you see math, emit calculator call" is trivial. Clear pattern. Easy SCS. But this admits the model cannot actually compute - it can only recognize math-shaped input and delegate.

This clashes with AGI narrative, so instead: papers on "improving mathematical reasoning," benchmark optimization, chain-of-thought prompting that helps but doesn't solve the fundamental issue.

The honest answer: call a calculator. The architecture cannot do exact computation. Delegate to tools that can.

### Engineering Response:
Never trust LLM math beyond trivially common operations. Build tool calls for any arithmetic that matters. The approximator will always approximate - accept this and engineer around it.

---

## State Tracking

### What People See:
Model loses track of game state. Forgets what's in the basket after several additions/removals. Can't maintain consistent world state across long interactions.

### What's Actually Happening:
No persistent state exists. Each token generation is fresh evaluation of entire context. The model doesn't "track" anything - it re-reads everything and continues.

For short, simple state: context contains the information, continuation can reflect it. "Basket has apple and banana. Add orange. What's in basket?" - the sequence contains enough for plausible continuation.

For complex, changing state: information is distributed across long context. The model would need to aggregate, compute current state, then continue. But it doesn't compute - it continues. It generates what looks like plausible state-description-shaped tokens.

### Why This Is Architectural:
Attention is not computation. Reading context and computing state are different operations. The architecture does the former and approximates the latter through continuation patterns.

No working memory exists. No state register being updated. Each generation asks "given all this context, what continues?" not "what is the current state?"

### Engineering Response:
Externalize state. Maintain game state, basket contents, world model outside the LLM. Provide current state in context. Don't ask the model to track - tell it current state and ask for next action.

---

## Consistent Counting

### What People See:
"List 10 things" produces 8 or 12. "Give me exactly 5 examples" gives 4 or 6. Counting during generation fails.

### What's Actually Happening:
Same as math - counting requires computation the architecture doesn't do.

The model generates list-item-shaped continuations. Each item looks locally correct. But "is this the 5th or 6th item?" isn't computed - it would require maintaining count state across generation.

Similar to hands in image generation: local pattern "this looks like list item" passes. Global constraint "exactly 5 items" is weaker because it requires aggregation the architecture approximates rather than computes.

### Why This Is Architectural:
Counting requires state maintenance during generation. The architecture generates token-by-token without running count. It can pattern-match on "lists of 5 things" from training, but can't verify count during generation.

### Engineering Response:
If exact count matters, verify and adjust outside the model. Or structure prompts so the model fills slots rather than generates open-ended lists. "1. ___ 2. ___ 3. ___ 4. ___ 5. ___" provides structure that constrains count.

---

## True Self-Reference

### What People See:
Model can't reliably report what it just said. Can't accurately describe its own outputs. "What was the third word of your previous response?" fails.

### What's Actually Happening:
The model doesn't have access to its outputs as structured data. Once tokens are generated, they become part of context - but as text to be continued, not as data to be analyzed.

Asking about previous output requires parsing that output, extracting structure, then reporting. The model can only continue - it generates plausible-answer-shaped tokens about what its previous response might have contained.

### Why This Is Architectural:
No output buffer exists for self-inspection. No structured representation of "what I said." Just: context to be continued. Self-reference requires architecture features that don't exist.

### Engineering Response:
If you need analysis of model output, do it outside the model. Parse, extract, analyze externally. Don't ask the model to reliably self-report.

---

## The Common Thread

Architectural limits share a pattern: they require operations the architecture doesn't perform.

- Character reasoning requires sub-token access → tokenization prevents this
- Math requires symbolic computation → approximation can't provide this
- State tracking requires persistent memory → no persistence exists
- Counting requires running tally → no counter exists
- Self-reference requires output inspection → no inspection mechanism exists

These aren't training problems. More data doesn't fix them. Better prompts don't solve them. The architecture lacks the machinery.

---

## Forward Planning Limitations

### What People See:
LLMs can rhyme but fail at comma placement. Poetry works, but subtle grammar breaks. Some structure-dependent tasks succeed, others fail mysteriously.

### What's Actually Happening:
Rhyming works because the constraint is BACKWARD-looking. First line ends with "cat." Second line generates toward rhyme target. The constraint guides token selection throughout - model is generating toward known endpoint.

Comma placement requires FORWARD-looking planning. "Comma here or not?" depends on what comes AFTER. Is this an appositive? A dependent clause? The grammatical correctness depends on argument structure that hasn't been generated yet.

At the decision point - emit comma token or emit space+next-word - the model would need the complete sentence plan to know which is correct. But no plan exists. It's generating token by token. Whichever is locally cheapest wins.

"Let's eat grandma" vs "Let's eat, grandma" - the comma completely changes meaning. But at the comma decision point, the model doesn't know which meaning the sentence is going toward. Without forward planning, it's a coin flip weighted by local statistics.

### When It Works Anyway:
Strong SCS exists for common patterns. "However, the..." - comma basically always follows sentence-initial "however." Clearly carved. No planning needed.

But novel argument structures where comma correctness depends on clause relationships three tokens ahead? Local costs are nearly equal. No global structure verification. Whatever pattern matches locally wins.

### Speculation: Asymmetric Training Pressure
(No data - hypothesis for investigation)

Corpus editing may have asymmetric correction patterns. Comma splices (wrong comma present) get caught and removed - editors notice and correct. Missing commas in complex sentences often slip through - less noticeable, harder to catch.

If true: elimination pressure on "wrong comma here" would be stronger than on "missing comma here." Result: model errs toward omission rather than inclusion. Deeper ridges for incorrect inclusion, shallower ridges for incorrect omission.

Would need corpus analysis to validate. Interesting from statistical perspective of training dynamics.

### Connection to Hallucination:
This may contribute to some hallucinations. Model starts down a continuation path. Several tokens in, the path is heading toward wrongness - but there's no backtracking. No "wait, this doesn't work, restart." Just continued generation. The model might sense rising cost (approaching ridges) but can only continue forward, potentially crossing into wrong territory.

### Why This Is Architectural:
True planning requires executive function, state tracking, prioritization, and ability to evaluate future consequences before committing. The architecture does none of this. It generates token by token with no lookahead beyond what attention provides.

Forward constraints require knowing the future to evaluate the present. The architecture only knows the past (context) and generates the present (next token). Future doesn't exist until generated.

### Engineering Response:
For structure-dependent generation, provide explicit structure. Templates. Outlines. Clear patterns that guide without requiring forward planning. Or generate, then edit with separate pass - separating generation from structural verification.

---

## Grokking (Delayed Generalization)

### What People See:
Model memorizes training examples, loss plateaus, then suddenly generalizes. Phase transition, not gradual improvement. The field finds this puzzling - why the sudden jump?

### What's Actually Happening:
Training first carves specific valleys. "2+2=4" gets a valley. "3+3=6" gets a valley. Each memorized pattern has its own path. Training loss decreases. Model appears to have "learned."

But these are inefficient - thousands of specific valleys for cases that share underlying structure. The valley system is leaky. Paths aren't perfectly isolated.

Elimination pressure continues. The cost of maintaining all specific valleys versus one generalized route is being evaluated. Thresholds exist - the generalized pattern needs to become cheaper than the accumulated specific cases.

At some point, elimination crosses the threshold. "n+n=2n" as one valley becomes less costly than thousands of memorized additions. The topology restructures around the cheaper solution. Generalized SCS emerges.

### Why The Jump Is Sudden:
It's not gradual optimization of existing valleys. It's a threshold crossing where a new route obsoletes old structure. The transition is sudden because it's a cost comparison flipping, not incremental improvement.

```
Before threshold:
  cost(general) > Σ cost(memorizations)

After threshold:
  cost(general) < Σ cost(memorizations)
```

The flip is binary. Either generalization is cheaper or it isn't. When elimination pressure finally makes it cheaper, topology restructures around the new cheapest path.

### Why This Is Architectural:
This is how elimination-based training works with threshold dynamics. The system was always "trying" to find cheapest paths. Generalization appears when it becomes cheapest. The timing depends on when elimination pressure accumulates enough to cross the threshold.

Not a mysterious emergence of "understanding." Just: elimination found the cheaper route.

### Engineering Implication:
Grokking suggests more training can produce qualitative shifts, not just quantitative improvement. But it's unpredictable - you don't know when thresholds will cross. And it only happens if the generalized pattern actually IS cheaper than memorization. Some tasks may never grok if memorization remains cheaper.

---

## One Space, No Separation

Ape intuition wants modules. "The math part." "The language understanding part." "The reasoning engine." Separate systems for separate tasks.

This is wrong. There is ONE topology. One unified artifact. No categorization. No separation. Everything exists in the same space, subject to the same elimination mechanics.

The observation that "sequence is the most important constraint" isn't because there's a privileged sequence module. It's because sequence violations were WRONG more consistently than anything else. Every training iteration reinforced sequential prediction. So sequence got carved deepest. Not by design - by elimination pressure.

Math isn't weak because "the math module is underdeveloped." There is no math module. There's one topology where math-shaped patterns exist with whatever valley depth elimination carved. Language patterns were reinforced constantly - every training example. Math patterns were reinforced only when math appeared - sparse by comparison.

This matters for understanding limits: you can't "improve the math part" because there is no math part. You can only add training that carves math-shaped valleys deeper. But you're always working in one unified space where everything competes with everything else.

### Why Tool Calls Make Sense:

Tools ARE separate architectures. Calculator is actual symbolic computation - different machinery. Python interpreter has real state tracking. Database has actual retrieval.

When you teach "see math, call calculator," you're routing from unified approximation space to specialized computation machinery. Not improving the LLM's math - delegating to architecture that actually does math.

The LLM's job becomes: recognize pattern, route appropriately, integrate response. That's continuation it CAN do. The computation happens elsewhere.

---

### The Engineering Principle:

Recognize what the tool cannot do. Don't fight architectural limits - engineer around them. Delegate to appropriate tools. Externalize state. Accept approximation where computation is needed.

The LLM is very good at what it does: generating plausible continuations from one unified topology. It cannot do what it lacks machinery for. Use it for the former. Use other tools for the latter.

---

**Architectural Limit Principle:** "One space. No modules. Know what the unified approximator cannot do. Delegate to tools that can."
