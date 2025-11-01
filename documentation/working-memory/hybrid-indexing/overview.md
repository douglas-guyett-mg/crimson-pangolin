# Hybrid Indexing

## Overview

Hybrid Indexing is the system that enables efficient retrieval of memory items through multiple search modalities. It combines dense vector embeddings (for semantic similarity), sparse term indexes (for keyword matching), and metadata indexes (for structured filtering) to provide flexible and accurate search capabilities.

## Purpose

Hybrid Indexing serves to:

1. **Enable Semantic Search**: Find items based on meaning, not just keywords
2. **Enable Keyword Search**: Find items based on exact terms and phrases
3. **Enable Metadata Filtering**: Filter items by structured attributes
4. **Optimize Performance**: Provide fast retrieval even with large memory stores
5. **Support Multiple Query Types**: Handle different query patterns efficiently

## Index Types

### Dense Index (Vector Embeddings)
- **Representation**: Vector embeddings of item text
- **Dimension**: Typically 384-1536 dimensions
- **Generation**: Using embedding model (e.g., sentence-transformers)
- **Storage**: Vector database or in-memory index
- **Query**: Vector similarity search (cosine, euclidean, etc.)
- **Advantages**: Semantic understanding, handles synonyms
- **Disadvantages**: Requires embedding model, slower than keyword search

### Sparse Index (Term Index)
- **Representation**: Inverted index of terms
- **Terms**: Extracted from item text (words, phrases, n-grams)
- **Storage**: Inverted index structure
- **Query**: Boolean or TF-IDF search
- **Advantages**: Fast, exact matching, interpretable
- **Disadvantages**: No semantic understanding, sensitive to exact wording

### Metadata Index
- **Representation**: Structured index of metadata fields
- **Fields**: agent_id, tags, source, timestamp, importance, etc.
- **Storage**: Hash tables, B-trees, or database indexes
- **Query**: Exact match, range queries, set membership
- **Advantages**: Fast filtering, supports complex queries
- **Disadvantages**: Only works for structured data

## Indexing Process

### Dense Indexing
1. **Text Extraction**: Extract text from MemoryItem
2. **Embedding Generation**: Generate vector embedding using embedding model
3. **Normalization**: Normalize vector to unit length
4. **Storage**: Store vector in vector database
5. **Metadata**: Store item ID, timestamp, importance with vector

### Sparse Indexing
1. **Text Extraction**: Extract text from MemoryItem
2. **Tokenization**: Split text into tokens
3. **Term Extraction**: Extract meaningful terms (remove stopwords, stem)
4. **Frequency Calculation**: Calculate term frequencies
5. **Index Update**: Add terms to inverted index

### Metadata Indexing
1. **Field Extraction**: Extract metadata fields
2. **Type Conversion**: Convert to appropriate types
3. **Index Update**: Add to metadata indexes
4. **Timestamp Indexing**: Index timestamps for range queries

## Search Operations

### Dense Search
Finds items semantically similar to query.

**Process**:
1. Generate embedding for query
2. Search vector database for nearest neighbors
3. Return top-k results with similarity scores

**Parameters**:
- `query`: Text query
- `k`: Number of results
- `threshold`: Minimum similarity score

**Returns**:
- List of items with similarity scores

### Sparse Search
Finds items matching query terms.

**Process**:
1. Tokenize and extract terms from query
2. Search inverted index for matching terms
3. Calculate TF-IDF scores
4. Return top-k results

**Parameters**:
- `query`: Text query
- `k`: Number of results
- `threshold`: Minimum relevance score

**Returns**:
- List of items with relevance scores

### Metadata Search
Filters items by metadata.

**Process**:
1. Parse filter conditions
2. Query metadata indexes
3. Return matching items

**Parameters**:
- `filters`: Dictionary of filter conditions
- `operator`: AND or OR for multiple filters

**Returns**:
- List of matching items

## Index Maintenance

### Index Updates
When items are written:
1. Generate dense embedding
2. Extract and index terms
3. Index metadata
4. Store item ID in all indexes

### Index Deletions
When items are deleted:
1. Remove from dense index
2. Remove from sparse index
3. Remove from metadata index

### Index Rebuilding
Periodically rebuild indexes:
1. Scan all items
2. Regenerate embeddings
3. Rebuild inverted index
4. Rebuild metadata indexes

## Design Principles

1. **Multi-Modal**: Supports multiple search modalities
2. **Efficient**: Optimized for fast retrieval
3. **Scalable**: Handles large memory stores
4. **Maintainable**: Indexes are kept in sync with data
5. **Flexible**: Supports custom index implementations

## Interaction with Other Components

- **Memory Plugins**: Call search operations
- **Memory Orchestrator**: Coordinates searches
- **Embedding Service**: Generates vector embeddings
- **Vector Database**: Stores and searches vectors
- **Turn Trace**: Logs search operations

## Configuration

Hybrid Indexing can be configured with:

- **embedding_model**: Model to use for embeddings (default: "sentence-transformers/all-MiniLM-L6-v2")
- **embedding_dimension**: Dimension of embeddings (default: 384)
- **vector_db_type**: Type of vector database (default: "in-memory")
- **sparse_index_type**: Type of sparse index (default: "inverted_index")
- **enable_dense_search**: Whether to enable dense search (default: true)
- **enable_sparse_search**: Whether to enable sparse search (default: true)
- **enable_metadata_search**: Whether to enable metadata search (default: true)

## Example Scenario

1. Item is written: "Paris is the capital of France"
2. **Dense Indexing**:
   - Generate embedding: [0.12, -0.45, 0.78, ...]
   - Store in vector database
3. **Sparse Indexing**:
   - Extract terms: ["paris", "capital", "france"]
   - Add to inverted index
4. **Metadata Indexing**:
   - Index agent_id, timestamp, tags
5. User searches: "French cities"
6. **Dense Search**:
   - Generate query embedding
   - Find similar vectors
   - Return Paris item with score 0.87
7. **Sparse Search**:
   - Extract terms: ["french", "cities"]
   - Search inverted index
   - Return Paris item with score 0.65
8. **Metadata Search**:
   - Filter by agent_id
   - Return matching items
9. **Hybrid Results**:
   - Combine dense and sparse scores
   - Return ranked results

