# Attention Is All You Need: Translation Glossary
> Sheet 3: The Masking Matrix
> Mechanical description without mystification
> 
> Draft revision: 2025-02-02

---

## Purpose

This sheet explains how the decoder prevents positions from accessing future information. Not through "understanding" or "constraint checking." Through arithmetic: adding -∞ to specific positions before softmax.

---

## The Problem

During autoregressive generation, each position must predict the next token based only on previous tokens. Position 5 cannot see position 6, 7, 8, etc. when generating.

**Why this matters:**

Training data contains complete sequences. Without masking, position 5 could "cheat" - use information from position 6 to predict position 6. This would be:
1. Useless at inference (future doesn't exist yet)
2. Training on wrong task (predicting with oracle access)

Need mechanism to prevent positions from attending to future positions during training.

---

## The Mechanism

### Step 1: Create Mask Matrix

```python
# For sequence length T:
mask = upper_triangular(T, value=-infinity)
```

**Example for T=4:**

```
Position:  0    1    2    3
        ┌                  ┐
      0 │  0   -∞   -∞   -∞ │
      1 │  0    0   -∞   -∞ │
      2 │  0    0    0   -∞ │
      3 │  0    0    0    0 │
        └                  ┘
```

Upper triangle (above diagonal) is -∞. Diagonal and below is 0.

**What this means:**
- Position 0 can see: position 0 only (diagonal)
- Position 1 can see: positions 0, 1
- Position 2 can see: positions 0, 1, 2
- Position 3 can see: positions 0, 1, 2, 3

Each position can see itself and all previous positions. Cannot see future positions (masked with -∞).

### Step 2: Apply Mask to Attention Scores

```python
# Normal attention:
scores = Q @ K.T / sqrt(d_k)  # Shape: (T, T)

# Masked attention:
scores = (Q @ K.T / sqrt(d_k)) + mask  # Add mask
```

**Example:**

Without mask:
```
scores = [
    [2.1,  3.4,  1.8,  2.9],
    [1.5,  4.2,  3.1,  2.7],
    [3.3,  2.8,  4.5,  3.6],
    [2.2,  3.9,  2.4,  4.1]
]
```

Add mask:
```
scores + mask = [
    [2.1,  -∞,   -∞,   -∞ ],
    [1.5,  4.2,  -∞,   -∞ ],
    [3.3,  2.8,  4.5,  -∞ ],
    [2.2,  3.9,  2.4,  4.1]
]
```

### Step 3: Apply Softmax

```python
attention_weights = softmax(scores + mask)
```

**Critical property of softmax with -∞:**

```
softmax([2.1, -∞, -∞, -∞])
= [exp(2.1), exp(-∞), exp(-∞), exp(-∞)] / sum
= [exp(2.1), 0, 0, 0] / exp(2.1)
= [1.0, 0.0, 0.0, 0.0]
```

Because `exp(-∞) = 0`.

**Result after softmax:**

```
Position 0: [1.0,  0.0,  0.0,  0.0]  # Only sees position 0
Position 1: [0.18, 0.82, 0.0,  0.0]  # Only sees positions 0-1
Position 2: [0.27, 0.11, 0.62, 0.0]  # Only sees positions 0-2
Position 3: [0.08, 0.44, 0.09, 0.39] # Sees all positions 0-3
```

Wherever mask was -∞, attention weight is exactly 0.0. Cannot attend to future positions.

---

## Why This Works

### Mathematical Mechanism

The softmax function:
```
softmax(x_i) = exp(x_i) / Σ exp(x_j)
```

When one element is -∞:
```
exp(-∞) = lim_{x→-∞} exp(x) = 0
```

So positions masked with -∞ contribute:
- Numerator: 0
- Denominator: 0
- Final weight: 0 / (sum of others) = 0

Zero attention weight means zero contribution to output:
```
output = Σ (attention_weight_j × value_j)

If attention_weight_j = 0, then that term vanishes.
```

Position 5 cannot attend to position 6 because attention_weight to position 6 is forced to exactly zero by the mask.

### Why -∞ and Not Just 0?

If mask was 0 instead of -∞:

```
scores = [2.1, 3.4, 1.8, 2.9]
scores + [0, 0, 0, 0] = [2.1, 3.4, 1.8, 2.9]  # No change!
```

Need to subtract something that makes future positions have very negative scores. -∞ is the extreme case.

Could use large negative number (like -1e9), but -∞ is cleaner and guaranteed to produce exactly 0.0 after softmax.

---

## Complete Attention with Masking

```python
def masked_attention(Q, K, V, mask):
    """
    Q: Query matrix (T, d_k)
    K: Key matrix (T, d_k)
    V: Value matrix (T, d_v)
    mask: Mask matrix (T, T) with -∞ in upper triangle
    """
    
    # Compute attention scores
    scores = (Q @ K.T) / sqrt(d_k)  # (T, T)
    
    # Apply mask
    scores = scores + mask  # Broadcasting adds mask to each head
    
    # Softmax (exp(-∞) becomes 0)
    attention_weights = softmax(scores)  # (T, T)
    
    # Weighted sum of values
    output = attention_weights @ V  # (T, d_v)
    
    return output
```

**Step-by-step for position 2:**

1. Compute scores with all positions: [s₀, s₁, s₂, s₃]
2. Add mask: [s₀ + 0, s₁ + 0, s₂ + 0, s₃ + (-∞)] = [s₀, s₁, s₂, -∞]
3. Softmax: [w₀, w₁, w₂, 0.0]
4. Output: w₀×v₀ + w₁×v₁ + w₂×v₂ + 0.0×v₃

Position 3 contributes nothing. Mechanically excluded via the mask.

---

## Training vs Inference

### During Training

Full sequence available. Mask applied to prevent cheating:

```python
# Input: complete sequence [token_0, token_1, token_2, token_3]
# Target: predict each next token

# Position 0 predicts token_1 (can only see token_0)
# Position 1 predicts token_2 (can only see token_0, token_1)
# Position 2 predicts token_3 (can only see token_0, token_1, token_2)
# Position 3 predicts token_4 (can see all: token_0, token_1, token_2, token_3)

# Mask ensures each position only uses past information
```

All predictions computed in parallel (parallelization benefit of Transformers). Mask enforces proper causal structure.

### During Inference

Generate one token at a time:

```python
sequence = [token_0]

# Step 1: Generate token_1
# Input: [token_0]
# Mask not strictly needed (sequence length 1)
# Output: token_1

sequence = [token_0, token_1]

# Step 2: Generate token_2
# Input: [token_0, token_1]
# Mask prevents token_1 from seeing future (none exists)
# Output: token_2

# Continue until done...
```

During inference, mask prevents newer positions from seeing older positions incorrectly, but since sequence grows one token at a time, there is no future to accidentally see.

Mask is more critical during training where full sequence present simultaneously.

---

## Multi-Head Attention with Masking

Same mask applied to all heads:

```python
# For each head i:
Q_i = sequence @ W_Q_i
K_i = sequence @ W_K_i
V_i = sequence @ W_V_i

scores_i = (Q_i @ K_i.T) / sqrt(d_k)
scores_i = scores_i + mask  # Same mask for all heads

attention_i = softmax(scores_i) @ V_i
```

Each head computes different attention patterns, but all respect the same causal structure: no seeing the future.

---

## Encoder vs Decoder Masking

### Encoder (No Mask)

In encoder (or encoder-decoder cross-attention), positions can attend to all positions:

```python
# No mask needed
scores = Q @ K.T / sqrt(d_k)
attention_weights = softmax(scores)
```

Why: Encoder processes complete input. All positions available simultaneously. No generation happening. No future/past distinction.

### Decoder (Masked)

In decoder self-attention, must prevent future access:

```python
# Mask required
mask = upper_triangular(T, value=-infinity)
scores = (Q @ K.T / sqrt(d_k)) + mask
attention_weights = softmax(scores)
```

Why: Decoder generates sequentially. Position i should not know what token i+1 will be (it hasn't been generated yet).

### Cross-Attention (No Mask)

When decoder attends to encoder output, no mask needed:

```python
# Q from decoder, K and V from encoder
# No mask - can attend to all encoder positions
scores = Q @ K.T / sqrt(d_k)
attention_weights = softmax(scores)
```

Why: Encoder output is complete context. Decoder can reference any part of input when generating output. No causal constraint between input and output positions.

---

## Implementation Details

### Creating the Mask

```python
import numpy as np

def create_causal_mask(length):
    """
    Create upper triangular mask with -inf above diagonal
    """
    # Create matrix of zeros
    mask = np.zeros((length, length))
    
    # Set upper triangle to -inf
    mask = np.triu(mask, k=1)  # Upper triangle, 1 above diagonal
    mask = np.where(mask == 0, 0.0, -np.inf)
    
    return mask

# Example:
mask = create_causal_mask(4)
print(mask)
# [[  0. -inf -inf -inf]
#  [  0.   0. -inf -inf]
#  [  0.   0.   0. -inf]
#  [  0.   0.   0.   0.]]
```

### Broadcasting

Mask has shape (T, T). Attention scores might have shape (batch, heads, T, T).

Broadcasting adds mask to each batch and head automatically:

```python
# scores: (8, 8, 50, 50)  # batch=8, heads=8, length=50
# mask: (50, 50)

scores = scores + mask  # Broadcasts to (8, 8, 50, 50)
```

Same mask applied across all batches and all heads.

---

## Why Not Other Approaches?

### Alternative 1: Separate Forward Passes

Could run attention only on valid positions:

```
# Position 0: attend to [0]
# Position 1: attend to [0, 1]
# Position 2: attend to [0, 1, 2]
```

**Problem:** Can't parallelize. Must compute each position sequentially. Loses main benefit of Transformers.

### Alternative 2: Check During Attention

Could add conditional logic:

```python
if query_position <= key_position:
    # Can attend
else:
    # Cannot attend, set weight to 0
```

**Problem:** Conditional logic in tight loop. Very slow. GPU parallelization breaks. Mask addition is simple arithmetic (fast).

### The Mask Approach

Add -∞ matrix before softmax:
- Single addition operation
- Fully parallelizable
- GPU friendly
- Mathematically guaranteed to produce 0 weights

This is why masking is standard approach. Fast and correct.

---

## Numerical Stability

### Potential Issue

Adding -∞ could cause NaN (not a number) errors in some implementations.

### Solution

Use very large negative number instead of true -∞:

```python
mask = np.where(mask == 0, 0.0, -1e9)
```

-1e9 is negative enough that:
```
exp(-1e9) ≈ 0 (for practical purposes)
```

But avoids potential -∞ arithmetic issues.

Many implementations use -1e9 or -1e10 instead of literal -∞.

---

## Summary

**Problem:** Prevent positions from seeing future during autoregressive generation.

**Solution:** Add mask matrix with -∞ in upper triangle.

**Mechanism:**
1. Create upper triangular mask with -∞ above diagonal
2. Add mask to attention scores
3. Apply softmax (exp(-∞) = 0)
4. Result: zero attention to future positions

**Why it works:** Mathematical property of exponential function. exp(-∞) = 0, so masked positions contribute nothing to output.

**Training:** Enables parallel computation of all positions while enforcing causal structure.

**Inference:** Prevents incorrectly attending to future (though sequence grows one token at a time anyway).

No intelligence. No "understanding" that future shouldn't be accessed. Just: arithmetic operation producing zero weights where needed.

---

**Masking Principle:** "Add -∞ where attention shouldn't flow. Softmax converts to zero. Pure arithmetic constraint."
