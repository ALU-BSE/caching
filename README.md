# Guided Learning Activity - Caching in Web Applications with Python

## Introduction

Welcome to this comprehensive guide on implementing caching in web applications using Python! Whether you're building a personal blog, an e-commerce platform, or a content management system, understanding and implementing caching is essential for creating performant web applications that can scale.

In this guide, you'll learn:
- What caching is and why it's important
- Different types of caching strategies
- How to implement various caching mechanisms in Python web applications
- Best practices for cache management
- How to measure the impact of your caching implementations
- Common pitfalls and how to avoid them

By the end of this guide, you'll have hands-on experience implementing different caching strategies and will be able to apply these concepts to your own projects.

## Table of Contents

1. [Understanding Caching Fundamentals](#understanding-caching-fundamentals)
2. [Types of Caching](#types-of-caching)
3. [Setting Up Your Development Environment](#setting-up-your-development-environment)
4. [In-Memory Caching with Python](#in-memory-caching-with-python)
5. [Database Query Caching](#database-query-caching)
6. [HTTP Caching](#http-caching)
7. [Implementing Redis for Distributed Caching](#implementing-redis-for-distributed-caching)
8. [Cache Invalidation Strategies](#cache-invalidation-strategies)
9. [Caching in Flask Applications](#caching-in-flask-applications)
10. [Caching in Django Applications](#caching-in-django-applications)
11. [Monitoring and Optimizing Your Cache](#monitoring-and-optimizing-your-cache)
12. [Final Challenge: Building a Cached API](#final-challenge-building-a-cached-api)
13. [Additional Resources](#additional-resources)

## Understanding Caching Fundamentals

### What is Caching?

Caching is the process of storing copies of data in a temporary storage location (a cache) so future requests for that data can be served faster. The primary purpose of caching is to increase data retrieval performance by reducing the need to access the underlying slower storage layer.

### Why is Caching Important?

1. **Performance Improvement**: Caching reduces latency and improves application response time.
2. **Reduced Server Load**: By serving cached data, you reduce the load on your database and application servers.
3. **Bandwidth Savings**: Less data needs to be transferred between servers and clients.
4. **Improved User Experience**: Faster load times lead to better user satisfaction.
5. **Cost Efficiency**: Reduced computational needs can lower infrastructure costs.

### Key Caching Concepts

- **Cache Hit**: When requested data is found in the cache.
- **Cache Miss**: When requested data is not found in the cache and must be fetched from the original source.
- **Cache Expiration**: When cached data is considered stale and needs to be refreshed.
- **Cache Invalidation**: The process of removing or updating cached data when the original data changes.
- **Cache Eviction**: When items are removed from the cache due to space constraints.
- **Cache Hit Ratio**: The percentage of requests that are served from the cache versus those that require fetching from the original source.

### Exercise 1: Analyzing Caching Potential

**Objective**: Identify opportunities for caching in a sample application.

1. Clone this sample Flask blog application:
```bash
git clone https://github.com/yourusername/sample-flask-blog
cd sample-flask-blog
pip install -r requirements.txt
```

2. Run the application and use a tool like Chrome DevTools or Firefox Developer Tools to analyze the network performance.

3. Create a document identifying:
   - Which resources could benefit from caching
   - What type of caching would be appropriate for each resource
   - Expected performance improvements if caching was implemented

## Types of Caching

### Browser Caching

Browser caching stores static assets (CSS, JavaScript, images) locally in the user's browser, reducing the need to download them on subsequent visits.

### Application Caching

Application caching occurs within your web application and includes:

- **Memory Caching**: Storing data in the application's memory for quick access
- **Database Query Caching**: Storing the results of expensive database queries
- **Computed Results Caching**: Storing the results of complex calculations

### Distributed Caching

Distributed caching uses external systems like Redis or Memcached to store cached data, allowing multiple application instances to share the same cache.

### Content Delivery Networks (CDNs)

CDNs cache content at strategically placed physical nodes across different geographical locations to deliver content to users more quickly and reliably.

### Exercise 2: Comparing Caching Strategies

**Objective**: Understand the trade-offs between different caching strategies.

Create a comparison table of different caching strategies with the following criteria:
- Implementation complexity
- Performance impact
- Scalability
- Data consistency challenges
- Appropriate use cases

## Setting Up Your Development Environment

To follow along with the hands-on exercises in this guide, you'll need to set up a Python development environment with the necessary tools.

### Prerequisites

- Python 3.8 or higher
- pip (Python package installer)
- Virtual environment tool (venv or conda)
- Git
- A code editor (VS Code, PyCharm, etc.)

### Setup Steps

1. Create a new project directory:
```bash
mkdir caching-workshop
cd caching-workshop
```

2. Create and activate a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install the required packages:
```bash
pip install Flask Flask-Caching redis SQLAlchemy requests pytest
```

4. Create a basic project structure:
```bash
mkdir -p app/templates app/static app/models app/utils tests
touch app/__init__.py app/models/__init__.py app/utils/__init__.py
touch app/app.py
```

### Exercise 3: Setting Up a Basic Flask Application

**Objective**: Create a simple Flask application that we'll use to implement various caching strategies.

1. Create a basic Flask app in `app/app.py`:

```python
from flask import Flask, render_template, jsonify
import time

app = Flask(__name__)

@app.route('/')
def index():
    # Simulate a slow operation
    time.sleep(2)
    return render_template('index.html')

@app.route('/api/data')
def get_data():
    # Simulate expensive data retrieval
    time.sleep(1)
    data = {
        'items': [
            {'id': 1, 'name': 'Item 1'},
            {'id': 2, 'name': 'Item 2'},
            {'id': 3, 'name': 'Item 3'}
        ],
        'timestamp': time.time()
    }
    return jsonify(data)

if __name__ == '__main__':
    app.run(debug=True)
```

2. Create a simple template in `app/templates/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Caching Demo</title>
</head>
<body>
    <h1>Caching Workshop</h1>
    <div id="data-container">Loading...</div>
    
    <script>
        // Simple script to fetch data from our API
        fetch('/api/data')
            .then(response => response.json())
            .then(data => {
                document.getElementById('data-container').innerHTML = 
                    `<pre>${JSON.stringify(data, null, 2)}</pre>`;
            });
    </script>
</body>
</html>
```

3. Run the application:
```bash
python app/app.py
```

4. Visit http://localhost:5000 in your browser and note the loading time.

## In-Memory Caching with Python

The simplest form of caching is to store data in memory. Python provides several ways to implement in-memory caching.

### Using Python's `functools.lru_cache`

The `lru_cache` decorator from the `functools` module provides a simple way to cache function results in memory using a Least Recently Used (LRU) cache strategy.

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def get_expensive_data(param):
    # Expensive operation here
    import time
    time.sleep(2)  # Simulate long processing
    return f"Result for {param}"

# First call - will be slow
result1 = get_expensive_data("test")
print(result1)

# Second call with same parameter - will be fast (cached)
result2 = get_expensive_data("test")
print(result2)

# Different parameter - will be slow again
result3 = get_expensive_data("another_test")
print(result3)
```

### Creating a Simple Cache Class

For more control, you can implement your own caching mechanism:

```python
class SimpleCache:
    def __init__(self, max_size=100):
        self.cache = {}
        self.max_size = max_size
    
    def get(self, key, default=None):
        return self.cache.get(key, default)
    
    def set(self, key, value, ttl=None):
        # Basic eviction strategy - if cache is full, remove oldest item
        if len(self.cache) >= self.max_size:
            # In a real implementation, you'd want a more sophisticated strategy
            self.cache.pop(next(iter(self.cache)))
        
        self.cache[key] = {
            'value': value,
            'expires_at': time.time() + ttl if ttl else None
        }
    
    def delete(self, key):
        if key in self.cache:
            del self.cache[key]
    
    def is_valid(self, key):
        if key not in self.cache:
            return False
        
        item = self.cache[key]
        if item['expires_at'] and time.time() > item['expires_at']:
            self.delete(key)
            return False
        
        return True

# Usage
cache = SimpleCache()
cache.set('user:1', {"name": "John", "email": "john@example.com"}, ttl=60)  # Cache for 60 seconds

if cache.is_valid('user:1'):
    user = cache.get('user:1')
    print(user)
```

### Exercise 4: Implementing Function-Level Caching

**Objective**: Improve the performance of a function that performs expensive calculations.

1. Create a new file `app/utils/calculator.py` with a function that performs a "costly" operation:

```python
import time
from functools import lru_cache

def fibonacci_uncached(n):
    """Calculate the nth Fibonacci number (inefficient recursive implementation)"""
    if n <= 1:
        return n
    return fibonacci_uncached(n-1) + fibonacci_uncached(n-2)

@lru_cache(maxsize=128)
def fibonacci_cached(n):
    """Calculate the nth Fibonacci number with caching"""
    if n <= 1:
        return n
    return fibonacci_cached(n-1) + fibonacci_cached(n-2)

def benchmark_fibonacci():
    """Compare the performance of cached vs uncached implementations"""
    n = 30
    
    # Benchmark uncached version
    start = time.time()
    result_uncached = fibonacci_uncached(n)
    uncached_time = time.time() - start
    
    # Benchmark cached version
    start = time.time()
    result_cached = fibonacci_cached(n)
    cached_time = time.time() - start
    
    return {
        'n': n,
        'result': result_cached,
        'uncached_time': uncached_time,
        'cached_time': cached_time,
        'speedup_factor': uncached_time / cached_time if cached_time > 0 else 'infinite'
    }
```

2. Create a route in `app/app.py` to display the benchmark results:

```python
from app.utils.calculator import benchmark_fibonacci

@app.route('/benchmark')
def benchmark():
    results = benchmark_fibonacci()
    return render_template('benchmark.html', results=results)
```

3. Create a template in `app/templates/benchmark.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Fibonacci Caching Benchmark</title>
    <style>
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
        .highlight { background-color: #e6ffe6; }
    </style>
</head>
<body>
    <h1>Fibonacci Caching Benchmark</h1>
    <table>
        <tr>
            <th>Parameter (n)</th>
            <td>{{ results.n }}</td>
        </tr>
        <tr>
            <th>Result (Fibonacci number)</th>
            <td>{{ results.result }}</td>
        </tr>
        <tr>
            <th>Uncached Time (seconds)</th>
            <td>{{ "%.4f"|format(results.uncached_time) }}</td>
        </tr>
        <tr class="highlight">
            <th>Cached Time (seconds)</th>
            <td>{{ "%.4f"|format(results.cached_time) }}</td>
        </tr>
        <tr class="highlight">
            <th>Speedup Factor</th>
            <td>{% if results.speedup_factor == 'infinite' %}∞{% else %}{{ "%.2f"|format(results.speedup_factor) }}x{% endif %}</td>
        </tr>
    </table>
    
    <h2>Explanation</h2>
    <p>
        The uncached Fibonacci implementation recalculates each value multiple times, leading to exponential time complexity.
        The cached version using <code>@lru_cache</code> stores results of previous calculations, dramatically improving performance.
    </p>
</body>
</html>
```

4. Run the application and visit http://localhost:5000/benchmark to see the dramatic difference caching makes.

## Database Query Caching

Database queries are often expensive operations that benefit greatly from caching. Let's explore how to implement database query caching in Python.

### Setting Up a Database

First, let's set up a simple SQLite database for our exercises:

```python
# app/models/database.py
import sqlite3
import time
from contextlib import contextmanager

DATABASE_PATH = 'app.db'

def init_db():
    """Initialize the database with sample data"""
    with get_db_connection() as conn:
        conn.execute('''
            CREATE TABLE IF NOT EXISTS products (
                id INTEGER PRIMARY KEY,
                name TEXT NOT NULL,
                description TEXT,
                price REAL NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        
        # Check if we need to insert sample data
        if conn.execute('SELECT COUNT(*) FROM products').fetchone()[0] == 0:
            # Insert 1000 sample products
            for i in range(1, 1001):
                conn.execute(
                    'INSERT INTO products (name, description, price) VALUES (?, ?, ?)',
                    (f'Product {i}', f'Description for product {i}', i * 10.99)
                )

@contextmanager
def get_db_connection():
    """Context manager for database connections"""
    conn = sqlite3.connect(DATABASE_PATH)
    conn.row_factory = sqlite3.Row
    try:
        yield conn
    finally:
        conn.close()
```

### Implementing Query Caching

Now, let's create a simple query caching layer:

```python
# app/models/products.py
import time
from app.models.database import get_db_connection

# Simple in-memory cache
_cache = {}

def get_products(page=1, per_page=10, use_cache=True):
    """Get a paginated list of products, with optional caching"""
    cache_key = f'products_page_{page}_per_page_{per_page}'
    
    # Check cache first if caching is enabled
    if use_cache and cache_key in _cache:
        cache_entry = _cache[cache_key]
        # Check if cache is still valid (30 seconds TTL)
        if time.time() - cache_entry['timestamp'] < 30:
            print("Cache hit!")
            return cache_entry['data']
        else:
            print("Cache expired!")
    else:
        print("Cache miss!")
    
    # Cache miss or expired, query the database
    start_time = time.time()
    
    with get_db_connection() as conn:
        offset = (page - 1) * per_page
        query = '''
            SELECT id, name, price 
            FROM products 
            ORDER BY id 
            LIMIT ? OFFSET ?
        '''
        rows = conn.execute(query, (per_page, offset)).fetchall()
        
        # Convert to list of dictionaries
        products = [dict(row) for row in rows]
        
        # Get total count for pagination
        total = conn.execute('SELECT COUNT(*) FROM products').fetchone()[0]
    
    query_time = time.time() - start_time
    
    result = {
        'products': products,
        'pagination': {
            'page': page,
            'per_page': per_page,
            'total': total,
            'pages': (total + per_page - 1) // per_page
        },
        'query_time': query_time
    }
    
    # Update cache
    if use_cache:
        _cache[cache_key] = {
            'data': result,
            'timestamp': time.time()
        }
    
    return result
```

### Adding Database Routes to Our Flask App

```python
# Update app/app.py
from app.models.database import init_db
from app.models.products import get_products

# Initialize database on startup
init_db()

@app.route('/products')
def product_list():
    page = int(request.args.get('page', 1))
    per_page = int(request.args.get('per_page', 10))
    use_cache = request.args.get('cache', 'true').lower() != 'false'
    
    result = get_products(page, per_page, use_cache)
    
    return render_template('products.html', 
                          products=result['products'], 
                          pagination=result['pagination'],
                          query_time=result['query_time'],
                          use_cache=use_cache)
```

### Create a Products Template

```html
<!-- app/templates/products.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Products - Caching Demo</title>
    <style>
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
        .pagination { margin: 20px 0; }
        .cache-info { background-color: #f9f9f9; padding: 10px; margin: 10px 0; }
    </style>
</head>
<body>
    <h1>Products List</h1>
    
    <div class="cache-info">
        <p>Query executed in <strong>{{ "%.4f"|format(query_time) }} seconds</strong></p>
        <p>Cache is currently <strong>{{ "ENABLED" if use_cache else "DISABLED" }}</strong></p>
        <p>
            <a href="?cache=true&page={{ pagination.page }}">Enable Cache</a> | 
            <a href="?cache=false&page={{ pagination.page }}">Disable Cache</a>
        </p>
    </div>
    
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Price</th>
            </tr>
        </thead>
        <tbody>
            {% for product in products %}
            <tr>
                <td>{{ product.id }}</td>
                <td>{{ product.name }}</td>
                <td>${{ "%.2f"|format(product.price) }}</td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
    
    <div class="pagination">
        {% if pagination.page > 1 %}
            <a href="?page={{ pagination.page - 1 }}&cache={{ 'true' if use_cache else 'false' }}">Previous</a>
        {% endif %}
        
        Page {{ pagination.page }} of {{ pagination.pages }}
        
        {% if pagination.page < pagination.pages %}
            <a href="?page={{ pagination.page + 1 }}&cache={{ 'true' if use_cache else 'false' }}">Next</a>
        {% endif %}
    </div>
</body>
</html>
```

### Exercise 5: Implementing a Cached Product Detail Page

**Objective**: Create a product detail page with caching to improve load times.

1. Add a new function to `app/models/products.py`:

```python
def get_product_by_id(product_id, use_cache=True):
    """Get a product by ID with caching"""
    cache_key = f'product_{product_id}'
    
    # Check cache first if caching is enabled
    if use_cache and cache_key in _cache:
        cache_entry = _cache[cache_key]
        # Check if cache is still valid (1 minute TTL)
        if time.time() - cache_entry['timestamp'] < 60:
            print(f"Cache hit for product {product_id}!")
            return cache_entry['data']
        else:
            print(f"Cache expired for product {product_id}!")
    else:
        print(f"Cache miss for product {product_id}!")
    
    # Cache miss or expired, query the database
    start_time = time.time()
    
    with get_db_connection() as conn:
        # Simulate a complex query with SLEEP
        query = '''
            SELECT * FROM products WHERE id = ?
        '''
        # Simulate a complex join or slow query
        time.sleep(0.5)  
        product = conn.execute(query, (product_id,)).fetchone()
        
        if product is None:
            return None
        
        product = dict(product)
    
    query_time = time.time() - start_time
    
    result = {
        'product': product,
        'query_time': query_time
    }
    
    # Update cache
    if use_cache:
        _cache[cache_key] = {
            'data': result,
            'timestamp': time.time()
        }
    
    return result
```

2. Add a product detail route to `app/app.py`:

```python
from app.models.products import get_product_by_id

@app.route('/products/<int:product_id>')
def product_detail(product_id):
    use_cache = request.args.get('cache', 'true').lower() != 'false'
    result = get_product_by_id(product_id, use_cache)
    
    if not result:
        return "Product not found", 404
    
    return render_template('product_detail.html', 
                          product=result['product'],
                          query_time=result['query_time'],
                          use_cache=use_cache)
```

3. Create a product detail template:

```html
<!-- app/templates/product_detail.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{{ product.name }} - Caching Demo</title>
    <style>
        .product-detail { border: 1px solid #ddd; padding: 20px; margin: 20px 0; }
        .cache-info { background-color: #f9f9f9; padding: 10px; margin: 10px 0; }
    </style>
</head>
<body>
    <h1>Product Detail</h1>
    
    <div class="cache-info">
        <p>Query executed in <strong>{{ "%.4f"|format(query_time) }} seconds</strong></p>
        <p>Cache is currently <strong>{{ "ENABLED" if use_cache else "DISABLED" }}</strong></p>
        <p>
            <a href="?cache=true">Enable Cache</a> | 
            <a href="?cache=false">Disable Cache</a>
        </p>
    </div>
    
    <div class="product-detail">
        <h2>{{ product.name }}</h2>
        <p><strong>ID:</strong> {{ product.id }}</p>
        <p><strong>Price:</strong> ${{ "%.2f"|format(product.price) }}</p>
        <p><strong>Description:</strong> {{ product.description }}</p>
        <p><strong>Added:</strong> {{ product.created_at }}</p>
    </div>
    
    <p><a href="/products">Back to Products List</a></p>
</body>
</html>
```

4. Update the products list template to link to detail pages:

```html
<!-- Modify the product row in products.html -->
<td><a href="/products/{{ product.id }}">{{ product.name }}</a></td>
```

5. Test the application by navigating between product pages with caching enabled and disabled to see the performance difference.

## HTTP Caching

HTTP caching leverages standard HTTP headers to instruct browsers and proxy servers on how to cache content.

### Key HTTP Cache Headers

- **Cache-Control**: Specifies caching directives for both requests and responses
- **ETag**: A unique identifier for a specific version of a resource
- **Last-Modified**: Indicates when the resource was last modified
- **Expires**: Provides a date/time after which the response is considered stale
- **Vary**: Specifies which request headers the server uses for selecting a response

### Implementing HTTP Caching in Flask

Let's add HTTP caching to our Flask application:

```python
# Add to app/app.py
from datetime import datetime, timedelta
from flask import make_response, request

@app.route('/static-content')
def static_content():
    """Example of content that rarely changes"""
    content = "<h1>Static Content</h1><p>This content rarely changes and can be cached for a long time.</p>"
    
    # Create response
    response = make_response(render_template('static_content.html', content=content))
    
    # Set Cache-Control header
    # max-age=3600: Cache for 1 hour (3600 seconds)
    # public: Can be cached by browsers and intermediate caches
    response.headers['Cache-Control'] = 'public, max-age=3600'
    
    # Set Expires header
    response.headers['Expires'] = (datetime.utcnow() + timedelta(hours=1)).strftime('%a, %d %b %Y %H:%M:%S GMT')
    
    return response

@app.route('/dynamic-content')
def dynamic_content():
    """Example of content that changes frequently"""
    # Generate dynamic content with current timestamp
    now = datetime.utcnow()
    content = f"<h1>Dynamic Content</h1><p>Generated at: {now.strftime('%H:%M:%S')}</p>"
    
    # Create response
    response = make_response(render_template('dynamic_content.html', content=content))
    
    # Set Cache-Control header
    # no-cache: Must revalidate with server before using cached version
    # must-revalidate: Don't use stale cached version without checking with server first
    response.headers['Cache-Control'] = 'no-cache, must-revalidate'
    
    # Set Last-Modified header
    response.headers['Last-Modified'] = now.strftime('%a, %d %b %Y %H:%M:%S GMT')
    
    return response

@app.route('/conditional-content')
def conditional_content():
    """Example of conditional caching with ETag"""
    # Generate content
    content = "<h1>Conditional Content</h1><p>This content uses ETags for efficient caching.</p>"
    
    # Generate an ETag (in a real app, this would be based on the content)
    etag = f"v1-{hash(content)}"
    
    # Check if the client sent an If-None-Match header matching our ETag
    if request.headers.get('If-None-Match') == etag:
        # The client already has the current version
        return '', 304  # 304 Not Modified
    
    # Create response
    response = make_response(render_template('conditional_content.html', content=content))
    
    # Set ETag and Cache-Control headers
    response.headers['ETag'] = etag
    response.headers['Cache-Control'] = 'public, max-age=300'  # Cache for 5 minutes
    
    return response
```

Create the corresponding templates:

```html
<!-- app/templates/static_content.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Static Content - HTTP Caching Demo</title>
</head>
<body>
    {{ content|safe }}
    <p>Refresh the page to test caching. This content should be cached by your browser.</p>
    <p><a href="/">Back to Home</a></p>
</body>
</html>
```

```html
<!-- app/templates/dynamic_content.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Dynamic Content - HTTP Caching Demo</title>
</head>
<body>
    {{ content|safe }}
    <p>Refresh the page to test caching. This content should refresh each time.</p>
    <p><a href="/">Back to Home</a></p>
</body>
</html>
```

```html
<!-- app/templates/conditional_content.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Conditional Content - HTTP Caching Demo</title>
</head>
<body>
    {{ content|safe }}
    <p>Refresh the page to test caching. The server should respond with 304 Not Modified if the content hasn't changed.</p>
    <p><a href="/">Back to Home</a></p>
</body>
</html>
```

### Exercise 6: Analyzing HTTP Caching Behavior

**Objective**: Use browser developer tools to observe HTTP caching behavior.

1. Update the main index page to include links to the caching demo pages:

```html
<!-- Update app/templates/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Caching Demo</title>
</head>
<body>
    <h1>Caching Workshop</h1>
    
    <h2>Application Features</h2>
    <ul>
        <li><a href="/benchmark">Fibonacci Caching Benchmark</a></li>
        <li><a href="/products">Product List (Database Caching)</a></li>
    </ul>
    
    <h2>HTTP Caching Examples</h2>
    <ul>
        <li><a href="/static-content">Static Content (Long-term Caching)</a></li>
        <li><a href="/dynamic-content">Dynamic Content (No Caching)</a></li>
        <li><a href="/conditional-content">Conditional Content (ETag-based)</a></li>
    </ul>
    
    <div id="data-container">Loading API data...</div>
    
    <script>
        // Simple script to fetch data from our API
        fetch('/api/data')
            .then(response => response.json())
            .then(data => {
                document.getElementById('data-container').innerHTML = 
                    `<pre>${JSON.stringify(data, null, 2)}</pre>`;
            });
    </script>
</body>
</html>
```

2. Open your browser's developer tools (F12 in most browsers).
3. Navigate to the Network tab and check the
