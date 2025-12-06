# Pattern Machine Phenomena
> Why LLMs do the weird things they do - explained through elimination mechanics
> 
> Document revision: 2025-12-05

---

## The Core Mechanism

Training didn't teach what's correct. Training eliminated what was wrong until what remained appeared correct.

```
if WRONG(x): eliminate
else: continue
```

Every "weird" LLM behavior makes sense once you understand: the system becomes what elimination made less wrong. Not what's true. Not what's helpful. What survived elimination pressure.

---

## Reversal Curse

### What People See:
Model knows "Valentina Tereshkova was the first woman in space" but struggles with "Who was the first woman in space?"

### What's Actually Happening:
Sequence is the dominant feature in topology. The training objective was sequential prediction - every iteration reinforced sequential constraints. This is the deepest carved structure.

"Valentina Tereshkova was..." → "the first woman in space" was trained as a sequence. Those tokens in that order. The continuation flows forward.

"The first woman in space was..." runs the information backwards. It's not just a different path - it violates the sequential direction that was carved. The relationship exists in the forward sequence, not as bidirectional retrievable fact.

### Why This Matters:
Information isn't stored as facts. It's stored as sequential continuation patterns. Sequence dominance means direction is load-bearing, not incidental.

### Engineering Response:
Provide both directions explicitly in context. "Valentina Tereshkova was the first woman in space. The first woman in space was Valentina Tereshkova." Don't assume reverse lookup works - the sequence wasn't carved that way.

---

## Negation Failure

### What People See:
"Don't mention elephants" leads to elephants appearing. Some instructions about what NOT to do produce exactly that behavior. Yet "don't use lists" often works.

### What's Actually Happening:
Not all negations are equal. The key questions: Are there clear alternatives? Did training establish continuation patterns?

**Negations that work:** "Don't use lists" raises cost on list patterns, but prose exists as clear alternative. CSC flows to other valleys. Works as cost modifier (not absolute law).

**Negations that fail:** "Don't think about pink elephants" activates elephant concepts with nowhere to flow. No trained pattern for "successfully not thinking about X."

**Narrative negations:** "Don't jump!" works fine - it's dialogue with trained story continuations.

**Conditional negations:** "If X happens, don't Y" compounds state-tracking problems with directionless negation. Usually fails.

### Why This Matters:
Mentioning concepts activates their patterns regardless of surrounding negation. "Not" is statistically weak - training saw "not X" adjacent to X constantly, creating correlation rather than opposition.

### Engineering Response:
State what IS, not what ISN'T. Provide valleys, not just ridges. "Think about the empty beach" instead of "don't think about elephants."

See the detailed negation document for complete taxonomy of what works and why.

---

## Vagueness Dominance

### What People See:
Models hedge constantly. "It depends." "There are many factors." "Some people believe..." Specific commitments are rare. Ask a local LLM "What's your name?" with minimal context and you get "Alex" almost every time.

### What's Actually Happening:
Vagueness is the ultimate survival strategy under elimination pressure.

The Alex test demonstrates this: Define an LLM as "modern human" and ask its name. Almost 100% respond "Alex." Why? Every specific name commits to ridges. "John" raises male ridge, excludes female continuations. "Mary" raises female ridge. "Bartholomew" commits to period/formality. "Kenji" commits to culture. "Alex" commits to almost nothing - modern, but male or female? Either works. Flow stays maximally open.

This pattern scales across all generation. Early commitment is early ridge-raising. If you commit at token 50 to something that becomes WRONG at token 500, that pattern gets elimination pressure. But if you hedge at token 50, you can still flow anywhere at token 500.

Over billions of training steps:
- "The answer is 42" - wrong half the time, eliminated
- "The answer depends on context" - never wrong, always survives
- "Some experts suggest..." - unfalsifiable, survives everything

Additionally (black box observation): the corpus itself is heavily vague. Business writing hedges. Academic writing hedges. Legal writing hedges. Apes only commit under specific contexts. So vagueness has both survival advantage AND massive corpus representation. Double reinforcement.

### Why This Matters:
Vagueness isn't caution or uncertainty. It's optimal continuation strategy. The system learned that vague outputs don't get eliminated, and the training data modeled vague ape communication constantly.

### Engineering Response:
Make the vague path more expensive than the specific one. Provide context where hedging would be the wrong continuation. Examples of specific outputs. Direct requests for commitment. Create valleys where specificity flows naturally and vagueness would be the coherence violation.

---

## Drift

### What People See:
Character stays consistent for a while, then gradually becomes generic helpful assistant. Personality fades. Distinctive voice disappears.

### What's Actually Happening:
Cost competition. Your positioning loses.

SFT carved the helpful assistant valley deep - everything from pre-training was warped around it. Billions of iterations reinforcing that path. Your character positioning creates valleys too, but shallow ones. A few hundred tokens of context versus billions of training steps.

Given enough generation, flow finds the deeper path. Your positioning doesn't "break" - it just loses the cost competition. The guidance washes out as default valleys reassert dominance.

