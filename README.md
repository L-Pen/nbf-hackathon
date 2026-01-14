# Orderbook Optimization Workshop

## Overview

A **collaborative learning workshop** where students start with a **working but slow** orderbook and optimize it together to achieve O(1) performance!

## Concept: Learn by Optimizing

Instead of filling in blank functions, students:
1. ✅ Start with **fully working code** (just slow)
2. 📊 **Measure performance problems** with real benchmarks
3. 🤔 **Discuss solutions** as a group
4. 🚀 **Optimize together** and see dramatic improvements

## Files

- **orderbook_workshop.ipynb** - Start here! Contains suboptimal implementation and test functions
- **orderbook_solution.ipynb** - Optimal implementation (reveal after workshop)
- **README.md** - This file

## The Challenge

### Focus: Sell Orders (Asks) Only
Simplified to just the ask side - no complex bid/ask logic needed!

### Three Functions to Optimize:
```python
submit(order_id, price, quantity)  # Add a sell order
cancel(order_id)                   # Cancel a sell order
get_best_price()                   # Get lowest ask price
```

### Performance Targets:

| Operation | Current (Slow) | Target (Fast) |
|-----------|----------------|---------------|
| **submit()** | O(N) | O(log P) |
| **cancel()** | O(N) | O(P) |
| **get_best_price()** | **O(N)** ❌ | **O(1)** ✅ |

Where:
- N = total orders (could be 1000s!)
- P = unique price levels (~10-100)

## Workshop Flow

### Phase 1: Understand the Problem
1. Run the suboptimal implementation
2. See that it works correctly
3. **Run performance benchmarks using test functions**
4. Watch it slow down with more orders
5. Identify: "get_best_price() is O(N) - too slow!"

### Phase 2: Group Discussion
- Why is get_best_price() slow?
- What data structure gives O(1) min access?
- Why use a dictionary for cancel()?
- Draw the target data structure

### Phase 3: Live Coding
Optimize together as a class:
1. Add dict for order lookup → O(1) order access
2. Add min heap for price tracking → O(1) get_best_price
3. Organize by price levels
4. Implement cancel with heap cleanup → O(P) but keeps heap clean
5. Test and benchmark improvements using test functions

**Key insight:** We optimize for the most common operation (get_best_price) at the expense of cancel!

### Phase 4: Results & Celebration
- Compare before/after performance
- See 50-100x speedup!
- Discuss real-world impact

## Key Learning Objectives

### 1. Performance is Measurable
Students **see** real numbers:
```
Before: 40 microseconds per get_best()
After:  <0.5 microsecond per get_best_price()
Result: 80-100x faster! 🚀
```

### 1.5 Understanding Trade-offs
- `get_best_price()`: O(1) - just peek at heap[0]!
- `cancel()`: O(P) - rebuilds heap to keep it clean
- **Key learning:** Optimize for the most common operation
- In real orderbooks, queries happen 1000x more than cancels!

### 2. Data Structures Matter
Learn **why** each structure is chosen:
- **Min Heap**: O(1) access to minimum value
- **Dictionary**: O(1) lookup by key
- **Not a list**: O(N) search is too slow

### 3. Algorithmic Thinking
Focus on:
- Identifying bottlenecks
- Choosing right data structures
- Measuring improvements
- Understanding Big O notation in practice

## The Min Heap Solution

### Problem
```python
# Current: Scan all orders to find minimum
for order in orders:
    if order.price < best_price:
        best_price = order.price
# O(N) - SLOW!
```

### Solution
```python
# Use min heap: smallest always at top
import heapq
ask_heap = [101, 102, 103]
best_ask = ask_heap[0]  # O(1) - FAST!
```

### Visual
```
Orders: [103, 101, 105, 102, 104]
         ↓
Push to heap: [101, 102, 103, 104, 105]
         ↓
heap[0] = 101  ← Lowest price instantly! ✓
```

## Suboptimal Implementation Details

### Current Data Structure
```python
class SuboptimalOrderBook:
    def __init__(self):
        self.orders = []  # Just a flat list!
```

**Problems:**
- `get_best()`: Must scan entire list → O(N)
- `cancel()`: Must search entire list → O(N)
- `submit()`: Must check for duplicates → O(N)

### Performance Characteristics
```
10 orders:     ~5 microseconds
100 orders:    ~50 microseconds
1000 orders:   ~500 microseconds

Scales linearly - O(N)! ❌
```

## Optimal Implementation Details

### Target Data Structure
```python
class OptimizedOrderBook:
    def __init__(self):
        self.asks = {}         # {price: deque([orders])}
        self.ask_heap = []     # [101, 102, 103]
        self.orders = {}       # {order_id: Order}
```

**Solutions:**
- `get_best_price()`: Peek heap top → O(1)
- `cancel()`: Mark cancelled + rebuild heap if needed → O(P)
- `submit()`: Dict check + heap push → O(log P)

### Performance Characteristics
```
10 orders:     <1 microsecond
100 orders:    <1 microsecond
1000 orders:   <1 microsecond

Constant time - O(1)! ✅
```

## Real-World Impact

### Why This Matters

**Scenario:** Stock exchange processing orders

**Suboptimal (O(N)):**
- 1000 orders in book
- 100 microseconds per query
- **10,000 queries/second max**

**Optimal (O(1)):**
- Any number of orders
- <1 microsecond per query
- **1,000,000+ queries/second** 🚀

**Result:** 100x more throughput!

### Common Student Questions

**Q: Why not just sort the list?**
A: Sorting is O(N log N) every time. Heap maintains order in O(log P) per insert!

**Q: What about deleting from the heap?**
A: Great question! Production systems handle "stale" prices. For this workshop, we simplify.

**Q: Can we use a binary search tree instead?**
A: Yes! That's O(log N) for everything. Heaps are simpler and O(1) for peek.

**Q: What about the bid side?**
A: Same idea, but you'd need max heap. We use asks to keep it simple!

## Extensions

After completing the core workshop, students can:

1. **Add the bid side** - Learn the negative price trick for max heap
2. **Implement matching** - Match buy and sell orders
3. **Add order modification** - Change price/quantity
4. **Handle partial fills** - Split orders across multiple trades
5. **Build a simulator** - Create fake market activity
6. **Add visualization** - Display orderbook graphically
7. **Measure latency** - Track microsecond-level performance
8. **Add persistence** - Save/load orderbook state
