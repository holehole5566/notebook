# Chapter 4 Summary: Storage and Retrieval

## Core Theme
How databases store data and retrieve it efficiently - the fundamental operations of any database system.

## OLTP Storage Engines

### Log-Structured Storage (LSM-Trees)
**Concept**: Append-only writes with periodic merging and compaction

**Key Components**:
- **Memtable**: In-memory sorted structure (red-black tree, skip list)
- **SSTables**: Sorted String Tables - immutable, sorted files on disk
- **Compaction**: Background merging of segments to remove duplicates

**Advantages**:
- High write throughput (sequential writes)
- Good compression (sorted data compresses well)
- Lower write amplification

**Examples**: RocksDB, Cassandra, HBase, Scylla

### B-Trees
**Concept**: Fixed-size pages with in-place updates

**Key Features**:
- **Pages**: Fixed-size blocks (4KB-16KB) that can be overwritten
- **Balanced tree**: O(log n) lookup time
- **Write-Ahead Log (WAL)**: For crash recovery
- **Branching factor**: Hundreds of references per page

**Advantages**:
- Predictable read performance
- Efficient range queries
- Mature and well-understood

**Examples**: Most relational databases (PostgreSQL, MySQL, etc.)

### Comparison: LSM vs B-Trees
- **LSM**: Better for write-heavy workloads, sequential I/O
- **B-Trees**: Better for read-heavy workloads, predictable performance
- **Write amplification**: Both have it, but different patterns
- **Disk space**: LSM generally more efficient due to compression

## Indexing Strategies

### Primary vs Secondary Indexes
- **Primary**: Uniquely identifies records
- **Secondary**: Additional access paths, may have duplicates

### Storage Approaches
- **Clustered index**: Data stored within index structure
- **Heap file**: Index points to separate data storage
- **Covering index**: Index includes some non-key columns

## Analytics Storage (OLAP)

### Column-Oriented Storage
**Concept**: Store each column separately rather than rows together

**Advantages**:
- Only read columns needed for query
- Better compression (similar values grouped)
- Vectorized processing possible

**Compression Techniques**:
- **Bitmap encoding**: Sparse bitmaps with run-length encoding
- **Dictionary encoding**: Map values to smaller identifiers

### Query Execution Optimization
**Vectorized Processing**:
- Process batches of values instead of row-by-row
- Utilize CPU SIMD instructions
- Operate on compressed data directly

**Query Compilation**:
- Generate machine code for specific queries
- Eliminate interpretation overhead
- Better CPU pipeline utilization

### Materialized Views and Data Cubes
- **Materialized views**: Precomputed query results
- **Data cubes**: Multi-dimensional aggregates
- Trade-off: Faster reads vs more storage and write overhead

## Advanced Indexing

### Multi-Dimensional Indexes
- **Geospatial**: R-trees, space-filling curves
- **Concatenated indexes**: Multiple columns as single key
- Limited to specific query patterns

### Full-Text Search
- **Inverted indexes**: Term → document list mapping
- **N-grams**: Substring indexing for fuzzy matching
- **Edit distance**: Levenshtein automata for typo tolerance

### Vector Embeddings (Semantic Search)
**Purpose**: Find semantically similar content using ML embeddings

**Index Types**:
- **Flat indexes**: Brute force comparison (accurate but slow)
- **IVF indexes**: Partition space into clusters (approximate)
- **HNSW**: Hierarchical navigable small world graphs (approximate)

**Applications**: Semantic search, recommendation systems, similarity matching

## Key Insights

### Storage Engine Selection
- **OLTP**: Choose based on read/write patterns
  - Write-heavy: LSM-trees
  - Read-heavy: B-trees
  - Mixed: Consider workload specifics

- **OLAP**: Column-oriented storage almost always preferred
  - Compression crucial for performance
  - Vectorized execution for CPU efficiency

### Performance Considerations
- **Sequential vs Random I/O**: Still matters even with SSDs
- **Write amplification**: Affects both throughput and SSD wear
- **Compression**: Often more important than raw storage format
- **Cache locality**: Critical for modern CPU performance

### Evolution Trends
- **Separation of storage and compute**: Cloud-native architectures
- **Specialized engines**: Different engines for different workloads
- **In-memory processing**: When datasets fit in RAM
- **Hybrid approaches**: Combining multiple storage techniques

The choice of storage engine significantly impacts application performance and should align with specific workload characteristics and access patterns.