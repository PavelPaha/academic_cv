---
layout: homepage
title: InferD - Distributed LLM Inference Engine
---

# InferD: Distributed LLM Inference Engine

**Project Type:** Engineering / Distributed Systems  
**Code:** [GitHub Repository (branch: defence)](https://github.com/sellerbto/InferD/tree/defence)

---

## 1. Overview & Inspiration

**InferD** is a framework for distributed inference of Large Language Models (LLMs) across a decentralized network of consumer-grade GPUs. Instead of requiring one massive server to hold the entire model, the model is split into pieces (stages), and different nodes in the network execute different parts of the computation graph.

### Origins of Ideas
The project synthesizes concepts from two key research areas:
*   **Pipeline Parallelism (from [Petals](https://github.com/bigscience-workshop/petals)):** The core idea of splitting a Transformer model into sequential blocks (Embedding $\to$ Layers 1-N $\to$ ... $\to$ Layers M-End $\to$ Head) and passing hidden states between nodes over the internet.
*   **Swarm Parallelism (from [SWARM](https://arxiv.org/abs/2301.11913)):** The concept of dynamic, fault-tolerant groups of nodes that can join/leave and balance the load autonomously without a central coordinator.

---

## 2. Core Architecture

The system operates as a **Peer-to-Peer (P2P) network**. There is no central master server controlling the inference.

### The Role of DHT (Distributed Hash Table)
We use a **Kademlia DHT** as the backbone for service discovery and state synchronization. Unlike centralized databases, the DHT is replicated across all participants.

**What exactly is stored in the DHT?**
In InferD, the DHT stores the **Routing Table**, which maps **Stage IDs** to **Node Addresses** and their current **Load**:
```json
{
  "stage_0": [
    {"node_id": "A", "ip": "192.168.1.5", "load": 0.2, "capacity": 10},
    {"node_id": "B", "ip": "192.168.1.6", "load": 0.8, "capacity": 12}
  ],
  "stage_1": [ ... ]
}
```
*   Nodes announce themselves: "I am hosting Stage 3."
*   Clients query: "Who is hosting Stage 3?" $\to$ DHT returns the list above.

---

## 3. How Inference Works (Pathfinding)

When a user sends a prompt, the request traverses the network "hop-by-hop".

1.  **Client Request:** The client sends text to a node hosting **Stage 0** (Embedding).
2.  **Forward Pass & Pathfinding:**
    *   Node A computes its layers.
    *   It now needs to send the hidden states to **Stage 1**.
    *   **Greedy Routing:** Node A queries the DHT for all nodes hosting Stage 1. It compares their `load` metrics and selects the **least loaded peer** (greedy heuristic).
    *   *Note: We prepared a D* Lite algorithm for more complex latency-aware routing, but the active implementation uses this robust greedy approach.*
3.  **Transmission:** Tensors are serialized (Base64) and sent via stateless HTTP RPC.
4.  **Completion:** This repeats until the final stage (LM Head) generates the token and returns it.

---

## 4. Dynamic Load Balancing

One of the main features of InferD is its ability to adapt to network conditions.

**The Problem:** In a decentralized network, some stages might become bottlenecks (e.g., 10 nodes host Stage 1, but only 1 node hosts Stage 2).

**The Solution (Balancer):**
Each node runs a background `Balancer` process that periodically checks the global load distribution via DHT.
*   **Logic:** If the node's current stage is idle (low load) but another stage is overloaded (high load), the node initiates a **Migration**.
*   **Action:** The node downloads/reloads the weights for the overloaded stage and announces its new role to the DHT.
*   **Result:** The swarm automatically "heals" bottlenecks by shifting compute resources to where they are needed most.

---

## 5. Implementation Details

*   **Model Partitioning:** `split_model.py` automatically slices HuggingFace models (e.g., Qwen) into:
    *   `FirstStage`: Embeddings + initial layers.
    *   `StageInner`: Middle transformer blocks.
    *   `LastStage`: Final layers + LM Head.
*   **Communication:** Stateless HTTP requests with JSON payloads (Base64 encoded tensors).
*   **Fault Tolerance:** If a node goes offline during inference, the `PathFinder` retries with a different peer. Dead nodes are eventually expired from the DHT.

---
[< Back to Home](./)
