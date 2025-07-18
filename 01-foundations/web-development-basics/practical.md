
# 🔨 Web Development Basics - Practical

## 🛠️ Hands-on Exercises

### Exercise 1: Understanding HTTP Requests

Use browser developer tools to inspect network requests:

```bash
# Open any website in your browser
# Press F12 to open Developer Tools
# Go to Network tab
# Refresh the page
# Observe the requests being made
```

**Tasks:**
1. Identify different types of requests (HTML, CSS, JS, images)
2. Note HTTP status codes
3. Examine request and response headers
4. Measure page load times

### Exercise 2: DNS Lookup Practice

Use command line tools to understand DNS:

```bash
# Look up IP address for a domain
nslookup google.com

# More detailed DNS information
dig google.com

# Trace the route to a server
traceroute google.com
```

### Exercise 3: HTTP Methods with curl

Practice different HTTP methods:

```bash
# GET request
curl -X GET https://jsonplaceholder.typicode.com/posts/1

# POST request
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{"title": "Test Post", "body": "This is a test", "userId": 1}'

# PUT request
curl -X PUT https://jsonplaceholder.typicode.com/posts/1 \
  -H "Content-Type: application/json" \
  -d '{"id": 1, "title": "Updated Post", "body": "Updated content", "userId": 1}'

# DELETE request
curl -X DELETE https://jsonplaceholder.typicode.com/posts/1
```

### Exercise 4: Simple Web Server

Create a basic web server using Python:

```python
# simple_server.py
import http.server
import socketserver
import os

PORT = 8000

class MyHTTPRequestHandler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/':
            self.path = '/index.html'
        return super().do_GET()

# Create a simple HTML file
html_content = """
<!DOCTYPE html>
<html>
<head>
    <title>My First Web Server</title>
</head>
<body>
    <h1>Hello, Web World!</h1>
    <p>This is served by my Python web server.</p>
</body>
</html>
"""

with open('index.html', 'w') as f:
    f.write(html_content)

# Start the server
with socketserver.TCPServer(("", PORT), MyHTTPRequestHandler) as httpd:
    print(f"Server running at http://localhost:{PORT}")
    httpd.serve_forever()
```

### Exercise 5: Understanding URLs

Break down URL components:

```
https://www.example.com:443/path/to/page?param1=value1&param2=value2#section

Protocol: https
Subdomain: www
Domain: example.com
Port: 443
Path: /path/to/page
Query Parameters: param1=value1&param2=value2
Fragment: section
```

**Practice URLs:**
- `https://api.github.com/users/octocat`
- `http://localhost:3000/admin/dashboard?tab=users&page=2`
- `https://shop.example.com/products/shoes?color=red&size=42#reviews`

### Exercise 6: Web Headers Analysis

Common HTTP headers and their purposes:

```bash
# Request Headers
curl -H "User-Agent: MyApp/1.0" \
     -H "Accept: application/json" \
     -H "Authorization: Bearer token123" \
     https://api.example.com/data

# Response Headers to look for:
# Content-Type: application/json
# Cache-Control: max-age=3600
# Set-Cookie: session=abc123
# X-Powered-By: Express
```

### Exercise 7: Status Code Practice

Test different HTTP status codes:

```bash
# 200 OK
curl -I https://www.google.com

# 404 Not Found
curl -I https://www.google.com/nonexistent

# 301 Moved Permanently
curl -I http://www.google.com

# 500 Internal Server Error (simulate with broken endpoint)
curl -I https://httpstat.us/500
```

## 🎯 Mini Project: HTTP Inspector Tool

Create a simple command-line tool to inspect HTTP requests:

```python
#!/usr/bin/env python3
import requests
import sys
from urllib.parse import urlparse

def inspect_url(url):
    """Inspect a URL and display detailed information"""
    
    # Parse URL
    parsed = urlparse(url)
    print(f"🔍 URL Analysis for: {url}")
    print(f"Protocol: {parsed.scheme}")
    print(f"Domain: {parsed.netloc}")
    print(f"Path: {parsed.path}")
    print(f"Query: {parsed.query}")
    print(f"Fragment: {parsed.fragment}")
    print("-" * 50)
    
    try:
        # Make request
        response = requests.get(url)
        
        # Display response info
        print(f"Status Code: {response.status_code}")
        print(f"Response Time: {response.elapsed.total_seconds():.2f}s")
        print(f"Content Type: {response.headers.get('content-type', 'Unknown')}")
        print(f"Content Length: {len(response.content)} bytes")
        
        # Display headers
        print("\n📋 Response Headers:")
        for header, value in response.headers.items():
            print(f"  {header}: {value}")
            
    except Exception as e:
        print(f"❌ Error: {e}")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python inspector.py <url>")
        sys.exit(1)
    
    url = sys.argv[1]
    if not url.startswith(('http://', 'https://')):
        url = 'https://' + url
    
    inspect_url(url)
```

## 📝 Practice Assignments

### Assignment 1: Web Architecture Diagram
Create a diagram showing the flow of a web request from browser to server and back.

### Assignment 2: Performance Analysis
Use browser dev tools to analyze the performance of three different websites and document your findings.

### Assignment 3: HTTP Method Testing
Create a test plan for a REST API using different HTTP methods and test it with curl or a tool like Postman.

## 🔍 Self-Assessment Questions

1. What happens when you type a URL in your browser and press Enter?
2. What's the difference between HTTP and HTTPS?
3. Explain the purpose of different HTTP status codes.
4. What is the role of DNS in web browsing?
5. How do cookies work in web applications?
6. What are the main components of an HTTP request?
7. How does caching improve web performance?

## 📚 Additional Resources

- [MDN Web Docs - HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [HTTP Status Dogs](https://httpstatusdogs.com/)
- [How DNS Works](https://howdns.works/)
- [Web Performance 101](https://3perf.com/talks/web-perf-101/)

---

**Next:** Proceed to Git Version Control module
