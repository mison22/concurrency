# Python asyncio Quick Reference - Interview Cheat Sheet

## Table of Contents

- [Basic Syntax](#basic-syntax)
- [Pattern 1: Run Multiple Operations Concurrently](#pattern-1-run-multiple-operations-concurrently)
- [Pattern 2: Worker Pool with Queue](#pattern-2-worker-pool-with-queue)
- [Pattern 3: Timeout Handling](#pattern-3-timeout-handling)
- [Pattern 4: Schedule Task (Fire and Forget)](#pattern-4-schedule-task-fire-and-forget)
- [Pattern 5: Rate Limiting](#pattern-5-rate-limiting)
- [Pattern 6: Nested Concurrency](#pattern-6-nested-concurrency)
- [Common Operations](#common-operations)
  - [asyncio.gather()](#asynciogather)
  - [asyncio.create_task()](#asynciocreate_task)
  - [asyncio.Queue](#asyncioqueue)
  - [asyncio.Lock](#asynciolock)
  - [asyncio.sleep()](#asynciosleep)
- [Error Handling](#error-handling)
- [Quick Decision Tree](#quick-decision-tree)
- [Common Mistakes to Avoid](#common-mistakes-to-avoid)
- [Interview Tips](#interview-tips)
- [Quick Syntax Reference](#quick-syntax-reference)
- [CrowdVolt-Specific Patterns](#crowdvolt-specific-patterns)
- [Remember](#remember)

---

## Basic Syntax

```python
import asyncio

# Define async function
async def my_function():
    await asyncio.sleep(1)
    return "result"

# Run async function
asyncio.run(my_function())
```

---

## Pattern 1: Run Multiple Operations Concurrently

**Use when:** You have multiple independent I/O operations (API calls, database queries)

```python
async def fetch_data(id):
    await asyncio.sleep(1)  # Simulates I/O
    return f"Data {id}"

async def main():
    # Run all concurrently
    results = await asyncio.gather(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3)
    )
    # OR from a list
    tasks = [fetch_data(i) for i in range(5)]
    results = await asyncio.gather(*tasks)
```

**Key points:**
- Total time = longest operation (NOT sum of all)
- Returns results in order of input (not completion order)
- Fails fast by default (use `return_exceptions=True` to continue on error)

---

## Pattern 2: Worker Pool with Queue

**Use when:** You have many tasks and want N workers processing them

```python
async def worker(worker_id, queue):
    while True:
        task = await queue.get()
        
        # Shutdown signal
        if task is None:
            queue.task_done()
            break
        
        # Process task
        print(f"Worker {worker_id} processing {task}")
        await asyncio.sleep(1)
        queue.task_done()

async def main():
    queue = asyncio.Queue()
    
    # Add tasks
    for i in range(10):
        await queue.put(i)
    
    # Create N workers
    workers = [
        asyncio.create_task(worker(i, queue))
        for i in range(3)  # 3 workers
    ]
    
    # Wait for all tasks to complete
    await queue.join()
    
    # Stop workers
    for _ in range(3):
        await queue.put(None)
    
    await asyncio.gather(*workers)
```

**Key points:**
- `queue.get()` waits if queue empty
- `queue.task_done()` MUST be called after processing
- `queue.join()` waits until all items processed
- Send `None` to signal shutdown

---

## Pattern 3: Timeout Handling

**Use when:** Operations might take too long, you want to limit wait time

```python
async def fetch_api(id):
    await asyncio.sleep(3)  # Slow operation
    return f"Data {id}"

# Option 1: Timeout single operation
async def with_timeout():
    try:
        result = await asyncio.wait_for(
            fetch_api(1),
            timeout=2.0
        )
    except asyncio.TimeoutError:
        result = "TIMEOUT"
    return result

# Option 2: Timeout multiple operations individually
async def multiple_with_timeout():
    async def safe_fetch(id):
        try:
            return await asyncio.wait_for(fetch_api(id), timeout=2.0)
        except asyncio.TimeoutError:
            return "TIMEOUT"
    
    results = await asyncio.gather(*[
        safe_fetch(i) for i in range(5)
    ])
    return results
```

**Key points:**
- `asyncio.wait_for(coro, timeout=X)` raises `asyncio.TimeoutError`
- Wrap in try-except to handle gracefully
- Timeout applies per operation, not total

---

## Pattern 4: Schedule Task (Fire and Forget)

**Use when:** You want to start something but not wait for it immediately

```python
async def background_task():
    await asyncio.sleep(5)
    print("Background task done")

async def main():
    # Schedule task (doesn't wait)
    task = asyncio.create_task(background_task())
    
    # Do other work
    await asyncio.sleep(1)
    print("Did other work")
    
    # Later, wait for it
    await task
```

**Key points:**
- `create_task()` schedules but doesn't block
- Task starts running immediately
- Can await the task later to get result

---

## Pattern 5: Rate Limiting

**Use when:** You need to limit requests per time window (API rate limits)

```python
import time

class RateLimiter:
    def __init__(self, max_requests, time_window):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = []
        self.lock = asyncio.Lock()
    
    async def acquire(self):
        async with self.lock:
            now = time.time()
            
            # Remove old requests
            self.requests = [
                r for r in self.requests 
                if now - r < self.time_window
            ]
            
            # Wait if at limit
            if len(self.requests) >= self.max_requests:
                wait_time = self.time_window - (now - self.requests[0])
                if wait_time > 0:
                    await asyncio.sleep(wait_time)
                    # Clean up again after wait
                    now = time.time()
                    self.requests = [
                        r for r in self.requests 
                        if now - r < self.time_window
                    ]
            
            # Record this request
            self.requests.append(time.time())

# Usage
limiter = RateLimiter(max_requests=5, time_window=10.0)
await limiter.acquire()  # Call before each request
```

**Key points:**
- Use `asyncio.Lock()` to prevent race conditions
- Sliding window approach (tracks actual timestamps)
- `async with lock:` ensures thread-safety

---

## Pattern 6: Nested Concurrency

**Use when:** Each item needs multiple operations, and you have multiple items

```python
async def process_payment(order_id):
    await asyncio.sleep(1)
    return f"Payment {order_id}"

async def transfer_ticket(order_id):
    await asyncio.sleep(1)
    return f"Transfer {order_id}"

async def process_order(order):
    # Run payment and transfer concurrently for this order
    payment, transfer = await asyncio.gather(
        process_payment(order['id']),
        transfer_ticket(order['id'])
    )
    return {"order": order['id'], "payment": payment, "transfer": transfer}

async def main():
    orders = [{"id": 1}, {"id": 2}, {"id": 3}]
    
    # Process all orders concurrently
    results = await asyncio.gather(*[
        process_order(order) for order in orders
    ])
    return results
```

**Key points:**
- Inner `gather()` runs operations for one item concurrently
- Outer `gather()` processes all items concurrently
- 3 orders × 2 operations = 1 second total (not 6 seconds)

---

## Common Operations

### asyncio.gather()
```python
# Run multiple coroutines concurrently
results = await asyncio.gather(coro1(), coro2(), coro3())

# From a list
tasks = [fetch(i) for i in range(5)]
results = await asyncio.gather(*tasks)

# Don't fail on error
results = await asyncio.gather(*tasks, return_exceptions=True)
for r in results:
    if isinstance(r, Exception):
        print(f"Error: {r}")
```

### asyncio.create_task()
```python
# Schedule task without waiting
task1 = asyncio.create_task(coro1())
task2 = asyncio.create_task(coro2())

# Do other work...

# Later, wait for tasks
result1 = await task1
result2 = await task2
```

### asyncio.Queue
```python
queue = asyncio.Queue()

# Producer
await queue.put(item)

# Consumer
item = await queue.get()
# ... process item ...
queue.task_done()

# Wait for all items processed
await queue.join()
```

### asyncio.Lock
```python
lock = asyncio.Lock()

# Use lock to protect shared state
async with lock:
    # Only one coroutine can be here at a time
    shared_state += 1
```

### asyncio.sleep()
```python
# NON-blocking sleep (allows other coroutines to run)
await asyncio.sleep(1)

# NEVER use time.sleep() in async functions!
# time.sleep(1)  # ❌ Blocks entire event loop
```

---

## Error Handling

### Try-Except in Async
```python
async def safe_fetch(url):
    try:
        result = await fetch(url)
        return result
    except Exception as e:
        print(f"Error: {e}")
        return None
```

### Gather with Exception Handling
```python
# Option 1: Stop on first error (default)
try:
    results = await asyncio.gather(*tasks)
except Exception as e:
    print(f"One task failed: {e}")

# Option 2: Collect exceptions, don't stop
results = await asyncio.gather(*tasks, return_exceptions=True)
successes = [r for r in results if not isinstance(r, Exception)]
errors = [r for r in results if isinstance(r, Exception)]
```

---

## Quick Decision Tree

**"I have 5 API calls to make"**
→ Use `asyncio.gather()` (Pattern 1)

**"I have 100 tasks and want 5 workers"**
→ Use Worker Queue (Pattern 2)

**"API might be slow, need timeout"**
→ Use `asyncio.wait_for()` (Pattern 3)

**"Start task now, wait later"**
→ Use `asyncio.create_task()` (Pattern 4)

**"API has rate limit"**
→ Use Rate Limiter (Pattern 5)

**"Each order needs payment + transfer, have many orders"**
→ Use Nested Concurrency (Pattern 6)

**"Need to protect shared variable"**
→ Use `asyncio.Lock()`

---

## Common Mistakes to Avoid

### ❌ Forgetting await
```python
# WRONG
result = asyncio.sleep(1)  # Returns coroutine object, doesn't sleep

# RIGHT
result = await asyncio.sleep(1)
```

### ❌ Using time.sleep()
```python
# WRONG - blocks entire event loop
async def bad():
    time.sleep(1)

# RIGHT - allows other coroutines to run
async def good():
    await asyncio.sleep(1)
```

### ❌ Not calling task_done()
```python
# WRONG - queue.join() hangs forever
item = await queue.get()
process(item)
# Missing: queue.task_done()

# RIGHT
item = await queue.get()
process(item)
queue.task_done()
```

### ❌ Sequential instead of concurrent
```python
# WRONG - takes 3 seconds
result1 = await fetch(1)
result2 = await fetch(2)
result3 = await fetch(3)

# RIGHT - takes 1 second
results = await asyncio.gather(
    fetch(1), fetch(2), fetch(3)
)
```

### ❌ Race condition on shared state
```python
# WRONG - race condition
async def increment():
    global counter
    temp = counter
    await asyncio.sleep(0.001)
    counter = temp + 1

# RIGHT - use lock
async def increment():
    async with lock:
        global counter
        temp = counter
        await asyncio.sleep(0.001)
        counter = temp + 1
```

---

## Interview Tips

### Think Out Loud
- "I'm using gather here because we want concurrent execution"
- "This needs a lock because multiple coroutines access shared state"
- "I'll use a queue here to distribute work among workers"

### Ask Clarifying Questions
- "Should we handle the case where all APIs timeout?"
- "Do we need results in order or can they come back in any order?"
- "What's the expected scale - 10 requests or 10,000?"

### Start Simple
- Get basic version working first
- Then add error handling
- Then optimize

### Test Mentally
- "This should take 3 seconds, not 8, because we're running concurrently"
- "With 3 workers and 10 tasks, each worker should process ~3-4 tasks"

---

## Quick Syntax Reference

```python
# Run coroutine
asyncio.run(main())

# Concurrent execution
await asyncio.gather(coro1(), coro2())

# Schedule task
task = asyncio.create_task(coro())

# Timeout
await asyncio.wait_for(coro(), timeout=5.0)

# Sleep
await asyncio.sleep(1)

# Queue operations
await queue.put(item)
item = await queue.get()
queue.task_done()
await queue.join()

# Lock
async with lock:
    # critical section
    pass

# Error handling
try:
    result = await coro()
except asyncio.TimeoutError:
    result = "TIMEOUT"
```

---

## CrowdVolt-Specific Patterns

### Order Matching
```python
async def match_and_process_orders(bids, asks):
    # Find matches
    matches = [(b, a) for b in bids for a in asks 
               if b['price'] >= a['price']]
    
    # Process all matches concurrently
    results = await asyncio.gather(*[
        process_match(bid, ask) 
        for bid, ask in matches
    ])
    return results

async def process_match(bid, ask):
    # Payment and transfer concurrently
    await asyncio.gather(
        process_payment(bid, ask),
        transfer_ticket(bid, ask)
    )
```

### Multi-Platform Integration
```python
async def fetch_inventory(event_id):
    # Fetch from all platforms concurrently
    results = await asyncio.gather(
        fetch_ticketmaster(event_id),
        fetch_axs(event_id),
        fetch_dice(event_id),
        return_exceptions=True  # Don't fail if one platform down
    )
    
    total = sum(r['count'] for r in results 
                if not isinstance(r, Exception))
    return total
```

### Payment Processing with Retry
```python
async def process_payment(amount, max_retries=3):
    for attempt in range(max_retries):
        try:
            return await asyncio.wait_for(
                charge_card(amount),
                timeout=5.0
            )
        except (asyncio.TimeoutError, PaymentError):
            if attempt == max_retries - 1:
                raise
            await asyncio.sleep(2 ** attempt)  # Exponential backoff
```

---

## Remember

✅ Use `await` before async operations
✅ Use `asyncio.sleep()` not `time.sleep()`
✅ Call `queue.task_done()` after processing
✅ Use locks for shared state
✅ Think concurrent by default
✅ Handle timeouts and errors
✅ Test with print statements to verify concurrency

Good luck! 🚀