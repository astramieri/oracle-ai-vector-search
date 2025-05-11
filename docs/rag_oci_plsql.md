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
