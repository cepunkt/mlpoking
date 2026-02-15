# Attention Is All You Need: Translation Glossary
> Sheet 1: Architecture & Mechanism (Version 2)
> Mechanical description without mystification
> 
> Draft revision: 2025-02-02

---

## Purpose

This glossary translates terminology from "Attention Is All You Need" (Vaswani et al., 2017) into direct mechanical description. No teleology. No mystification. Just what the math does.

Each entry shows: what the paper says, what actually happens, the arithmetic involved.

---

## Core Training Mechanism

### Paper Says: "Training" / "Learning" / "Optimization"

**What Actually Happens:**

Gradient Anti-Ascent (GAA).

```python
# For each training example:
loss = -log P_θ(target_token | context)
gradients = compute_gradients(loss, all_weights)

# Update all weight matrices and biases:
for weight in model:
    weight = weight - learning_rate * gradient
```

At each step, weights cannot move toward higher loss. The gradient points UP the loss surface. The negative sign prevents movement in that direction.

Billions of iterations. Each time: "can't go up from here." Parameters drift through high-dimensional space until they settle somewhere loss is low.

**Why "Anti-Ascent" not "Descent":**

In 2D diagrams, gradient descent looks like seeking a goal - ball rolling toward valley bottom. Teleological framing sneaks in.

In high dimensions: at each point, parameters prevented from moving UP. That's the only constraint. Not "moving toward optimum" but "prevented from getting worse."

After billions of steps of "can't go worse," whatever configuration results is what gets used.

---

## Architecture Components

### Hidden State → Stingy Scrooge (Statistical Selection)

**Paper terminology:** "Hidden state h_t as function of previous hidden state h_{t-1}"

**What actually happens:**

At each token position, values computed through matrix operations get passed to next layer. These values represent current state of computation. "Hidden" because internal to network, not visible to user.

The "stingy scrooge" framing: token selection is statistical. Given current state, next token selected from probability distribution:

```python
logits = compute_logits(current_state)  # numbers
probs = softmax(logits)                 # probability distribution
next_token = sample(probs)              # statistical selection
```

Probability favors low-cost (high-probability) tokens. The system is "miserly" - always prefers cheaper continuations in probability space.

**Governing math:** Boltzmann distribution.

```
P(token) ∝ exp(-E(token))
```

Where E is "energy" or "cost" - low energy means high probability. The system samples from this distribution. More probable tokens get selected more often.

---

### Attention Mechanism → Weighted Context Aggregation

**Paper terminology:** "Attention mechanism allowing modeling of dependencies"

**What actually happens:**

Matrix operations computing weighted sums.

```python
# Given context of T tokens:
Q = context @ W_Q  # Query: what each position is looking for
K = context @ W_K  # Key: what each position offers
V = context @ W_V  # Value: information each position carries

# Compute attention weights:
scores = Q @ K.T / sqrt(d_k)  # dot products, scaled
weights = softmax(scores)     # normalize to probabilities

# Weighted sum:
output = weights @ V
```

Step by step:
1. Linear projections create Q, K, V matrices
2. Dot products measure similarity between queries and keys
3. Softmax normalizes to probability distribution
4. Weighted sum aggregates values according to probabilities

No "attending." No "dependencies." Just: matrix multiplication, softmax normalization, weighted averaging.

**Why it works:**

Dot product Q·K measures similarity. High similarity = high weight = position contributes more to output. Low similarity = low weight = position contributes less.

The weight matrices (W_Q, W_K, W_V) encode which relationships matter. These numbers got adjusted during training via GAA.

---

### Self-Attention → Context Aggregating Against Itself

**Paper terminology:** "Self-attention relating different positions of single sequence"

**What actually happens:**

Same operation as attention, but Q, K, V all derive from same input sequence.

```python
# Input: sequence of token embeddings
# Output: sequence of aggregated representations

Q = sequence @ W_Q
K = sequence @ W_K  
V = sequence @ W_V

# Each position computes weighted sum of all positions
attention_output = softmax(Q @ K.T / sqrt(d_k)) @ V
```

Position i's output is weighted combination of all positions' values, where weights come from similarity between position i's query and each position's key.

This creates position-to-position relationships within the sequence. Token 5 might "attend strongly" to tokens 1 and 3, meaning: the similarity Q₅·K₁ and Q₅·K₃ was high, so V₁ and V₃ contribute more to position 5's output.

**Why "self":**

Not comparing to external information. The sequence evaluates against itself. Every position computing weighted combination of every other position.

---

### Multi-Head Attention → Parallel Independent Aggregations

**Paper terminology:** "Multi-head attention allows jointly attending to information from different representation subspaces"

