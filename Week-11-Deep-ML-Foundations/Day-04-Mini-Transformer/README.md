# Day 4: Build a Mini-Transformer (in C#)

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** MiniGPT
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Implement Causal Self-Attention in TorchSharp.
- Construct a full Transformer Decoder Block.
- Connect token embeddings and positional encodings.
- See how this architecture scales to GPT-level models.

---

## 💻 Code Sample: Mini-GPT in TorchSharp

This is a simplified, educational implementation of a GPT-style decoder block in C#.

```csharp
using TorchSharp;
using static TorchSharp.torch;
using static TorchSharp.torch.nn;

// =====================================================
// 1. Causal Self-Attention
// =====================================================
public class CausalSelfAttention : Module<Tensor, Tensor>
{
    private readonly Module<Tensor, Tensor> _c_attn;
    private readonly Module<Tensor, Tensor> _c_proj;
    private readonly int _n_head;
    private readonly int _d_model;

    public CausalSelfAttention(int d_model, int n_head) : base(nameof(CausalSelfAttention))
    {
        _d_model = d_model;
        _n_head = n_head;

        // Combined projection for Query, Key, Value
        _c_attn = Linear(d_model, 3 * d_model);
        _c_proj = Linear(d_model, d_model);
        
        RegisterComponents();
    }

    public override Tensor forward(Tensor x)
    {
        var B = x.size(0); // Batch size
        var T = x.size(1); // Sequence length
        var C = x.size(2); // Embedding dim (d_model)

        // Calculate Q, K, V
        var qkv = _c_attn.forward(x).split(C, dim: 2);
        var q = qkv[0].view(B, T, _n_head, C / _n_head).transpose(1, 2);
        var k = qkv[1].view(B, T, _n_head, C / _n_head).transpose(1, 2);
        var v = qkv[2].view(B, T, _n_head, C / _n_head).transpose(1, 2);

        // Attention (Q*K^T) / sqrt(d_k)
        // Note: PyTorch/TorchSharp has a highly optimized scaled_dot_product_attention built in!
        var y = nn.functional.scaled_dot_product_attention(
            q, k, v, 
            is_causal: true // Applies the causal mask so it can't look into the future!
        );

        y = y.transpose(1, 2).contiguous().view(B, T, C);

        // Output projection
        return _c_proj.forward(y);
    }
}

// =====================================================
// 2. Transformer Block
// =====================================================
public class TransformerBlock : Module<Tensor, Tensor>
{
    private readonly Module<Tensor, Tensor> _ln_1;
    private readonly CausalSelfAttention _attn;
    private readonly Module<Tensor, Tensor> _ln_2;
    private readonly Module<Tensor, Tensor> _mlp;

    public TransformerBlock(int d_model, int n_head) : base(nameof(TransformerBlock))
    {
        _ln_1 = LayerNorm(d_model);
        _attn = new CausalSelfAttention(d_model, n_head);
        _ln_2 = LayerNorm(d_model);
        
        _mlp = Sequential(
            Linear(d_model, 4 * d_model),
            GELU(),
            Linear(4 * d_model, d_model)
        );

        RegisterComponents();
    }

    public override Tensor forward(Tensor x)
    {
        // Residual connections: x = x + Attention(LayerNorm(x))
        x = x + _attn.forward(_ln_1.forward(x));
        x = x + _mlp.forward(_ln_2.forward(x));
        return x;
    }
}

// =====================================================
// 3. The Full Mini-GPT Model
// =====================================================
public class MiniGPT : Module<Tensor, Tensor>
{
    private readonly Embedding _wte; // Word Token Embeddings
    private readonly Embedding _wpe; // Word Position Embeddings
    private readonly ModuleList<TransformerBlock> _blocks;
    private readonly Module<Tensor, Tensor> _ln_f; // Final LayerNorm
    private readonly Module<Tensor, Tensor> _lm_head; // Output to vocabulary

    public MiniGPT(int vocabSize, int d_model, int n_head, int n_layer, int maxSeqLen) : base(nameof(MiniGPT))
    {
        _wte = Embedding(vocabSize, d_model);
        _wpe = Embedding(maxSeqLen, d_model);
        
        _blocks = new ModuleList<TransformerBlock>();
        for (int i = 0; i < n_layer; i++)
            _blocks.Add(new TransformerBlock(d_model, n_head));

        _ln_f = LayerNorm(d_model);
        _lm_head = Linear(d_model, vocabSize, hasBias: false);

        RegisterComponents();
    }

    public override Tensor forward(Tensor idx)
    {
        var B = idx.size(0);
        var T = idx.size(1);

        // Create position indices (0, 1, 2, ... T-1)
        var pos = torch.arange(0, T, dtype: ScalarType.Int64, device: idx.device).unsqueeze(0);

        // Add Token Embeddings + Positional Embeddings
        var tok_emb = _wte.forward(idx);
        var pos_emb = _wpe.forward(pos);
        var x = tok_emb + pos_emb;

        // Pass through Transformer blocks
        foreach (var block in _blocks)
            x = block.forward(x);

        x = _ln_f.forward(x);
        
        // Project back to vocabulary size to get logits
        return _lm_head.forward(x);
    }
}
```

---

## 🔑 Key Takeaways

- You just built the exact architecture that powers GPT-3! The only difference is the size (`d_model = 768` vs `12288`, `n_layer = 12` vs `96`).
- **Residual Connections (`x = x + ...`)** are visible in the `TransformerBlock.forward` method.
- **Causal Masking** is critical. The `is_causal: true` flag prevents the model from cheating by looking at future words during training.

---

## ➡️ Next

Continue to **[Day 5: 2026 Architectures](../Day-05-Modern-Architectures/README.md)**
