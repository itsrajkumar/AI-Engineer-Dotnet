# Week 11: Deep ML Foundations

Welcome to Week 11! This marks the beginning of the **Advanced AI Expansion Curriculum**. Now that you've mastered building applications *using* AI (API consumers, RAG, Agents), it's time to pull back the curtain and understand how the models themselves work. 

Over the next few weeks, we will transition from using high-level abstractions like MEAI and MAF to using low-level tensor libraries like **TorchSharp** (the .NET bindings for PyTorch).

## 🎯 Learning Objectives

By the end of this week, you will be able to:
- Understand the core mathematics of Machine Learning (Loss, Gradient Descent, Backpropagation).
- Build a Multi-Layer Perceptron (MLP) from scratch in C#.
- Deconstruct the Transformer architecture ("Attention is All You Need").
- Build a mini-GPT model in TorchSharp.
- Understand modern 2026 architectures like Mixture of Experts (MoE) and Mamba.

## 📅 Daily Schedule

| Day | Topic | Description |
|-----|-------|-------------|
| **[Day 1](Day-01-ML-Fundamentals/README.md)** | ML Fundamentals | Supervised/Unsupervised learning, Loss functions, Gradient Descent, Backpropagation. |
| **[Day 2](Day-02-Neural-Networks-From-Scratch/README.md)** | Neural Networks in C# | Perceptrons, MLPs, Activation Functions, Forward/Backward pass using TorchSharp. |
| **[Day 3](Day-03-Transformer-Architecture/README.md)** | Transformer Architecture | Self-Attention, Multi-Head Attention, Positional Encoding, LayerNorm. |
| **[Day 4](Day-04-Mini-Transformer/README.md)** | Build a Mini-GPT | Implementing a decoder-only transformer in TorchSharp. |
| **[Day 5](Day-05-Modern-Architectures/README.md)** | 2026 Architectures | Mixture of Experts (Llama 4), Vision Transformers, State Space Models (Mamba). |

## 🛠️ Prerequisites for Week 11

- Strong C# skills
- Basic understanding of linear algebra (matrices, vectors)
- Install **TorchSharp** in your projects:
  ```powershell
  dotnet add package TorchSharp
  ```

Let's begin the journey into the math and code that powers modern AI!

➡️ **[Start Day 1](Day-01-ML-Fundamentals/README.md)**
