# Python Concurrency Solutions & Study Guide

## Table of Contents

- [Problem 1: Concurrent API Fetcher](#problem-1-concurrent-api-fetcher)
  - [Solution](#solution)
  - [Key Concepts Explained](#key-concepts-explained)
  - [Study Notes](#study-notes)
- [Problem 2: Worker Queue](#problem-2-worker-queue)
  - [Solution](#solution-1)
  - [Key Concepts Explained](#key-concepts-explained-1)
  - [Study Notes](#study-notes-1)
- [Problem 3: Timeout Handler](#problem-3-timeout-handler)
  - [Solution](#solution-2)
  - [Key Concepts Explained](#key-concepts-explained-2)
  - [Study Notes](#study-notes-2)
- [Problem 4: Rate Limiter](#problem-4-rate-limiter)
  - [Solution](#solution-3)
  - [Key Concepts Explained](#key-concepts-explained-3)
  - [Study Notes](#study-notes-3)
- [Problem 5: CrowdVolt Order Matcher](#problem-5-crowdvolt-order-matcher)
  - [Solution](#solution-4)
  - [Key Concepts Explained](#key-concepts-explained-4)
  - [Study Notes](#study-notes-4)
- [Study Guide Summary](#study-guide-summary)
  - [Pattern Cheat Sheet](#pattern-cheat-sheet)
  - [Common Interview Questions](#common-interview-questions)
  - [Before Interview Checklist](#before-interview-checklist)
  - [During Interview Tips](#during-interview-tips)

---

## Problem 1: Concurrent API Fetcher

### Solution
```python
import asyncio
import time

async def fetch_api(api_id, delay):
    print(f"Starting fetch from API {api_id} ({delay}s)")
    await asyncio.sleep(delay)
    print(f"Completed API {api_id}")
    return f"Data from API {api_id}"

async def main():
    # Define API delays
    api_delays = [
        (1, 1.0),
        (2, 2.0),
        (3, 1.5),
        (4, 0.5),
        (5, 3.0)
    ]
    
    # Create tasks for all APIs
    tasks = [fetch_api(api_id, delay) for api_id, delay in api_delays]
    
    # Run all concurrently
    results = await asyncio.gather(*tasks)
    
    print(f"\nAll results: {results}")
    return results

if __name__ == "__main__":
    start = time.time()
    asyncio.run(main())
    print(f"Total time: {time.time() - start:.2f}s")
```

### Key Concepts Explained

**Why this works:**
1. `asyncio.gather(*tasks)` runs all coroutines concurrently
2. Total time is ~3s (longest delay), NOT 8s (sum of all delays)
3. Each `await asyncio.sleep()` releases control back to event loop

**Common mistakes:**
```python
# WRONG - Sequential (8 seconds)
for api_id, delay in api_delays:
    result = await fetch_api(api_id, delay)

# RIGHT - Concurrent (3 seconds)
results = await asyncio.gather(*tasks)
```

**What happens under the hood:**
1. Event loop starts all coroutines nearly simultaneously
2. When each hits `await asyncio.sleep()`, control returns to event loop
3. Event loop switches between coroutines as they become ready
4. All complete in time of slowest operation (3s)

### Study Notes

**asyncio.gather() syntax:**
```python
# Option 1: Unpack list
tasks = [coro1(), coro2(), coro3()]
results = await asyncio.gather(*tasks)

# Option 2: Direct arguments
results = await asyncio.gather(coro1(), coro2(), coro3())

# With error handling
results = await asyncio.gather(*tasks, return_exceptions=True)
```

**Key differences:**
- `await coro()` - waits for single coroutine
- `await asyncio.gather(*coros)` - waits for ALL, runs concurrently
- `asyncio.create_task(coro())` - schedules but doesn't wait

---

## Problem 2: Worker Queue

### Solution
```python
import asyncio

async def worker(worker_id, queue):
    while True:
        # Get task from queue
        task = await queue.get()
        
        # None signals shutdown
        if task is None:
            queue.task_done()
            break
        
        # Process task
        print(f"Worker {worker_id} processing task {task}")
        await asyncio.sleep(1)  # Simulate work
        
        # Mark task as done
        queue.task_done()

async def main():
    # Create queue
    queue = asyncio.Queue()
    
    # Add tasks to queue
    for i in range(10):
        await queue.put(i)
    
    # Create 3 workers
    workers = [
        asyncio.create_task(worker(i, queue))
        for i in range(3)
    ]
    
    # Wait for all tasks to be processed
    await queue.join()
    
    # Stop workers by sending None
    for _ in range(3):
        await queue.put(None)
    
    # Wait for workers to finish
    await asyncio.gather(*workers)
    
    print("All tasks completed!")

if __name__ == "__main__":
    asyncio.run(main())
```

### Key Concepts Explained

**Queue workflow:**
1. **Producer** adds items: `await queue.put(item)`
2. **Consumer** gets items: `item = await queue.get()`
3. **Consumer** marks done: `queue.task_done()`
4. **Wait for completion:** `await queue.join()`

**Why use queue.task_done()?**
- Tells queue "I finished processing this item"
- `queue.join()` waits until all items have `task_done()` called
- Without it, `queue.join()` hangs forever

**Worker shutdown pattern:**
```python
# Sentinel value (None) signals worker to stop
if task is None:
    queue.task_done()  # Important: mark the None as done!
    break
```

**Why create_task() instead of gather()?**
```python
# create_task() schedules workers to run in background
workers = [asyncio.create_task(worker(i, queue)) for i in range(3)]

# Workers start running immediately
# Main code continues

# Later, wait for workers to finish
await asyncio.gather(*workers)
```

### Study Notes

**Queue methods:**
```python
queue = asyncio.Queue(maxsize=10)  # Optional max size

await queue.put(item)      # Add item (waits if full)
item = await queue.get()   # Get item (waits if empty)
queue.task_done()          # Mark item as processed
await queue.join()         # Wait for all items to be processed

queue.qsize()              # Current size (approximate)
queue.empty()              # Check if empty
queue.full()               # Check if full
```

**Worker pool pattern (memorize this):**
```python
# 1. Create queue
queue = asyncio.Queue()

# 2. Add tasks
for item in items:
    await queue.put(item)

# 3. Create workers
workers = [asyncio.create_task(worker(i, queue)) for i in range(num_workers)]

# 4. Wait for completion
await queue.join()

# 5. Stop workers
for _ in range(num_workers):
    await queue.put(None)

await asyncio.gather(*workers)
```

---

## Problem 3: Timeout Handler

### Solution
```python
import asyncio

async def fetch_api(api_id, delay):
    await asyncio.sleep(delay)
    return f"Data from API {api_id}"

async def fetch_with_timeout(api_id, delay, timeout):
    try:
        result = await asyncio.wait_for(
            fetch_api(api_id, delay),
            timeout=timeout
        )
        print(f"API {api_id}: Success ({delay}s)")
        return result
    except asyncio.TimeoutError:
        print(f"API {api_id}: TIMEOUT (would take {delay}s)")
        return "TIMEOUT"

async def main():
    delays = [1.0, 3.0, 0.5, 4.0, 1.5]
    timeout = 2.0
    
    # Create tasks with timeout
    tasks = [
        fetch_with_timeout(i, delay, timeout)
        for i, delay in enumerate(delays)
    ]
    
    # Run all concurrently
    results = await asyncio.gather(*tasks)
    
    print("\nResults:")
    for i, result in enumerate(results):
        print(f"  API {i}: {result}")
    
    successes = sum(1 for r in results if r != "TIMEOUT")
    print(f"\nTotal successful: {successes}/{len(results)}")

if __name__ == "__main__":
    asyncio.run(main())
```

### Key Concepts Explained

**asyncio.wait_for() syntax:**
```python
try:
    result = await asyncio.wait_for(coroutine, timeout=2.0)
except asyncio.TimeoutError:
    # Handle timeout
    pass
```

**Why wrap in try-except?**
- `asyncio.wait_for()` raises `asyncio.TimeoutError` if timeout exceeded
- Without try-except, entire program crashes
- With try-except, we handle gracefully and continue

**Alternative approach - using as_completed():**
```python
async def main_alternative():
    delays = [1.0, 3.0, 0.5, 4.0, 1.5]
    timeout = 2.0
    
    tasks = [fetch_api(i, delay) for i, delay in enumerate(delays)]
    results = []
    
    for coro in asyncio.as_completed(tasks, timeout=timeout):
        try:
            result = await coro
            results.append(result)
        except asyncio.TimeoutError:
            results.append("TIMEOUT")
    
    return results
```

**Key difference:**
- `wait_for()` applies timeout to EACH coroutine individually
- `as_completed(timeout=X)` applies timeout to ENTIRE set

### Study Notes

**Timeout patterns:**
```python
# Pattern 1: Timeout individual operation
try:
    result = await asyncio.wait_for(fetch(), timeout=5.0)
except asyncio.TimeoutError:
    result = "TIMEOUT"

# Pattern 2: Timeout with default value
async def fetch_with_default(url, default="N/A", timeout=5.0):
    try:
        return await asyncio.wait_for(fetch(url), timeout=timeout)
    except asyncio.TimeoutError:
        return default

# Pattern 3: Partial results (as_completed)
for coro in asyncio.as_completed(tasks, timeout=10):
    try:
        result = await coro
        results.append(result)
    except asyncio.TimeoutError:
        break  # Stop waiting for rest
```

**Error handling with gather:**
```python
# Option 1: Stop on first error (default)
results = await asyncio.gather(*tasks)

# Option 2: Collect exceptions, don't stop
results = await asyncio.gather(*tasks, return_exceptions=True)

# Check for exceptions
for result in results:
    if isinstance(result, Exception):
        print(f"Error: {result}")
```

---

## Problem 4: Rate Limiter

### Solution
```python
import asyncio
import time

class RateLimiter:
    def __init__(self, max_requests, time_window):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = []  # List of timestamps
        self.lock = asyncio.Lock()
    
    async def acquire(self):
        async with self.lock:
            now = time.time()
            
            # Remove requests outside the time window
            self.requests = [
                req_time for req_time in self.requests
                if now - req_time < self.time_window
            ]
            
            # If at limit, wait until oldest request expires
            if len(self.requests) >= self.max_requests:
                oldest_request = self.requests[0]
                wait_time = self.time_window - (now - oldest_request)
                
                if wait_time > 0:
                    await asyncio.sleep(wait_time)
                
                # After waiting, remove old requests again
                now = time.time()
                self.requests = [
                    req_time for req_time in self.requests
                    if now - req_time < self.time_window
                ]
            
            # Record this request
            self.requests.append(time.time())

async def make_request(limiter, request_id, start_time):
    await limiter.acquire()
    elapsed = time.time() - start_time
    print(f"Request {request_id} at {elapsed:.2f}s")
    await asyncio.sleep(0.1)  # Simulate request
    return f"Response {request_id}"

async def main():
    limiter = RateLimiter(max_requests=3, time_window=5.0)
    start_time = time.time()
    
    # Make 10 requests
    tasks = [make_request(limiter, i, start_time) for i in range(10)]
    results = await asyncio.gather(*tasks)
    
    print(f"\nTotal time: {time.time() - start_time:.2f}s")
    print(f"Completed {len(results)} requests")

if __name__ == "__main__":
    asyncio.run(main())
```

### Key Concepts Explained

**Why use asyncio.Lock?**
- Multiple coroutines might call `acquire()` simultaneously
- Lock prevents race conditions on `self.requests` list
- Without lock, count could be wrong

**Rate limiting algorithm:**
1. Remove old timestamps (outside time window)
2. Check if at limit
3. If yes, calculate wait time and sleep
4. Record new timestamp

**Sliding window approach:**
```python
# Example: max 3 requests per 5 seconds
# Time:    0s  1s  2s  3s  4s  5s  6s  7s
# Request: 1   2   3   -   -   4   5   6
#          |----5s----|
#                     |----5s----|
```

**Lock patterns:**
```python
# Pattern 1: Context manager (preferred)
async with self.lock:
    # Critical section
    pass

# Pattern 2: Manual (don't use unless necessary)
await self.lock.acquire()
try:
    # Critical section
finally:
    self.lock.release()
```

### Study Notes

**Common rate limiting strategies:**

**1. Fixed Window**
```python
# Simple but has burst problem
class FixedWindowLimiter:
    def __init__(self, max_requests, window_seconds):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.count = 0
        self.window_start = time.time()
    
    async def acquire(self):
        now = time.time()
        if now - self.window_start >= self.window_seconds:
            # New window
            self.count = 0
            self.window_start = now
        
        if self.count >= self.max_requests:
            wait = self.window_seconds - (now - self.window_start)
            await asyncio.sleep(wait)
            self.count = 0
            self.window_start = time.time()
        
        self.count += 1
```

**2. Token Bucket** (most common in production)
```python
class TokenBucket:
    def __init__(self, rate, capacity):
        self.rate = rate  # tokens per second
        self.capacity = capacity
        self.tokens = capacity
        self.last_update = time.time()
        self.lock = asyncio.Lock()
    
    async def acquire(self):
        async with self.lock:
            now = time.time()
            # Add tokens based on time passed
            elapsed = now - self.last_update
            self.tokens = min(self.capacity, self.tokens + elapsed * self.rate)
            self.last_update = now
            
            if self.tokens < 1:
                wait_time = (1 - self.tokens) / self.rate
                await asyncio.sleep(wait_time)
                self.tokens = 0
            else:
                self.tokens -= 1
```

**3. Sliding Window** (our solution - good balance)
- Tracks actual request timestamps
- More accurate than fixed window
- Simpler than token bucket

**Lock vs Queue for coordination:**
- **Lock:** Protects shared data, prevents concurrent access
- **Queue:** Coordinates work distribution, communicates between coroutines

---

## Problem 5: CrowdVolt Order Matcher

### Solution
```python
import asyncio
import time

async def process_payment(bid_id, ask_id):
    print(f"  Payment for match Bid {bid_id} -> Ask {ask_id}")
    await asyncio.sleep(1)

async def transfer_ticket(bid_id, ask_id):
    print(f"  Transfer for match Bid {bid_id} -> Ask {ask_id}")
    await asyncio.sleep(1)

async def process_match(bid, ask):
    print(f"Processing match: Bid {bid['id']} (${bid['price']}) -> Ask {ask['id']} (${ask['price']})")
    
    # Run payment and transfer concurrently
    await asyncio.gather(
        process_payment(bid['id'], ask['id']),
        transfer_ticket(bid['id'], ask['id'])
    )
    
    print(f"Match completed: Bid {bid['id']} -> Ask {ask['id']}")
    
    return {
        "bid_id": bid['id'],
        "ask_id": ask['id'],
        "price": ask['price'],  # Trade at ask price
        "status": "completed"
    }

def find_matches(bids, asks):
    """Find all valid matches where bid price >= ask price"""
    matches = []
    
    for bid in bids:
        for ask in asks:
            if bid['price'] >= ask['price']:
                matches.append((bid, ask))
    
    return matches

async def main():
    bids = [
        {"id": 1, "price": 100},
        {"id": 2, "price": 95},
        {"id": 3, "price": 85}
    ]
    asks = [
        {"id": 4, "price": 90},
        {"id": 5, "price": 98},
        {"id": 6, "price": 88}
    ]
    
    # Find all matches
    matches = find_matches(bids, asks)
    print(f"Found {len(matches)} potential matches\n")
    
    if not matches:
        print("No matches found")
        return []
    
    # Process all matches concurrently
    tasks = [process_match(bid, ask) for bid, ask in matches]
    results = await asyncio.gather(*tasks)
    
    print(f"\nTotal matches: {len(results)}")
    return results

if __name__ == "__main__":
    start = time.time()
    results = asyncio.run(main())
    print(f"Total time: {time.time() - start:.2f}s")
    print(f"\nCompleted matches: {results}")
```

### Key Concepts Explained

**Two levels of concurrency:**
```python
# Level 1: Payment and transfer for single match (concurrent)
await asyncio.gather(
    process_payment(bid_id, ask_id),      # 1 second
    transfer_ticket(bid_id, ask_id)       # 1 second
)
# Total: 1 second (not 2)

# Level 2: All matches processed concurrently
tasks = [process_match(bid, ask) for bid, ask in matches]
results = await asyncio.gather(*tasks)
# If 3 matches, takes 1 second total (not 3)
```

**Why this matters for CrowdVolt:**
- Multiple bids/asks can match simultaneously
- Each match needs payment + ticket transfer
- Both operations can happen in parallel
- System can process many orders per second

**Order matching logic:**
```python
# Simple version (our solution)
for bid in bids:
    for ask in asks:
        if bid['price'] >= ask['price']:
            matches.append((bid, ask))

# Production version would be more sophisticated:
# - Sort by price (best prices first)
# - Match highest bid with lowest ask
# - Remove matched orders from pool
# - Handle partial fills
```

### Study Notes

**Nested concurrency pattern:**
```python
async def process_item(item):
    # Run multiple operations concurrently for this item
    await asyncio.gather(
        operation1(item),
        operation2(item),
        operation3(item)
    )

async def process_all_items(items):
    # Process all items concurrently
    tasks = [process_item(item) for item in items]
    results = await asyncio.gather(*tasks)
    return results
```

**CrowdVolt marketplace patterns:**

**1. Order Book Update**
```python
async def update_order_book(new_order):
    async with order_book_lock:
        # Add order
        order_book.add(new_order)
        
        # Find matches
        matches = find_matches(order_book.bids, order_book.asks)
        
        # Remove matched orders
        for bid, ask in matches:
            order_book.remove(bid)
            order_book.remove(ask)
    
    # Process matches outside lock (don't block order book)
    await asyncio.gather(*[
        process_match(bid, ask) for bid, ask in matches
    ])
```

**2. Multi-Platform Ticket Transfer**
```python
async def transfer_ticket(ticket_id, from_user, to_user):
    # Determine platform
    platform = get_platform(ticket_id)
    
    if platform == "ticketmaster":
        await transfer_ticketmaster(ticket_id, from_user, to_user)
    elif platform == "axs":
        await transfer_axs(ticket_id, from_user, to_user)
    elif platform == "dice":
        await transfer_dice(ticket_id, from_user, to_user)

async def transfer_batch(transfers):
    # Transfer tickets on all platforms concurrently
    tasks = [
        transfer_ticket(t['ticket_id'], t['from'], t['to'])
        for t in transfers
    ]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    # Check for failures
    failures = [r for r in results if isinstance(r, Exception)]
    return len(failures) == 0
```

**3. Payment Processing with Retry**
```python
async def process_payment_with_retry(amount, max_retries=3):
    for attempt in range(max_retries):
        try:
            result = await asyncio.wait_for(
                charge_payment(amount),
                timeout=5.0
            )
            return result
        except (asyncio.TimeoutError, PaymentError) as e:
            if attempt == max_retries - 1:
                raise
            await asyncio.sleep(2 ** attempt)  # Exponential backoff
```

---

## Study Guide Summary

### Pattern Cheat Sheet

**1. Run multiple operations concurrently**
```python
results = await asyncio.gather(op1(), op2(), op3())
```

**2. Worker pool with queue**
```python
queue = asyncio.Queue()
workers = [asyncio.create_task(worker(i, queue)) for i in range(N)]
# Add items to queue
await queue.join()
# Stop workers
for _ in workers: await queue.put(None)
await asyncio.gather(*workers)
```

**3. Timeout handling**
```python
try:
    result = await asyncio.wait_for(coro(), timeout=5.0)
except asyncio.TimeoutError:
    result = "TIMEOUT"
```

**4. Rate limiting**
```python
async with self.lock:
    # Check and update rate limit state
    if at_limit:
        await asyncio.sleep(wait_time)
    # Record request
```

**5. Nested concurrency**
```python
async def process_item(item):
    await asyncio.gather(sub_op1(item), sub_op2(item))

results = await asyncio.gather(*[process_item(i) for i in items])
```

### Common Interview Questions

**Q: "When would you use threading vs asyncio?"**
A: Use asyncio for I/O-bound tasks (APIs, databases, file I/O) because it's more efficient and easier to reason about. Use threading for blocking I/O operations that don't have async alternatives, or when working with legacy code. Use multiprocessing for CPU-bound tasks to bypass the GIL.

**Q: "What's a race condition? Give an example."**
A: When multiple coroutines access shared state and the outcome depends on timing. Example:
```python
# BAD - race condition
async def increment():
    global counter
    temp = counter
    await asyncio.sleep(0.001)
    counter = temp + 1

# GOOD - use lock
async def increment():
    async with lock:
        global counter
        temp = counter
        await asyncio.sleep(0.001)
        counter = temp + 1
```

**Q: "Explain the GIL"**
A: Global Interpreter Lock. CPython's memory management isn't thread-safe, so the GIL ensures only one thread executes Python bytecode at a time. This makes threading ineffective for CPU-bound tasks. Asyncio doesn't solve this (single-threaded), but multiprocessing does (separate processes = separate GILs).

**Q: "How does asyncio work under the hood?"**
A: Event loop tracks all coroutines. When a coroutine hits `await`, it yields control back to the event loop. The loop switches to another ready coroutine. When I/O completes, the loop marks that coroutine as ready and switches back to it. It's cooperative multitasking.

**Q: "What's the difference between asyncio.create_task() and await?"**
A: 
- `await coro()` - runs coroutine and waits for result (blocking)
- `task = asyncio.create_task(coro())` - schedules coroutine but doesn't wait (non-blocking)
- `await task` - later, wait for the task to complete

### Before Interview Checklist

- [ ] Can write asyncio.gather() from memory
- [ ] Can implement worker queue pattern
- [ ] Can handle timeouts with wait_for()
- [ ] Understand when to use locks
- [ ] Can explain GIL in 30 seconds
- [ ] Know difference between threading/asyncio/multiprocessing
- [ ] Practiced all 5 problems without AI
- [ ] Can explain CrowdVolt's order matching use case

### During Interview Tips

1. **Start simple**: "Let me first write a basic version, then we can add error handling"
2. **Think out loud**: "I'm using gather here because we want concurrent execution"
3. **Ask questions**: "Should we handle the case where all APIs timeout?"
4. **Test mentally**: "This should take 3 seconds, not 8, because we're running concurrently"
5. **Admit unknowns**: "I haven't used X before, but I think it works like Y"

Good luck! 🚀