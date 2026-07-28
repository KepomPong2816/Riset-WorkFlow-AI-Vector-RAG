# Dokumen Arsitektur Teknis: Workflow AI Wrapper (Hybrid SQL + Vector RAG)

## 1. Pendahuluan

Sistem dibagi menjadi empat komponen utama:

| Komponen | Tanggung Jawab |
|----------|----------------|
| Laravel | Authentication, Authorization, Intent Parsing, Query Builder, Business Logic |
| Vector RAG | Embedding, Vector Search, Retrieval Dokumen |
| Go Wrapper | Prompt Builder, Guardrail, LLM Communication, Response Validation |
| LLM | Mengubah data SQL dan dokumen hasil retrieval menjadi narasi |

---

> Seluruh data transaksi tetap berasal dari database SQL. Vector RAG
> hanya digunakan untuk mengambil referensi dari dokumen tidak
> terstruktur (SOP, PDF, kebijakan, manual).

---

## 2. Arsitektur Sistem

``` mermaid
graph TD

User([User / Chatbot])
User -->|Prompt| LaravelBackend

subgraph Laravel
    LaravelBackend[Laravel Backend]
    Intent[Intent Parser]
    Auth[Authorization Guard]
    Query[SQL Query]
    DB[(MySQL)]
    Payload[JSON Payload]

    LaravelBackend --> Intent
    Intent --> Auth
    Auth --> Query
    Query --> DB
    DB --> Payload
end

Auth --> Retrieval

subgraph VectorRAG
    Retrieval[Retriever]
    Embedding[Embedding Model]
    Vector[(Vector Database)]
    Docs[Retrieved Documents]

    Retrieval --> Embedding
    Embedding --> Vector
    Vector --> Docs
end

Payload --> Builder
Docs --> Builder

subgraph GoWrapper
    Builder[Prompt Builder]
    LLM[LLM API]
    Validator[Response Validation]

    Builder --> LLM
    LLM --> Validator
end

Validator --> LaravelBackend
LaravelBackend --> User
```

## Penjelasan Alur

1.  User mengirim prompt.
2.  Laravel melakukan authentication, authorization, dan intent parsing.
3.  Laravel mengambil data terstruktur dari MySQL.
4.  Jika diperlukan referensi dokumen, Laravel memicu proses retrieval
    ke Vector RAG.
5.  Retriever mencari dokumen paling relevan menggunakan embedding.
6.  JSON Payload dan Retrieved Documents dikirim ke Go Wrapper.
7.  Go Wrapper menyusun prompt (System Prompt + Guardrail + User
    Prompt + SQL Payload + Retrieved Documents).
8.  Prompt dikirim ke LLM.
9.  LLM menghasilkan narasi berdasarkan data SQL dan dokumen hasil
    retrieval.
10. Go melakukan validasi respons.
11. Laravel memformat hasil akhir.
12. Respons dikirim ke pengguna.

---

## 3. Prompt Construction

Prompt terdiri dari: - System Prompt - Guardrail - User Prompt - JSON
Payload (SQL) - Retrieved Documents (Vector RAG)

``` mermaid
flowchart LR
System --> Builder
Guardrail --> Builder
User --> Builder
Payload --> Builder
Docs --> Builder
Builder[Prompt Builder] --> LLM
```

---

## 4. Payload

``` json
{
  "metadata": {},
  "configuration": {},
  "user_original_query": "...",
  "factual_payload": {},
  "retrieved_documents": [
    {
      "title": "SOP Refund",
      "content": "Potongan dokumen yang relevan..."
    }
  ]
}
```

`factual_payload` berasal dari SQL, sedangkan `retrieved_documents`
berasal dari Vector Database.

---

## 5. Referensi 

- Linkedin Aricle [Linkedin](https://www.linkedin.com/pulse/from-vector-databases-hybrid-rag-enterprise-gen-ai-nitin-karandikar-odzjc/).
- Google Skill [Google](https://www.skills.google/paths/1282/course_templates/1097/documents/530393?locale=id).