**What actually happens:**

Run the attention operation multiple times in parallel with different weight matrices.

```python
# 8 heads example:
for i in range(8):
    Q_i = sequence @ W_Q_i  # Each head has own projection matrices
    K_i = sequence @ W_K_i
    V_i = sequence @ W_V_i
    
    head_i = softmax(Q_i @ K_i.T / sqrt(d_k)) @ V_i

# Concatenate all heads and project:
output = concat(head_1, head_2, ..., head_8) @ W_O
```

Each head computes attention independently. Different W_Q, W_K, W_V means different relationships get encoded. Head 1 might learn positional patterns. Head 2 might learn semantic patterns. Head 3 something else.

All computed in parallel. Results concatenated and projected through final weight matrix W_O.

**Why multiple heads:**

More capacity. Eight heads = eight different ways to aggregate context. More flexible than single aggregation. But it's just the same operation repeated with different weight matrices.

---

### Query / Key / Value → Components of Weighted Aggregation

**Paper terminology:** "Queries, keys, and values where output is weighted sum of values"

**What actually happens:**

Linear projections of input creating three different views:

```python
Q = input @ W_Q  # Query matrix (T × d_k)
K = input @ W_K  # Key matrix (T × d_k)
V = input @ W_V  # Value matrix (T × d_v)
```

- **Query (Q):** For each position, vector representing "what I'm looking for"
- **Key (K):** For each position, vector representing "what I offer"
- **Value (V):** For each position, information to be aggregated

The dot product Q·K measures compatibility. High dot product = high compatibility = position contributes more.

Softmax over Q·K produces weights. These weights applied to V produce output.

**Concrete example:**

Token at position 5 has query Q₅. This gets compared against keys of all positions (K₁, K₂, ..., K_T) via dot products. Softmax over these scores produces weights. These weights aggregate the values (V₁, V₂, ..., V_T).

Position 5's output = weighted combination of all values, weighted by query-key similarity.

---

### Encoder / Decoder → Processing Stages

**Paper terminology:** "Encoder maps input to continuous representations; decoder generates output sequence"

**What actually happens:**

**Encoder:** Stack of identical layers, each containing:
1. Self-attention (each position aggregates all input positions)
2. Feed-forward network (nonlinear transformation)
3. Residual connections and normalization

Input sequence goes through all layers. Output is processed representation of entire input.

**Decoder:** Similar stack, but with masking and cross-attention:
1. Masked self-attention (position can only see past, not future)
2. Cross-attention to encoder output
3. Feed-forward network
4. Residual connections and normalization

Generates output one token at a time. Each new token becomes input for next step.

**Why separate stages:**

Machine translation task structure. Encoder processes source language. Decoder generates target language. Cross-attention lets decoder reference encoded source while generating.

For decoder-only models (like GPT), only the decoder stack exists. No separate encoder.

---

### Learned Parameters → Weight Matrices and Biases

**Paper terminology:** "Model parameters learned through training"

**What actually happens:**

Numbers in matrices and bias vectors. These encode token-to-token relations.

For attention:
- W_Q: How to compute queries (d_model × d_k matrix)
- W_K: How to compute keys (d_model × d_k matrix)
- W_V: How to compute values (d_model × d_v matrix)
- W_O: How to project concatenated heads (h·d_v × d_model matrix)

For feed-forward:
- W_1: First layer weights (d_model × d_ff matrix)
- b_1: First layer bias (d_ff vector)
- W_2: Second layer weights (d_ff × d_model matrix)
- b_2: Second layer bias (d_model vector)

Training adjusts these numbers via GAA. At each training step, gradients computed. Numbers shifted in direction opposite to gradient (prevented from moving toward higher loss).

After billions of such adjustments across training examples, numbers settle into configuration that produces low loss on training data.

**What these numbers do:**

They define linear transformations. When you compute Q = input @ W_Q, you're multiplying input vectors by this matrix of numbers. Different numbers = different transformation = different Q values = different attention patterns.

The "learning" is: numbers changed until the transformations they encode produce correct predictions on training data.

---

### Positional Encoding → Position Information Added to Input

**Paper terminology:** "Positional encodings injecting information about position"

**What actually happens:**

Fixed patterns added to input embeddings before first layer.

```python
# For each position pos and dimension i:
PE(pos, 2i)   = sin(pos / 10000^(2i/d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))

# Add to input:
input_with_position = token_embedding + PE(position)
```

Different positions get different patterns. Position 0 has different PE values than position 10.

**Why necessary:**

