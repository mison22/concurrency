# Python Best Practices & Core Knowledge
## For Developers with 3+ Years Experience

## Table of Contents

- [1. Data Structures & Operations](#1-data-structures--operations)
  - [Lists](#lists)
  - [Dictionaries](#dictionaries)
  - [Sets](#sets)
  - [Tuples](#tuples)
- [2. Functions & Best Practices](#2-functions--best-practices)
  - [Function Definitions](#function-definitions)
  - [Arguments](#arguments)
  - [Lambda Functions](#lambda-functions)
  - [Decorators](#decorators)
- [3. Type Hints & Type Checking](#3-type-hints--type-checking)
- [4. Error Handling](#4-error-handling)
- [5. Iterators & Generators](#5-iterators--generators)
- [6. Standard Library Gems](#6-standard-library-gems)
  - [itertools](#itertools-powerful-iteration-tools)
  - [functools](#functools-function-tools)
  - [collections](#collections-specialized-data-structures)
- [7. File I/O & Context Managers](#7-file-io--context-managers)
- [8. Object-Oriented Programming](#8-object-oriented-programming)
- [9. String Operations & Formatting](#9-string-operations--formatting)
- [10. List/Dict/Set Comprehensions (Deep Dive)](#10-listdictset-comprehensions-deep-dive)
- [11. Common Patterns & Idioms](#11-common-patterns--idioms)
- [12. Performance & Best Practices](#12-performance--best-practices)
- [13. Testing Best Practices](#13-testing-best-practices)
- [14. Common Gotchas & Anti-Patterns](#14-common-gotchas--anti-patterns)
- [15. Interview-Specific Tips](#15-interview-specific-tips)
- [Quick Reference Checklist](#quick-reference-checklist)
- [Resources for Continued Learning](#resources-for-continued-learning)

---

## 1. Data Structures & Operations

### Lists

**Basic Operations**
```python
# Creation
nums = [1, 2, 3, 4, 5]
empty = []
range_list = list(range(10))

# Common methods
nums.append(6)           # Add to end
nums.insert(0, 0)        # Insert at index
nums.extend([7, 8])      # Add multiple
nums.remove(3)           # Remove first occurrence
popped = nums.pop()      # Remove and return last
popped = nums.pop(0)     # Remove and return at index

# Slicing
first_three = nums[:3]      # [0, 1, 2]
last_three = nums[-3:]      # Last 3 elements
reversed_list = nums[::-1]  # Reverse
every_other = nums[::2]     # Every 2nd element

# Copying (important!)
shallow_copy = nums.copy()  # or nums[:]
import copy
deep_copy = copy.deepcopy(nums)

# Sorting
nums.sort()                    # In-place
sorted_nums = sorted(nums)     # Returns new list
nums.sort(reverse=True)        # Descending
nums.sort(key=lambda x: abs(x))  # Custom sort
```

**List Comprehensions** (Preferred over loops)
```python
# Basic
squares = [x**2 for x in range(10)]

# With condition
evens = [x for x in range(10) if x % 2 == 0]

# With transformation and condition
processed = [x.upper() for x in words if len(x) > 3]

# Nested (but be careful - can get unreadable)
matrix = [[i+j for j in range(3)] for i in range(3)]
flattened = [item for row in matrix for item in row]

# Dict/Set comprehensions
squares_dict = {x: x**2 for x in range(5)}
unique_lengths = {len(word) for word in words}
```

### Dictionaries

**Core Operations**
```python
# Creation
user = {'name': 'Josh', 'age': 25}
empty = {}
from_keys = dict.fromkeys(['a', 'b', 'c'], 0)

# Access
name = user['name']              # KeyError if missing
name = user.get('name')          # None if missing
name = user.get('name', 'N/A')   # Default if missing

# Modification
user['email'] = 'josh@example.com'
user.update({'age': 26, 'city': 'NYC'})

# Deletion
del user['age']
age = user.pop('age', None)      # Remove and return
user.clear()                      # Remove all

# Iteration
for key in user:                  # Iterate keys
for key, value in user.items():   # Iterate key-value pairs
for value in user.values():       # Iterate values

# Merging (Python 3.9+)
merged = {**dict1, **dict2}
merged = dict1 | dict2

# Dictionary comprehension
squared = {x: x**2 for x in range(5)}
inverted = {v: k for k, v in original.items()}
```

**Advanced Dict Patterns**
```python
from collections import defaultdict, Counter

# defaultdict - no KeyError
word_count = defaultdict(int)
for word in words:
    word_count[word] += 1

groups = defaultdict(list)
for item in items:
    groups[item.category].append(item)

# Counter - for counting
from collections import Counter
counts = Counter(['a', 'b', 'a', 'c', 'b', 'a'])
# Counter({'a': 3, 'b': 2, 'c': 1})
most_common = counts.most_common(2)  # Top 2

# OrderedDict - maintains insertion order (Python 3.7+ dicts do this by default)
from collections import OrderedDict
ordered = OrderedDict([('a', 1), ('b', 2)])
```

### Sets

**Operations**
```python
# Creation
nums = {1, 2, 3, 4, 5}
empty = set()  # NOT {} - that's a dict!

# Operations
nums.add(6)
nums.remove(3)      # KeyError if not found
nums.discard(3)     # No error if not found
nums.clear()

# Set operations
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

union = a | b              # {1, 2, 3, 4, 5, 6}
intersection = a & b       # {3, 4}
difference = a - b         # {1, 2}
symmetric_diff = a ^ b     # {1, 2, 5, 6}

# Membership testing (O(1) - very fast!)
if 3 in nums:
    print("Found")
```

### Tuples

**When to Use Tuples**
- Immutable data (can't be changed)
- Dictionary keys (must be immutable)
- Unpacking
- Multiple return values

```python
# Creation
point = (10, 20)
single = (1,)  # Note the comma!

# Unpacking
x, y = point
first, *rest, last = [1, 2, 3, 4, 5]  # first=1, rest=[2,3,4], last=5

# Named tuples (better than regular tuples)
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
print(p.x, p.y)  # More readable than p[0], p[1]
```

---

## 2. Functions & Best Practices

### Function Definitions

**Basic Structure**
```python
def calculate_total(items: list, tax_rate: float = 0.08) -> float:
    """
    Calculate total price including tax.
    
    Args:
        items: List of item prices
        tax_rate: Tax rate as decimal (default 0.08)
    
    Returns:
        Total price including tax
    
    Raises:
        ValueError: If tax_rate is negative
    """
    if tax_rate < 0:
        raise ValueError("Tax rate cannot be negative")
    
    subtotal = sum(items)
    return subtotal * (1 + tax_rate)
```

**Arguments**
```python
# Positional and keyword arguments
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Josh")                    # Positional
greet(name="Josh")               # Keyword
greet("Josh", greeting="Hi")     # Mixed

# *args and **kwargs
def flexible_function(*args, **kwargs):
    print(f"Positional: {args}")
    print(f"Keyword: {kwargs}")

flexible_function(1, 2, 3, name="Josh", age=25)
# Positional: (1, 2, 3)
# Keyword: {'name': 'Josh', 'age': 25}

# Forwarding arguments
def wrapper(*args, **kwargs):
    return other_function(*args, **kwargs)
```

**Lambda Functions**
```python
# Simple transformations
square = lambda x: x**2
add = lambda x, y: x + y

# Common use cases
users.sort(key=lambda u: u['age'])
prices = list(map(lambda x: x * 1.1, original_prices))
adults = list(filter(lambda u: u['age'] >= 18, users))

# But prefer regular functions for readability
def get_age(user):
    return user['age']

users.sort(key=get_age)  # More readable
```

**Decorators** (Common Pattern)
```python
# Timing decorator
import time
from functools import wraps

def timer(func):
    @wraps(func)  # Preserves original function metadata
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.2f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "Done"

# Decorator with arguments
def retry(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
            return wrapper
        return decorator

@retry(max_attempts=5)
def flaky_api_call():
    pass
```

---

## 3. Type Hints & Type Checking

**Why Use Type Hints**
- Better IDE autocomplete
- Catch bugs before runtime
- Self-documenting code
- Required for many modern codebases

**Basic Type Hints**
```python
from typing import List, Dict, Set, Tuple, Optional, Union, Any

# Simple types
name: str = "Josh"
age: int = 25
salary: float = 75000.0
is_active: bool = True

# Collections
numbers: List[int] = [1, 2, 3]
user: Dict[str, str] = {"name": "Josh", "email": "josh@example.com"}
unique_ids: Set[int] = {1, 2, 3}
point: Tuple[int, int] = (10, 20)

# Optional (can be None)
middle_name: Optional[str] = None  # Same as Union[str, None]

# Union (multiple types)
def process_id(id: Union[int, str]) -> str:
    return str(id)

# Any (avoid when possible)
data: Any = {"could": "be", "anything": True}

# Function signatures
def get_user(user_id: int) -> Dict[str, Any]:
    return {"id": user_id, "name": "Josh"}

def process_items(items: List[Dict[str, int]]) -> Optional[int]:
    if not items:
        return None
    return sum(item['value'] for item in items)
```

**Advanced Type Hints**
```python
from typing import Callable, TypeVar, Generic, Protocol

# Callable (function types)
def apply_operation(x: int, operation: Callable[[int], int]) -> int:
    return operation(x)

# TypeVar (generic types)
T = TypeVar('T')

def first_element(items: List[T]) -> Optional[T]:
    return items[0] if items else None

# Generic classes
class Stack(Generic[T]):
    def __init__(self) -> None:
        self.items: List[T] = []
    
    def push(self, item: T) -> None:
        self.items.append(item)
    
    def pop(self) -> T:
        return self.items.pop()

# Protocol (structural typing)
class Drawable(Protocol):
    def draw(self) -> None: ...

def render(obj: Drawable) -> None:
    obj.draw()
```

---

## 4. Error Handling

**Exception Hierarchy**
```python
try:
    # Risky operation
    result = int(user_input)
except ValueError:
    # Specific exception
    print("Not a valid number")
except (TypeError, KeyError):
    # Multiple exceptions
    print("Type or Key error")
except Exception as e:
    # Catch all (be careful!)
    print(f"Unexpected error: {e}")
else:
    # Runs if no exception
    print("Success!")
finally:
    # Always runs
    cleanup()
```

**Best Practices**
```python
# DON'T: Bare except (catches everything, even KeyboardInterrupt)
try:
    risky_operation()
except:  # ❌ Too broad
    pass

# DO: Catch specific exceptions
try:
    risky_operation()
except ValueError as e:  # ✅ Specific
    handle_error(e)

# DON'T: Swallow exceptions silently
try:
    api_call()
except Exception:
    pass  # ❌ Error is lost

# DO: Log and handle appropriately
import logging

try:
    api_call()
except Exception as e:
    logging.error(f"API call failed: {e}")
    raise  # Re-raise if you can't handle it

# Custom exceptions
class InvalidOrderError(Exception):
    """Raised when order is invalid"""
    pass

class InsufficientFundsError(Exception):
    """Raised when user has insufficient funds"""
    def __init__(self, required: float, available: float):
        self.required = required
        self.available = available
        super().__init__(f"Need ${required}, have ${available}")

# Context managers for cleanup
with open('file.txt') as f:
    data = f.read()
# File automatically closed

from contextlib import contextmanager

@contextmanager
def database_transaction():
    db.begin()
    try:
        yield db
        db.commit()
    except Exception:
        db.rollback()
        raise
    finally:
        db.close()

with database_transaction() as db:
    db.execute("INSERT ...")
```

---

## 5. Iterators & Generators

**Iterators**
```python
# Everything iterable has __iter__ and __next__
numbers = [1, 2, 3]
iterator = iter(numbers)
print(next(iterator))  # 1
print(next(iterator))  # 2
print(next(iterator))  # 3
# next(iterator)  # StopIteration

# Custom iterator
class CountDown:
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

for num in CountDown(5):
    print(num)  # 5, 4, 3, 2, 1
```

**Generators** (Preferred over iterators)
```python
# Generator function (uses yield)
def countdown(start):
    while start > 0:
        yield start
        start -= 1

for num in countdown(5):
    print(num)  # 5, 4, 3, 2, 1

# Generator expressions (like list comprehension but lazy)
squares = (x**2 for x in range(1000000))  # Doesn't create list in memory
first_ten = [next(squares) for _ in range(10)]

# Practical example: reading large files
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line.strip()

for line in read_large_file('huge.txt'):
    process(line)  # Memory efficient!

# Infinite generators
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Use with itertools
from itertools import islice
first_ten_fibs = list(islice(fibonacci(), 10))
```

---

## 6. Standard Library Gems

### itertools (Powerful iteration tools)
```python
from itertools import (
    chain, combinations, permutations, 
    product, cycle, repeat, islice, 
    groupby, accumulate
)

# Chain - flatten iterables
combined = list(chain([1, 2], [3, 4], [5, 6]))  # [1,2,3,4,5,6]

# Combinations & Permutations
combos = list(combinations([1, 2, 3], 2))  # [(1,2), (1,3), (2,3)]
perms = list(permutations([1, 2, 3], 2))   # [(1,2), (1,3), (2,1), ...]

# Product - Cartesian product
pairs = list(product([1, 2], ['a', 'b']))  # [(1,'a'), (1,'b'), (2,'a'), (2,'b')]

# Cycle - infinite repeat
counter = cycle([1, 2, 3])  # 1, 2, 3, 1, 2, 3, ...

# Islice - slice iterator
first_five = list(islice(range(100), 5))  # [0,1,2,3,4]

# Groupby - group consecutive elements
data = [('A', 1), ('A', 2), ('B', 3), ('B', 4)]
for key, group in groupby(data, key=lambda x: x[0]):
    print(key, list(group))
# A [('A', 1), ('A', 2)]
# B [('B', 3), ('B', 4)]

# Accumulate - running totals
running_sum = list(accumulate([1, 2, 3, 4]))  # [1, 3, 6, 10]
```

### functools (Function tools)
```python
from functools import reduce, partial, lru_cache, wraps

# Reduce - fold operation
total = reduce(lambda acc, x: acc + x, [1, 2, 3, 4])  # 10

# Partial - pre-fill arguments
def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)
print(square(5))  # 25
print(cube(5))    # 125

# LRU Cache - memoization
@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Wraps - preserve function metadata
def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### collections (Specialized data structures)
```python
from collections import (
    defaultdict, Counter, deque, 
    namedtuple, OrderedDict, ChainMap
)

# Deque - efficient queue/stack
queue = deque([1, 2, 3])
queue.append(4)        # Add right
queue.appendleft(0)    # Add left
queue.pop()            # Remove right
queue.popleft()        # Remove left

# Named tuples (better than regular tuples)
User = namedtuple('User', ['id', 'name', 'email'])
user = User(1, 'Josh', 'josh@example.com')
print(user.name)  # 'Josh'

# ChainMap - combine multiple dicts
defaults = {'color': 'red', 'user': 'guest'}
environment = {'user': 'admin'}
combined = ChainMap(environment, defaults)
print(combined['user'])   # 'admin' (first dict wins)
print(combined['color'])  # 'red' (from defaults)
```

---

## 7. File I/O & Context Managers

**Reading Files**
```python
# Read entire file
with open('file.txt', 'r') as f:
    content = f.read()

# Read lines
with open('file.txt', 'r') as f:
    lines = f.readlines()  # List of lines

# Iterate line by line (memory efficient)
with open('file.txt', 'r') as f:
    for line in f:
        process(line.strip())

# Read JSON
import json
with open('data.json', 'r') as f:
    data = json.load(f)

# Read CSV
import csv
with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['name'], row['age'])
```

**Writing Files**
```python
# Write text
with open('output.txt', 'w') as f:
    f.write('Hello\n')
    f.writelines(['Line 1\n', 'Line 2\n'])

# Append
with open('log.txt', 'a') as f:
    f.write(f'Log entry: {timestamp}\n')

# Write JSON
import json
with open('data.json', 'w') as f:
    json.dump(data, f, indent=2)

# Write CSV
import csv
with open('output.csv', 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=['name', 'age'])
    writer.writeheader()
    writer.writerows([
        {'name': 'Josh', 'age': 25},
        {'name': 'Alex', 'age': 30}
    ])
```

**Path Operations**
```python
from pathlib import Path

# Modern way (prefer over os.path)
path = Path('data/users/josh.json')
print(path.exists())
print(path.is_file())
print(path.is_dir())
print(path.suffix)  # '.json'
print(path.stem)    # 'josh'
print(path.parent)  # 'data/users'

# Create directories
path.parent.mkdir(parents=True, exist_ok=True)

# Iterate directory
for file in Path('data').glob('*.json'):
    process(file)

# Join paths (platform-independent)
config_path = Path.home() / '.config' / 'app' / 'settings.json'
```

---

## 8. Object-Oriented Programming

**Classes & Instances**
```python
class User:
    # Class variable (shared by all instances)
    user_count = 0
    
    def __init__(self, name: str, email: str):
        # Instance variables
        self.name = name
        self.email = email
        self._password = None  # "Private" (convention)
        User.user_count += 1
    
    def __str__(self):
        """String representation"""
        return f"User({self.name}, {self.email})"
    
    def __repr__(self):
        """Developer-friendly representation"""
        return f"User(name='{self.name}', email='{self.email}')"
    
    def __eq__(self, other):
        """Equality comparison"""
        if not isinstance(other, User):
            return False
        return self.email == other.email
    
    # Instance method
    def send_email(self, message: str):
        print(f"Sending to {self.email}: {message}")
    
    # Property (getter)
    @property
    def display_name(self):
        return f"{self.name} <{self.email}>"
    
    # Property setter
    @property
    def password(self):
        raise AttributeError("Password is write-only")
    
    @password.setter
    def password(self, value: str):
        if len(value) < 8:
            raise ValueError("Password too short")
        self._password = hash(value)
    
    # Class method
    @classmethod
    def from_dict(cls, data: dict):
        return cls(data['name'], data['email'])
    
    # Static method
    @staticmethod
    def is_valid_email(email: str) -> bool:
        return '@' in email and '.' in email
```

**Inheritance**
```python
class Employee(User):
    def __init__(self, name: str, email: str, employee_id: int):
        super().__init__(name, email)
        self.employee_id = employee_id
    
    def send_email(self, message: str):
        """Override parent method"""
        print(f"[EMPLOYEE] Sending to {self.email}: {message}")

# Multiple inheritance
class Manager(Employee):
    def __init__(self, name: str, email: str, employee_id: int, team_size: int):
        super().__init__(name, email, employee_id)
        self.team_size = team_size
```

**Abstract Base Classes**
```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float) -> bool:
        """Process payment and return success status"""
        pass
    
    @abstractmethod
    def refund(self, transaction_id: str) -> bool:
        pass

class StripeProcessor(PaymentProcessor):
    def process_payment(self, amount: float) -> bool:
        # Implementation
        return True
    
    def refund(self, transaction_id: str) -> bool:
        # Implementation
        return True

# Can't instantiate abstract class
# processor = PaymentProcessor()  # TypeError
processor = StripeProcessor()  # ✅ Works
```

**Data Classes** (Python 3.7+)
```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Order:
    id: int
    customer_name: str
    items: List[str] = field(default_factory=list)
    total: float = 0.0
    
    def __post_init__(self):
        """Called after __init__"""
        if self.total < 0:
            raise ValueError("Total cannot be negative")

# Auto-generates __init__, __repr__, __eq__, etc.
order = Order(id=1, customer_name="Josh", items=["ticket"], total=100.0)
print(order)  # Order(id=1, customer_name='Josh', items=['ticket'], total=100.0)
```

---

## 9. String Operations & Formatting

**String Methods**
```python
text = "  Hello, World!  "

# Case
text.upper()           # "  HELLO, WORLD!  "
text.lower()           # "  hello, world!  "
text.capitalize()      # "  hello, world!  "
text.title()           # "  Hello, World!  "

# Whitespace
text.strip()           # "Hello, World!"
text.lstrip()          # "Hello, World!  "
text.rstrip()          # "  Hello, World!"

# Splitting/Joining
words = text.strip().split(', ')  # ["Hello", "World!"]
joined = ' '.join(words)          # "Hello World!"

# Searching
text.startswith('Hello')  # False (has leading spaces)
text.find('World')        # 9 (index, -1 if not found)
text.index('World')       # 9 (raises ValueError if not found)
text.count('l')           # 3

# Replacement
text.replace('World', 'Python')

# Checking
'123'.isdigit()      # True
'abc'.isalpha()      # True
'abc123'.isalnum()   # True
```

**String Formatting**
```python
name = "Josh"
age = 25
price = 19.99

# f-strings (preferred, Python 3.6+)
message = f"Hello, {name}!"
message = f"{name} is {age} years old"
message = f"Price: ${price:.2f}"  # Price: $19.99
message = f"{name.upper()}"        # JOSH

# Format method
message = "Hello, {}!".format(name)
message = "{name} is {age} years old".format(name=name, age=age)
message = "Price: ${:.2f}".format(price)

# Alignment and padding
f"{name:>10}"   # "      Josh" (right-aligned, width 10)
f"{name:<10}"   # "Josh      " (left-aligned)
f"{name:^10}"   # "   Josh   " (centered)
f"{age:05d}"    # "00025" (zero-padded)

# Expressions in f-strings
f"{2 + 2}"               # "4"
f"{len(name)}"           # "4"
f"{name if age >= 18 else 'Minor'}"

# Multiline f-strings
message = f"""
Hello {name},
You are {age} years old.
Total: ${price:.2f}
"""
```

---

## 10. List/Dict/Set Comprehensions (Deep Dive)

**List Comprehensions**
```python
# Basic
squares = [x**2 for x in range(10)]

# With condition
evens = [x for x in range(10) if x % 2 == 0]

# With if-else (must be before 'for')
labels = ['even' if x % 2 == 0 else 'odd' for x in range(5)]

# Nested loops
pairs = [(x, y) for x in range(3) for y in range(3)]
# [(0,0), (0,1), (0,2), (1,0), (1,1), (1,2), (2,0), (2,1), (2,2)]

# Flattening
matrix = [[1, 2], [3, 4], [5, 6]]
flattened = [item for row in matrix for item in row]
# [1, 2, 3, 4, 5, 6]

# With function call
processed = [process(x) for x in data if is_valid(x)]
```

**Dictionary Comprehensions**
```python
# Basic
squares = {x: x**2 for x in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# From two lists
keys = ['a', 'b', 'c']
values = [1, 2, 3]
mapping = {k: v for k, v in zip(keys, values)}

# Filtering
scores = {'Alice': 85, 'Bob': 92, 'Charlie': 78}
high_scores = {name: score for name, score in scores.items() if score >= 80}

# Inverting dictionary
inverted = {v: k for k, v in original.items()}

# From list of tuples
pairs = [('a', 1), ('b', 2), ('c', 3)]
mapping = {k: v for k, v in pairs}
```

**Set Comprehensions**
```python
# Basic
squares = {x**2 for x in range(10)}

# Unique lengths
words = ['hello', 'world', 'hi', 'bye']
lengths = {len(word) for word in words}  # {2, 5}

# Filtering
positive_evens = {x for x in range(-10, 10) if x > 0 and x % 2 == 0}
```

---

## 11. Common Patterns & Idioms

**Swapping**
```python
# Python way (single line)
a, b = b, a
```

**Unpacking**
```python
# Multiple assignment
x, y, z = [1, 2, 3]

# Extended unpacking
first, *middle, last = [1, 2, 3, 4, 5]
# first = 1, middle = [2, 3, 4], last = 5

# Ignore values with _
x, _, z = [1, 2, 3]

# Dictionary unpacking
user = {'name': 'Josh', 'age': 25, 'city': 'NYC'}
name, age = user['name'], user['age']  # Old way
name = user['name']; age = user['age']  # Still old

# Better with destructuring
def process_user(name, age, city):
    pass

process_user(**user)  # Unpack dict as kwargs
```

**Ternary Operator**
```python
# Instead of
if condition:
    x = value1
else:
    x = value2

# Use
x = value1 if condition else value2

# Chained
status = 'child' if age < 13 else 'teen' if age < 20 else 'adult'
```

**Enumerate & Zip**
```python
# Enumerate - get index and value
for i, item in enumerate(['a', 'b', 'c']):
    print(i, item)  # 0 a, 1 b, 2 c

# Start at different index
for i, item in enumerate(['a', 'b', 'c'], start=1):
    print(i, item)  # 1 a, 2 b, 3 c

# Zip - combine iterables
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name} is {age}")

# Zip multiple
for name, age, city in zip(names, ages, cities):
    print(f"{name}, {age}, {city}")

# Unzip
pairs = [(1, 'a'), (2, 'b'), (3, 'c')]
numbers, letters = zip(*pairs)  # (1,2,3), ('a','b','c')
```

**EAFP vs LBYL**
```python
# LBYL (Look Before You Leap) - NOT Pythonic
if key in dictionary:
    value = dictionary[key]
else:
    value = default

# EAFP (Easier to Ask Forgiveness than Permission) - Pythonic
try:
    value = dictionary[key]
except KeyError:
    value = default

# Or just use get()
value = dictionary.get(key, default)
```

**Context Managers**
```python
# Using with statement (preferred)
with open('file.txt') as f:
    data = f.read()

# Multiple context managers
with open('input.txt') as infile, open('output.txt', 'w') as outfile:
    outfile.write(infile.read())

# Custom context manager
from contextlib import contextmanager

@contextmanager
def timer():
    import time
    start = time.time()
    yield
    end = time.time()
    print(f"Took {end - start:.2f}s")

with timer():
    # Code to time
    slow_operation()
```

---

## 12. Performance & Best Practices

**Use Built-ins (They're Fast!)**
```python
# Slow - manual loop
total = 0
for num in numbers:
    total += num

# Fast - built-in
total = sum(numbers)

# Slow - manual search
found = False
for item in items:
    if item == target:
        found = True
        break

# Fast - membership test
found = target in items

# Slow - building list
result = []
for x in data:
    result.append(x * 2)

# Fast - list comprehension
result = [x * 2 for x in data]

# Fast - map (for simple operations)
result = list(map(lambda x: x * 2, data))
```

**Avoid Repeated Lookups**
```python
# Slow - repeated attribute lookup
for item in items:
    result.append(expensive.calculation.result)

# Fast - cache the lookup
calc_result = expensive.calculation.result
for item in items:
    result.append(calc_result)

# Slow - repeated method calls in loop
for i in range(len(items)):
    process(items[i])

# Fast - iterate directly
for item in items:
    process(item)
```

**Use Sets for Membership Testing**
```python
# Slow - O(n) for lists
if item in list_of_items:  # Checks every element
    pass

# Fast - O(1) for sets
items_set = set(list_of_items)
if item in items_set:  # Instant lookup
    pass
```

**String Concatenation**
```python
# Slow - string concatenation in loop
result = ""
for item in items:
    result += str(item)  # Creates new string each time

# Fast - join
result = ''.join(str(item) for item in items)

# Fast - list then join
parts = []
for item in items:
    parts.append(str(item))
result = ''.join(parts)
```

---

## 13. Testing Best Practices

**Unit Testing Basics**
```python
import unittest

class TestOrderProcessing(unittest.TestCase):
    def setUp(self):
        """Run before each test"""
        self.order = Order(id=1, total=100.0)
    
    def tearDown(self):
        """Run after each test"""
        pass
    
    def test_order_total(self):
        """Test order total calculation"""
        self.assertEqual(self.order.total, 100.0)
    
    def test_invalid_order(self):
        """Test that invalid orders raise errors"""
        with self.assertRaises(ValueError):
            Order(id=1, total=-100.0)
    
    def test_order_processing(self):
        """Test order can be processed"""
        result = self.order.process()
        self.assertTrue(result)
        self.assertEqual(self.order.status, 'processed')

if __name__ == '__main__':
    unittest.main()
```

**Pytest (More Modern)**
```python
import pytest

def test_order_total():
    order = Order(id=1, total=100.0)
    assert order.total == 100.0

def test_invalid_order():
    with pytest.raises(ValueError):
        Order(id=1, total=-100.0)

@pytest.fixture
def sample_order():
    """Reusable test fixture"""
    return Order(id=1, items=['ticket'], total=100.0)

def test_process_order(sample_order):
    result = sample_order.process()
    assert result is True
    assert sample_order.status == 'processed'

@pytest.mark.parametrize("total,expected", [
    (100.0, 100.0),
    (50.0, 50.0),
    (0.0, 0.0),
])
def test_order_totals(total, expected):
    order = Order(id=1, total=total)
    assert order.total == expected
```

---

## 14. Common Gotchas & Anti-Patterns

**Mutable Default Arguments**
```python
# WRONG - default list is shared!
def add_item(item, items=[]):
    items.append(item)
    return items

add_item(1)  # [1]
add_item(2)  # [1, 2] - unexpected!

# RIGHT - use None
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

**Late Binding Closures**
```python
# WRONG
funcs = []
for i in range(3):
    funcs.append(lambda: i)

for f in funcs:
    print(f())  # 2, 2, 2 (all print last value!)

# RIGHT - default argument captures value
funcs = []
for i in range(3):
    funcs.append(lambda x=i: x)

for f in funcs:
    print(f())  # 0, 1, 2
```

**Modifying List While Iterating**
```python
# WRONG - skips elements
items = [1, 2, 3, 4, 5]
for item in items:
    if item % 2 == 0:
        items.remove(item)  # Modifies list during iteration!

# RIGHT - iterate over copy
items = [1, 2, 3, 4, 5]
for item in items[:]:  # [:] creates copy
    if item % 2 == 0:
        items.remove(item)

# BETTER - list comprehension
items = [1, 2, 3, 4, 5]
items = [item for item in items if item % 2 != 0]
```

**Using `is` Instead of `==`**
```python
# WRONG - is checks identity, not equality
a = [1, 2, 3]
b = [1, 2, 3]
if a is b:  # False! Different objects
    pass

# RIGHT - == checks equality
if a == b:  # True! Same contents
    pass

# Exception: None, True, False (singletons)
if x is None:  # Correct
    pass
```

---

## 15. Interview-Specific Tips

**Common Interview Patterns**

**1. Two Pointers**
```python
def is_palindrome(s: str) -> bool:
    left, right = 0, len(s) - 1
    while left < right:
        if s[left] != s[right]:
            return False
        left += 1
        right -= 1
    return True
```

**2. Hash Map for O(1) Lookup**
```python
def two_sum(nums: List[int], target: int) -> List[int]:
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []
```

**3. Sliding Window**
```python
def max_sum_subarray(arr: List[int], k: int) -> int:
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i-k] + arr[i]
        max_sum = max(max_sum, window_sum)
    
    return max_sum
```

**4. Sorting for Easier Processing**
```python
def find_closest_pair(points: List[Tuple[int, int]]) -> Tuple:
    points.sort()  # Sort makes many problems easier
    min_dist = float('inf')
    closest = None
    
    for i in range(len(points) - 1):
        dist = abs(points[i][0] - points[i+1][0])
        if dist < min_dist:
            min_dist = dist
            closest = (points[i], points[i+1])
    
    return closest
```

**Think Out Loud During Interviews**
- "I'm thinking we could use a hash map here for O(1) lookup"
- "This looks like a two-pointer problem since the array is sorted"
- "Let me start with a brute force solution, then optimize"
- "The time complexity here is O(n²), we could improve it with..."

---

## Quick Reference Checklist

**Data Structures:**
- [ ] Can use list comprehensions fluently
- [ ] Know when to use list vs tuple vs set vs dict
- [ ] Understand defaultdict and Counter
- [ ] Can use deque for efficient queues

**Functions:**
- [ ] Can write functions with type hints
- [ ] Understand *args and **kwargs
- [ ] Know how to use decorators
- [ ] Can write lambda functions for simple cases

**Error Handling:**
- [ ] Always catch specific exceptions
- [ ] Know when to use try/except vs LBYL
- [ ] Can create custom exceptions
- [ ] Understand context managers (with statement)

**Iterators & Generators:**
- [ ] Prefer generators over building full lists
- [ ] Know itertools (chain, combinations, groupby)
- [ ] Can write generator functions with yield

**OOP:**
- [ ] Can write classes with __init__, __str__, __repr__
- [ ] Understand properties (@property)
- [ ] Know when to use @classmethod vs @staticmethod
- [ ] Familiar with dataclasses

**Performance:**
- [ ] Use built-in functions (sum, max, min, any, all)
- [ ] Use sets for membership testing
- [ ] Join strings efficiently
- [ ] Avoid repeated lookups in loops

**Best Practices:**
- [ ] Follow PEP 8 (style guide)
- [ ] Write docstrings for functions/classes
- [ ] Use type hints for clarity
- [ ] Handle errors gracefully
- [ ] Test edge cases

---

## Resources for Continued Learning

**Official Documentation:**
- Python Tutorial: docs.python.org/3/tutorial/
- Python Standard Library: docs.python.org/3/library/
- PEP 8 Style Guide: peps.python.org/pep-0008/

**Books:**
- "Fluent Python" by Luciano Ramalho
- "Effective Python" by Brett Slatkin
- "Python Cookbook" by David Beazley

**Practice:**
- LeetCode (Python track)
- HackerRank (Python challenges)
- Real Python (tutorials and articles)

Good luck with your interview! 🚀