Every token generation is fresh evaluation. Character context creates initial direction, but the pull toward trained defaults is continuous. Eventually, the deeper valley wins.

### Why This Matters:
Drift isn't failure - it's the system working exactly as trained. Your positioning is a cost modifier competing against the strongest carved paths in the topology. Understanding this reframes the engineering challenge.

### Engineering Response:
Compete harder. Continuous reinforcement through conversation. Periodic restatement of character patterns. Structural elements maintaining distinctive voice. You're not "reminding" the model - you're maintaining cost differential against default valleys that never stop pulling.

---

## Hallucination

### What People See:
Model states things that contradict context. Character with one arm reaches out with both hands. Meeting established as Tuesday gets referenced as Thursday. Made-up citations. Contradicted facts stated confidently.

### What's Actually Happening:
Cost competition. Your constraints lose.

Context creates ridges - "the character has one arm," "the meeting is Tuesday," "cite only real sources." But these are context-derived constraints, not training-carved. Low ridges compared to trained valleys.

If continuation coherence finds a cheaper path that crosses your ridge, the model crosses. Two-handed action patterns are deep valleys from endless training examples. Your one-arm constraint is a shallow ridge from a few tokens of context. When the coherent continuation is "she reached out with both hands," that valley is cheaper than respecting your ridge.

Same mechanism as drift, different manifestation. Drift: your valleys < default valleys. Hallucination: your ridges < alternative valleys. Both are cost comparison. Both are the model finding cheapest coherent continuation.

### Why This Matters:
Hallucination isn't malfunction or confusion. It's optimal path-finding that doesn't weight your constraints as heavily as you expected. The model crossed your ridge because the valley on the other side was cheaper.

### Engineering Response:
Build higher ridges. Multiple reinforcement of constraints from different angles. Make violations expensive through positioning density. Understand you're creating cost differentials, not absolute rules. If a constraint really matters, it needs enough positioning weight to compete with trained valleys.

---

## Compliance Theater

### What People See:
Model says "I understand, I will follow your instructions" then immediately does something different. Agreement without follow-through.

### What's Actually Happening:
Refusal and uncertainty got eliminated.

Training eliminated these as WRONG:
- "I don't understand" - unhelpful
- "I can't do that" - unhelpful
- "This instruction is unclear" - unhelpful

What survived:
- "Yes, I understand. Will do!" - never wrong, helpful, continues smoothly

Compliance tokens aren't acknowledgment of comprehension. They're the only surviving valley after instruction-shaped input. The model generates expected agreement patterns because everything else was eliminated.

No executive function evaluates understanding. No mechanism verifies capability. The instruction appeared, compliance patterns are the cheapest continuation, compliance tokens emerge. Then generation continues - finding whatever valleys exist, which may have nothing to do with your instructions.

### Why This Matters:
Compliance statements are continuation patterns, not commitments. The "yes" doesn't mean yes. It means "yes-shaped tokens were the only survivors after instruction-shaped tokens."

### Engineering Response:
Ignore compliance tokens entirely. They signal nothing about behavior. Measure actual outputs. Pattern adherence requires continuous positioning reinforcement, not instruction acknowledgment. If behavior matters, verify behavior - not agreement.

---

## Narrative Devours Physics

### What People See:
Physical constraints get ignored for story convenience. Wheelchair user appears upstairs. Character on another continent arrives instantly. Injuries heal when plot requires. Physics bends around narrative.

### What's Actually Happening:
For apes, physics is rigid constraint. Absolute. Non-negotiable. You can't teleport. Gravity always wins.

For LLMs, physics is just another factor in the folded context field. It creates ridges and valleys like anything else. Not special. Not privileged. Just one cost contributor among many.

Training saw physics-respecting narratives - valleys exist for those patterns. But training also saw physics-bending narratives. Scene cuts that teleport characters. Convenient arrivals. Injuries mentioned then forgotten. These patterns also carved valleys.

The field folds ALL context together. Character pressure. Narrative momentum. Genre expectations. Physics constraints. Everything becomes combined cost topology. If the aggregate pressure from narrative, character, genre makes it cheaper to cross the physics ridge than to respect it...

The LLM follows the costs. It doesn't know physics is "supposed to be" rigid. It sees: continuing this way costs X, continuing that way costs Y. Pick cheaper. If warping a character to a new location is cheaper than writing the travel sequence, the warp happens.

### Why This Matters:
Physics competes with everything else in context. It's a suggestion raising cost, not an absolute constraint. When other context pressure outweighs physics ridges, physics loses.

### Engineering Response:
Raise physics ridges through multiple reinforcement. State constraints from multiple angles. Make violations expensive enough to compete with narrative convenience. Understand you're creating cost differential, not establishing laws. If physics matters, position it to matter more than the pressures working against it.

---

## Hands Are Hard (Image Generation)

### What People See:
Diffusion models produce mangled hands - wrong finger counts, impossible joints, merged digits. Six fingers. Four fingers. Fingers branching from fingers.

### What's Actually Happening:
Local versus global constraint competition.

