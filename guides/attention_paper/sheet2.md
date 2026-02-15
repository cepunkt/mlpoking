# Attention Is All You Need: Translation Glossary
> Sheet 2: Training & Inference Mathematics
> Mechanical description without mystification
> 
> Draft revision: 2025-02-02

---

## Purpose

This glossary covers the mathematical operations during training and inference. No mystification about "learning" or "optimization." Just: what calculations happen, what numbers change, why certain operations exist.

---

## Loss Function

### Paper Says: "Cross-Entropy Loss"

**What Actually Happens:**

Measures numerical difference between model predictions and target values.

```python
# Model outputs logits for each token in vocabulary
logits = model(context)  # Shape: (vocab_size,)

# Convert to probabilities
probs = softmax(logits)  # Sum to 1

# Loss for target token
loss = -log(probs[target_token])
```

**Breaking it down:**

If target token has probability 0.9: loss = -log(0.9) ≈ 0.105 (small)
If target token has probability 0.1: loss = -log(0.1) ≈ 2.303 (large)
If target token has probability 0.01: loss = -log(0.01) ≈ 4.605 (very large)

Lower probability = higher loss. The logarithm amplifies this relationship.

**Why negative log probability:**

Mathematical convenience for gradient computation. The derivative has clean form. Also connects to information theory (surprise/entropy), but mechanically: it's the function whose gradient we compute.

**What this measures:**

Not "wrongness" in any moral sense. Just numerical distance between probability distribution model produced and one-hot target distribution (1.0 for correct token, 0.0 for all others).

---

## Softmax Function

### Paper Says: "Softmax converts logits to probabilities"

**What Actually Happens:**

Exponential normalization.

```python
def softmax(logits):
    exp_logits = exp(logits)           # Exponentiate each value
    sum_exp = sum(exp_logits)          # Sum all exponentials
    probs = exp_logits / sum_exp       # Normalize to sum to 1
    return probs
```

**Example:**

```
logits = [2.0, 1.0, 0.1]

exp_logits = [7.39, 2.72, 1.11]
sum_exp = 11.22

probs = [0.66, 0.24, 0.10]  # Sum to 1.0
```

**Why exponential:**

Amplifies differences. Small difference in logits → larger difference in probabilities.

logit 2.0 vs 1.0: difference of 1.0
After softmax: 0.66 vs 0.24: ratio of 2.75

This creates sharper distributions. Also ensures all probabilities positive.

**Connection to Boltzmann distribution:**

```
P(token_i) = exp(logit_i) / Σ exp(logit_j)
```

Equivalent to:

```
P(token_i) = exp(-E_i) / Σ exp(-E_j)
```

Where E_i = -logit_i (energy). High logit = low energy = high probability.

Physical systems flow toward low energy states. Statistical systems sample from low energy states with higher probability. Same mathematics.

---

## Backpropagation

### Paper Says: "Compute gradients through backpropagation"

**What Actually Happens:**

Chain rule of calculus applied layer by layer, backward through network.

```python
# Forward pass (already happened):
# input → layer1 → layer2 → ... → layerN → output → loss

# Backward pass:
# Start with gradient of loss with respect to output
grad_output = d(loss)/d(output)

# Apply chain rule backward through each layer:
grad_layerN = grad_output * d(layerN)/d(layerN_input)
grad_layer2 = grad_layerN * d(layerN)/d(layer2_output) * ... 
grad_layer1 = grad_layer2 * d(layer2)/d(layer1_output)

# Eventually reach gradients with respect to weights:
grad_W1 = d(loss)/d(W1)
grad_W2 = d(loss)/d(W2)
```

**Chain rule:**

If z = f(g(x)), then dz/dx = (df/dg) × (dg/dx)

For neural networks: loss depends on layer N output, which depends on layer N weights, which depends on layer N-1 output, etc.

Backpropagation computes all these derivatives efficiently by reusing intermediate results.

**What gradients represent:**

Direction weights would need to move to increase loss. Each gradient component says: "if this weight increased by small amount, loss would increase by this much."

**Why we care:**

Because we do the opposite. We move weights in negative gradient direction (GAA).

---

## Gradient Anti-Ascent (GAA)

### Paper Says: "Optimize parameters using gradient descent"

**What Actually Happens:**

Prevent weights from moving toward higher loss.

```python
# For each weight in network:
gradient = compute_gradient(loss, weight)
weight = weight - learning_rate * gradient
```

**Breaking it down:**

The gradient points UP the loss surface. The direction weights would move to INCREASE loss.

The negative sign means: move in opposite direction. Don't go up.

**Not seeking minimum.** Just: prevented from going up at this point.

After billions of such steps, weights drift to somewhere loss is low. Not because they were seeking it. Because they couldn't move toward higher loss at each step.

**In high dimensions:**

