# RAG with OCI Gen AI and PL/SQL

Process Overview:

- Step 1
    - Load the document
    - Transform the document to text
    - Split the text into chunks
- Step 2
    - Embedding Models and Vectorization
        - Load multiple ONNX models into the database
        - Compare vector embedding using different models
        - Creating Vector Embedding using PL/SQL packages
- Step 3    
    - Similarity Search and Response Generation
        - Select the text chunks that has relevant information for the user question based on vector search        
- Step 4
    - Build the prompt
        - LLM Prompt Engineering enables you to craft input queries of instructions to create more accurate and desirable outputs
    - Simple User Interface example using Streamlit
- Step 5
    - Invoke the Chain

## 1. Create tables

```
CREATE TABLE my_books (
    file_id       INTEGER,
    file name     VARCHAR2(900),
    file_size     INTEGER,
    file_type     VARCHAR2(100),
    file_content  BLOB        
);

CREATE TABLE vector_store (
    file_id       INTEGER,
    embed_id      NUMBER ,
    embed_data    VARCHAR2(4000),
    embed_vector  VECTOR    
);
```