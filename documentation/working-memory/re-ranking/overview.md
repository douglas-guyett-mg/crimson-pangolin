# Re-ranking

## Overview

Re-ranking is the system that improves search result quality by combining scores from multiple sources (dense search, sparse search, metadata) and applying contextual weights to produce a final ranking. It ensures that the most relevant results appear first, taking into account both semantic similarity and other factors like recency and importance.

## Purpose

Re-ranking serves to:

1. **Combine Multiple Signals**: Merge scores from dense, sparse, and metadata searches
2. **Apply Context**: Weight results based on current context and task
3. **Improve Relevance**: Ensure most relevant results appear first
4. **Handle Diversity**: Balance relevance with diversity in results
5. **Support Customization**: Allow custom re-ranking strategies

## Re-ranking Factors

### Dense Score
- **Source**: Vector similarity search
- **Range**: 0-1 (normalized)
- **Interpretation**: Semantic similarity to query
- **Weight**: Configurable (default: 0.4)

### Sparse Score
- **Source**: Keyword/term matching
- **Range**: 0-1 (normalized)
- **Interpretation**: Keyword relevance to query
- **Weight**: Configurable (default: 0.3)

### Recency Score
- **Source**: Item timestamp
- **Range**: 0-1 (normalized)
- **Interpretation**: How recent the item is
- **Weight**: Configurable (default: 0.1)
- **Calculation**: Exponential decay based on age

### Importance Score
- **Source**: Item metadata
- **Range**: 0-1 (from metadata)
- **Interpretation**: Importance of the item
- **Weight**: Configurable (default: 0.1)
- **Calculation**: Direct from metadata

### Source Weight
- **Source**: Item metadata (source field)
- **Range**: 0-1 (based on source type)
- **Interpretation**: Reliability of source
- **Weight**: Configurable (default: 0.05)
- **Values**: user_input=1.0, tool_output=0.8, inference=0.6

### Diversity Score
- **Source**: Similarity to other results
- **Range**: 0-1 (normalized)
- **Interpretation**: How different from other results
- **Weight**: Configurable (default: 0.05)
- **Calculation**: Penalize results similar to higher-ranked results

## Re-ranking Process

### Stage 1: Score Normalization
Normalize all scores to 0-1 range:
- Dense score: already 0-1
- Sparse score: already 0-1
- Recency score: calculate from timestamp
- Importance score: already 0-1
- Source weight: map source to 0-1
- Diversity score: calculate from other results

### Stage 2: Weight Application
Apply weights to each score:
```
weighted_score = (dense * w_dense) + (sparse * w_sparse) + 
                 (recency * w_recency) + (importance * w_importance) +
                 (source * w_source) + (diversity * w_diversity)
```

### Stage 3: Ranking
Sort results by weighted score in descending order.

### Stage 4: Diversity Adjustment
Optionally adjust ranking to improve diversity:
- Penalize results similar to higher-ranked results
- Re-rank to balance relevance and diversity

## Operations

### rerank(results, query, context)
Re-ranks search results.

**Parameters**:
- `results`: List of search results with scores
- `query`: Original query
- `context`: Current context (task, agent, etc.)

**Returns**:
- Re-ranked results
- Final scores
- Re-ranking metadata

### calculate_recency_score(timestamp)
Calculates recency score for an item.

**Parameters**:
- `timestamp`: Item timestamp

**Returns**:
- Recency score (0-1)

### calculate_diversity_score(item, other_results)
Calculates diversity score for an item.

**Parameters**:
- `item`: Item to score
- `other_results`: Other results in ranking

**Returns**:
- Diversity score (0-1)

### set_weights(weights)
Sets re-ranking weights.

**Parameters**:
- `weights`: Dictionary of weights

**Returns**:
- Confirmation of weight update

### get_weights()
Gets current re-ranking weights.

**Returns**:
- Dictionary of current weights

## Design Principles

1. **Multi-Factor**: Considers multiple factors in ranking
2. **Configurable**: Weights can be customized
3. **Transparent**: Ranking decisions are explainable
4. **Efficient**: Re-ranking is fast
5. **Adaptive**: Can adapt to context

## Interaction with Other Components

- **Hybrid Indexing**: Receives results from indexing
- **Memory Orchestrator**: Calls re-ranking for search results
- **Turn Trace**: Logs re-ranking decisions
- **Context Manager**: Provides context for weighting

## Configuration

Re-ranking can be configured with:

- **weight_dense**: Weight for dense score (default: 0.4)
- **weight_sparse**: Weight for sparse score (default: 0.3)
- **weight_recency**: Weight for recency score (default: 0.1)
- **weight_importance**: Weight for importance score (default: 0.1)
- **weight_source**: Weight for source weight (default: 0.05)
- **weight_diversity**: Weight for diversity score (default: 0.05)
- **recency_decay**: How quickly recency decays (default: exponential)
- **diversity_threshold**: Minimum diversity to maintain (default: 0.3)
- **enable_diversity**: Whether to apply diversity adjustment (default: true)

## Example Scenario

1. User searches: "weather forecast"
2. Hybrid indexing returns 10 results:
   - Result A: dense=0.9, sparse=0.7, recency=0.8, importance=0.6, source=0.8
   - Result B: dense=0.7, sparse=0.9, recency=0.5, importance=0.8, source=1.0
   - Result C: dense=0.8, sparse=0.6, recency=0.9, importance=0.7, source=0.8
   - ... (7 more results)
3. **Score Normalization**: All scores already normalized
4. **Weight Application**:
   - Result A: (0.9*0.4) + (0.7*0.3) + (0.8*0.1) + (0.6*0.1) + (0.8*0.05) = 0.77
   - Result B: (0.7*0.4) + (0.9*0.3) + (0.5*0.1) + (0.8*0.1) + (1.0*0.05) = 0.74
   - Result C: (0.8*0.4) + (0.6*0.3) + (0.9*0.1) + (0.7*0.1) + (0.8*0.05) = 0.73
5. **Ranking**: Results sorted by weighted score: A (0.77), B (0.74), C (0.73), ...
6. **Diversity Adjustment**: Check if top results are too similar
   - If A and C are very similar, penalize C
   - Re-rank to improve diversity
7. **Final Ranking**: Return re-ranked results to user

