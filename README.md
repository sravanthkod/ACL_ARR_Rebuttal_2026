## ACL ARR 2026 Rebuttal

***

### **Response to Reviewer SpdU**


Here are the updated rebuttal responses for Reviewer SpdU and Reviewer 1reK including the new data for Genie and Nexa ML.

I have also kept the BiTA comparison table for Reviewer xkXK as per your previous instruction.

Response to Reviewer SpdU
code
Markdown
download
content_copy
expand_less
**Re: Comparison with Open Source Baselines**
We appreciate the suggestion to compare against frameworks like `llama.cpp` or `MLC-LLM`. To address this, we benchmarked our model against three popular on-device inference engines—`llama.cpp`, `Genie`, and `Nexa ML`—on the Samsung Galaxy S25.

**Table 1: Generation Speed (tokens/sec) Comparison**
| Metric | llama.cpp | Genie | Nexa ML | **Ours (NPU w/ DS2D)** |
| :--- | :---: | :---: | :---: | :---: |
| **Decode Speed** | 11.3 | 18.0 | 22.0 | **44.8** |
| **Speedup (Ours vs X)** | **~4.0x** | **~2.5x** | **~2.0x** | **-** |
| **Peak Memory** | ~4.3 GB | - | - | **~2.5 GB** |

**Conclusion:**
While optimized frameworks like `Nexa ML` and `Genie` outperform `llama.cpp` (reaching ~22 tok/s), they still lag significantly behind our solution (~45 tok/s). This confirms that general-purpose engines cannot fully exploit the proprietary NPU stack (Hexagon DSP) to the same extent as our hardware-aware "One-for-All" framework. Furthermore, our solution operates with significantly lower memory footprint (2.5GB vs 4.3GB on CPU), which is critical for multi-task availability.

**Re: Clarity on Masking for CTG**
We agree that the masking description was brief.
*   **Mechanism:** We utilize a tree-attention mechanism. The KV-cache is split into a "shared prefix" and "independent branches."
*   **Implementation:** We construct a 4D attention mask where tokens in specific branches can attend to the shared prefix and their own generated tokens, but are masked (`-inf`) from attending to parallel branches.
*   **Revision:** We will include the specific mask construction pseudocode and a diagram illustrating the tensor slicing indices in the final version.
Response to Reviewer xkXK
code
Markdown
download
content_copy
expand_less
**Re: Technical Clarifications (FlashAttention & Winograd)**
We thank the reviewer for the precise technical corrections.
*   **FlashAttention:** We agree that FlashAttention primarily optimizes memory access patterns rather than reducing FLOPs. Our usage of "computational overhead" referred to the system latency caused by memory bandwidth bottlenecks. We will correct this in the final text.
*   **Linear-to-Conv:** We acknowledge that Winograd optimization typically applies to 3x3 convolutions. For 1x1 convolutions, the speedup we observed stems from the specific NPU architecture (Hexagon DSP), which has dedicated hardware paths and superior cache utilization for `Conv2d` kernels compared to generic `Linear` kernels. We will revise the text to attribute gains to NPU-specific kernel utilization.

**Re: Novelty of "LoRA-as-Input"**
The reviewer notes that swapping weights is standard. While true for server-side frameworks (PyTorch), it is a major challenge in **embedded inference** (e.g., QNN, TFLite).
*   **The Constraint:** Mobile inference engines compile models into **immutable, static graphs** for performance. "Swapping" adapters typically requires recompiling the graph (taking minutes) or loading a new model (doubling memory).
*   **The Innovation:** We architected the graph to accept weights as **dynamic input tensors** rather than fixed constants. This allows instantaneous task switching within a *single* frozen graph without recompilation or memory spikes.

**Re: Relevance of DS2D/CTG to Multi-LoRA**
These components are not orthogonal but are required to make the "One-for-All" system viable. The Multi-LoRA architecture introduces system overhead. DS2D and CTG are the necessary "accelerators" that recover the latency budget, allowing the complex multi-adapter system to run at real-time interactive speeds (44+ tok/sec vs 11-22 tok/sec on other engines).

