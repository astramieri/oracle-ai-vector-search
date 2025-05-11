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
    embed_id      NUMBER,
    embed_data    VARCHAR2(4000),
    embed_vector  VECTOR    
);
```

## 2. Load the document

```
INSERT INTO my_books (
    file_name,
    file_size,
    file_type,
    file_content
) VALUES (
    '23ai_release_notes.pdf',
    DBMS_LOG.GETLENGTH(TO_BLOB(B_FILENAME('DIR', ''23ai_release_notes.pdf'))),
    'PDF',
    TO_BLOB(B_FILENAME('DIR', ''23ai_release_notes.pdf'))
)
```

## 3. Convert the document to plain text

```
DBMS_VECTOR_CHAIN.UTL_TO_TEXT (
    DATA   IN CLOB | BLOB
    PARAMS IN JSON default NULL
) RETURN CLOB;
```

## 4. Split the text into chunks

```
DBMS_VECTOR_CHAIN.UTL_TO_CHUNKS (
    DATA   IN CLOB | BLOB
    PARAMS IN JSON DEFAULT NULL
) RETURN VECTOR_ARRAY_T;
```

It returns an array of ```CLOBs``` where each ```CLOB``` contains a chunk along with the metadata in JSON format.

```
{
    "chunk_id" : NUMBER,
    "chunk_offset" : NUMBER,
    "chunk_length" : NUMBER,
    "chunk_data" : VARCHAR2(4000)
}
```

```
SELECT
    JSON_VALUE(c.column_value, '$.chunk_id' RETURNING NUMBER) as id,
    JSON_VALUE(c.column_value, '$.chunk_offset' RETURNING NUMBER) as offset,
    JSON_VALUE(c.column_value, '$.chunk_length' RETURNING NUMBER) as length,
    JSON_VALUE(c.column_value, '$.chunk_data' RETURNING NUMBER) as data
FROM my_books b
     DBMS_VECTOR_CHAIN.UTL_TO_CHUNKS (
        DBMS_VECTOR_CHAIN.UTL_TO_TEXT(b.file_content)) c
WHERE ROWNUM < 4;
```

## 5. Load ONNX model

```
BEGIN
    DBMS_DATA_MINING.LOAD_ONNX_MODEL(
        directory  => 'DIR',
        file_name  => 'tinybert.onnx',
        model_name => 'TINYBERT_MODEL'
    );
END;
```

## 6. Create vector embedding

```
INSERT INTO vector_store (
    file_id,
    embed_id,
    embed_data,
    embed_vector   
)
    SELECT 
        id,
        embed_id,
        text_chunk,
        embed_vector
    FROM 
        my_books b
        CROSS JOIN TABLE (
            DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDINGS(
                DBMS_VECTOR_CHAIN.UTL_TO_CHUNKS(
                    DBMS_VECTOR_CHAIN.UTL_TO_TEXT(b.file_content),
                    JSON('{
                        "by" : "words",
                        "max" : "300",
                        "split" : "sentece",
                        "normalize" : "all"
                    }')),
                JSON('{
                    "provider" : "database",
                    "model" : "TINYBERT_MODEL"
                    }')
            )
        ) t
        CROSS JOIN JSON_TABLE (
            t.column_value, '$[*]' COLUMNS (
                embed_id NUMBER PATH '$.embed_id',
                text_chunk VARCHAR2(4000) APTH '$.embed_data',
                embed_vector CLOB PATH '$.embed_vector'
            )
        ) e
```

7. Vectorize the user question

```
SELECT
    VECTOR_EMBEDDING(tinybert_model USING 'What is the result of the release version' as data) as embedding
```

8. Perform the vector search

```
WITH query_vector AS (
    SELECT VECTOR_EMBEDDING(tinybert_model USING 'List some limitations' AS DATA) as embedding
)
    SELECT 
        embed_id,
        embed_data
    FROM
        vector_store,
        query_vector
    ORDER BY 
        VECTOR_DISTANCE(embed_vector, query_vector.embedding, COSINE)
    FETCH APPORX FIRST 4 ROWS ONLY;
```