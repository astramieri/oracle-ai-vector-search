# Vector Indexes And Memory

**Vector Indexes** are specialized indexing data structures that can make your queries more efficient against your vectors. They are designed for similarity search in high-dimensional vector space. Without indexes, searching would require comparing against every vector.

Vector Indexes use these techniques to *drastically* reduce the search space:
- Clustering
- Partitioning
- Neighbor Graphs

**NOTE. They do require that you enable the Vector Pool in the SGA.**

## Types of Vector Indexes

Oracle AI Vector Search supports two types of indexes:
- **In-Memory Neighbor Graph Vector Index**
    - *Hierarchical Navigable Small World (HNSW)* is the only type supported
    - HNSW are very efficient for vector approximate similarity search
    - In a RAC environment you cannot create HNSW index
    - Store in memory (SGA) 
    - Best choice when data fits in memory
- **Neighbor Partition Vector Index**
    - *Inverted File Flat (IVF) Index* is the only type supported
    - They balances high search quality with reasonable speed
    - Better suited for larget datasets
    - Supports DML operations

**NOTE.** You can only create **one type of vector index per vector column.** 

*Example 1 (HNSW). Immagine highways (upper layers) connecting major cities and local roads (lower layers) for detailed navigation. HNSW starts the search on "highways" to quicly get close to the target, the uses "local roads" for precise results.*

![Hierarchical Navigable Small World (HNSW)](../imgs/vector_index_hnsw.png)

*Example 2 (IVF). Think of a libraray with books organized by subject (partitions). If you're looking for a specific topic, you only need to search the relevant section, not the entire library.*

![Inverted File Flat (IVF)](../imgs/vector_index_ivf.png)

## Vector Pool Area

The Vector Pool is a memory area in the System Global Area (SGA) designed to HNSW vector indexes and associated metadata. 

It is configured using the ```VECTOR_MEMORY_SIZE``` parameter. It can be modified at CDB level (Container Database) or PDB level (Pluggable Database).

```ALTER SYSTEM SET VECTOR_MEMORY_SIZE=1G SCOPE=BOTH;```

**Note.**    Large vector indexes do need lots of RAM and RAM constrains the vector index size. You should use IVF indexes when there is not enough RAM. IVF index is used both the buffer cache as well as disk.

![Vector Pool Area](../imgs/vector_pool_area.png)

## Memory Considerations

The size of a vector depends upon the embedding model that you use to create those embeddings. 

Most popular vectors are between 1.5 and 12 KB in size.

Oracle AI Vector Search supports:
- INT8
- FLOAT32
- FLOAT64

Oracle AI Vector Search supports vectors with up to 65,535 dimensions.

*In this example, we're using the FLOAT32 format, which is 4 bytes in size. And so the size would be the number of dimensions multiplied by how many bytes for that format.*

![Vector Size](../imgs/vector_size.png)

## Indexes Memory Size

In-Memory Neighbor Graph Indexes are stored:
- On-Disk
- In-Memory

The In-Memory size formula is:

    (1.3) * (size of vector format) * (# of dimensions) * (# of rows)

*NOTE: 1.3 is an approximation for the overhead and graph layers*

![In-Memory Indexes Size](../imgs/indexes_memory_size.png)