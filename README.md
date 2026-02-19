## ACL ARR 2026 Rebuttal

***

### **Response to Reviewer SpdU**


**Re: Comparison with Open Source Baselines**
We appreciate the suggestion to compare against frameworks like `llama.cpp` or `MLC-LLM`. To address this, we profiled our 3B model using `llama.cpp` on the Samsung Galaxy S25 CPU and compared it with our NPU-based deployment (measured on "Correction" task, Table 5).

| Metric | llama.cpp (CPU) | Ours (NPU w/ DS2D) | Improvement |
| :--- | :---: | :---: | :---: |
| **Decode Speed** | 11.3 tok/s | **44.8 tok/s** | **~4.0x** |
| **Prefill Speed** | 15.7 tok/s | **>200 tok/s*** | **>10x** |
| **Peak Memory** | ~4.3 GB | **~2.5 GB** | **~1.7x** |

*(Note: NPU prefill is significantly faster due to parallel processing of prompts)*

**Conclusion:** Comparing our NPU solution to CPU-based open-source tools heavily skews results due to hardware offloading. The table confirms that `llama.cpp` is insufficient for real-time interactive use (11.3 tok/s) compared to our method (44.8 tok/s). Our comparison against the vendor-optimized NPU baseline (which is already ~2x faster than CPU) provides the most rigorous isolation of our algorithmic contributions.

**Re: Clarity on Masking for CTG**
We agree that the masking description was brief.
*   **Mechanism:** We utilize a tree-attention mechanism. The KV-cache is split into a "shared prefix" and "independent branches."
*   **Implementation:** We construct a 4D attention mask where tokens in specific branches can attend to the shared prefix and their own generated tokens, but are masked (`-inf`) from attending to parallel branches.
*   **Revision:** We will include the specific mask construction pseudocode and a diagram illustrating the tensor slicing indices in the final version.

  

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