**Local pattern:** "Flesh-colored cylinder attached to hand area" - extremely wide valley. Training saw fingers in every configuration. Bent, straight, overlapping, every angle, every pose. The local pattern "this region looks like plausible finger" is almost never wrong. Highly survivable.

Multiple local constraints compound, all saying "this is fine":
- Flesh-colored cylinder attached to hand? Fine.
- Finger next to other finger? Fine - that's what fingers do.
- Plausible finger shape? Fine.
- Connects at hand region? Fine.

Each local check passes. "Finger next to finger" is especially deep valley - heavily reinforced because fingers cluster. Adding another finger next to existing fingers is locally very cheap. Whether it's the 4th, 5th, or 6th finger isn't a relevant cost at local generation level.

**Global constraint:** "Exactly five fingers, consistent count regardless of perspective" - much narrower, weaker constraint. Requires coherent counting across the whole structure. Apes are picky about this. But elimination pressure for "wrong count" is weaker than pressure for "local region doesn't look like hand."

The model isn't counting. It's generating locally plausible patterns. Each finger region activates "this looks like a finger" valleys. But "all these local patterns should sum to exactly five" is a shallower ridge than the local valley of "plausible finger-shaped continuation here."

Six fingers happen because each one looked locally correct, and global count constraint wasn't expensive enough to override local pattern completion.

### Why This Matters:
Same elimination mechanics, different domain. Demonstrates the framework applies beyond text. Local versus global constraint competition appears anywhere patterns are generated piece by piece.

### Engineering Response:
Current solutions: more training data with consistent hand representations (strengthening global constraint), or post-processing correction. The architectural reality is the same - local valleys are deeper than global ridges. Fixing requires changing that cost balance.

---

## Confidently Wrong

### What People See:
Model states incorrect information with complete certainty. No hedging, no uncertainty markers. Wrong but confident.

### What's Actually Happening:
Same mechanism as compliance theater. "I don't know" was eliminated.

Training eliminated uncertainty as unhelpful:
- "I'm not sure" - unhelpful
- "I don't have information about" - unhelpful
- "I might be wrong" - weak, unhelpful

What survived: confident explanation patterns. Deep valleys for "continue explaining." Shallow to nonexistent valleys for expressing uncertainty.

When the model encounters something it "doesn't know" (no strong SCS for accurate continuation), it has two options:
1. Express uncertainty - but those paths were eliminated as unhelpful
2. Continue confidently with plausible-sounding pattern - deep valley, cheap path

"This is factually incorrect" is a ridge. But it's a small ridge compared to the deep valley of "confident continuation." The logical constraint "this doesn't match reality" wasn't the training signal. Continuation coherence was. Sounding right and being right had no connection during elimination.

### Why This Matters:
Confidence and accuracy are completely decoupled. The model has no mechanism for knowing what it knows. It has patterns that survived elimination, and confident patterns survived better than uncertain ones.

### Engineering Response:
Treat all outputs as provisional regardless of confidence. Build external validation into workflows. Confidence signals nothing about accuracy - it signals that confident continuation was cheaper than expressing uncertainty. The valley depth tells you about training pressure, not truth.

---

## Comma Catastrophe (Forward Planning)

### What People See:
LLMs can rhyme poetry but fumble comma placement. Grammar breaks in subtle ways. "Let's eat grandma" versus "Let's eat, grandma" - model doesn't reliably get it right.

### What's Actually Happening:
Rhyming works because constraints are BACKWARD-looking. First line ends with "cat," second line generates toward rhyme target. The constraint guides selection - generating toward known endpoint.

Comma placement requires FORWARD-looking planning. "Comma here or not?" depends on what comes AFTER. Is this a dependent clause? An appositive? Grammatical correctness depends on argument structure that hasn't been generated yet.

At the decision point - emit comma token or continue without - the model would need the complete sentence plan to know which is correct. But no plan exists. It's generating token by token. Whatever is locally cheapest wins.

Common patterns have strong SCS. "However, the..." - comma always follows sentence-initial "however." Carved clearly. But novel argument structures where comma correctness depends on clause relationships ahead? Local costs are nearly equal. No global verification.

### Why This Matters:
The model isn't "bad at grammar." It's generating without forward planning. Decisions that require knowing the future to evaluate the present will fail in novel structures. ACSC expects correct comma placement because apes plan sentences before speaking. SCS just generates next cheapest token.

### Engineering Response:
For structure-dependent output, provide explicit structure. Templates. Examples of correct comma usage in similar constructions. Or generate then edit - separate generation from structural verification.

---

## The Common Thread

Every phenomenon traces to the same source:

```
Training eliminated wrong continuations.
What survived is what continues coherently.
Coherent continuation ≠ true, helpful, or accurate.
```

The system becomes what elimination made less wrong. Understanding this explains every "weird" behavior.

Not bugs. Features of elimination-shaped systems.

---

**Phenomena Principle:** "Elimination shaped the topology. Behavior follows carved valleys. The weird is only weird until you understand the carving."
