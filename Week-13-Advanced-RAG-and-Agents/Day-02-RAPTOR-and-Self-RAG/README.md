# Day 2: RAPTOR and Self-RAG

> **Type:** 📖 Theory | **Time:** ~2 hours
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Implement RAPTOR for hierarchical chunking.
- Understand the Corrective RAG (CRAG) flow.
- Implement Self-Reflective RAG (Self-RAG) using agents.

---

## 📖 RAPTOR (Recursive Abstractive Processing for Tree-Organized Retrieval)

Like GraphRAG, **RAPTOR** is designed to solve the "Global Question" problem, but without the complexity of a Knowledge Graph.

Instead of building a graph, RAPTOR builds a **Tree**:
1. Take all your base text chunks (Level 0).
2. Cluster semantically similar chunks together.
3. Have an LLM write a summary for each cluster. These summaries become Level 1 chunks.
4. Cluster the Level 1 chunks together, and have the LLM summarize them to create Level 2 chunks.
5. Repeat until the entire corpus is summarized into a single root node.

**At Query Time:**
You embed the user's question, and search against *all levels* of the tree simultaneously. 
- If they ask a specific question, the vector search finds the Leaf node (Level 0).
- If they ask a broad question, the vector search matches with a higher-level summary node (Level 2).

## 📖 Self-RAG & Corrective RAG

Naive RAG assumes that whatever the vector database returns is useful. This leads to hallucinations when the retrieval step fails.

**Agentic RAG** (Self-RAG, Corrective RAG) solves this by placing an Agent in a loop to evaluate the retrieval *before* generating the final answer.

### The Corrective RAG Flow (CRAG)

1. **Retrieve:** Get top-K chunks from the Vector DB.
2. **Evaluate (Judge Agent):** "Are these chunks relevant to the user's question?"
3. **Branch:**
   - If **Relevant**: Proceed to generation.
   - If **Irrelevant**: Discard chunks. Use a Web Search Tool to find the answer on the internet instead.
   - If **Ambiguous**: Rewrite the user's query and try retrieving again.

### The Self-RAG Flow

Self-RAG adds self-reflection to the Generation step as well.
1. The generator writes an answer based on the retrieved context.
2. A **Critic Agent** reads the generated answer and checks for hallucinations: "Is every claim in this answer explicitly supported by the retrieved context?"
3. If the Critic finds a hallucination, it forces the Generator to rewrite the answer.

---

## 💻 Conceptual Implementation with MAF

```csharp
public async Task<string> SafeRagAnswerAsync(string userQuery)
{
    // Step 1: Retrieve
    var context = await VectorSearchAsync(userQuery);

    // Step 2: Evaluate Relevance
    var isRelevant = await JudgeAgent.InvokeAsync(
        $"Query: {userQuery}\nContext: {context}\nIs this context sufficient? Reply YES or NO."
    );

    if (isRelevant.Content.Contains("NO"))
    {
        // Step 3: Fallback (e.g., Web Search)
        context = await WebSearchAsync(userQuery);
    }

    // Step 4: Generate
    var answer = await GeneratorAgent.InvokeAsync(
        $"Answer this query based ONLY on the context: {userQuery}\nContext: {context}"
    );

    // Step 5: Self-Reflection
    var critique = await CriticAgent.InvokeAsync(
        $"Does this answer contain hallucinations based on the context? Answer: {answer.Content}\nContext: {context}"
    );

    if (critique.Content.Contains("Hallucination found"))
    {
        return "I'm sorry, I couldn't find a reliable answer to your question in my knowledge base.";
    }

    return answer.Content;
}
```

---

## 🔑 Key Takeaways

- **RAPTOR** solves global reasoning by building a hierarchical tree of summaries.
- **CRAG / Self-RAG** solve hallucinations by actively judging the retrieved context and the generated answer before showing it to the user.
- These techniques turn a single API call into a complex workflow, massively increasing accuracy at the cost of higher latency and token usage.

---

## ➡️ Next

Continue to **[Day 3: Agent Swarms](../Day-03-Agent-Swarms/README.md)**
