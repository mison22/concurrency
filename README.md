# Python Concurrency Study Materials

Study materials and practice problems for Python coding tests with a focus on concurrency and asynchronous programming.

## 📚 Repository Contents

This repository contains five comprehensive guides to help you prepare for Python coding interviews, particularly those focusing on concurrency, async/await, and performance optimization.

### 1. **[`python_concurrency_prep.md`](python_concurrency_prep.md)** - Core Concepts Guide
**Purpose:** Fundamental concepts and theory for concurrency in Python

**Contains:**
- Threading vs. Multiprocessing vs. Async/Await comparisons
- asyncio fundamentals with examples
- Common async patterns (concurrent API calls, task management, timeouts, worker pools)
- Race conditions and locks
- CrowdVolt-specific scenarios
- Interview tips and best practices

**Best for:** Understanding the theory and concepts before diving into practice

---

### 2. **[`asyncio_cheat_sheet.md`](asyncio_cheat_sheet.md)** - Quick Reference Guide
**Purpose:** Fast lookup for asyncio syntax, patterns, and common operations

**Contains:**
- 6 common async patterns with code examples
- Quick syntax reference for common operations
- Decision tree for choosing the right pattern
- Common mistakes to avoid
- Interview tips and mental models
- CrowdVolt-specific patterns

**Best for:** Quick reference during practice or as a refresher before the interview

---

### 3. **[`concurrency_practice_problems.md`](concurrency_practice_problems.md)** - Practice Problems
**Purpose:** Hands-on coding problems to test your skills

**Contains:**
- 5 problems with increasing difficulty (Easy → Hard)
- Problem statements with requirements
- Expected output examples
- Starter code templates
- Hints for each problem
- Practice strategy recommendations
- Self-check checklists

**Problems:**
1. **Concurrent API Fetcher** (Easy) - Basic asyncio.gather usage
2. **Worker Queue** (Medium) - Queue and worker pool pattern
3. **Timeout Handler** (Medium) - Error handling with timeouts
4. **Rate Limiter** (Hard) - Locks and rate limiting logic
5. **Order Matcher** (Hard) - Multi-level concurrency

**Best for:** Practicing coding skills and testing your understanding

---

### 4. **[`concurrency_solutions.md`](concurrency_solutions.md)** - Detailed Solutions
**Purpose:** Step-by-step solutions with explanations

**Contains:**
- Complete solutions for all 5 practice problems
- Key concepts explained for each solution
- Common mistakes highlighted
- Alternative approaches
- Study notes and patterns
- Interview questions and answers
- Before-interview checklist

**Best for:** Reviewing solutions after attempting problems, understanding patterns deeply

---

### 5. **[`python_best_practices.md`](python_best_practices.md)** - Python Fundamentals
**Purpose:** Comprehensive Python knowledge for developers with 3+ years experience

**Contains:**
- Data structures (lists, dicts, sets, tuples) with best practices
- Functions, decorators, and advanced patterns
- Type hints and type checking
- Error handling strategies
- Iterators and generators
- Standard library gems (itertools, functools, collections)
- Object-oriented programming
- String operations and formatting
- Common patterns and idioms
- Performance optimization tips
- Testing best practices
- Common gotchas and anti-patterns

**Best for:** Brushing up on Python fundamentals and best practices

---

## 🎯 How to Use This Repository

### Recommended Study Path

#### **Week Before Interview**

1. **Day 1: Concepts & Theory**
   - Read [`python_concurrency_prep.md`](python_concurrency_prep.md) thoroughly
   - Take notes on concepts you're unfamiliar with
   - Understand the difference between threading, multiprocessing, and asyncio

2. **Day 2-3: Practice Problems**
   - Start with [`concurrency_practice_problems.md`](concurrency_practice_problems.md)
   - Attempt Problem 1-3 without looking at solutions
   - Time yourself (each problem has a suggested time limit)
   - Check your solutions against [`concurrency_solutions.md`](concurrency_solutions.md)
   - Review what you got wrong and why

3. **Day 4: Advanced Problems**
   - Attempt Problems 4-5 (Rate Limiter, Order Matcher)
   - These are harder and test multiple concepts together
   - Review solutions and understand the patterns

4. **Day 5: Review & Reference**
   - Use [`asyncio_cheat_sheet.md`](asyncio_cheat_sheet.md) as a quick reference
   - Practice writing common patterns from memory
   - Review [`python_best_practices.md`](python_best_practices.md) for any Python fundamentals you're rusty on

#### **Day of Interview**

1. **Morning Warm-up:**
   - Quickly redo Problem 1 (10 minutes)
   - Review [`asyncio_cheat_sheet.md`](asyncio_cheat_sheet.md) patterns
   - Go through the interview tips in [`concurrency_solutions.md`](concurrency_solutions.md)

2. **Before Interview:**
   - Read the "Tips for the Pairing Session" section in [`python_concurrency_prep.md`](python_concurrency_prep.md)
   - Review common mistakes to avoid
   - Be ready to explain your thought process

---

## 📋 Study Checklist

Use this checklist to track your preparation:

