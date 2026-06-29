# Day 3: RLHF and DPO

> **Type:** 📖 Theory | **Time:** ~2 hours
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Understand the limitations of Supervised Fine-Tuning.
- Explain Reinforcement Learning from Human Feedback (RLHF).
- Understand Direct Preference Optimization (DPO).
- Compare RLHF vs DPO.

---

## 📖 The Alignment Problem

Supervised Fine-Tuning (SFT) is great for teaching a model a format, but it struggles with **Nuance and Safety**.

Imagine the prompt: *"Write a funny joke about a politician."*
- Response A: A lighthearted, universally funny joke.
- Response B: A highly offensive, partisan attack.

In SFT, if both exist in the dataset, the model might generate either. We need a way to tell the model: **"Response A is better than Response B."**

### 1. RLHF (Reinforcement Learning from Human Feedback)

This was the breakthrough that made ChatGPT possible in 2022.

**Step 1: Reward Model Training**
- We take a prompt and have the SFT model generate 4 different responses.
- Human labelers rank them from best to worst.
- We train a second model (the **Reward Model**) to act as a judge. It takes a prompt and a response, and outputs a score (e.g., +10 for good, -10 for bad).

**Step 2: PPO (Proximal Policy Optimization)**
- We use the Reward Model to automatically grade the SFT model in real-time.
- The SFT model generates a response -> Reward model grades it -> The SFT model updates its weights to maximize the score using Reinforcement Learning (PPO).

*Problem:* PPO is incredibly unstable, mathematically complex, and requires holding multiple massive models in VRAM simultaneously.

### 2. DPO (Direct Preference Optimization)

In 2023-2024, Stanford researchers proved that the complex RLHF/PPO pipeline was unnecessary. You can optimize for preferences *directly*.

**How DPO Works:**
- You collect the same human preference data: (Prompt, Winner Response, Loser Response).
- Instead of training a separate Reward Model, you use a clever mathematical trick to treat the Language Model *as its own reward model*.
- You update the weights using a standard classification loss (like in SFT) to increase the probability of the Winner and decrease the probability of the Loser.

### RLHF vs DPO in 2026

- **DPO** (and its variants like ORPO, GRPO) are the overwhelming standard for open-source and enterprise fine-tuning because they are vastly cheaper, more stable, and require fewer GPUs.
- **RLHF (PPO)** is still used by frontier labs (OpenAI, Anthropic) because, at the extreme high-end, it sometimes scales better for complex reasoning tasks.

---

## 🔑 Key Takeaways

- SFT teaches the model *how* to talk; RLHF/DPO teaches the model *what we prefer* it to say.
- **RLHF** requires a complex multi-model reinforcement learning pipeline.
- **DPO** skips the reward model and optimizes preferences directly using standard loss functions. It is the modern standard for enterprise alignment.

---

## ➡️ Next

Continue to **[Day 4: Parameter-Efficient Fine-Tuning (PEFT)](../Day-04-Parameter-Efficient-Fine-Tuning/README.md)**
