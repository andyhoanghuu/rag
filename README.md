**Llama 3 Offline AI Model Report**
===================================

1.  **Using Offline AI models**
    

Tools for running an offline AI model:

*   Ollama: [https://ollama.com/](https://ollama.com/)
    
*   LM Studio: [https://lmstudio.ai/](https://lmstudio.ai/)
    

\-> Download phiên bản cho Windows và cài đặt.

1.  **Tool Ollama**
    

*   Run command
    

**ollama run llama3.1**

Tìm các phiên bản model và version khác tại: [https://ollama.com/library](https://ollama.com/library)

Ollama sẽ download model về máy và được lưu tại:**C:\\Users\\{username}\\.ollama\\models**

Lựa chọn model khi chat:

Các conversion sẽ được lưu trong DB file **db.sqlite**

**C:\\Users\\{username}\\AppData\\Local\\Ollama**

1.  **Tool LM Studio**
    

*   Search AI Model trong phần Model Search và tải xuống model
    

Khi tải xong, model sẽ được lưu tại:

**C:\\Users\\{username**}**\\.lmstudio\\models\\lmstudio-community**

Lựa chọn model khi chat:

Các conversion sẽ được lưu tại:

**C:\\Users\\{username}\\.lmstudio\\conversations**

1.  **Training Model**
    

1.  **LLM Finetune with Unsloth**
    

**Website:** [**https://unsloth.ai/**](https://unsloth.ai/)

Thay vì phải **Full Fine-Tuning** với chi phí rất cao,  **Unsloth** cho phép fine-tuning thêm các phần nhỏ, sau đó merge chúng lại với model gốc.

Có 2 loại Fine-tune mà Unsloth cho phép, đó là **Lora** & **QLora**.

 • **LoRA**: Fine-tunes small, trainable matrices in **16-bit** without updating all model weights.

 • **QLoRA**: Combines LoRA with **4-bit** quantisation to handle very large models with minimal resources.

**Step to Finetune:**

1.   Choose the Right Model + Method
    
2.   Prepare Your Dataset: You will need to create a dataset, usually with 2 columns - question and answer. The quality and amount will largely reflect the result of your fine-tuning, so it's imperative to get this part right.
    
3.  Select training parameters
    
4.  Train, save weight.
    
5.  Test the finetuned model.
    

**Do máy local khá yếu, ta có thể test fine-tune trên Google Colab bằng các docs bên dưới:**

[**https://unsloth.ai/docs/get-started/fine-tuning-llms-guide/tutorial-how-to-finetune-llama-3-and-use-in-ollama**](https://unsloth.ai/docs/get-started/fine-tuning-llms-guide/tutorial-how-to-finetune-llama-3-and-use-in-ollama)

[**https://colab.research.google.com/github/unslothai/notebooks/blob/main/nb/Llama3\_(8B)-Ollama.ipynb**](https://colab.research.google.com/github/unslothai/notebooks/blob/main/nb/Llama3_(8B)-Ollama.ipynb)


Sau khi test với việc training 1 dữ liệu internal, ta có kết quả:

1.  **Build RAG**
    

(Giải pháp kết hợp hoặc thay thế Fine-tuning)

**Retrieval-Augmented Generation (RAG)** là một khung AI giúp tối ưu hóa mô hình ngôn ngữ lớn (LLM) bằng cách truy xuất thông tin từ các nguồn dữ liệu bên ngoài (cơ sở tri thức nội bộ, Internet) trước khi tạo câu trả lời.

Thay vì phải training trực tiếp vào model, ta có thể xây dựng một hệ thống truy xuất dữ liệu để model AI có thể truy cập và trả lời những câu hỏi nằm trong nguồn dữ liệu mong muốn.

Về cơ bản, có 2 phase chính.

### **Phase 1: Indexing (Chuẩn bị)**

Document → Chia nhỏ chunks → Embedding model → Vectors → Vector DB

**Ví dụ:**

Doc: "Python là ngôn ngữ lập trình phổ biến"

→ Embedding model (OpenAI, Cohere, etc.)

→ Vector: \[0.12, -0.34, 0.56, ...\]

→ Lưu vào Vector DB (Pinecone, Weaviate, Chroma)

### **Phase 2: Retrieval (Khi có query)**

User query → Embedding model → Query vector → 

Tìm kiếm vectors gần nhất → Lấy documents tương ứng(context)→ Truyền vào custom prompt → LLM → Trả lời kết quả cho User

**Ví dụ:**

Query: "Học Python như thế nào?"

→ Query vector: \[0.15, -0.31, 0.59, ...\]

→ Vector DB tìm top-k vectors gần nhất

→ Trả về context: \["Python là ngôn ngữ...", "Khóa học Python...", ...\]

Custom prompt:      “You are a helpful assistant.

      Answer ONLY from the provided transcript context.

      If the context is insufficient, just say you don't know.

      {context}

      Question: {question}”

### **Tại sao cần Vector DB?**

1.  Semantic Search: Tìm theo ý nghĩa, không chỉ từ khóa
    
2.  Performance: Tối ưu cho similarity search trên millions vectors
    
3.  Scalability: Xử lý data lớn, distributed
    
4.  Features: Metadata filtering, hybrid search, real-time updates
    

### **2.1. Triển khai RAG sử dụng n8n**

**Hướng dẫn:**Import this file to n8n: 

[https://github.com/andyhoanghuu/rag/blob/master/RAG\_flow\_with\_supabase.json](https://github.com/andyhoanghuu/rag/blob/master/RAG_flow_with_supabase.json)


### **2.2. Triển khai RAG với Python**

**B1**: Cài đặt Ollama

**B2**: Tải và run model Qwen 3.5

ollama run qwen3.5

hoặc

ollama run qwen3.5:9b

**B3**: Tải các thư viện của Python

pip install langchain langchain-community langchain-core langchain-ollama chromadb sentence-transformers pypdf python-dotenv unstructured\[pdf\] tiktoken

*   langchain, langchain-community, langchain-core: The core LangChain framework for building LLM applications.
    
*   langchain-ollama: Specific integration for using Ollama models with LangChain.
    
*   chromadb: The local vector database for storing document embeddings.
    
*   sentence-transformers: Used for an alternative local embedding method (explained later).
    
*   pypdf: A library for loading PDF documents.
    
*   python-dotenv: For managing environment variables (optional but good practice).
    
*   unstructured\[pdf\]: An alternative, powerful document loader, especially for complex PDFs.
    
*   tiktoken: Used by LangChain for token counting.
    

**B4**: Chuẩn bị tài liệu để AI refer.

Ví dụ: [https://www.shb.com.vn/wp-content/uploads/2021/12/HƯỚNG-DẪN-SỬ-DỤNG-THẺ-TÍN-DỤNG\_2-1.pdf](https://www.shb.com.vn/wp-content/uploads/2021/12/HƯỚNG-DẪN-SỬ-DỤNG-THẺ-TÍN-DỤNG_2-1.pdf)

**B5**: Tải file code Python sau và chạy:[https://github.com/andyhoanghuu/rag/blob/master/rag\_local.py](https://github.com/andyhoanghuu/rag/blob/master/rag_local.py)

