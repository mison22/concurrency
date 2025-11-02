# 30-Minute Python Concurrency Practice Problems

Do these problems in order. Time yourself. Don't use AI assistance. Focus on getting it working first, then refine.

## Table of Contents

- [Problem 1: Concurrent API Fetcher (10 minutes)](#problem-1-concurrent-api-fetcher-10-minutes)
- [Problem 2: Worker Queue (15 minutes)](#problem-2-worker-queue-15-minutes)
- [Problem 3: Timeout Handler (15 minutes)](#problem-3-timeout-handler-15-minutes)
- [Problem 4: Rate Limiter (20 minutes)](#problem-4-rate-limiter-20-minutes)
- [Problem 5: CrowdVolt Order Matcher (25 minutes)](#problem-5-crowdvolt-order-matcher-25-minutes)
- [Solutions Check](#solutions-check)
- [Practice Strategy](#practice-strategy)
- [Common Mistakes to Avoid](#common-mistakes-to-avoid)

---

## Problem 1: Concurrent API Fetcher (10 minutes)
**Difficulty:** Easy
**Concepts:** asyncio basics, gather, sleep

### Problem Statement
Write an async function that simulates fetching data from 5 different APIs concurrently. Each API takes a different amount of time (1s, 2s, 1.5s, 0.5s, 3s). Return all results and print the total time taken.

### Requirements
- Use `asyncio.sleep()` to simulate API calls
- Run all requests concurrently (not sequentially)
- Print each result as it completes
- Return list of all results

### Expected Output
```
Starting fetch from API 4 (0.5s)
Starting fetch from API 1 (1.0s)
Starting fetch from API 3 (1.5s)
Starting fetch from API 2 (2.0s)
Starting fetch from API 5 (3.0s)
Completed API 4
Completed API 1
Completed API 3
Completed API 2
Completed API 5
Total time: ~3 seconds (not 8 seconds!)
```

### Starter Code
```python
import asyncio
import time

async def fetch_api(api_id, delay):
    # Your code here
    pass

async def main():
    # Your code here
    pass

if __name__ == "__main__":
    start = time.time()
    asyncio.run(main())
    print(f"Total time: {time.time() - start:.2f}s")
```

### Hints
- Use `asyncio.gather()` to run multiple coroutines
- Use `await asyncio.sleep(delay)` to simulate API call
- Print statements help track progress

---

## Problem 2: Worker Queue (15 minutes)
**Difficulty:** Medium
**Concepts:** asyncio.Queue, workers, task_done

### Problem Statement
Implement a task processing system with 3 workers that process items from a queue. Items are numbers 0-9. Each worker should process tasks as they become available. Workers should print which task they're processing.

### Requirements
- Create a queue with 10 items (numbers 0-9)
- Create 3 worker coroutines that process items from the queue
- Each task takes 1 second to process
- Workers should stop when queue is empty
- Print which worker is processing which task

### Expected Output
```
Worker 0 processing task 0
Worker 1 processing task 1
Worker 2 processing task 2
Worker 0 processing task 3
Worker 1 processing task 4
Worker 2 processing task 5
...
All tasks completed!
```

### Starter Code
```python
import asyncio

async def worker(worker_id, queue):
    # Your code here
    pass

async def main():
    queue = asyncio.Queue()
    
    # Add tasks to queue
    # Your code here
    
    # Create workers
    # Your code here
    
    # Wait for completion
    # Your code here
    
    print("All tasks completed!")

if __name__ == "__main__":
    asyncio.run(main())
```

### Hints
- `await queue.get()` to get an item
- `queue.task_done()` to mark item as processed
- `await queue.join()` to wait for all items to be processed
- Use `while True` loop in worker, break when item is None
- Add None to queue for each worker to signal shutdown

---

## Problem 3: Timeout Handler (15 minutes)
**Difficulty:** Medium
**Concepts:** asyncio.wait_for, exception handling, as_completed

### Problem Statement
Fetch data from 5 APIs, but some might be slow. Set a 2-second timeout. Return results from APIs that complete in time, and "TIMEOUT" for those that don't.

### Requirements
- 5 simulated API calls with delays: [1s, 3s, 0.5s, 4s, 1.5s]
- 2-second timeout for each API call
- Return results in original order (not completion order)
- Handle timeout exceptions gracefully

### Expected Output
```
Results: 
API 0: Success (1.0s)
API 1: TIMEOUT (would take 3.0s)
API 2: Success (0.5s)
API 3: TIMEOUT (would take 4.0s)
API 4: Success (1.5s)
Total successful: 3/5
```

### Starter Code
```python
import asyncio

async def fetch_api(api_id, delay):
    await asyncio.sleep(delay)
    return f"Data from API {api_id}"

async def fetch_with_timeout(api_id, delay, timeout):
    # Your code here - handle timeout
    pass

async def main():
    delays = [1.0, 3.0, 0.5, 4.0, 1.5]
    timeout = 2.0
    
    # Your code here
    pass

if __name__ == "__main__":
    asyncio.run(main())
```

### Hints
- Use `asyncio.wait_for(coroutine, timeout=2.0)`
- Catch `asyncio.TimeoutError`
- Use list comprehension with gather to run all concurrently
- Track successes vs timeouts

---

## Problem 4: Rate Limiter (20 minutes)
**Difficulty:** Hard
**Concepts:** Locks, timestamps, rate limiting logic

### Problem Statement
Implement a rate limiter that allows maximum 3 requests per 5 seconds. Test it by making 10 requests and observe the rate limiting in action.

### Requirements
- Max 3 requests per 5-second window
- Implement using asyncio.Lock
- Track request timestamps
- When limit reached, wait before allowing next request
- Print timestamp of each request to verify rate limiting works

### Expected Output
```
Request 1 at 0.00s
Request 2 at 0.00s
Request 3 at 0.00s
Request 4 at 5.01s  <- Had to wait!
Request 5 at 5.01s
Request 6 at 5.01s
Request 7 at 10.02s <- Had to wait again!
...
```

### Starter Code
```python
import asyncio
import time

class RateLimiter:
    def __init__(self, max_requests, time_window):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = []  # Store timestamps
        self.lock = asyncio.Lock()
    
    async def acquire(self):
        # Your code here
        pass

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

if __name__ == "__main__":
    asyncio.run(main())
```

### Hints
- Store timestamps in `self.requests` list
- Remove timestamps older than `time_window`
- If at limit, calculate how long to wait
- Use `await asyncio.sleep(wait_time)` to throttle
- Use lock to prevent race conditions

---

## Problem 5: CrowdVolt Order Matcher (25 minutes)
**Difficulty:** Hard
**Concepts:** Multiple concepts combined, realistic scenario

### Problem Statement
Simulate a simplified order matching system. You have bids and asks coming in. When a bid price >= ask price, match them and process the order concurrently.

### Requirements
- Accept list of bids: `[{"id": 1, "price": 100}, {"id": 2, "price": 95}]`
- Accept list of asks: `[{"id": 3, "price": 90}, {"id": 4, "price": 98}]`
- Find all matches where bid_price >= ask_price
- Process each match concurrently (simulate payment + transfer taking 1s each)
- Return list of completed matches
- Handle case where no matches exist

### Expected Output
```
Found 2 potential matches
Processing match: Bid 1 ($100) -> Ask 3 ($90)
Processing match: Bid 2 ($95) -> Ask 3 ($90)
  Payment for match Bid 1 -> Ask 3
  Payment for match Bid 2 -> Ask 3
  Transfer for match Bid 1 -> Ask 3
  Transfer for match Bid 2 -> Ask 3
Match completed: Bid 1 -> Ask 3
Match completed: Bid 2 -> Ask 3
Total matches: 2
Total time: ~2s (not 4s!)
```

### Starter Code
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
    
    # Your code here - process payment and transfer concurrently
    # Then return match result
    pass

def find_matches(bids, asks):
    # Your code here - find all valid matches
    pass

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
    
    # Your code here
    pass

if __name__ == "__main__":
    start = time.time()
    asyncio.run(main())
    print(f"\nTotal time: {time.time() - start:.2f}s")
```

### Hints
- Use nested loops to find matches (bid_price >= ask_price)
- Use `asyncio.gather()` to run payment and transfer concurrently
- Use `asyncio.gather()` again to process all matches concurrently
- Return dict with bid_id, ask_id, status

---

## Solutions Check

After attempting each problem, verify your solution works by checking:

### Problem 1 Checklist
- [ ] Total time is ~3 seconds (not 8 seconds)
- [ ] All 5 API results are returned
- [ ] Uses `asyncio.gather()`
- [ ] Uses `await asyncio.sleep()`

### Problem 2 Checklist
- [ ] All 10 tasks are processed
- [ ] Workers process tasks concurrently (not sequentially)
- [ ] Each worker processes multiple tasks
- [ ] Program exits cleanly

### Problem 3 Checklist
- [ ] Returns 5 results (3 successes, 2 timeouts)
- [ ] Results are in original order
- [ ] Catches `asyncio.TimeoutError`
- [ ] Total time is ~2 seconds (not 10+ seconds)

### Problem 4 Checklist
- [ ] First 3 requests happen immediately
- [ ] Request 4 waits ~5 seconds
- [ ] Rate limiting pattern repeats (3 requests, wait, 3 requests, wait)
- [ ] Total time is ~15 seconds for 10 requests

### Problem 5 Checklist
- [ ] Correctly identifies matches (bid >= ask price)
- [ ] Processes payment and transfer concurrently (2s not 4s)
- [ ] Processes multiple matches concurrently
- [ ] Returns completed match details

---

## Practice Strategy

**Day 1-2 Before Interview:**
- Do Problems 1-3 (should take ~40 minutes total)
- Review solutions, understand what you got wrong
- Redo any you struggled with

**Day of Interview:**
- Quickly redo Problem 1 (warm-up)
- Review the concurrency prep guide concepts
- Do Problem 4 or 5 if you have time

**During Interview:**
- If you get stuck, explain your thinking process
- It's okay to ask for hints
- Start with a simple solution, then optimize

---

## Common Mistakes to Avoid

1. **Forgetting `await`** - Most common mistake
   ```python
   # WRONG
   result = asyncio.sleep(1)
   
   # RIGHT
   result = await asyncio.sleep(1)
   ```

2. **Using `time.sleep()` instead of `asyncio.sleep()`**
   ```python
   # WRONG - blocks event loop
   time.sleep(1)
   
   # RIGHT - allows other coroutines to run
   await asyncio.sleep(1)
   ```

3. **Not awaiting `queue.join()`**
   ```python
   # WRONG
   queue.join()
   
   # RIGHT
   await queue.join()
   ```

4. **Creating event loop incorrectly**
   ```python
   # WRONG (Python 3.10+)
   loop = asyncio.get_event_loop()
   loop.run_until_complete(main())
   
   # RIGHT
   asyncio.run(main())
   ```

5. **Not handling exceptions in gather**
   ```python
   # If you want to continue even if one fails
   results = await asyncio.gather(*tasks, return_exceptions=True)
   ```

Good luck with your practice! Time yourself and don't use AI assistance. The struggle is part of the learning process.