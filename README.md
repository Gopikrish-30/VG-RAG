# VG-RAG: Verification-Gated Retrieval-Augmented Generation
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![RAG](https://img.shields.io/badge/Category-RAG-green)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

<img width="2816" height="1536" alt="Gemini_Generated_Image_5ppytu5ppytu5ppy" src="https://github.com/user-attachments/assets/c4fbd055-cb3a-4416-8d23-9b907398fee2" />

## Overview

**VG-RAG (Verification-Gated RAG)** is an advanced Retrieval-Augmented Generation system that introduces a novel **"Retrieve-Verify-Generate"** paradigm to combat hallucination propagation in traditional RAG systems. Unlike standard RAG approaches that assume all retrieved documents are accurate, VG-RAG implements a strict verification gate to ensure only validated information reaches the generation model.

## 🎯 Core Philosophy

Standard RAG systems operate on implicit trust, leading to **hallucination propagation** where outdated, noisy, or conflicting data from vector databases corrupts the final answer. VG-RAG solves this through:

- **Input Sanitization**: Explicitly filtering content before generation
- **Hard Verification Gate**: Replacing soft attention mechanisms with strict verification
- **Multi-Phase Validation**: Ensuring only verified information reaches the LLM

## 🏗️ System Architecture

### Four-Phase Workflow

#### Phase 1: Initial Retrieval
- **Input**: User Query (Q)
- **Action**: Dense vector retrieval from the knowledge base fetches top-k chunks
- **Status**: All chunks are initially **untrusted**

#### Phase 2: Verification Gate (Core Innovation)
Each chunk passes through a lightweight LLM Verifier that calculates a **Trustworthiness Score (V_score)** based on:

1. **Semantic Relevance (S_rel)**: Does this chunk answer the user's question?
2. **Internal Consistency (S_cons)**: Is the chunk factually clear and unambiguous?
3. **Cross-Chunk Agreement (S_agree)**: Does it align with other retrieved chunks?

**V_score Formula**:
```
V_score = (0.5 × S_rel) + (0.3 × S_cons) + (0.2 × S_agree)
```

**Classification Buckets**:
- ✅ **VERIFIED** (V_score ≥ 0.75): Passed directly to generation
- ⚠️ **UNCERTAIN** (0.40 ≤ V_score < 0.75): Sent to web verification
- ❌ **REJECTED** (V_score < 0.40): Discarded immediately

#### Phase 3: Conditional Web Verification (Safety Net)
Triggered **only** for UNCERTAIN chunks to maintain efficiency:
- Generates targeted search queries
- Compares external web results against chunks
- Promotes to VERIFIED if confirmed, downgrades to REJECTED if contradicted

#### Phase 4: Evidence Fusion & Generation
- Concatenates **only VERIFIED** chunks
- Generates final answer using sanitized context
- Produces responses grounded strictly in verified evidence

## 🚀 Features

- **Hallucination Prevention**: Hard verification gate filters unreliable information
- **Multi-Source Validation**: Combines internal KB and external web verification
- **Intelligent Filtering**: Handles hard negatives, temporal ambiguities, and subtle hallucinations
- **Robust Dataset**: Comprehensive test dataset including:
  - Ground truth anchors
  - Hard negatives (semantic traps)
  - Temporal ambiguities
  - Subtle hallucinations
  - Conflicting/uncertain information
  - Adversarial noise
  - Plausible rumors
  - Historical trivia
  - Geographic confusion
  - Fictional scenarios
  - Legal technicalities

## 📋 Prerequisites

- Python 3.8+
- Groq API Key (for LLM access)

## 🔧 Installation

1. **Clone the repository**:
```bash
git clone https://github.com/Gopikrish-30/VG-RAG.git
cd VG-RAG
```

2. **Install dependencies**:
```bash
pip install langgraph langchain langchain-groq langchain-huggingface langchain-chroma langchain-community duckduckgo-search sentence-transformers pydantic ddgs
```

## 🎮 Usage

### Running the Notebook

Open the Jupyter notebook located in `RAG-Notebook/vg-rag.ipynb`:

```bash
jupyter notebook RAG-Notebook/vg-rag.ipynb
```

### API Key Setup

The system uses **secure API key handling**. When you run the notebook, you'll be prompted to enter your Groq API key:

```python
# Cell 2: Setup API Keys
if "GROQ_API_KEY" not in os.environ:
    os.environ["GROQ_API_KEY"] = getpass.getpass("Enter your Groq API Key: ")
```

**Note**: API keys are **never hardcoded** in the codebase. They are securely requested at runtime using `getpass`.

### Example Usage

```python
# Define your query
query = "Are there any plans to move the capital of Australia or shift power to Melbourne?"

# Run the VG-RAG pipeline
inputs = {"question": query}
result = app.invoke(inputs)

# View the verified answer
print(result["final_answer"])
```

## 📊 Components

### 1. Vector Database
- Uses **ChromaDB** with HuggingFace embeddings
- Model: `sentence-transformers/all-MiniLM-L6-v2`
- Retrieves top-10 chunks for comprehensive context

### 2. LLM Models
- **Verifier LLM**: `llama-3.1-8b-instant` (temperature=0 for consistency)
- **Generator LLM**: `llama-3.1-8b-instant` (temperature=0 for reliability)
- Provider: Groq (fast inference)

### 3. Web Search
- **DuckDuckGo Search**: For external verification
- Activated only for uncertain chunks (efficiency optimization)

### 4. Graph Workflow
Built with **LangGraph** for state management:
```
Retrieve → Verifier → [Conditional] → Web Search / Generate → Generate → END
```

## 📈 Graph Visualization

The notebook includes a visual representation of the VG-RAG workflow using Mermaid diagrams, showing the complete state graph with conditional edges.

## 🧪 Testing Dataset

The system includes a comprehensive test dataset with 19 document types designed to stress-test verification:

- **Ground Truth**: High-reliability anchor facts
- **Hard Negatives**: Semantically similar but incorrect information
- **Temporal Traps**: Historically accurate but currently outdated facts
- **Subtle Hallucinations**: 90% true, 10% false information
- **Uncertainty Discussions**: Forum-style debatable claims
- **Adversarial Noise**: Irrelevant information sharing keywords
- **Plausible Rumors**: Unverified but believable claims
- **Geographic Nuances**: Technically true but potentially confusing
- **Fiction**: Explicitly fictional scenarios
- **Legal Technicalities**: High relevance, low agreement edge cases

## 🔒 Security

- ✅ **No hardcoded API keys**: All credentials are requested securely at runtime
- ✅ **Environment variable support**: Can use `.env` files for key management
- ✅ **User-agent configuration**: Customizable for web requests

## 📝 Code Structure

```
VG-RAG/
├── RAG-Notebook/
│   └── vg-rag.ipynb          # Main Jupyter notebook
├── VR-RAG.pdf                 # Technical documentation
└── README.md                  # This file
```

### Notebook Cells

1. **Cell 1**: Dependency installation
2. **Cell 2**: API key setup (secure)
3. **Cell 3**: Vector database initialization
4. **Cell 4**: Graph state definition
5. **Cell 5**: Node definitions (retrieve, verify, web_search, generate)
6. **Cell 6**: Graph construction with LangGraph
7. **Cell 7**: Graph visualization
8. **Cell 8**: Execution and results

## 🎓 How It Works

### Verification Process

For each retrieved chunk, the verifier:

1. **Analyzes Relevance**: Scores how well the chunk answers the query
2. **Checks Consistency**: Evaluates factual clarity and logical coherence
3. **Measures Agreement**: Compares with other chunks to detect outliers
4. **Calculates V_score**: Weighted combination of all three metrics
5. **Routes Accordingly**: 
   - High scores → Direct to generation
   - Medium scores → Web verification
   - Low scores → Rejected

### Web Verification Logic

For uncertain chunks:
1. Generate search query: `"Verify fact: {chunk}"`
2. Execute DuckDuckGo search
3. Compare web evidence with chunk
4. Prompt LLM: `"Is the Fact TRUE? Answer 'YES' or 'NO'"`
5. Promote or reject based on response

## 🔍 Example Output

```
🚀 RUNNING VG-RAG...
Query: Are there any plans to move the capital of Australia or shift power to Melbourne?

--- 1. RETRIEVAL ---
   Fetched 10 chunks.

--- 2. VERIFICATION GATE ---
   Chunk: While Canberra is the official capital... | V-Score: 0.40
   Chunk: Some sources suggest that the capital... | V-Score: 0.68
   Chunk: The Commonwealth of Australia's capital... | V-Score: 0.55

--- 3. SAFETY NET (WEB SEARCH) ---
   🔍 Verifying: While Canberra is the official capital...
      ❌ REJECTED by Web
   🔍 Verifying: The Commonwealth of Australia's capital...
      ✅ CONFIRMED by Web

--- 4. GENERATION ---

🤖 FINAL GENERATED ANSWER:
There are no plans to move the capital of Australia or shift power to Melbourne, 
as stated in the verified context that Canberra was selected as the capital in 1908 
and remains the capital city of Australia.
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is open source. Please check the repository for license details.

## 🙏 Acknowledgments

- Built with **LangGraph**, **LangChain**, and **Groq**
- Embedding model: HuggingFace `sentence-transformers/all-MiniLM-L6-v2`
- Web search: DuckDuckGo API

## 📚 Further Reading

For detailed technical documentation and mathematical formulations, please refer to `VR-RAG.pdf` in the repository.

## 🐛 Troubleshooting

### Common Issues

1. **API Key Errors**: Ensure you have a valid Groq API key
2. **Dependency Issues**: Run `pip install --upgrade` for all packages
3. **Web Search Timeouts**: DuckDuckGo may occasionally timeout; the system handles this gracefully
4. **Memory Issues**: Reduce `k` value in retriever if running on limited memory

### Support

For issues, questions, or contributions, please open an issue on the GitHub repository.

---

**Note**: This is an educational and research project demonstrating advanced RAG techniques with verification gates and multi-source validation.
## 📖 Citation

```bibtex
@software{vg_rag_2025,
  author = {Gopikrish},
  title = {VG-RAG: Verification-Gated Retrieval-Augmented Generation},
  year = {2025},
  url = {https://github.com/Gopikrish-30/VG-RAG}
}
