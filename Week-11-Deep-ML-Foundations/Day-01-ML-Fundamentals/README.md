# Day 1: ML Fundamentals Refresher

> **Type:** 📖 Theory | **Time:** ~2 hours
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Distinguish between Supervised, Unsupervised, and Reinforcement Learning.
- Understand Loss Functions (MSE, Cross-Entropy, Contrastive Loss).
- Explain Gradient Descent and modern optimizers (Adam, AdamW).
- Understand Backpropagation conceptually.
- Prepare for TorchSharp implementation in .NET.

---

## 📖 The Core Pillars of ML

Machine learning is fundamentally about finding patterns in data using mathematics.

### 1. Types of Learning

- **Supervised Learning:** Learning a mapping from inputs $X$ to outputs $y$ given labeled pairs $(X, y)$. (e.g., predicting house prices, classifying sentiment).
- **Unsupervised Learning:** Finding structure in unlabeled data $X$. (e.g., clustering, dimensionality reduction, Autoencoders).
- **Reinforcement Learning:** An agent learns to perform actions in an environment to maximize cumulative reward. (Crucial for RLHF used in modern LLMs).
- **Self-Supervised Learning:** A form of unsupervised learning where the data itself provides the supervision. (e.g., LLMs predicting the next word in a sentence: "The cat sat on the [blank]").

### 2. Loss Functions

The "Loss" measures how far off the model's prediction $\hat{y}$ is from the true target $y$. The goal of training is to minimize this loss.

- **Mean Squared Error (MSE):** Used for regression (predicting continuous numbers).
  $L = \frac{1}{n} \sum (y_i - \hat{y}_i)^2$
- **Cross-Entropy Loss:** Used for classification (predicting discrete categories or probabilities). This is what LLMs use to predict the next token!
- **Contrastive Loss:** Used for training Embedding models. It pulls similar concepts together in vector space and pushes dissimilar ones apart.

### 3. Gradient Descent & Optimizers

Once we have a Loss, how do we update the model's weights to make the loss smaller? **Gradient Descent.**

Imagine standing on a foggy mountain (the loss landscape) trying to reach the bottom. You look at the slope under your feet (the gradient) and take a step downhill.

- **Learning Rate ($\eta$):** How big of a step you take. Too small = takes forever. Too big = you overshoot the valley.
- **SGD (Stochastic Gradient Descent):** Taking steps using small batches of data.
- **Adam / AdamW:** The modern standard. It adapts the learning rate for each individual weight based on past gradients (momentum), preventing getting stuck in shallow valleys.

### 4. Backpropagation

Backpropagation ("backprop") is the algorithm used to calculate the gradients (the slope) for every single weight in the network, working backwards from the loss at the output layer all the way to the input layer using the calculus Chain Rule.

---

## 💻 Conceptualizing in .NET

While Python is the lingua franca of ML, we can do this in C# using **TorchSharp**.

```csharp
using TorchSharp;
using static TorchSharp.torch;

// Create a tensor (a multi-dimensional matrix)
var x = tensor(new float[] { 1.0f, 2.0f, 3.0f });

// Define a simple linear model (y = Wx + b)
var linear = nn.Linear(3, 1);

// Forward pass
var prediction = linear.forward(x);

// Calculate Loss (MSE)
var target = tensor(new float[] { 10.0f });
var loss = nn.functional.mse_loss(prediction, target);

// Backward pass (calculate gradients)
loss.backward();

// The optimizer will now update the weights!
```

---

## 🔑 Key Takeaways

- **Loss** tells you how bad the model is.
- **Backpropagation** figures out exactly which weights are to blame.
- **Optimizers (AdamW)** adjust the weights to lower the loss.
- **Self-Supervised Learning** (next-token prediction) is the foundation of modern LLMs.

---

## ➡️ Next

Continue to **[Day 2: Neural Networks from Scratch](../Day-02-Neural-Networks-From-Scratch/README.md)**
