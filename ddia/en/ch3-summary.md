# Chapter 3 Summary: Data Models and Query Languages

## Core Theme
Data models profoundly affect how we think about problems. Each layer represents data in terms of the layer below it.

## Relational vs Document Models

### Object-Relational Mismatch
- **Problem**: Awkward translation between objects and relational tables
- **ORM Solutions**: Reduce boilerplate but have limitations (N+1 queries, complexity)
- **Document Advantage**: Better locality, closer to object model for tree-structured data

### Normalization Trade-offs

**Normalized Data**:
- Store human-meaningful info once, reference by ID
- Faster writes, slower reads (requires joins)
- Better for OLTP, consistency easier

**Denormalized Data**:
- Duplicate human-meaningful info
- Faster reads, slower writes (multiple copies to update)
- Better for analytics, but consistency challenges

### Relationship Types
- **One-to-Many**: Document model handles well (JSON arrays)
- **Many-to-One**: Requires references/joins
- **Many-to-Many**: Best handled with normalized approach

## Data Warehouse Schemas

### Star Schema
- **Fact Table**: Events/transactions at center
- **Dimension Tables**: Who, what, where, when, how, why
- **Snowflake Schema**: Further normalized dimensions

### One Big Table (OBT)
- Denormalized approach folding dimensions into fact table
- More storage, potentially faster queries
- Acceptable for analytics (historical data doesn't change)

## Graph Data Models

### When to Use Graphs
- Many-to-many relationships are common
- Need to traverse multiple hops
- Anything potentially related to everything

### Property Graphs
- **Vertices**: Unique ID, label, properties, edges
- **Edges**: Unique ID, tail/head vertices, label, properties
- Flexible schema, efficient traversal in both directions

### Query Languages

**Cypher**:
```cypher
MATCH (person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (:Location {name:'United States'})
RETURN person.name
```

**SPARQL** (for RDF/Triple-stores):
```sparql
SELECT ?personName WHERE {
  ?person :name ?personName.
  ?person :bornIn / :within* / :name "United States".
}
```

**Datalog**:
- Rule-based approach building complex queries incrementally
- Recursive rules enable graph traversal
- More verbose but very expressive

**GraphQL**:
- Client specifies exact data structure needed
- Limited (no recursion) for security
- Good for frontend-backend communication

## Event Sourcing and CQRS

### Event Sourcing
- Store all changes as immutable events
- Events are facts about what happened (past tense)
- Materialized views derived from event log

### CQRS (Command Query Responsibility Segregation)
- Separate write-optimized and read-optimized representations
- Commands validated before becoming events
- Multiple materialized views for different query patterns

### Advantages
- Clear intent and auditability
- Reproducible view generation
- Easy to add new views from existing events
- Fewer irreversible actions

### Challenges
- External data consistency (exchange rates)
- Personal data deletion (GDPR)
- Side effects during reprocessing

## Dataframes and Arrays

### Dataframes
- Table-like structure with rich operations
- Popular in data science (R, Pandas, Spark)
- Bridge between relational data and matrices
- Support transformations like pivot tables, one-hot encoding

### Use Cases
- Machine learning data preparation
- Statistical analysis
- Time series data (financial)
- Scientific computing

## Key Insights

### Model Selection Criteria
- **Document**: Tree-structured data, schema flexibility
- **Relational**: Complex queries, many-to-many relationships
- **Graph**: Highly connected data, path traversal
- **Event Sourcing**: Complex business domains, auditability
- **Dataframes**: Analytics, ML, scientific computing

### Convergence Trend
- Relational databases adding JSON support
- Document databases adding joins
- Graph capabilities in SQL improving
- Hybrid approaches becoming common

### Schema Approaches
- **Schema-on-Write**: Traditional relational (explicit, enforced)
- **Schema-on-Read**: Document databases (implicit, flexible)
- Trade-off between flexibility and guarantees

The choice of data model significantly impacts application architecture, query complexity, and system performance. Modern systems often benefit from hybrid approaches combining multiple models.