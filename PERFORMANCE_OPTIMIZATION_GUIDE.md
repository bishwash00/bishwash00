# Performance Optimization Guide

A comprehensive guide to identifying and fixing slow or inefficient code.

## Table of Contents
1. [Common Performance Issues](#common-performance-issues)
2. [Algorithm Optimization](#algorithm-optimization)
3. [Data Structure Selection](#data-structure-selection)
4. [Database Query Optimization](#database-query-optimization)
5. [Frontend Performance](#frontend-performance)
6. [Profiling Tools](#profiling-tools)
7. [Best Practices](#best-practices)

## Common Performance Issues

### 1. Nested Loops (O(n²) complexity)

**❌ Inefficient Code:**
```javascript
// O(n²) - checking duplicates with nested loops
function hasDuplicates(arr) {
    for (let i = 0; i < arr.length; i++) {
        for (let j = i + 1; j < arr.length; j++) {
            if (arr[i] === arr[j]) {
                return true;
            }
        }
    }
    return false;
}
```

**✅ Optimized Code:**
```javascript
// O(n) - using a Set for O(1) lookups
function hasDuplicates(arr) {
    const seen = new Set();
    for (const item of arr) {
        if (seen.has(item)) {
            return true;
        }
        seen.add(item);
    }
    return false;
}

// Or even simpler:
function hasDuplicates(arr) {
    return new Set(arr).size !== arr.length;
}
```

**Why it's better:** Reduces time complexity from O(n²) to O(n).

---

### 2. Repeated Array Searches

**❌ Inefficient Code:**
```python
# Searching array multiple times - O(n) per search
def filter_common_elements(list1, list2):
    result = []
    for item in list1:
        if item in list2:  # O(n) search each time
            result.append(item)
    return result
```

**✅ Optimized Code:**
```python
# Using set for O(1) lookups
def filter_common_elements(list1, list2):
    set2 = set(list2)  # O(n) once
    return [item for item in list1 if item in set2]  # O(1) per lookup

# Or using set intersection
def filter_common_elements(list1, list2):
    return list(set(list1) & set(list2))
```

**Why it's better:** Reduces time complexity from O(n × m) to O(n + m).

---

### 3. String Concatenation in Loops

**❌ Inefficient Code:**
```python
# Creating new string objects repeatedly
def build_string(items):
    result = ""
    for item in items:
        result += str(item) + ","  # Creates new string each iteration
    return result
```

**✅ Optimized Code:**
```python
# Using join which is optimized for string concatenation
def build_string(items):
    return ",".join(str(item) for item in items)

# Or using list accumulation
def build_string(items):
    parts = []
    for item in items:
        parts.append(str(item))
    return ",".join(parts)
```

**Why it's better:** Strings are immutable; concatenation creates new objects. `join()` is optimized for this pattern.

---

### 4. Unnecessary DOM Queries

**❌ Inefficient Code:**
```javascript
// Querying DOM repeatedly
function updateItems() {
    for (let i = 0; i < 100; i++) {
        document.getElementById('item-' + i).style.color = 'red';
        document.getElementById('item-' + i).style.fontSize = '14px';
    }
}
```

**✅ Optimized Code:**
```javascript
// Cache DOM queries and batch updates using CSS
function updateItems() {
    for (let i = 0; i < 100; i++) {
        const item = document.getElementById('item-' + i);
        item.style.cssText = 'color: red; font-size: 14px;';
    }
}

// Or use CSS classes (best approach)
function updateItems() {
    for (let i = 0; i < 100; i++) {
        const item = document.getElementById('item-' + i);
        item.classList.add('updated-style');
    }
}
```

**Why it's better:** Reduces DOM queries and reflows.

---

## Algorithm Optimization

### Problem: Finding if Array Contains Target Sum

**❌ Inefficient - O(n²):**
```javascript
function hasPairWithSum(arr, target) {
    for (let i = 0; i < arr.length; i++) {
        for (let j = i + 1; j < arr.length; j++) {
            if (arr[i] + arr[j] === target) {
                return true;
            }
        }
    }
    return false;
}
```

**✅ Optimized - O(n):**
```javascript
function hasPairWithSum(arr, target) {
    const complements = new Set();
    for (const num of arr) {
        if (complements.has(target - num)) {
            return true;
        }
        complements.add(num);
    }
    return false;
}
```

---

### Problem: Fibonacci Sequence

**❌ Inefficient - Exponential O(2ⁿ):**
```python
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

**✅ Optimized with Memoization - O(n):**
```python
def fibonacci(n, memo=None):
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo)
    return memo[n]

# Or iterative approach
def fibonacci(n):
    if n <= 1:
        return n
    prev, curr = 0, 1
    for _ in range(2, n + 1):
        prev, curr = curr, prev + curr
    return curr
```

---

## Data Structure Selection

### When to Use What

| Operation | Array | Linked List | Hash Table | Binary Search Tree |
|-----------|-------|-------------|------------|--------------------|
| Access by index | O(1) ✅* | O(n) ❌ | N/A | N/A |
| Search | O(n) | O(n) | O(1) ✅ | O(log n) |
| Insert at end | O(1)† ✅ | O(1) ✅ | O(1) ✅ | O(log n) |
| Insert at beginning | O(n)‡ | O(1) ✅ | O(1) ✅ | O(log n) |
| Delete | O(n) | O(1) ✅ | O(1) ✅ | O(log n) |
| Ordered traversal | O(n) ✅ | O(n) ✅ | N/A | O(n) ✅ |

*Assumes contiguous memory (most implementations)  
†O(1) amortized - may resize occasionally  
‡Requires shifting all elements in dynamic arrays

### Example: Frequent Lookups

**❌ Using Array:**
```javascript
const users = [];

function findUser(id) {
    return users.find(u => u.id === id); // O(n)
}
```

**✅ Using Map:**
```javascript
const users = new Map();

function findUser(id) {
    return users.get(id); // O(1)
}
```

---

## Database Query Optimization

### 1. N+1 Query Problem

**❌ Inefficient:**
```javascript
// Fetching users, then posts for each user separately
const users = await db.query('SELECT * FROM users');
for (const user of users) {
    user.posts = await db.query('SELECT * FROM posts WHERE user_id = ?', [user.id]);
}
// Total queries: 1 + N
```

**✅ Optimized:**
```javascript
// Single query with JOIN
const usersWithPosts = await db.query(`
    SELECT users.*, posts.*
    FROM users
    LEFT JOIN posts ON users.id = posts.user_id
`);
// Total queries: 1
```

### 2. Missing Indexes

**❌ Without Index:**
```sql
-- Full table scan
SELECT * FROM users WHERE email = 'user@example.com';
```

**✅ With Index:**
```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Now the query uses index
SELECT * FROM users WHERE email = 'user@example.com';
```

### 3. SELECT * Problem

**❌ Fetching Unnecessary Data:**
```sql
SELECT * FROM users WHERE id = 1;
```

**✅ Fetch Only What You Need:**
```sql
SELECT id, name, email FROM users WHERE id = 1;
```

---

## Frontend Performance

### 1. Debouncing User Input

**❌ Calling API on Every Keystroke:**
```javascript
input.addEventListener('input', (e) => {
    fetchSearchResults(e.target.value); // API call on every keystroke
});
```

**✅ Debounced API Calls:**
```javascript
function debounce(func, delay) {
    let timeoutId;
    return function (...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
}

input.addEventListener('input', debounce((e) => {
    fetchSearchResults(e.target.value);
}, 300));
```

### 2. Virtual Scrolling for Large Lists

**❌ Rendering 10,000 DOM Elements:**
```javascript
function renderList(items) {
    const container = document.getElementById('list');
    items.forEach(item => {
        const div = document.createElement('div');
        div.textContent = item;
        container.appendChild(div);
    });
}
```

**✅ Virtual Scrolling (Render Only Visible):**
```javascript
function renderVirtualList(items, containerHeight, itemHeight) {
    const container = document.getElementById('list');
    const visibleCount = Math.ceil(containerHeight / itemHeight);
    
    container.addEventListener('scroll', () => {
        const scrollTop = container.scrollTop;
        const startIndex = Math.floor(scrollTop / itemHeight);
        const endIndex = startIndex + visibleCount;
        
        // Only render visible items
        renderItems(items.slice(startIndex, endIndex), startIndex);
    });
}
```

### 3. Lazy Loading Images

**❌ Loading All Images Upfront:**
```html
<img src="large-image-1.jpg" />
<img src="large-image-2.jpg" />
<img src="large-image-3.jpg" />
```

**✅ Lazy Loading:**
```html
<img src="placeholder.jpg" data-src="large-image-1.jpg" loading="lazy" />
<img src="placeholder.jpg" data-src="large-image-2.jpg" loading="lazy" />
<img src="placeholder.jpg" data-src="large-image-3.jpg" loading="lazy" />
```

---

## Profiling Tools

### JavaScript
- **Chrome DevTools Performance Tab**: Record runtime performance
- **Console.time()**: Measure code execution time
```javascript
console.time('operation');
// ... code to measure
console.timeEnd('operation');
```

### Python
- **cProfile**: Built-in profiler
```python
import cProfile
cProfile.run('my_function()')
```

- **timeit**: Measure small code snippets
```python
import timeit
timeit.timeit('sum(range(100))', number=10000)
```

### Database
- **EXPLAIN**: Analyze query execution plan
```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

### General Tools
- **New Relic / DataDog**: Application performance monitoring
- **Lighthouse**: Web performance auditing
- **k6 / JMeter**: Load testing

---

## Best Practices

### 1. **Use Appropriate Data Structures**
   - Need fast lookups? Use Hash Maps/Sets
   - Need sorted data? Use Trees
   - Need FIFO/LIFO? Use Queues/Stacks

### 2. **Avoid Premature Optimization**
   - Profile first, optimize second
   - Focus on algorithmic improvements before micro-optimizations

### 3. **Cache Expensive Operations**
   - Memoize function results
   - Cache database queries
   - Use CDN for static assets

### 4. **Minimize I/O Operations**
   - Batch database queries
   - Use connection pooling
   - Compress data transfers

### 5. **Optimize Critical Path**
   - Identify bottlenecks with profiling
   - Optimize the slowest 20% that causes 80% of delays
   - Consider async operations for I/O

### 6. **Use Lazy Loading**
   - Load resources only when needed
   - Defer non-critical JavaScript
   - Implement pagination for large datasets

### 7. **Leverage Parallelism**
   - Use Web Workers for heavy computations
   - Implement multi-threading where applicable
   - Consider async/await patterns

### 8. **Regular Performance Audits**
   - Monitor performance metrics continuously
   - Set performance budgets
   - Test with realistic data volumes

---

## Quick Reference: Big O Complexity

| Complexity | Name | Example Operations |
|------------|------|-------------------|
| O(1) | Constant | Array access, Hash table lookup |
| O(log n) | Logarithmic | Binary search, Balanced tree operations |
| O(n) | Linear | Array traversal, Linear search |
| O(n log n) | Linearithmic | Efficient sorting (Merge, Quick, Heap sort) |
| O(n²) | Quadratic | Nested loops, Bubble sort |
| O(2ⁿ) | Exponential | Recursive Fibonacci, Subset generation |
| O(n!) | Factorial | Permutation generation, Traveling salesman |

**Goal**: Aim for O(1), O(log n), or O(n) when possible.

---

## Conclusion

Performance optimization is about:
1. **Measuring** - Use profiling tools to identify bottlenecks
2. **Understanding** - Know your algorithms and data structures
3. **Optimizing** - Focus on the critical path
4. **Validating** - Verify improvements with benchmarks

Remember: **Premature optimization is the root of all evil** - Donald Knuth

Focus on writing clean, maintainable code first. Optimize when you have data showing it's necessary.

---

## Additional Resources

- [Big O Cheat Sheet](https://www.bigocheatsheet.com/)
- [Web.dev Performance](https://web.dev/performance/)
- [Database Indexing Guide](https://use-the-index-luke.com/)
- [JavaScript Performance Optimization](https://developer.mozilla.org/en-US/docs/Web/Performance)
- [Python Performance Tips](https://wiki.python.org/moin/PythonSpeed/PerformanceTips)
