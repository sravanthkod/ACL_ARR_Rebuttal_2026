## ACL ARR 2026 Rebuttal

***

### **Response to Reviewer SpdU**


**Re: Comparison with Open Source Baselines**
We appreciate the suggestion to compare against frameworks like `llama.cpp` or `MLC-LLM`. To address this, we profiled our model using `llama.cpp` on the Samsung Galaxy S25 CPU.
*   **Performance Gap:** The `llama.cpp` CPU implementation yielded a generation speed of **11.3 tokens/sec** and a prefill speed of **15.7 tokens/sec**. In contrast, our NPU-optimized baseline (`w/o DS2D`) achieves **~19.9 tokens/sec**, and our full method (`w/ DS2D`) reaches **~44.8 tokens/sec** (see Table 5).
*   **Memory Efficiency:** The CPU execution consumed **~4.3 GB** of peak memory, whereas our quantized NPU solution operates under **2.5 GB**.
*   **Conclusion:** Comparing our NPU solution to CPU-based open-source tools would heavily skew results due to hardware offloading. Our comparison against the vendor-optimized NPU baseline (which is already 2x faster than the CPU) allows us to isolate and demonstrate the specific gains from our algorithmic contributions (DS2D and Multi-LoRA).

**Re: Clarity on Masking for CTG**
We agree that the description of the masking mechanism was brief.
*   **Mechanism:** We utilize a tree-attention mechanism. The KV-cache is logically split into a "shared prefix" (system prompt + history) and "independent branches" (stylistic variants).
*   **Implementation:** We construct a 4D attention mask where tokens in specific branches can attend to the shared prefix and their own generated tokens, but are masked (`-inf`) from attending to parallel branches. This prevents "contamination" between stylistic variants while physically sharing the memory for the prefix.
*   **Revision:** In the camera-ready version, we will include the specific mask construction pseudocode and a diagram illustrating the tensor slicing indices.

  

**Re: Clarity on Masking for CTG**
We agree that the description of the masking mechanism was brief in the main text.
*   **Mechanism:** We utilize a tree-attention mechanism. The KV-cache is logically split into a "shared prefix" (system prompt + history) and "independent branches" (stylistic variants).
*   **Implementation:** We construct a 4D attention mask where tokens in specific branches can attend to the shared prefix and their own generated tokens, but are masked (`-inf`) from attending to parallel branches. This prevents "contamination" between stylistic variants while physically sharing the memory for the prefix.
*   **Revision:** In the camera-ready version, we will leverage the additional page allowance to include the specific mask construction pseudocode and a diagram illustrating the tensor slicing indices.


***

### **Response to Reviewer xkXK**


**Re: Technical Clarifications (FlashAttention & Winograd)**
We thank the reviewer for the precise technical corrections.
*   **FlashAttention:** We agree that FlashAttention primarily optimizes memory access patterns rather than reducing FLOPs. Our usage of "computational overhead" referred to the system latency caused by memory I/O bottlenecks. We will correct this to "memory bandwidth overhead" in the final text.
*   **Linear-to-Conv:** We acknowledge that Winograd optimization typically applies to 3x3 convolutions. For 1x1 convolutions, the speedup we observed stems from the specific NPU architecture (Hexagon DSP), which has dedicated hardware paths and superior cache utilization for `Conv2d` kernels compared to generic `Linear` kernels. We will revise the text to attribute gains to NPU-specific kernel utilization rather than Winograd algorithms.

**Re: Novelty of "LoRA-as-Input"**
The reviewer notes that swapping weights is standard. While true for dynamic server-side frameworks (e.g., PyTorch), it is a major challenge in **embedded inference engines** (e.g., QNN, TFLite).
*   **The Constraint:** Mobile inference engines compile models into **immutable, static graphs** for performance. "Swapping" adapters typically requires recompiling the graph (taking minutes) or loading a new model (doubling memory).
*   **The Innovation:** We architected the graph to accept weights as **dynamic input tensors** rather than fixed constants. This allows instantaneous task switching within a *single* frozen graph without recompilation or memory spikes—a critical enabler for mobile interaction that is not natively supported by default mobile inference stacks.

**Re: Relevance of DS2D/CTG to Multi-LoRA**
These components are not orthogonal but are required to make the "One-for-All" system viable.
*   **System Latency:** The Multi-LoRA architecture introduces system overhead (memory bandwidth pressure from loading adapter inputs).
*   **Synergy:** DS2D and CTG are the necessary "accelerators" that recover the latency budget, allowing the complex multi-adapter system to run at real-time interactive speeds (40+ tok/sec), which would otherwise be too slow for commercial deployment.

**Re: DS2D vs. BiTA**
DS2D differs from BiTA by eliminating the need for dynamic training or architecture changes. DS2D uses **static forecast embeddings** derived from the frozen model itself, specifically optimized for the "no-draft-model" memory constraint of edge devices where RAM is at a premium.


***

### **Response to Reviewer 1reK**


**Re: Baselines and Reproducibility**
*   **Baselines:** As noted in our response to other reviewers, external baselines (CPU/GPU implementations) are significantly slower than NPU execution. Our chosen baseline (Vendor-Optimized NPU *without* our algorithms) provides the most rigorous isolation of our algorithmic contributions.
*   **Reproducibility:** We acknowledge the reliance on specific hardware (Galaxy S24/S25). However, our core contributions—the "LoRA-as-Input" graph structure, the DS2D tree logic, and CTG masking—are hardware-agnostic algorithmic patterns. We will release configuration files detailing the branch structures and mask patterns to aid implementation in other edge frameworks (like ExecuTorch).

**Re: Paper Length & Content**
*   **Page Count:** We submitted this as a Long Paper (allowing up to 8 pages) but aimed for conciseness. We will utilize the remaining allowable space in the camera-ready version to expand on the ablation studies (specifically isolating Multi-LoRA overhead vs. DS2D gains) and provide the missing details on quantization setups.
*   **Terminology:** We will standardize the terms "Task" (e.g., Summarization) vs. "Use Case" (e.g., Health Summary) and define all abbreviations (QAT, CTG, etc.) upon first use as suggested.

**Re: DS2D Impact on Quality**
We clarify that DS2D does not degrade output quality. It uses a verification step where the target model validates the drafted tokens. If the drafted tokens do not match the target model's verification, they are rejected. Therefore, DS2D accelerates generation **losslessly** compared to the base model's greedy/sampling distribution. We will make this explicit in the final text.


