# Day 4: Parameter-Efficient Fine-Tuning (PEFT)

> **Type:** 📖 Theory | **Time:** ~2 hours
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Understand why Full Fine-Tuning is often impossible.
- Explain the mechanics of LoRA (Low-Rank Adaptation).
- Understand Quantized LoRA (QLoRA) for consumer hardware.
- Review .NET options for PEFT fine-tuning.

---

## 📖 The Hardware Bottleneck

If you want to Supervised Fine-Tune (SFT) an 8 Billion parameter model, you need to load:
1. The model weights (16GB in FP16).
2. The optimizer states (e.g., Adam needs 2 states per weight = 32GB).
3. The gradients (16GB).
4. The activations for the forward pass.

Total VRAM required for an 8B model: **~80GB+**. 
A consumer RTX 4090 only has 24GB.

### 1. The Solution: LoRA

**Low-Rank Adaptation (LoRA)** is a mathematical trick to bypass this limit.

Instead of updating all 8 Billion original weights, we:
1. **Freeze** the original base model (it requires no optimizer states or gradients, saving massive memory).
2. Attach a tiny, randomly initialized "Adapter" (two small matrices) alongside the frozen layers.
3. Only calculate gradients and update the weights of the tiny Adapter.

Because the Adapter is "Low-Rank", it might only have 10 Million parameters instead of 8 Billion. The VRAM requirement drops from 80GB to 18GB!

At the end of training, you have a 40MB adapter file. When deploying, you merge this 40MB file back into the original 16GB model weights.

### 2. Pushing the Limits: QLoRA

**QLoRA (Quantized LoRA)** takes this a step further. 
Instead of loading the frozen base model in FP16 (16-bit), it loads it in 4-bit quantization (compressing it 4x). 
The model weights now take only 4GB instead of 16GB. You then train a 16-bit LoRA adapter on top of it. 

With QLoRA, you can fine-tune an 8B model on a standard gaming laptop, or a 70B model on a single 80GB enterprise GPU.

---

## 💻 PEFT in the .NET Ecosystem

In 2026, you don't necessarily have to jump to Python to fine-tune.

1. **TorchSharp:** You can implement the LoRA matrices manually (or use community PEFT libraries built on TorchSharp) to fine-tune directly in C#.
2. **Azure AI Foundry:** If you don't want to deal with hardware, you can upload your JSONL dataset to Azure AI Foundry via the C# SDK and trigger a cloud-based LoRA fine-tuning job with a single API call.
3. **ONNX Runtime Training:** Microsoft provides C# bindings to fine-tune ONNX models locally using LoRA adapters.

---

## 🔑 Key Takeaways

- **Full Fine-Tuning** is expensive and requires massive VRAM.
- **LoRA** freezes the model and trains a tiny adapter matrix, saving memory and disk space.
- **QLoRA** quantizes the frozen base model to 4-bit, enabling fine-tuning on consumer GPUs.
- You can train multiple LoRA adapters (e.g., one for C# coding, one for SQL generation) and swap them instantly at runtime over the same base model.

---

## ➡️ Next

Continue to **[Day 5: Synthetic Data Generation](../Day-05-Synthetic-Data-Generation/README.md)**