**Re: DS2D vs. BiTA**
The key distinction is **Dynamic vs. Fixed Branching**. While BiTA uses a fixed tree structure for all inputs, DS2D dynamically optimizes the branch configuration (depth/width) specific to the active **Use Case**. Since different LoRA tasks have different acceptance rates, DS2D outperforms BiTA by tailoring the speculation tree to the specific task difficulty.

**Table: Gauss L 3B Model Generation speed (toks/sec) on Samsung GS25 - BiTA vs DS2D**
| Task / Use Case | BiTA | **DS2D (Ours)** | **Improvement** |
| :--- | :---: | :---: | :---: |
| **Correction** | 45.0 | **49.6** | **+10.2%** |
| **Composer** | 37.3 | **40.4** | **+8.3%** |
| **Style** | 42.0 | **45.0** | **+7.1%** |
| **Summarization** | 36.9 | **42.0** | **+13.8%** |

As shown above, DS2D achieves consistently higher throughput by adapting to the task entropy, whereas BiTA's fixed structure is suboptimal for harder tasks like Summarization.
Response to Reviewer 1reK
code
Markdown
download
content_copy
expand_less
**Re: Baselines and Reproducibility**
*   **Baselines:** As noted in our response to Reviewer SpdU, we profiled multiple on-device frameworks (`llama.cpp`, `Genie`, `Nexa ML`) on the Galaxy S25 CPU/GPU.

**Table 1: Generation Speed (tokens/sec) Comparison**
| Metric | llama.cpp | Genie | Nexa ML | **Ours (NPU w/ DS2D)** |
| :--- | :---: | :---: | :---: | :---: |
| **Decode Speed** | 11.3 | 18.0 | 22.0 | **44.8** |
| **Peak Memory** | ~4.3 GB | - | - | **~2.5 GB** |

Our solution is **~2x faster** than the best open-source baseline (Nexa ML) and **~4x faster** than `llama.cpp`. This validates our choice to compare against the **Vendor-Optimized NPU baseline** (which runs at ~20 tok/s) to rigorously isolate our algorithmic contributions (DS2D & Multi-LoRA) rather than just hardware gains.

*   **Reproducibility:** We acknowledge the reliance on specific hardware. However, our core contributions—the "LoRA-as-Input" graph structure, the DS2D tree logic, and CTG masking—are hardware-agnostic. We will release configuration files detailing the branch structures and mask patterns to aid implementation in other edge frameworks.

**Re: Paper Length & Content**
*   **Page Count:** We submitted as a Long Paper but aimed for conciseness. We will utilize the remaining allowable space to expand on ablation studies and quantization details.
*   **Terminology:** We will standardize "Task" vs. "Use Case" and define all abbreviations (QAT, CTG, etc.) upon first use.

**Re: DS2D Impact on Quality**
We clarify that DS2D does not degrade output quality. It uses a verification step where the target model validates drafted tokens. If tokens do not match the target model's verification, they are rejected. Therefore, DS2D accelerates generation **losslessly** compared to the base model's distribution.


***

### **Response to Reviewer xkXK**


**Re: Technical Clarifications (FlashAttention & Winograd)**
We thank the reviewer for the precise technical corrections.
*   **FlashAttention:** We agree that FlashAttention primarily optimizes memory access patterns rather than reducing FLOPs. Our usage of "computational overhead" referred to the system latency caused by memory bandwidth bottlenecks. We will correct this in the final text.
*   **Linear-to-Conv:** We acknowledge that Winograd optimization typically applies to 3x3 convolutions. For 1x1 convolutions, the speedup we observed stems from the specific NPU architecture (Hexagon DSP), which has dedicated hardware paths and superior cache utilization for `Conv2d` kernels compared to generic `Linear` kernels. We will revise the text to attribute gains to NPU-specific kernel utilization rather than Winograd.