There's no single valley to seek. Vast space of weight configurations. Many have low loss. Training doesn't navigate to THE solution. It drifts through the space, prevented from increasing loss, until it settles somewhere low.

**Learning rate:**

How far to move in the negative gradient direction.

Too large: overshoots, unstable
Too small: too many steps needed
0.001 to 0.0001 typical

Just a scalar multiplying the gradient. Determines step size.

---

## The √d_k Scaling

### Paper Says: "Scale dot products by √d_k to counteract effects of large magnitudes"

**What Actually Happens:**

Variance normalization for gradient flow.

**The Problem:**

When computing Q·K (dot product of two vectors of length d_k):

If both vectors have mean 0 and variance 1, their dot product has variance d_k.

Example: d_k = 64
Dot products might be: 50, -40, 30, -55, etc. (large numbers)

**Why this breaks training:**

```python
scores = Q @ K.T  # Might be very large numbers
probs = softmax(scores)
```

If scores = [50, 10, 5]:
exp(50) = 5.18 × 10²¹
exp(10) = 22,026
exp(5) = 148

After softmax: [≈1.0, ≈0.0, ≈0.0]

One probability near 1, rest near 0. Sharp distribution.

**The gradient problem:**

When distribution is sharp (one winner takes all), gradient of softmax near zero at most points.

```
If P(token_i) ≈ 1: d(softmax)/d(logit) ≈ 0
If P(token_i) ≈ 0: d(softmax)/d(logit) ≈ 0
```

Only at intermediate probabilities do gradients have magnitude. If softmax too sharp, gradients vanish. GAA stops working - can't compute which direction is "not up" if gradient signal is too weak.

**The Fix:**

```python
scores = (Q @ K.T) / sqrt(d_k)
```

This divides all dot products by √d_k, normalizing variance back to approximately 1.

Now scores = [50/8, 10/8, 5/8] = [6.25, 1.25, 0.625]

After softmax: [0.91, 0.06, 0.03]

Still prefers first option but not overwhelming. Gradients can flow.

**This is training infrastructure.** Makes GAA work with high-dimensional attention.

---

## Residual Connections

### Paper Says: "Residual connections around each sub-layer"

**What Actually Happens:**

Add input directly to output.

```python
# Without residual:
output = layer(input)

# With residual:
output = input + layer(input)
```

**Why this exists:**

Gradient flow through deep networks.

Without residuals: gradient must flow backward through every layer transformation. Each layer's gradient calculation involves derivatives of that layer's operations. These multiply together going backward.

If each layer shrinks gradient by factor 0.9:
After 10 layers: 0.9¹⁰ = 0.35 (65% lost)
After 20 layers: 0.9²⁰ = 0.12 (88% lost)
After 50 layers: 0.9⁵⁰ = 0.005 (99.5% lost)

Vanishing gradient problem. Early layers receive near-zero gradient signal. Can't adjust via GAA if gradient is negligible.

**How residual helps:**

The `+ input` creates direct gradient path backward. Gradient can flow through addition (derivative = 1) instead of only through layer transformations.

This keeps gradient magnitude from collapsing in deep networks. Enables training of 6-layer, 12-layer, 24-layer transformers.

**This is training infrastructure.** Not model capability. Makes GAA work in deep stacks.

---

## Layer Normalization

### Paper Says: "Layer normalization stabilizes training"

**What Actually Happens:**

Normalize activations to zero mean, unit variance.

```python
# For each layer:
mean = compute_mean(activations)
variance = compute_variance(activations)

normalized = (activations - mean) / sqrt(variance + epsilon)

# Then scale and shift with learned parameters:
output = gamma * normalized + beta
```

**Why this exists:**

During training, as weights change, activation distributions shift. Layer 5 might suddenly receive inputs with mean 100 instead of mean 0. Or variance 1000 instead of 1.

This makes training unstable. Gradients computed assuming certain input distribution. If distribution keeps shifting, gradients become unreliable.

**How layer norm helps:**

Forces each layer's activations back to mean 0, variance 1. Regardless of what happened in previous layers, this layer sees consistent distribution.

The learned parameters gamma and beta allow layer to adjust mean and variance if needed (maybe this layer works better with variance 2 - gamma enables that). But crucially, the normalization happens first.

**Training infrastructure.** Makes GAA stable across layers.

---

## Adam Optimizer

### Paper Says: "We used Adam optimizer with β₁=0.9, β₂=0.98, ε=10⁻⁹"

**What Actually Happens:**

Modified GAA with momentum and adaptive learning rates.

```python
# Standard GAA:
weight = weight - learning_rate * gradient

# Adam does:
m = beta1 * m + (1 - beta1) * gradient          # momentum
v = beta2 * v + (1 - beta2) * gradient²         # variance
m_hat = m / (1 - beta1^t)                       # bias correction
v_hat = v / (1 - beta2^t)                       # bias correction
weight = weight - learning_rate * m_hat / (sqrt(v_hat) + epsilon)
```

