# Week 12: Model Training and Alignment

Welcome to Week 12! A raw neural network that has only been trained to predict the next word is actually quite useless as an assistant. It won't answer your questions; it will just try to complete the text (e.g., if you say "What is the capital of France?", the base model might respond with "What is the capital of Germany?").

This week, we explore **Alignment**: how we take a raw base model and turn it into a helpful, harmless, and honest assistant using techniques like Supervised Fine-Tuning (SFT) and Reinforcement Learning from Human Feedback (RLHF).

## 🎯 Learning Objectives

By the end of this week, you will be able to:
- Understand the massive data pipelines required for Pretraining.
- Prepare instruction-following datasets for Supervised Fine-Tuning.
- Understand how RLHF and DPO (Direct Preference Optimization) steer model behavior.
- Perform Parameter-Efficient Fine-Tuning (PEFT) using LoRA / QLoRA natively in C#.
- Build robust synthetic data generation pipelines using MAF.

## 📅 Daily Schedule

| Day | Topic | Description |
|-----|-------|-------------|
| **[Day 1](Day-01-Pretraining-and-Datasets/README.md)** | Pretraining & Datasets | How base models are trained, data quality, tokenization (Tiktoken/BPE). |
| **[Day 2](Day-02-Supervised-Fine-Tuning/README.md)** | Supervised Fine-Tuning (SFT) | Teaching the model the Q&A format using high-quality prompt-response pairs. |
| **[Day 3](Day-03-RLHF-and-DPO/README.md)** | RLHF & DPO | Aligning models to human preferences without expensive reinforcement learning. |
| **[Day 4](Day-04-Parameter-Efficient-Fine-Tuning/README.md)** | LoRA & QLoRA | Fine-tuning massive models on a single consumer GPU using adapters. |
| **[Day 5](Day-05-Synthetic-Data-Generation/README.md)** | Synthetic Data Generation | Using strong models to generate training data for smaller, specialized models. |

## 🛠️ Prerequisites for Week 12

- TorchSharp installed in your projects.
- Familiarity with the Hugging Face dataset format.
- (Optional but recommended) An NVIDIA GPU with at least 8GB VRAM for local fine-tuning experiments, though cloud notebooks (like Azure ML or Google Colab) are perfectly fine.

Let's learn how to teach these models to be helpful!

➡️ **[Start Day 1](Day-01-Pretraining-and-Datasets/README.md)**
