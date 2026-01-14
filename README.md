# Orderbook Optimization Workshop

## Overview

A collaborative learning workshop where students start with a working but inefficient orderbook and optimize it together to achieve better performance.

## Learning Approach

Students will:
1. Start with fully working but slow code
2. Measure performance problems with benchmarks
3. Discuss optimization strategies as a group
4. Implement optimizations together

## Files

- **orderbook_workshop.ipynb** - Starting point with suboptimal implementation and test functions
- **orderbook_solution.ipynb** - Optimized implementation for instructor reference
- **README.md** - This file

## The Challenge

### Focus: Sell Orders (Asks) Only
Simplified to just the ask side to reduce complexity.

### Three Core Functions
```python
submit(order_id, price, quantity)  # Add a sell order
cancel(order_id)                   # Cancel a sell order
get_best_price()                   # Get lowest ask price
```

### Performance Targets

| Operation | Current | Target |
|-----------|---------|--------|
| submit() | O(N) | O(log P) |
| cancel() | O(N) | O(P) |
| get_best_price() | O(N) | O(1) |

Where:
- N = total orders
- P = unique price levels

## Workshop Structure

### Phase 1: Problem Identification
1. Run the suboptimal implementation
2. Execute performance benchmarks
3. Observe performance degradation with scale
4. Identify bottleneck: get_best_price() is O(N)

### Phase 2: Solution Design
- Analyze why get_best_price() is slow
- Discuss data structures for O(1) minimum access
- Consider dictionary for O(1) lookup by ID
- Design target data structure

### Phase 3: Implementation
Implement optimizations collaboratively:
1. Add dictionary for order lookup
2. Add min heap for price tracking
3. Organize orders by price levels
4. Implement cancel with heap cleanup
5. Test and benchmark improvements

Key trade-off: Optimize for the most common operation (get_best_price) at the expense of cancel.

## Learning Objectives

### 1. Performance Measurement
Students observe quantitative improvements:
```
Before: 40 microseconds per get_best()
After:  <0.5 microsecond per get_best_price()
Result: 80-100x faster
```

### 2. Complexity Trade-offs
- get_best_price(): O(1) via heap peek
- cancel(): O(P) via heap rebuild
- Design principle: Optimize for common operations
- Real-world context: Queries occur 1000x more than cancellations

### 3. Data Structure Selection
Understanding the rationale for each choice:
- **Min Heap**: O(1) access to minimum value
- **Dictionary**: O(1) lookup by key
- **Flat List**: O(N) search is inefficient

### 4. Algorithmic Thinking
Focus areas:
- Identifying performance bottlenecks
- Selecting appropriate data structures
- Measuring improvements quantitatively
- Understanding Big O notation practically

## Technical Details

### Min Heap Solution

#### Problem
```python
# Current: Scan all orders to find minimum
for order in orders:
    if order.price < best_price:
        best_price = order.price
# O(N)
```

#### Solution
```python
# Use min heap: smallest always at top
import heapq
ask_heap = [101, 102, 103]
best_ask = ask_heap[0]  # O(1)
```

#### Visualization
```
Orders: [103, 101, 105, 102, 104]
         |
         v
Push to heap: [101, 102, 103, 104, 105]
         |
         v
heap[0] = 101  (Lowest price)
```

### Suboptimal Implementation

#### Data Structure
```python
class SuboptimalOrderBook:
    def __init__(self):
        self.orders = []  # Flat list
```

#### Performance Issues
- get_best(): Must scan entire list (O(N))
- cancel(): Must search entire list (O(N))
- submit(): Must check for duplicates (O(N))

#### Measured Performance
```
10 orders:     ~5 microseconds
100 orders:    ~50 microseconds
1000 orders:   ~500 microseconds

Linear scaling: O(N)
```

### Optimized Implementation

#### Data Structure
```python
class OptimizedOrderBook:
    def __init__(self):
        self.asks = {}         # {price: deque([orders])}
        self.ask_heap = []     # [101, 102, 103]
        self.orders = {}       # {order_id: Order}
```

#### Optimizations
- get_best_price(): Peek heap top (O(1))
- cancel(): Mark cancelled, rebuild heap if needed (O(P))
- submit(): Dict check + heap push (O(log P))

#### Measured Performance
```
10 orders:     <1 microsecond
100 orders:    <1 microsecond
1000 orders:   <1 microsecond

Constant time: O(1)
```

## Real-World Application

### Scenario: Stock Exchange

**Suboptimal Implementation (O(N)):**
- 1000 orders in book
- 100 microseconds per query
- Maximum: 10,000 queries/second

**Optimized Implementation (O(1)):**
- Any number of orders
- <1 microsecond per query
- Maximum: 1,000,000+ queries/second

**Impact:** 100x throughput improvement

### Common Questions

**Q: Why not sort the list?**
A: Sorting is O(N log N) per operation. Heap maintains order in O(log P) per insert.

**Q: What about deleting from the heap?**
A: Production systems handle stale prices differently. This workshop simplifies for clarity.

**Q: Can we use a binary search tree?**
A: Yes, that provides O(log N) for all operations. Heaps are simpler and provide O(1) for peek.

**Q: What about the bid side?**
A: Same approach, but requires max heap (use negative prices). This workshop focuses on asks for simplicity.

### Production Considerations
- Thread safety and locking mechanisms
- Lazy deletion strategies
- Memory pooling
- Event notification systems
- Audit logging
- Microsecond-precision timestamps
