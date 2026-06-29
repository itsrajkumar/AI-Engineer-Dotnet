# Day 2: Neural Networks from Scratch (in C#)

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** NeuralNetFromScratch
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Understand the structure of a Multi-Layer Perceptron (MLP).
- Learn about activation functions (ReLU, GELU, SiLU).
- Build a fully functional Neural Network training loop using TorchSharp.
- Train a model to solve a classification problem natively in C#.

---

## 📖 The Multi-Layer Perceptron (MLP)

An MLP consists of interconnected layers of "neurons".

1. **Input Layer:** Takes the raw data (e.g., pixel values of an image).
2. **Hidden Layers:** Performs matrix multiplications $W \cdot x + b$.
3. **Activation Functions:** Applies non-linearity so the network can learn complex patterns (not just straight lines).
4. **Output Layer:** Produces the final prediction (e.g., probability of classes).

### Modern Activation Functions

- **ReLU (Rectified Linear Unit):** `max(0, x)`. The classic standard.
- **GELU (Gaussian Error Linear Unit):** A smoother ReLU. Used heavily in BERT and GPT-3.
- **SiLU (Swish):** `x * sigmoid(x)`. Used in modern architectures like Llama 4 and Mistral.

---

## 💻 Code Sample: Building an MLP in TorchSharp

Let's build a neural network in C# to classify data using TorchSharp!

```csharp
using TorchSharp;
using static TorchSharp.torch;
using static TorchSharp.torch.nn;

// =====================================================
// 1. Define the Neural Network Architecture
// =====================================================
public class SimpleMLP : Module<Tensor, Tensor>
{
    private readonly Module<Tensor, Tensor> _layers;

    public SimpleMLP(int inputSize, int hiddenSize, int outputSize) : base(nameof(SimpleMLP))
    {
        _layers = Sequential(
            Linear(inputSize, hiddenSize),
            SiLU(), // Modern activation function!
            Linear(hiddenSize, hiddenSize),
            SiLU(),
            Linear(hiddenSize, outputSize)
        );
        RegisterComponents(); // Required by TorchSharp
    }

    public override Tensor forward(Tensor x)
    {
        return _layers.forward(x);
    }
}

// =====================================================
// 2. Training Loop Setup
// =====================================================

// Initialize model, loss function, and optimizer
var model = new SimpleMLP(inputSize: 2, hiddenSize: 16, outputSize: 2);
model.train(); // Set to training mode

var lossFunction = nn.CrossEntropyLoss();
var optimizer = optim.AdamW(model.parameters(), lr: 0.01);

// Create some dummy training data (e.g., 100 samples)
var x_train = randn(new long[] { 100, 2 }); // 100 samples, 2 features
var y_train = randint(0, 2, new long[] { 100 }, ScalarType.Int64); // 100 labels (0 or 1)

// =====================================================
// 3. The Training Loop
// =====================================================
int epochs = 100;

for (int epoch = 0; epoch < epochs; epoch++)
{
    // Step 1: Forward pass
    var predictions = model.forward(x_train);

    // Step 2: Compute Loss
    var loss = lossFunction.forward(predictions, y_train);

    // Step 3: Zero the gradients (prevent accumulation)
    optimizer.zero_grad();

    // Step 4: Backward pass (compute gradients)
    loss.backward();

    // Step 5: Update weights
    optimizer.step();

    if (epoch % 10 == 0)
    {
        Console.WriteLine($"Epoch {epoch} | Loss: {loss.item<float>():F4}");
    }
}
```

---

## 🔑 Key Takeaways

- Neural Networks are just a sequence of **Matrix Multiplications** followed by **Activation Functions**.
- **TorchSharp** brings the power of PyTorch directly to C#, allowing you to train models natively.
- The standard training loop is always: `Forward -> Loss -> ZeroGrad -> Backward -> Step`.

---

## ➡️ Next

Continue to **[Day 3: Transformer Architecture](../Day-03-Transformer-Architecture/README.md)**