Attention is permutation-invariant. Without positional encoding, "cat sat mat" and "mat sat cat" would be identical to the attention mechanism. The position information breaks this - makes position 1 different from position 2.

**Why sine/cosine:**

Creates unique pattern for each position. Also allows model to learn relative positions - PE(pos+k) can be expressed as linear function of PE(pos).

Not learned during training. Fixed mathematical function.

---

### Residual Connection → Bypass Path for Gradients

**Paper terminology:** "Residual connections around each sub-layer"

**What actually happens:**

Add input directly to output: `output = LayerNorm(input + Sublayer(input))`

```python
# Without residual:
output = layer(input)

# With residual:
output = input + layer(input)
output = layer_norm(output)
```

**Why it matters:**

Gradient flow during training. In deep networks without residuals, gradients diminish exponentially as they backpropagate through layers. Signal becomes too weak to adjust early layers.

With residuals, gradient has direct path backward. This enables training of deep networks - gradients can flow through the bypass path even if they vanish through the layer transformation.

**This is training infrastructure**, not model capability. Residuals make GAA work in deep stacks.

---

### Layer Normalization → Activation Distribution Control

**Paper terminology:** "Layer normalization after residual connection"

**What actually happens:**

Normalize activations to zero mean, unit variance:

```python
mean = compute_mean(x)
variance = compute_variance(x)
normalized = (x - mean) / sqrt(variance + epsilon)
output = gamma * normalized + beta
```

Gamma and beta are learned scaling/shift parameters.

**Why it matters:**

Stabilizes training. Without normalization, activation distributions shift as weights change. This makes training unstable - gradients explode or vanish.

With normalization, each layer receives consistent input distribution regardless of what previous layers do. This enables stable gradient flow.

**This is training infrastructure**, not model capability.

---

### Feed-Forward Network → Nonlinear Transformation

**Paper terminology:** "Position-wise feed-forward network"

**What actually happens:**

Two linear transformations with ReLU in between:

```python
FFN(x) = max(0, x @ W_1 + b_1) @ W_2 + b_2
```

Expands to higher dimension (d_ff = 2048), applies nonlinearity, projects back down (d_model = 512).

Applied independently at each position - same transformation everywhere, but operating on different inputs.

**Why it's there:**

Adds capacity beyond what attention provides. Attention is linear aggregation (weighted sum). Feed-forward adds nonlinear transformation capacity. The ReLU enables learning nonlinear relationships.

Together: attention aggregates context, feed-forward transforms it nonlinearly.

---

## Training vs Inference

### Training

```python
for epoch in range(num_epochs):
    for batch in training_data:
        # Forward pass
        predictions = model(batch.input)
        
        # Compute loss
        loss = cross_entropy(predictions, batch.targets)
        
        # Backward pass (compute gradients)
        gradients = backprop(loss, model.weights)
        
        # Update weights (GAA)
        for weight, grad in zip(model.weights, gradients):
            weight = weight - learning_rate * grad
```

Each iteration: forward pass computes predictions, loss measures difference from targets, backprop computes gradients, weights update via GAA.

Billions of iterations. Each time weights prevented from moving toward higher loss. Eventually settle into configuration with low training loss.

### Inference

```python
# Weights frozen - no updates
output = model(input)
```

Just the forward pass. No loss computation. No gradients. No weight updates.

The weight matrices used are whatever GAA produced during training. Those numbers now fixed. They define the transformations applied at inference.

---

## What Training Produced

After training, the model has specific numbers in all its weight matrices. These numbers encode:

- Which query-key relationships activate strongly (high dot products)
- How values get aggregated (through learned V projections)  
- What nonlinear transformations occur (feed-forward weights)
- Position-to-position relationships across the sequence

Training via GAA found a configuration of these numbers that produces low loss on training data.

At inference, input sequences get processed through these fixed transformations. The attention weights that emerge (softmax over Q·K) determine how context aggregates. The final logits determine probability distribution over next tokens.

The model isn't "understanding" or "reasoning." It's processing input through learned linear transformations and nonlinear activations. The specific transformations were found via GAA to produce continuations matching training data patterns.

---

## Bottom Line

The paper describes architecture built for machine translation. This glossary describes what that architecture mechanically does:

- Matrix multiplications computing Q, K, V
- Dot products measuring similarity  
- Softmax producing probability distributions
- Weighted sums aggregating information
- Nonlinear transformations adding capacity
- All weights found via Gradient Anti-Ascent on training data

No mystification. No teleology. Just matrix operations with learned weights.

The weights changed through billions of GAA steps until they produced low loss. What configuration resulted is what gets used.

---

**Translation Principle:** "Describe the math. Not what it means. What it does."
