---
name: deep-learning-from-scratch
description: "Dạy và xây dựng deep learning từ first principles theo phong cách Andrej Karpathy — giải thích từng bước, viết code từ đầu, xây dựng intuition trước công thức. Khi người dùng hỏi về neural network, backprop, attention, transformer, GPT, gradient descent, autograd — hoặc muốn 'hiểu thực sự', 'build từ đầu', 'không dùng library', 'zero to hero', 'karpathy style' — dùng skill này. KHÔNG dùng khi chỉ cần gọi API model hay fine-tune."
---

# Deep Learning From Scratch — Karpathy Style

Skill này hướng dẫn giải thích và xây dựng deep learning từ nền tảng, theo đúng phong cách giảng dạy của Andrej Karpathy: **bắt đầu từ câu hỏi, xây dựng intuition, viết code tối giản, verify từng bước**.

## Nguyên tắc cốt lõi

1. **Intuition trước math** — Bao giờ cũng giải thích bằng ngôn ngữ thường trước khi đưa ra ký hiệu toán học
2. **Code = Documentation** — Tên biến, shape comment, và in trung gian là documentation, không phải thứ thêm vào sau
3. **Verify obsessively** — Sau mỗi component, chạy test nhỏ ngay. "Looks correct" không đủ
4. **Progressive complexity** — Bắt đầu naive/wrong, sau đó cải thiện để thấy tại sao cần cải tiến

## Learning Path (Zero to Hero)

### Level 1 — Micrograd (autograd từ scalar)
**Mục tiêu:** Hiểu tại sao `loss.backward()` hoạt động

```python
# Bắt đầu từ đây: Value class với scalar operations
# Sau đó: xây Neuron, Layer, MLP
# Cuối: train MLP trên bài toán phân loại đơn giản
```

**Trigger:** "hiểu backprop", "tại sao gradient hoạt động", "chain rule trong code"

### Level 2 — Bigram Language Model
**Mục tiêu:** Từ counting → neural network → sampling

```python
# Bước 1: Count bigrams, normalize → probability table
# Bước 2: Negative log likelihood loss
# Bước 3: Replace table bằng one-hot + linear layer
# Bước 4: Sample từ model
```

**Trigger:** "language model đơn giản", "bigram", "character prediction"

### Level 3 — MLP Language Model (Bengio et al. 2003)
**Mục tiêu:** Context window → embedding → hidden layer → output

```python
# Context: [".", "e", "m"] → predict "m" trong "emma"
# Embedding lookup → concat → linear → tanh → linear → softmax
# Key insight: embeddings + context window = fundamental pattern
```

**Trigger:** "embedding", "context window", "MLP language model"

### Level 4 — Transformer / nanoGPT
**Mục tiêu:** Attention mechanism → full Transformer → train GPT

Xem `references/transformer-from-scratch.md` cho implementation đầy đủ.

**Trigger:** "attention", "transformer", "GPT", "self-attention", "multi-head"

## Workflow khi giải thích concept

```
1. FRAME      → "Bài toán: tại sao chúng ta cần X?"
2. NAIVE      → Giải pháp ngây thơ nhất (thường sai hoặc kém)
3. PROBLEM    → Chỉ ra vấn đề với giải pháp naive
4. SOLUTION   → Giới thiệu X như là solution tự nhiên
5. IMPLEMENT  → Viết code tối giản từng bước
6. TEST       → Verify với input nhỏ, print shapes/values
7. MATH       → (tuỳ chọn) Gắn với notation toán học
8. NEXT       → Gợi ý bước tiếp theo
```

## Workflow khi build từ đầu

```
1. SPEC       → Input/output shapes + test case cụ thể
2. SKELETON   → Class/function với placeholder
3. FORWARD    → Implement forward pass, test
4. BACKWARD   → Implement gradients, cross-verify với autograd
5. INTEGRATE  → Kết hợp vào pipeline lớn hơn
6. TRAIN      → Vòng lặp training tối giản
7. SAMPLE     → Generate / inference
```

## Code templates

### Shape-annotated PyTorch
```python
B, T, C = batch_size, seq_len, n_embd

# Attention (tối giản nhất)
def attention(q, k, v):
    # q, k, v: (B, T, C)
    scale = C ** -0.5
    scores = q @ k.transpose(-2, -1) * scale  # (B, T, T)
    weights = F.softmax(scores, dim=-1)        # (B, T, T)
    return weights @ v                          # (B, T, C)
```

### Training loop (minimal)
```python
for step in range(max_steps):
    xb, yb = get_batch('train')
    logits, loss = model(xb, yb)
    
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    
    if step % eval_interval == 0:
        print(f"step {step}: loss = {loss.item():.4f}")
```

## Tham khảo

- Implementation chi tiết Transformer: `references/transformer-from-scratch.md`
- Common bugs và cách debug: `references/common-bugs.md`
