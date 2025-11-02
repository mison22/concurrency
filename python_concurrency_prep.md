# Python Concurrency Prep Guide for CrowdVolt

## Table of Contents

- [Core Concepts to Review](#core-concepts-to-review)
  - [Threading vs. Multiprocessing vs. Async/Await](#1-threading-vs-multiprocessing-vs-asyncawait)
- [Key Topics to Master](#key-topics-to-master)
  - [asyncio Fundamentals](#1-asyncio-fundamentals)
  - [Common Patterns You'll Likely See](#2-common-patterns-youll-likely-see)
  - [Threading Basics (May Come Up)](#3-threading-basics-may-come-up)
  - [Race Conditions and Locks](#4-race-conditions-and-locks)
- [CrowdVolt-Specific Scenarios](#crowdvolt-specific-scenarios)
  - [Order Book Updates](#scenario-1-order-book-updates)
  - [Multiple API Integrations](#scenario-2-multiple-api-integrations)
  - [Rate Limiting](#scenario-3-rate-limiting)
- [Common Interview Questions](#common-interview-questions)
  - [Conceptual Questions](#conceptual-questions)
  - [Coding Problems You Might See](#coding-problems-you-might-see)
- [Tips for the Pairing Session](#tips-for-the-pairing-session)
  - [During the Interview](#during-the-interview)
  - [Code Review Best Practices](#code-review-best-practices)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
- [Practice Problems to Try Before Interview](#practice-problems-to-try-before-interview)
- [Final Prep Checklist](#final-prep-checklist)

---

## Core Concepts to Review

### 1. Threading vs. Multiprocessing vs. Async/Await

**Threading** (`threading` module)
- Good for: I/O-bound tasks (API calls, file operations, network requests)
- Limited by: GIL (Global Interpreter Lock) - only one thread executes Python bytecode at a time
- Use case: Multiple API calls, web scraping, reading/writing files

**Multiprocessing** (`multiprocessing` module)
- Good for: CPU-bound tasks (heavy computations, data processing)
- Bypasses GIL: Each process has its own Python interpreter
- Use case: Image processing, mathematical computations, data transformations

**Async/Await** (`asyncio` module)
- Good for: I/O-bound tasks with many concurrent operations
- Single-threaded: Event loop manages coroutines
- Use case: Web servers, handling many concurrent connections, API requests
- **Most relevant for CrowdVolt** (marketplace order matching, real-time updates, payment processing)

---

## Key Topics to Master

### 1. asyncio Fundamentals

```python
import asyncio

# Basic async function
async def fetch_data(id):
    print(f"Fetching {id}...")
    await asyncio.sleep(1)  # Simulates I/O operation
    return f"Data {id}"

# Running async functions
async def main():
    # Sequential (slow)
    result1 = await fetch_data(1)
    result2 = await fetch_data(2)
    
    # Concurrent (fast)
    results = await asyncio.gather(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3)
    )
    print(results)

# Run the event loop
asyncio.run(main())
```

**Key concepts:**
- `async def` defines coroutine
- `await` pauses execution until result ready
- `asyncio.gather()` runs multiple coroutines concurrently
- `asyncio.create_task()` schedules coroutines
- `asyncio.run()` runs the event loop

### 2. Common Patterns You'll Likely See

**Pattern 1: Concurrent API Calls**
```python
import asyncio
import aiohttp

async def fetch_ticket(session, ticket_id):
    async with session.get(f"https://api.example.com/tickets/{ticket_id}") as response:
        return await response.json()

async def fetch_multiple_tickets(ticket_ids):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_ticket(session, tid) for tid in ticket_ids]
        results = await asyncio.gather(*tasks)
        return results

# Usage
ticket_ids = [1, 2, 3, 4, 5]
results = asyncio.run(fetch_multiple_tickets(ticket_ids))
```

**Pattern 2: Task Management**
```python
async def process_order(order_id):
    print(f"Processing order {order_id}")
    await asyncio.sleep(2)
    return f"Order {order_id} complete"

async def main():
    # Create tasks
    task1 = asyncio.create_task(process_order(1))
    task2 = asyncio.create_task(process_order(2))
    
    # Do other work while tasks run
    print("Doing other work...")
    
    # Wait for completion
    result1 = await task1
    result2 = await task2
    print(result1, result2)

asyncio.run(main())
```

**Pattern 3: Timeouts and Cancellation**
```python
async def slow_operation():
    await asyncio.sleep(10)
    return "Done"

async def main():
    try:
        # Timeout after 3 seconds
        result = await asyncio.wait_for(slow_operation(), timeout=3.0)
    except asyncio.TimeoutError:
        print("Operation timed out!")

asyncio.run(main())
```

**Pattern 4: Queue-Based Processing (Worker Pool)**
```python
import asyncio

async def worker(name, queue):
    while True:
        item = await queue.get()
        if item is None:
            break
        print(f"Worker {name} processing {item}")
        await asyncio.sleep(1)  # Simulate work
        queue.task_done()

async def main():
    queue = asyncio.Queue()
    
    # Create workers
    workers = [asyncio.create_task(worker(f"W{i}", queue)) for i in range(3)]
    
    # Add items to queue
    for item in range(10):
        await queue.put(item)
    
    # Wait for all items to be processed
    await queue.join()
    
    # Stop workers
    for _ in workers:
        await queue.put(None)
    
    await asyncio.gather(*workers)

asyncio.run(main())
```

### 3. Threading Basics (May Come Up)

```python
import threading
import time

def download_file(file_id):
    print(f"Downloading {file_id}...")
    time.sleep(2)
    print(f"Downloaded {file_id}")

# Create threads
threads = []
for i in range(5):
    t = threading.Thread(target=download_file, args=(i,))
    threads.append(t)
    t.start()

# Wait for all threads to complete
for t in threads:
    t.join()

print("All downloads complete")
```

**Thread-safe data structures:**
```python
from queue import Queue
import threading

# Thread-safe queue
q = Queue()

def producer():
    for i in range(5):
        q.put(i)
        print(f"Produced {i}")

def consumer():
    while True:
        item = q.get()
        if item is None:
            break
        print(f"Consumed {item}")
        q.task_done()

# Start threads
producer_thread = threading.Thread(target=producer)
consumer_thread = threading.Thread(target=consumer)

producer_thread.start()
consumer_thread.start()

producer_thread.join()
q.put(None)  # Signal consumer to stop
consumer_thread.join()
```

### 4. Race Conditions and Locks

```python
import asyncio

# Problem: Race condition
class Counter:
    def __init__(self):
        self.count = 0
    
    async def increment(self):
        current = self.count
        await asyncio.sleep(0.001)  # Simulates delay
        self.count = current + 1

# Solution: Use Lock
class SafeCounter:
    def __init__(self):
        self.count = 0
        self.lock = asyncio.Lock()
    
    async def increment(self):
        async with self.lock:
            current = self.count
            await asyncio.sleep(0.001)
            self.count = current + 1

async def test_race_condition():
    counter = SafeCounter()
    await asyncio.gather(*[counter.increment() for _ in range(100)])
    print(f"Count: {counter.count}")  # Should be 100

asyncio.run(test_race_condition())
```

---

## CrowdVolt-Specific Scenarios

### Scenario 1: Order Book Updates
```python
import asyncio

async def match_orders(bids, asks):
    """Match bids and asks concurrently"""
    # Check for matches
    matches = []
    for bid in bids:
        for ask in asks:
            if bid['price'] >= ask['price']:
                matches.append((bid, ask))
    return matches

async def process_match(bid, ask):
    """Process a matched order"""
    print(f"Matching bid {bid['id']} with ask {ask['id']}")
    # Simulate payment processing
    await asyncio.sleep(0.5)
    # Simulate ticket transfer
    await asyncio.sleep(0.5)
    return {"bid": bid['id'], "ask": ask['id'], "status": "complete"}

async def main():
    bids = [{"id": 1, "price": 100}, {"id": 2, "price": 95}]
    asks = [{"id": 3, "price": 90}, {"id": 4, "price": 98}]
    
    matches = await match_orders(bids, asks)
    
    # Process all matches concurrently
    results = await asyncio.gather(*[
        process_match(bid, ask) for bid, ask in matches
    ])
    
    print(f"Processed {len(results)} matches")

asyncio.run(main())
```

### Scenario 2: Multiple API Integrations
```python
import asyncio

async def fetch_ticketmaster(event_id):
    await asyncio.sleep(1)  # Simulate API call
    return {"platform": "ticketmaster", "tickets": 50}

async def fetch_axs(event_id):
    await asyncio.sleep(1.5)  # Simulate API call
    return {"platform": "axs", "tickets": 30}

async def fetch_dice(event_id):
    await asyncio.sleep(0.8)  # Simulate API call
    return {"platform": "dice", "tickets": 20}

async def aggregate_inventory(event_id):
    """Fetch inventory from all platforms concurrently"""
    results = await asyncio.gather(
        fetch_ticketmaster(event_id),
        fetch_axs(event_id),
        fetch_dice(event_id),
        return_exceptions=True  # Don't fail if one API fails
    )
    
    total_tickets = 0
    for result in results:
        if isinstance(result, Exception):
            print(f"API call failed: {result}")
        else:
            total_tickets += result['tickets']
    
    return total_tickets

# Usage
total = asyncio.run(aggregate_inventory("event_123"))
print(f"Total tickets available: {total}")
```

### Scenario 3: Rate Limiting
```python
import asyncio
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
            # Remove old requests outside time window
            self.requests = [req for req in self.requests 
                           if now - req < self.time_window]
            
            if len(self.requests) >= self.max_requests:
                # Wait until we can make another request
                sleep_time = self.time_window - (now - self.requests[0])
                await asyncio.sleep(sleep_time)
                self.requests = self.requests[1:]
            
            self.requests.append(now)

async def api_call(limiter, id):
    await limiter.acquire()
    print(f"Making API call {id} at {time.time()}")
    await asyncio.sleep(0.1)  # Simulate API call
    return f"Result {id}"

async def main():
    # Max 5 requests per 2 seconds
    limiter = RateLimiter(max_requests=5, time_window=2.0)
    
    # Try to make 10 requests
    results = await asyncio.gather(*[
        api_call(limiter, i) for i in range(10)
    ])
    print(results)

asyncio.run(main())
```

---

## Common Interview Questions

### Conceptual Questions
1. **"What's the difference between concurrency and parallelism?"**
   - Concurrency: Managing multiple tasks at once (interleaved)
   - Parallelism: Executing multiple tasks simultaneously (truly concurrent)

2. **"When would you use threading vs. asyncio?"**
   - Threading: I/O-bound with blocking operations, legacy code
   - Asyncio: I/O-bound with many operations, modern async libraries, better performance

3. **"What is the GIL?"**
   - Global Interpreter Lock: Prevents multiple threads from executing Python bytecode simultaneously
   - Protects memory management
   - Makes threading less effective for CPU-bound tasks

4. **"What's a race condition?"**
   - When multiple threads/coroutines access shared data and outcome depends on timing
   - Solution: Use locks, queues, or immutable data structures

### Coding Problems You Might See

**Problem 1: Concurrent Downloads**
"Write a function that downloads multiple files concurrently and returns results in order"

```python
import asyncio

async def download_file(url, index):
    # Simulate download
    await asyncio.sleep(1)
    return f"Content from {url}"

async def download_all(urls):
    tasks = [download_file(url, i) for i, url in enumerate(urls)]
    results = await asyncio.gather(*tasks)
    return results

# Test
urls = ["url1", "url2", "url3"]
results = asyncio.run(download_all(urls))
print(results)
```

**Problem 2: Worker Pool**
"Implement a worker pool that processes tasks from a queue"

```python
import asyncio

async def worker(worker_id, queue, results):
    while True:
        task = await queue.get()
        if task is None:
            queue.task_done()
            break
        
        # Process task
        result = await process_task(task)
        results.append(result)
        queue.task_done()

async def process_task(task):
    await asyncio.sleep(0.5)
    return f"Processed {task}"

async def main():
    queue = asyncio.Queue()
    results = []
    
    # Create workers
    num_workers = 3
    workers = [
        asyncio.create_task(worker(i, queue, results)) 
        for i in range(num_workers)
    ]
    
    # Add tasks
    for i in range(10):
        await queue.put(i)
    
    # Wait for all tasks to complete
    await queue.join()
    
    # Stop workers
    for _ in range(num_workers):
        await queue.put(None)
    
    await asyncio.gather(*workers)
    print(f"Results: {results}")

asyncio.run(main())
```

**Problem 3: Timeout Handling**
"Fetch data from multiple APIs with timeout, return what you can get"

```python
import asyncio

async def fetch_api(api_name, delay):
    await asyncio.sleep(delay)
    return f"Data from {api_name}"

async def fetch_with_timeout(apis, timeout=2.0):
    tasks = [fetch_api(name, delay) for name, delay in apis]
    
    results = []
    for task in asyncio.as_completed(tasks):
        try:
            result = await asyncio.wait_for(task, timeout=timeout)
            results.append(result)
        except asyncio.TimeoutError:
            results.append("Timeout")
    
    return results

# Test
apis = [("API1", 1), ("API2", 3), ("API3", 0.5)]
results = asyncio.run(fetch_with_timeout(apis))
print(results)
```

---

## Tips for the Pairing Session

### During the Interview

1. **Think Out Loud**
   - Explain your thought process
   - "I'm thinking we should use asyncio here because we're making multiple API calls"
   - "We need a lock here to prevent race conditions"

2. **Ask Clarifying Questions**
   - "Should this handle errors gracefully or fail fast?"
   - "Do we need to preserve order of results?"
   - "What's the expected scale - 10 requests or 10,000?"

3. **Start Simple, Then Optimize**
   - Get a working solution first
   - Then add error handling, timeouts, etc.
   - "Let me first get the basic version working, then we can add timeout handling"

4. **Test Your Code**
   - Suggest test cases
   - "Let me add a simple test to verify this works"
   - Run the code if possible

5. **Common Gotchas to Avoid**
   - Forgetting `await` keyword
   - Using `time.sleep()` instead of `asyncio.sleep()` in async functions
   - Not handling exceptions in `asyncio.gather()`
   - Creating event loop incorrectly

### Code Review Best Practices

- **Be collaborative**: "What do you think about this approach?"
- **Be open to feedback**: "That's a good point, let me refactor this"
- **Discuss trade-offs**: "This approach is simpler but may be slower at scale"

---

## Quick Reference Cheat Sheet

### asyncio Common Functions
```python
# Run coroutine
asyncio.run(main())

# Run multiple coroutines concurrently
await asyncio.gather(coro1(), coro2())

# Create task (schedule for execution)
task = asyncio.create_task(coro())

# Wait with timeout
await asyncio.wait_for(coro(), timeout=5.0)

# Sleep (non-blocking)
await asyncio.sleep(1)

# Get completed tasks as they finish
for coro in asyncio.as_completed(coros):
    result = await coro
```

### Threading Common Functions
```python
# Create thread
t = threading.Thread(target=func, args=(arg,))
t.start()
t.join()

# Thread-safe queue
from queue import Queue
q = Queue()
q.put(item)
item = q.get()

# Lock
lock = threading.Lock()
with lock:
    # critical section
```

---

## Practice Problems to Try Before Interview

1. **Concurrent URL Fetcher**: Write async function that fetches 20 URLs concurrently and returns results
2. **Order Processor**: Simulate order processing queue with 5 workers
3. **Rate-Limited API**: Implement rate limiter that allows max N requests per second
4. **Timeout Handler**: Fetch from 3 APIs, return results from those that complete within timeout
5. **Producer-Consumer**: Implement classic producer-consumer pattern with asyncio

---

## Final Prep Checklist

- [ ] Review asyncio basics (async/await, gather, create_task)
- [ ] Practice 2-3 coding problems with asyncio
- [ ] Understand when to use threading vs asyncio
- [ ] Know how to handle timeouts and errors
- [ ] Understand race conditions and locks
- [ ] Be ready to explain trade-offs of different approaches
- [ ] Set up Python 3 environment to test code snippets
- [ ] Review CrowdVolt use cases (order matching, API integrations, payment processing)

Good luck! Remember: they're evaluating both your technical skills AND how you collaborate in a pairing session. Communication is just as important as getting the code right.