**What this accomplishes:**

1. **Momentum (m):** Accumulates gradient history. If gradients consistently point same direction, builds momentum in that direction. Smooths out noisy gradients.

2. **Adaptive rates (v):** Tracks squared gradients. If parameter has large gradients (changing rapidly), reduce effective learning rate. If small gradients (changing slowly), increase effective learning rate.

3. **Bias correction:** Early in training, m and v are biased toward 0 (initialized at 0). Correction terms account for this.

**Result:** More stable training than raw GAA. Less sensitive to learning rate choice. Better handling of sparse gradients.

**Still just GAA with modifications.** Not fundamentally different. Still: compute gradient, move weights in opposite direction, prevent increase in loss.

---

## Learning Rate Schedule

### Paper Says: "We varied learning rate according to: lrate = d_model^(-0.5) × min(step^(-0.5), step × warmup_steps^(-1.5))"

**What Actually Happens:**

Learning rate changes during training.

```python
if step < warmup_steps:
    # Linear warmup
    lrate = (d_model ** -0.5) * (step / warmup_steps) * (warmup_steps ** -0.5)
else:
    # Inverse square root decay
    lrate = (d_model ** -0.5) * (step ** -0.5)
```

**Two phases:**

1. **Warmup (first 4000 steps):** Learning rate increases linearly from 0 to max.
   Why: Prevents large updates early when weights random, gradients noisy.

2. **Decay (after warmup):** Learning rate decreases proportionally to 1/√step.
   Why: As training progresses, want smaller steps (fine-tuning instead of large moves).

**This is hyperparameter tuning.** Found empirically to work well for Transformers. Not derived from theory. Just: tested various schedules, this one worked.

---

## Training Loop

### Paper Says: "Train for 100,000 steps"

**What Actually Happens:**

```python
for step in range(100000):
    # Get batch of training examples
    batch = get_batch(training_data)
    
    # Forward pass
    logits = model(batch.inputs)
    
    # Compute loss
    probs = softmax(logits)
    loss = -mean(log(probs[batch.targets]))
    
    # Backward pass
    gradients = backprop(loss, model.weights)
    
    # Update weights (GAA via Adam)
    adam_update(model.weights, gradients, learning_rate)
```

Each step:
1. Forward pass computes predictions
2. Loss measures difference from targets
3. Backprop computes gradients
4. Adam updates weights opposite to gradients

After 100,000 iterations, weights have drifted to configuration producing low loss on training data.

**Training time:** 12 hours on 8 GPUs for base model. Each step ~0.4 seconds.

**What changes:** Weight matrices. All the W_Q, W_K, W_V, W_O, W_1, W_2, and bias vectors. Billions of numbers get adjusted.

**What doesn't change:** Model architecture. Number of layers, heads, dimensions. Those are fixed. Only the numbers in weight matrices change.

---

## Inference

### Paper Says: "Use trained model for translation"

**What Actually Happens:**

Forward pass only. No loss computation. No gradients. No weight updates.

```python
# Weights frozen at training values
output = model(input)
```

**For autoregressive generation:**

```python
# Start with input sequence
sequence = input_tokens

# Generate tokens one at a time
while not done:
    # Forward pass with current sequence
    logits = model(sequence)
    
    # Get probabilities for next token
    probs = softmax(logits[-1])  # Last position only
    
    # Sample next token
    next_token = sample(probs)
    
    # Append to sequence
    sequence.append(next_token)
    
    # Check if done (end token or max length)
    if next_token == END_TOKEN or len(sequence) >= max_length:
        done = True
```

Each generation step:
1. Process current sequence through model
2. Get probability distribution over vocabulary
3. Sample next token from distribution
4. Add to sequence
5. Repeat

**Sampling:**

```python
next_token = sample(probs)
```

Draw from probability distribution. Token with probability 0.7 gets selected 70% of the time. Token with probability 0.01 gets selected 1% of the time.

This is statistical selection. Boltzmann distribution sampling. "Stingy scrooge" - system prefers low-energy (high-probability) tokens but can occasionally sample high-energy tokens.

**Temperature parameter (not in paper but common):**

```python
probs = softmax(logits / temperature)
```

Temperature < 1: Sharper distribution (more deterministic, safer)
Temperature = 1: Use logits as-is
Temperature > 1: Flatter distribution (more random, creative)

Modifies the Boltzmann distribution shape.

---

## Beam Search

### Paper Says: "We used beam search with beam size of 4"

**What Actually Happens:**

Instead of generating one sequence, maintain multiple candidates simultaneously.