**Re: Novelty of "LoRA-as-Input"**
The reviewer notes that swapping weights is standard. While true for dynamic server-side frameworks (e.g., PyTorch), it is a major challenge in **embedded inference** (e.g., QNN, TFLite).
*   **The Constraint:** Mobile inference engines compile models into **immutable, static graphs** for performance. "Swapping" adapters typically requires recompiling the graph (taking minutes) or loading a new model (doubling memory).
*   **The Innovation:** We architected the graph to accept weights as **dynamic input tensors** rather than fixed constants. This allows instantaneous task switching within a *single* frozen graph without recompilation or memory spikes.

**Re: Relevance of DS2D/CTG to Multi-LoRA**
These components are not orthogonal but are required to make the "One-for-All" system viable. The Multi-LoRA architecture introduces system overhead (memory bandwidth pressure). DS2D and CTG are the necessary "accelerators" that recover the latency budget, allowing the complex multi-adapter system to run at real-time interactive speeds.

**Re: DS2D vs. BiTA**
The key distinction is **Dynamic vs. Fixed Branching**. While BiTA uses a fixed tree structure for all inputs, DS2D dynamically optimizes the branch configuration (depth/width) specific to the active **Use Case**. Since different LoRA tasks have different acceptance rates (entropy), DS2D outperforms BiTA by tailoring the speculation tree to the specific task.

**Table: Gauss L 3B Model Generation speed (toks/sec) on Samsung GS25 - BiTA vs DS2D**

| Task / Use Case | BiTA | **DS2D (Ours)** | **Improvement** |
| :--- | :---: | :---: | :---: |
| **Correction** | 45.0 | **49.6** | **+10.2%** |
| **Composer** | 37.3 | **40.4** | **+8.3%** |
| **Style** | 42.0 | **45.0** | **+7.1%** |
| **Summarization** | 36.9 | **42.0** | **+13.8%** |

As shown above (benchmarked on Galaxy S25, 3B Model), DS2D achieves consistently higher throughput by adapting to the task difficulty, whereas BiTA's fixed structure is suboptimal for harder tasks like Summarization.


***

### **Response to Reviewer 1reK**


**Re: Baselines and Reproducibility**
*   **Baselines:** As noted in our response to other reviewers, external baselines (CPU/GPU implementations) are significantly slower than NPU execution. Our chosen baseline (Vendor-Optimized NPU *without* our algorithms) provides the most rigorous isolation of our algorithmic contributions.
*   **Reproducibility:** We acknowledge the reliance on specific hardware (Galaxy S24/S25). However, our core contributions—the "LoRA-as-Input" graph structure, the DS2D tree logic, and CTG masking—are hardware-agnostic algorithmic patterns.

**Re: Paper Length & Content**
*   **Page Count:** We submitted this as a Long Paper (allowing up to 8 pages) but aimed for conciseness. We will utilize the remaining allowable space in the camera-ready version to expand on the ablation studies (specifically isolating Multi-LoRA overhead vs. DS2D gains) and provide the missing details on quantization setups.
*   **Terminology:** We will standardize the terms "Task" (e.g., Summarization) vs. "Use Case" (e.g., Health Summary) and define all abbreviations (QAT, CTG, etc.) upon first use as suggested.

**Re: DS2D Impact on Quality**
We clarify that DS2D does not degrade output quality. It uses a verification step where the target model validates the drafted tokens. If the drafted tokens do not match the target model's verification, they are rejected. Therefore, DS2D accelerates generation **losslessly** compared to the base model's greedy/sampling distribution. We will make this explicit in the final text.


Init Time: 504147 us
Prompt Processing Time: 70191 us, Prompt Processing Rate : 470.145752 toks/sec
Token Generation Time: 343135 us, Token Generation Rate: 26.228821 toks/sec

