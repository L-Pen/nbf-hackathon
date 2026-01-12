# Orderbook Implementation with Heaps

## Overview
An efficient orderbook implementation using heaps for O(log P) best price lookups. Designed for students to complete in under 25 minutes (5 TODOs, each under 5 minutes).

## Files
- **orderbook_template.ipynb** - Student template with strategic TODOs
- **orderbook_solution.ipynb** - Complete solution
- **README_ORDERBOOK.md** - This file

## Key Concept: Negative Prices for Max Heap

Python's `heapq` module only implements **min heaps** (smallest element at the top). For bids, we need a **max heap** to get the highest price first.

**Solution:** Store negative prices!

```python
# Bid prices: $100, $99, $98
# Store in heap as: -100, -99, -98
heapq.heappush(bid_heap, -100.0)

# Min heap puts -100 at top (most negative)
# Negate to get actual price: -(-100) = $100 (highest bid!)
best_bid = -bid_heap[0]  # Returns 100.0
```

**For asks:** Use positive prices normally (we want lowest ask at top).

```python
# Ask prices: $101, $102, $103
heapq.heappush(ask_heap, 101.0)  # No negation

# Min heap puts 101 at top (smallest)
best_ask = ask_heap[0]  # Returns 101.0
```

## Data Structures

```python
bids = {price: deque([order1, order2, ...])}  # Buy orders
asks = {price: deque([order1, order2, ...])}  # Sell orders
bid_heap = [-100, -99, -98]                    # Negative prices!
ask_heap = [101, 102, 103]                     # Positive prices
orders = {order_id: Order}                     # Fast O(1) lookup
```

**Why this design:**
- **Heaps:** O(log P) for best price operations
- **Deques:** O(1) append/popleft for FIFO order at each price level
- **Dicts:** O(1) order lookup by ID

## TODOs (Each under 5 minutes)

### TODO 1: Submit Order (~3 min)
**Task:** Complete the SELL/asks section

**What's provided:**
- BUY side fully implemented as example
- All structure and error checking done

**What to do:**
```python
# Add these 3 lines for asks:
if price not in self.asks:
    self.asks[price] = deque()
    heapq.heappush(self.ask_heap, price)  # Positive!
self.asks[price].append(order)
```

### TODO 2: Cancel Order (~4 min)
**Task:** Remove order from queue and cleanup

**What's provided:**
- Lookup and validation logic
- Status update
- Price level dictionary selection

**What to do:**
```python
# Inside the try block:
queue.remove(order)
if len(queue) == 0:
    del price_levels[order.price]
```

**Why more complex:** Real cancellation needs to remove from queue, not just mark status.

### Get Order Status (Already complete!)
No TODO - this is provided as a freebie. Simple O(1) dict lookup.

### TODO 3: Get Best Bid (~3 min)
**Task:** Complete the validity check and cleanup

**What's provided:**
- Loop structure
- Price extraction with negation

**What to do:**
```python
if price in self.bids and len(self.bids[price]) > 0:
    return price

heapq.heappop(self.bid_heap)
if price in self.bids:
    del self.bids[price]
```

**Key point:** Remember prices are negative! Already handled: `price = -self.bid_heap[0]`

### TODO 4: Get Best Ask (~2 min)
**Task:** Same as get_best_bid but for asks

**What to do:** Copy TODO 3 logic, but use `self.ask_heap` and `self.asks` (no negation needed!)

### TODO 5: Match Order (~5 min)
**Task:** Implement SELL order matching logic

**What's provided:**
- Complete BUY side as template
- All structure and trade recording logic

**What to do:** Mirror the BUY logic for SELL:
```python
best_bid = self.get_best_bid()
if best_bid is None or best_bid < price:
    return trades

bid_orders = self.bids[best_bid]

for resting in list(bid_orders):
    # Same matching logic as BUY
    # Key difference: trade format is (resting, incoming, ...)
    trades.append((resting.order_id, order_id, best_bid, trade_qty))
```

## Time Complexity

| Operation | Complexity | Notes |
|-----------|-----------|-------|
| **Submit** | O(log P) | Heap push for new price level |
| **Cancel** | O(M) | Must scan deque to remove order |
| **Get Status** | O(1) | Direct dict lookup |
| **Get Best Bid/Ask** | O(1) amortized | Usually O(1), worst O(P) for cleanup |
| **Match** | O(M) | Process orders at best price level |

Where:
- **P** = number of unique price levels (typically 10-100)
- **M** = orders at a price level (typically 1-10)

## Why Heaps Are Better