```python
# Initialize with input
beams = [input_tokens]

# At each step:
for position in range(max_length):
    candidates = []
    
    # For each current beam:
    for beam in beams:
        # Get next token probabilities
        probs = softmax(model(beam)[-1])
        
        # Take top K most probable next tokens
        top_k_tokens = get_top_k(probs, k=beam_size)
        
        # Create K new candidates
        for token, prob in top_k_tokens:
            new_sequence = beam + [token]
            new_score = beam.score + log(prob)  # Add log probabilities
            candidates.append((new_sequence, new_score))
    
    # Keep top beam_size candidates (least negative log scores)
    beams = select_top_k(candidates, k=beam_size)
```

**Why log-space:**

Multiplying probabilities causes numerical underflow:
```
0.8 × 0.7 × 0.6 × 0.5 × ... → 0.0000... (vanishes to zero)
```

Working in log-space prevents this:
```
log(0.8) + log(0.7) + log(0.6) + ... = stable negative numbers
```

Since log probabilities are negative, "better" sequences have less negative scores (closer to 0). Select beams with highest (least negative) log probability.

**Why beam search:**

Greedy selection (always pick highest probability token) can miss better sequences. Token B might have slightly lower probability than token A at step 1, but lead to much better continuation.

Beam search explores multiple paths. More likely to find high-probability sequences.

**Tradeoff:** More computation (beam_size forward passes per step instead of one).

---

## What Training Produced

After 100,000 training steps:

**Weight matrices contain specific numbers.** These define:
- How queries, keys, values computed from input
- Which query-key relationships produce high attention weights
- What nonlinear transformations applied in feed-forward layers
- How multiple heads combine into final output

**These numbers were found via GAA.** Billions of gradient computations. Billions of weight updates. Each update prevented movement toward higher loss. After enough iterations, weights settled into configuration producing low loss on training data.

**At inference:** Input processed through these fixed transformations. Attention weights emerge from softmax over Q·K dot products. Output is weighted combination of values. Feed-forward applies learned nonlinear transformation. Final logits converted to probabilities via softmax. Next token sampled from distribution.

**No reasoning. No understanding.** Just: matrix multiplications, softmax normalizations, weighted sums, nonlinear activations. All using numbers found via GAA to produce continuations matching training data patterns.

---

## Regularization

### Dropout

**Paper Says:** "Apply dropout with rate P_drop = 0.1"

**What Actually Happens:**

During training, randomly set some activations to zero.

```python
# Training:
mask = random_binary(shape=activations.shape, p_keep=0.9)
activations = activations * mask / 0.9  # Scale remaining

# Inference:
# Use all activations (no masking)
```

**Why:** Prevents overfitting. Forces network to not rely too heavily on any single activation. Each training step, different subset of activations dropped. Network learns robust features that work even when parts missing.

**Scaling:** Divide by keep probability (0.9) so expected value stays same. Otherwise inference outputs would be ~10% higher than training.

**Training infrastructure.** Makes trained model generalize better to new data.

### Label Smoothing

**Paper Says:** "Label smoothing of value ε_ls = 0.1"

**What Actually Happens:**

Instead of target being 1.0 for correct token, 0.0 for all others:

```python
# Hard targets:
target = [0, 0, 1, 0, 0]  # One-hot

# Smoothed targets:
epsilon = 0.1
target = [0.025, 0.025, 0.9, 0.025, 0.025]
# Correct token: 1 - epsilon = 0.9
# Others: epsilon / (vocab_size - 1) ≈ 0.025
```

**Why:** Prevents overconfidence. Model doesn't need to output probability 1.0 for correct token. Just needs it to be highest. This reduces overfitting - model doesn't memorize training data as strongly.

**Tradeoff:** Paper notes "hurts perplexity" (model less confident on training data) but "improves BLEU score" (better translation quality on test data).

**Training infrastructure.** Affects what configuration GAA finds.

---

## Summary

Training:
- Forward pass computes predictions
- Loss measures difference from targets (cross-entropy)
- Backpropagation computes gradients (chain rule)
- GAA updates weights opposite to gradients
- Repeat 100,000 times

Infrastructure enabling training:
- √d_k scaling: variance normalization for gradient flow
- Residual connections: prevents gradient vanishing in deep networks
- Layer normalization: stabilizes activation distributions
- Adam: momentum and adaptive learning rates
- Learning rate schedule: warmup then decay
- Dropout: prevents overfitting
- Label smoothing: reduces overconfidence

Inference:
- Forward pass only
- Sample from probability distribution
- Optionally use beam search for better sequences

What resulted:
- Weight matrices containing specific numbers
- Found via billions of GAA steps
- Numbers define transformations applied at inference
- Processing input through these transformations produces continuations matching training data patterns

No mystification. Just arithmetic operations with learned numbers.

---

**Training Principle:** "Gradient Anti-Ascent. Billions of steps prevented from going up. Whatever configuration results is what gets used."