### Core Concepts
- [ ] Understand the difference between threading, multiprocessing, and asyncio
- [ ] Can explain when to use each approach
- [ ] Understand what the GIL is and its implications
- [ ] Know what a race condition is and how to prevent it

### asyncio Skills
- [ ] Can write `async def` functions correctly
- [ ] Know when to use `await`, `asyncio.gather()`, and `asyncio.create_task()`
- [ ] Can implement worker pools with queues
- [ ] Can handle timeouts with `asyncio.wait_for()`
- [ ] Can use locks to prevent race conditions
- [ ] Can implement rate limiting

### Practice Problems
- [ ] Completed Problem 1: Concurrent API Fetcher (under 15 min)
- [ ] Completed Problem 2: Worker Queue (under 20 min)
- [ ] Completed Problem 3: Timeout Handler (under 20 min)
- [ ] Completed Problem 4: Rate Limiter (under 30 min)
- [ ] Completed Problem 5: Order Matcher (under 35 min)

### Python Fundamentals
- [ ] Comfortable with list/dict comprehensions
- [ ] Can use type hints properly
- [ ] Understand error handling best practices
- [ ] Know common standard library modules

---

## 🚀 Quick Start

1. **First time?** Start with [`python_concurrency_prep.md`](python_concurrency_prep.md) to build your foundation
2. **Need quick reference?** Open [`asyncio_cheat_sheet.md`](asyncio_cheat_sheet.md)
3. **Want to practice?** Start with Problem 1 in [`concurrency_practice_problems.md`](concurrency_practice_problems.md)
4. **Stuck on a problem?** Check [`concurrency_solutions.md`](concurrency_solutions.md) but try to solve it first!

---

## 💡 Key Patterns to Master

These patterns appear frequently in concurrency interviews:

1. **Concurrent Execution**
   ```python
   results = await asyncio.gather(*[fetch(i) for i in items])
   ```

2. **Worker Pool**
   ```python
   workers = [asyncio.create_task(worker(i, queue)) for i in range(n)]
   await queue.join()
   ```

3. **Timeout Handling**
   ```python
   try:
       result = await asyncio.wait_for(operation(), timeout=5.0)
   except asyncio.TimeoutError:
       result = "TIMEOUT"
   ```

4. **Rate Limiting**
   ```python
   async with limiter.lock:
       if at_limit:
           await asyncio.sleep(wait_time)
   ```

5. **Nested Concurrency**
   ```python
   async def process_item(item):
       await asyncio.gather(sub_op1(item), sub_op2(item))
   
   results = await asyncio.gather(*[process_item(i) for i in items])
   ```

---

## ⚠️ Common Mistakes to Avoid

1. **Forgetting `await`** - Most common mistake
   ```python
   # ❌ WRONG
   result = asyncio.sleep(1)
   
   # ✅ RIGHT
   result = await asyncio.sleep(1)
   ```

2. **Using `time.sleep()` in async code**
   ```python
   # ❌ WRONG - blocks event loop
   time.sleep(1)
   
   # ✅ RIGHT - non-blocking
   await asyncio.sleep(1)
   ```

3. **Not calling `queue.task_done()`**
   ```python
   # ❌ WRONG - queue.join() hangs forever
   item = await queue.get()
   process(item)
   
   # ✅ RIGHT
   item = await queue.get()
   process(item)
   queue.task_done()
   ```

4. **Sequential instead of concurrent**
   ```python
   # ❌ WRONG - 3 seconds total
   result1 = await fetch(1)
   result2 = await fetch(2)
   result3 = await fetch(3)
   
   # ✅ RIGHT - 1 second total
   results = await asyncio.gather(fetch(1), fetch(2), fetch(3))
   ```

---

## 📖 Additional Resources

- **Official Python asyncio docs:** https://docs.python.org/3/library/asyncio.html
- **Real Python asyncio tutorial:** https://realpython.com/async-io-python/
- **PEP 484** (Type Hints): https://peps.python.org/pep-0484/
- **PEP 8** (Style Guide): https://peps.python.org/pep-0008/

---

## 🎓 Interview Tips

### During the Interview

1. **Think Out Loud**
   - "I'm using `gather()` here because we want concurrent execution"
   - "This needs a lock because multiple coroutines access shared state"

2. **Ask Clarifying Questions**
   - "Should we handle the case where all APIs timeout?"
   - "Do we need results in order or can they come back in any order?"
   - "What's the expected scale - 10 requests or 10,000?"

3. **Start Simple, Then Optimize**
   - Get basic version working first
   - Then add error handling, timeouts, etc.
   - "Let me first get the basic version working, then we can add timeout handling"

4. **Test Your Code Mentally**
   - "This should take 3 seconds, not 8, because we're running concurrently"
   - "With 3 workers and 10 tasks, each worker should process ~3-4 tasks"

---

## 📝 Notes

- All code examples are written for Python 3.7+
- Focus on `asyncio` as it's most relevant for modern Python concurrency interviews
- These materials are designed for developers with 3+ years of Python experience
- Practice problems should be attempted without AI assistance for best learning

---

## 🤝 Contributing

If you find errors or want to add improvements, feel free to submit issues or pull requests.

---

Good luck with your interview! Remember: communication and problem-solving approach are just as important as getting the code perfect. 🚀