### Without Heaps (simple dict):
```python
def get_best_bid(self):
    return max(self.bids.keys())  # O(P) - scans all prices!
```

### With Heaps:
```python
def get_best_bid(self):
    return -self.bid_heap[0]  # O(1) - instant access!
```

**Benefit:** O(P) → O(1) for best price queries, which happen frequently.

## Stale Price Cleanup

**Problem:** When orders are cancelled, we don't remove prices from the heap (too expensive).

**Solution:** Lazy cleanup in get_best_bid/ask:

```python
while self.bid_heap:
    price = -self.bid_heap[0]
    if price in self.bids and len(self.bids[price]) > 0:
        return price  # Valid price!

    # Stale price - remove it
    heapq.heappop(self.bid_heap)
    del self.bids[price]
```

**Why this works:** Stale prices are rare, so amortized complexity is still O(1).

## Matching Logic

### BUY Order Matching:
1. Get best (lowest) ask price
2. Check if ask ≤ buy price
3. Match against orders at that ask price in FIFO order
4. Execute trades at the **ask price** (resting order's price)
5. Update filled quantities and statuses

### SELL Order Matching:
1. Get best (highest) bid price
2. Check if bid ≥ sell price
3. Match against orders at that bid price in FIFO order
4. Execute trades at the **bid price** (resting order's price)
5. Update filled quantities and statuses

**Trade Format:**
- BUY order: `(buyer_id=incoming, seller_id=resting, price, qty)`
- SELL order: `(buyer_id=resting, seller_id=incoming, price, qty)`

## Example Walkthrough

```python
ob = OrderBook()

# Submit orders
ob.submit_order("B1", Side.BUY, 100.0, 50)   # Best bid: $100
ob.submit_order("A1", Side.SELL, 101.0, 40)  # Best ask: $101

# State of heaps:
# bid_heap = [-100.0]  (negative!)
# ask_heap = [101.0]   (positive)

# Get best prices
ob.get_best_bid()  # Returns 100.0 (negates -100.0)
ob.get_best_ask()  # Returns 101.0

# Match sell order willing to accept $99+
trades = ob.match_order("S1", Side.SELL, 99.0, 30)
# Matches with B1 at $100 (bid price)
# Returns: [("B1", "S1", 100.0, 30)]

# B1 now partially filled (30/50)
```

## Common Mistakes to Avoid

### 1. Forgetting to negate bid prices
```python
# WRONG
heapq.heappush(self.bid_heap, price)

# RIGHT
heapq.heappush(self.bid_heap, -price)
```

### 2. Negating ask prices (don't!)
```python
# WRONG
heapq.heappush(self.ask_heap, -price)

# RIGHT
heapq.heappush(self.ask_heap, price)
```

### 3. Not checking for stale prices
```python
# WRONG - might return empty price level
return self.bid_heap[0]

# RIGHT - clean up stale prices
while self.bid_heap:
    price = -self.bid_heap[0]
    if price in self.bids and len(self.bids[price]) > 0:
        return price
    heapq.heappop(self.bid_heap)
```

### 4. Wrong trade format for SELL orders
```python
# WRONG - incoming is seller, not buyer!
trades.append((order_id, resting.order_id, ...))

# RIGHT - resting is buyer for SELL orders
trades.append((resting.order_id, order_id, ...))
```

## Testing Your Implementation

The notebook includes 5 test cells:
1. **Submit orders** - Tests both BUY and SELL submissions
2. **Order status** - Tests lookup functionality
3. **Cancel order** - Tests cancellation and best price update
4. **Match sell** - Tests SELL order matching with partial fills
5. **Match buy** - Tests BUY order matching across quantities

Expected output shows orderbook state, trades executed, and final order statuses.

## Extensions (After Completing TODOs)

Once you've finished the basic implementation, try:

1. **Multi-level matching** - Match across multiple price levels
2. **Order modification** - Implement as cancel + resubmit
3. **Market orders** - Execute at best available price regardless
4. **Stop orders** - Trigger when price reaches threshold
5. **Performance testing** - Benchmark with 10,000+ orders
6. **Spread tracking** - Add get_spread() method

## Production Considerations

This implementation is educational. Real exchanges need:
- **Lock-free data structures** for concurrency
- **Direct price removal from heap** (using indexed heaps)
- **Event-driven architecture** for scalability
- **Audit logs** for regulatory compliance
- **Risk management** before order acceptance
- **Market data distribution** in microseconds

But for understanding orderbook mechanics? This is perfect! 🎯
