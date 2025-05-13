# Vector DDL

You can have tables with different vector formats:
- more than one column of ```VECTOR``` data type
- vector columns with different formats

## Example 

```
CREATE TABLE IF NOT EXISTS t5 (
    v1  VECTOR(3, float32),
    v2  VECTOR(2, float64),
    v3  VECTOR(1, int8),
    v4  VECTOR(1, *),
    v5  VECTOR(*, float32),
    v6  VECTOR(*, *),
    v7  VECTOR
);

ALTER TABLE t5 ADD v8 VECTOR(2, float32);

INSERT INTO t5 VALUES (
    '[1.1, 2.2, 3.3]',
    '[1.1, 2.2]',
    '[7]',
    '[9]',
    '[1.1, 2.2, 3.3, 4.4, 5.5]',
    '[1.1, 2.2]',
    '[1.1, 2.2, 3.3, 4.4, 5.5, 6.6]',
    '[5.5, 6.6]'
);

COMMIT;
```

## Prohibited Operations

You cannot define ```VECTOR``` columns in/as:
- External Tables
- Index Organized Tables (IOTs) (neither as primary key nor as non-key column)
- Clusters/Cluster Tables
- Global Temp Tables
- (Sub)Partitioning Key
- Primary Key
- Foreign Key
- Unique Contrastraint
- Check Constraints
- Default Value
- Modify Column
- MSSM tablespace (only SYS user can create VECTORs ad Basicfiles in MSSM tablespace)
- Continuous Query Notification (CQN)
- Non-vector indexes such as B-tree, Reverse Key, Text, Spatial Indexes, etc.

**Note.**  Oracle does not support ```distinct```, ```count distinct```, ```order by```, ```group by```, ```join``` condition, or comparison operators such as less than, greater than, or equal to with vector columns.