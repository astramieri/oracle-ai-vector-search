# DML Operations on Vectors

Oracle Database 23ai has a new ```vector``` data type. The new data type was created in order to support vector search.

The ```vector``` data type definition includes:
- number of dimensions (optional)
- dimension format (optional)
    - BYNARY
    - INT8
    - FLOAT32 (default)
    - FLOAT64

Declaration formats:
- ```vector``` or ```vector(*, *)```
    - vectors can have arbitrary number of dimensions and formats
- ```vector(num_of_dims)``` or ```vector(num_of_dims, *)```
    - vectors must have the specified number of dimensions (or an error is thrown)
    - every vector will have its dimension stored without format modification
- ```vector(*, dim_format)```
    - vectors can have an arbitrary number of dimensions
    - every vector will be converted to the specified format

**IMPORTANT**. If you are storing a mixed number of dimensions and formats, **you are not allowed to create vector indexes or to use distance functions** because they require the same number of dimensions as well as the same format for the entire column.

A vector can be ```NULL``` but its dimensions cannot (for example, you cannot have a ```VECTOR``` with a ```NULL``` dimension such as ```[1.1, NULL, 2.2]```).

## Vector Data Type Example

```
CREATE TABLE IF NOT EXISTS t2 (
    id          NUMBER,
    embedding   VECTOR(768, INT8)
);
```
