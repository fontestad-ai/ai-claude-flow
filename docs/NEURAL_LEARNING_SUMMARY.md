# Neural Learning Architecture - Quick Reference Summary

## System Overview

Claude Flow implements a **multi-layered neural learning system** that continuously learns from operation outcomes through hooks-based pattern extraction and model training.

## Key Components

### 1. Graph Neural Network (GNN) Domain Mapper
**Location**: `src/neural/NeuralDomainMapper.ts`
- **Architecture**: 3-layer GNN (GCN → GAT → GCN)
- **Node Features**: 64-dimensional vectors
- **Domain Embeddings**: 32-dimensional learning representations
- **Capabilities**: Domain cohesion analysis, dependency detection, boundary optimization

### 2. SAFLA - Self-Aware Feedback Loop Algorithm
**Location**: `docs/reasoningbank/models/safla/`
- **Pattern Count**: 2,000 trained patterns
- **Embedding Dimension**: 1,024
- **Knowledge Graph**: 3,999 relationship links
- **Pattern Categories**: 5 domains (self-learning, feedback-optimization, confidence-adjustment, distillation, recursive-cycles)
- **Confidence Range**: 0.55 (learning) to 0.95 (expert)
- **Average Success Rate**: 90.3%

### 3. Neural Hooks System
**Location**: `src/services/agentic-flow-hooks/neural-hooks.ts`
- **Hook Types**: 5 main hooks (pre-train, post-train, pattern-detected, prediction, adaptation)
- **Priority Levels**: 90-100
- **Training Triggers**: Post-operation success/failure detection
- **Data Augmentation**: Historical pattern injection with confidence weighting

### 4. Pattern Storage & Retrieval
**Types**: Success, Failure, Optimization, Behavior
- **Matching Algorithm**: Similarity-based (threshold 0.7)
- **Storage**: Memory cache + SQLite3 database
- **TTL**: Session (24 hours), persistent (7 days), or permanent
- **Lifecycle**: Create → Detect → Store → Retrieve → Apply → Validate → Update

### 5. Memory Management
**Cache System**:
- **Max Size**: 10,000 entries
- **Max Memory**: 100 MB
- **Eviction**: LRU (Least Recently Used)
- **Hit Rate Tracking**: Efficiency metrics

**Object Pool**:
- Reduces garbage collection pressure
- Configurable pool size (default 1,000)
- Object reuse optimization

## Training Pipeline

### Data Flow
```
Operation Execution
    ↓
Post-Operation Hooks
    ↓
Pattern Extraction
    ↓
Dataset Augmentation (add successful patterns)
    ↓
Dataset Balancing (equalize labels)
    ↓
Neural Model Training (GNN forward/backward pass)
    ↓
Model Validation & Promotion
    ↓
Weight Storage & Pattern Update
```

### Training Parameters
- **Learning Rate**: 0.001-0.01 (adaptive decay)
- **Batch Size**: 8-64 (configurable by scale)
- **Epochs**: 20-200 (early stopping at patience=10-20)
- **Optimizer**: Adam (AdamW for large-scale)
- **Loss Function**: MSE, Cross-Entropy, Contrastive, Triplet
- **Regularization**: L1=0.0001-0.0002, L2=0.0001-0.0002, Dropout=0.05-0.2

## Model Types (27+)

**Implemented**:
- Feedforward Networks (Coordination)
- LSTM Networks (Pattern Recognition)
- Transformer Networks (Performance Prediction)
- Domain Mapper GNN (Relationship Analysis)
- CNN/RNN Models (Pattern Recognition)

**Domain-Specific Variants**:
- Code Reasoning
- Problem Solving
- Domain Expertise
- Google Research Patterns
- SAFLA Self-Learning

## Performance Metrics

### Achievements
- **84.8%** SWE-Bench solve rate
- **32.3%** token reduction from pattern reuse
- **2.8-4.4x** speed improvement (parallel execution)
- **90.3%** average pattern success rate
- **85.0%** model accuracy after 100 epochs
- **<5ms** pattern lookup latency

### Learning Curve
```
Epoch 1:    55-65% confidence (learning phase)
Epoch 10:   65-75% confidence (accumulating)
Epoch 50:   75-85% confidence (converging)
Epoch 100:  85-95% confidence (expert level)
```

### Pattern Reuse
- Cold start: 15% pattern match rate
- After 100 ops: 45% pattern match
- After 1000 ops: 72% pattern match
- Mature system: 85%+ pattern match

## Configuration Profiles

### Scale-Based Presets
- **Small** (< 50 domains): Quick learning, lower threshold
- **Medium** (50-200): Balanced, moderate threshold
- **Large** (> 200): Slower learning, high threshold
- **Real-Time**: Fast feedback, minimal suggestions

### Confidence Thresholds
- **0.9+**: Expert recommendation (trust completely)
- **0.75-0.9**: Suggested (medium trust)
- **0.5-0.75**: Consider (validate first)
- **<0.5**: Ignore (insufficient data)

## Integration Points

### Hooks System
- Automatic domain analysis (neural-pattern-detected)
- Continuous training integration (post-neural-train)
- Periodic analysis (30-120 second intervals)
- Side effect generation (notifications, metrics, logs)

### Memory Management
- LRU cache for fast retrieval
- Object pooling for efficiency
- SQLite3 for persistence
- Automatic archival of old patterns

## Safety Mechanisms

### Validation
- Data quality checks (no NaN/Infinity)
- Input/output validation
- Batch size > 0, epochs > 0
- Model consistency checking

### Adaptation Safety
- Parameter changes require validation
- Architecture changes require confidence > 0.8
- Strategy changes require confidence > 0.75 AND impact < 0.5
- Rollback capability via checkpoints

## Model Promotion Criteria

Models promoted to production when:
1. Average accuracy > 0.85 (over 10 runs)
2. Consistency: std deviation < 0.05
3. No regression from previous best
4. Validation loss < training loss
5. Performance stable for >= 20 evaluations

## File Locations

**Core Neural System**:
- `src/neural/NeuralDomainMapper.ts` (1,667 lines)
- `src/neural/integration.ts` (819 lines)
- `src/services/agentic-flow-hooks/neural-hooks.ts` (758 lines)
- `src/services/agentic-flow-hooks/types.ts` (507+ lines)

**Training & Models**:
- `bin/training.js` - Training CLI
- `bin/training-pipeline.js` - Pipeline execution
- `docs/reasoningbank/models/safla/` - SAFLA training
- `docs/reasoningbank/models/*/train-*.js` - Domain-specific models

**Initialization**:
- `src/modes/NeuralInit.ts` (248 lines)
- `src/hive-mind/core/Memory.ts` - Memory management

## Key Insights

1. **Post-Operation Learning**: Patterns extracted automatically after successful operations
2. **Confidence Evolution**: Patterns start at 0.5 confidence and evolve to 0.95 through repeated success
3. **Multi-Model Approach**: 27+ specialized neural models for different domains
4. **Context-Aware**: Patterns stored with full context for intelligent retrieval
5. **Continuous Adaptation**: Models automatically adjust parameters/architecture based on performance
6. **Safety First**: Confidence thresholds prevent risky pattern application
7. **Efficient Memory**: LRU cache + object pooling minimize garbage collection overhead
8. **Knowledge Graph**: Pattern relationships enable discovery of pattern combinations
9. **Early Stopping**: Prevents overfitting while maintaining convergence speed
10. **Production Ready**: Validated patterns with > 0.85 accuracy promoted automatically

## Getting Started

1. **View Full Analysis**: `/home/user/ai-claude-flow/docs/neural-learning-architecture.md`
2. **Configure Neural System**: Choose scale-based preset (small/medium/large/real-time)
3. **Enable Hooks**: Register neural hooks via `agenticHookManager.register()`
4. **Train Models**: Patterns extracted automatically post-operation
5. **Monitor Performance**: Track accuracy trends and pattern reuse rates
6. **Apply Patterns**: Automatic retrieval and application via hooks

---

**Full documentation**: `/home/user/ai-claude-flow/docs/neural-learning-architecture.md` (1,258 lines)
**Last Updated**: October 2025
