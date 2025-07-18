
# 🚀 Intermediate Computer Fundamentals

## Table of Contents
- [Computer Architecture](#computer-architecture)
- [Data Representation](#data-representation)
- [Programming Concepts](#programming-concepts)
- [Algorithms and Data Structures](#algorithms-and-data-structures)
- [Computer Networks](#computer-networks)
- [Database Systems](#database-systems)

---

## 🏗️ Computer Architecture

### Von Neumann Architecture

```
    ┌─────────────────────────────────────────┐
    │              Memory Unit                │
    │  ┌─────────────┐  ┌─────────────────┐   │
    │  │ Instructions│  │      Data       │   │
    │  └─────────────┘  └─────────────────┘   │
    └─────────────┬───────────────────────────┘
                  │
    ┌─────────────▼───────────────────────────┐
    │           Control Unit                  │
    └─────────────┬───────────────────────────┘
                  │
    ┌─────────────▼───────────────────────────┐
    │      Arithmetic Logic Unit (ALU)       │
    └─────────────┬───────────────────────────┘
                  │
    ┌─────────────▼───────────────────────────┐
    │          Input/Output Unit              │
    └─────────────────────────────────────────┘
```

### CPU Instruction Cycle

| Phase | Description | Example |
|-------|-------------|---------|
| **Fetch** | Get instruction from memory | Load ADD instruction |
| **Decode** | Interpret the instruction | Identify as addition operation |
| **Execute** | Perform the operation | Add two numbers |
| **Store** | Save the result | Store result in register |

### 🔄 Example: Adding Two Numbers
```assembly
; Assembly language example
LOAD A, 5      ; Load value 5 into register A
LOAD B, 3      ; Load value 3 into register B
ADD A, B       ; Add B to A
STORE A, SUM   ; Store result in memory location SUM
```

---

## 🔢 Data Representation

### Number Systems

| System | Base | Digits | Example |
|--------|------|--------|---------|
| **Binary** | 2 | 0, 1 | 1011₂ = 11₁₀ |
| **Octal** | 8 | 0-7 | 13₈ = 11₁₀ |
| **Decimal** | 10 | 0-9 | 11₁₀ |
| **Hexadecimal** | 16 | 0-9, A-F | B₁₆ = 11₁₀ |

### Binary Arithmetic Examples:

```
Addition:
  1011  (11 in decimal)
+ 0101  (5 in decimal)
------
 10000  (16 in decimal)

Multiplication:
  1011  (11 in decimal)
×   10  (2 in decimal)
------
  0000
 1011
------
 10110  (22 in decimal)
```

### Data Types and Storage

```
┌─────────────────────────────────────────┐
│              Data Types                 │
├─────────────┬───────────────────────────┤
│ Primitive   │ Composite                 │
├─────────────┼───────────────────────────┤
│ • Integer   │ • Arrays                  │
│ • Float     │ • Structures              │
│ • Character │ • Classes                 │
│ • Boolean   │ • Objects                 │
└─────────────┴───────────────────────────┘
```

### Memory Allocation Example:
```c
// C programming example
int age = 25;           // 4 bytes
float salary = 50000.5; // 4 bytes
char grade = 'A';       // 1 byte
double pi = 3.14159;    // 8 bytes

// Array allocation
int numbers[5] = {1, 2, 3, 4, 5}; // 20 bytes (5 × 4)
```

---

## 💻 Programming Concepts

### Programming Paradigms

| Paradigm | Approach | Languages | Example Use |
|----------|----------|-----------|-------------|
| **Procedural** | Step-by-step functions | C, Pascal | System programming |
| **Object-Oriented** | Objects and classes | Java, C++ | Software development |
| **Functional** | Mathematical functions | Haskell, Lisp | Data analysis |
| **Event-Driven** | Event handling | JavaScript | Web development |

### Control Structures

#### 1. Sequential Structure
```python
# Python example
print("Step 1: Initialize variables")
x = 10
y = 20
result = x + y
print(f"Step 2: Result is {result}")
```

#### 2. Selection Structure
```python
# If-else example
age = 18
if age >= 18:
    print("You can vote!")
elif age >= 16:
    print("You can drive!")
else:
    print("You're still young!")
```

#### 3. Iteration Structure
```python
# Loop examples
# For loop
for i in range(5):
    print(f"Count: {i}")

# While loop
count = 0
while count < 3:
    print(f"Loop {count}")
    count += 1
```

### Functions and Modularity

```python
# Function definition
def calculate_area(length, width):
    """Calculate area of a rectangle"""
    return length * width

# Function call
room_area = calculate_area(10, 12)
print(f"Room area: {room_area} square feet")

# Function with default parameters
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

print(greet("Alice"))        # Output: Hello, Alice!
print(greet("Bob", "Hi"))    # Output: Hi, Bob!
```

---

## 📊 Algorithms and Data Structures

### Algorithm Complexity

```
Time Complexity Comparison:
    
O(1)     ←─────────── Constant
O(log n) ←─────────── Logarithmic
O(n)     ←─────────── Linear
O(n²)    ←─────────── Quadratic
O(2ⁿ)    ←─────────── Exponential

Best ────────────────→ Worst
```

### Common Sorting Algorithms

| Algorithm | Time Complexity | Space | Best Use Case |
|-----------|----------------|-------|---------------|
| **Bubble Sort** | O(n²) | O(1) | Learning purposes |
| **Quick Sort** | O(n log n) | O(log n) | General purpose |
| **Merge Sort** | O(n log n) | O(n) | Stable sorting |
| **Heap Sort** | O(n log n) | O(1) | Memory constrained |

### Bubble Sort Example:
```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr

# Example usage
numbers = [64, 34, 25, 12, 22, 11, 90]
sorted_numbers = bubble_sort(numbers.copy())
print(f"Original: {numbers}")
print(f"Sorted: {sorted_numbers}")
```

### Data Structures

#### Arrays
```python
# Array operations
fruits = ["apple", "banana", "orange"]

# Access
print(fruits[0])  # apple

# Insert
fruits.append("grape")
fruits.insert(1, "mango")

# Delete
fruits.remove("banana")
del fruits[0]

print(fruits)  # ['mango', 'orange', 'grape']
```

#### Linked Lists
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None
    
    def append(self, data):
        new_node = Node(data)
        if not self.head:
            self.head = new_node
            return
        current = self.head
        while current.next:
            current = current.next
        current.next = new_node
    
    def display(self):
        elements = []
        current = self.head
        while current:
            elements.append(current.data)
            current = current.next
        return elements

# Usage
ll = LinkedList()
ll.append(1)
ll.append(2)
ll.append(3)
print(ll.display())  # [1, 2, 3]
```

---

## 🌐 Computer Networks

### Network Types

```
┌─────────────────────────────────────────┐
│            Network Types                │
├─────────────┬───────────────────────────┤
│ By Scale    │ By Topology               │
├─────────────┼───────────────────────────┤
│ • PAN       │ • Bus                     │
│ • LAN       │ • Star                    │
│ • MAN       │ • Ring                    │
│ • WAN       │ • Mesh                    │
│ • Internet  │ • Hybrid                  │
└─────────────┴───────────────────────────┘
```

### OSI Model

| Layer | Name | Function | Examples |
|-------|------|----------|----------|
| 7 | **Application** | User interface | HTTP, FTP, SMTP |
| 6 | **Presentation** | Data formatting | SSL, JPEG, GIF |
| 5 | **Session** | Session management | NetBIOS, RPC |
| 4 | **Transport** | End-to-end delivery | TCP, UDP |
| 3 | **Network** | Routing | IP, ICMP |
| 2 | **Data Link** | Frame delivery | Ethernet, Wi-Fi |
| 1 | **Physical** | Bit transmission | Cables, Radio |

### TCP/IP Example:
```python
# Simple TCP client
import socket

# Create socket
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect to server
client.connect(('localhost', 8080))

# Send data
message = "Hello, Server!"
client.send(message.encode('utf-8'))

# Receive response
response = client.recv(1024)
print(f"Server response: {response.decode('utf-8')}")

client.close()
```

### IP Addressing

```
IPv4 Address Structure:
192.168.1.100
│   │   │ │
│   │   │ └── Host part
│   │   └──── Subnet
│   └──────── Network part
└──────────── Network class

IPv6 Example:
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

---

## 🗄️ Database Systems

### Database Models

| Model | Structure | Example | Use Case |
|-------|-----------|---------|----------|
| **Relational** | Tables with rows/columns | MySQL, PostgreSQL | Business applications |
| **Document** | JSON-like documents | MongoDB | Web applications |
| **Graph** | Nodes and edges | Neo4j | Social networks |
| **Key-Value** | Key-value pairs | Redis | Caching |

### SQL Examples:

#### Creating a Database and Table:
```sql
-- Create database
CREATE DATABASE company;
USE company;

-- Create table
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE
);

-- Insert data
INSERT INTO employees (name, department, salary, hire_date)
VALUES 
    ('John Doe', 'IT', 75000.00, '2023-01-15'),
    ('Jane Smith', 'HR', 65000.00, '2023-02-20'),
    ('Mike Johnson', 'IT', 80000.00, '2023-03-10');

-- Query data
SELECT name, salary 
FROM employees 
WHERE department = 'IT' 
ORDER BY salary DESC;
```

### Database Normalization:

```
┌─────────────────────────────────────────┐
│          Normal Forms                   │
├─────────────┬───────────────────────────┤
│ 1NF         │ Atomic values             │
│ 2NF         │ No partial dependencies   │
│ 3NF         │ No transitive dependencies│
│ BCNF        │ Every dependency is key   │
└─────────────┴───────────────────────────┘
```

### ACID Properties:

| Property | Description | Example |
|----------|-------------|---------|
| **Atomicity** | All or nothing | Bank transfer succeeds or fails completely |
| **Consistency** | Valid state always | Account balance never negative |
| **Isolation** | Concurrent transactions don't interfere | Multiple users accessing same account |
| **Durability** | Changes persist | Data survives system crash |

---

## 🎯 Performance Optimization

### Code Optimization Techniques:

```python
# Inefficient approach
def find_max_slow(numbers):
    max_val = numbers[0]
    for i in range(len(numbers)):
        if numbers[i] > max_val:
            max_val = numbers[i]
    return max_val

# Optimized approach
def find_max_fast(numbers):
    return max(numbers)  # Built-in function is optimized

# Time comparison
import time

large_list = list(range(1000000))

start = time.time()
result1 = find_max_slow(large_list)
time1 = time.time() - start

start = time.time()
result2 = find_max_fast(large_list)
time2 = time.time() - start

print(f"Slow method: {time1:.4f} seconds")
print(f"Fast method: {time2:.4f} seconds")
```

### Memory Management:
```python
# Memory-efficient generator
def fibonacci_generator(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

# Usage
for num in fibonacci_generator(10):
    print(num, end=" ")
# Output: 0 1 1 2 3 5 8 13 21 34
```

---

## 🔐 Security Fundamentals

### Common Security Threats:

| Threat | Description | Prevention |
|--------|-------------|------------|
| **Malware** | Malicious software | Antivirus, updates |
| **Phishing** | Fake communications | User education |
| **SQL Injection** | Database attacks | Parameterized queries |
| **XSS** | Script injection | Input validation |

### Password Security Example:
```python
import hashlib
import secrets

def hash_password(password):
    # Generate random salt
    salt = secrets.token_hex(16)
    
    # Combine password and salt
    pwd_hash = hashlib.pbkdf2_hmac('sha256', 
                                   password.encode('utf-8'), 
                                   salt.encode('utf-8'), 
                                   100000)
    
    return salt + pwd_hash.hex()

def verify_password(stored_password, provided_password):
    salt = stored_password[:32]
    stored_hash = stored_password[32:]
    
    pwd_hash = hashlib.pbkdf2_hmac('sha256',
                                   provided_password.encode('utf-8'),
                                   salt.encode('utf-8'),
                                   100000)
    
    return pwd_hash.hex() == stored_hash

# Example usage
password = "mySecurePassword123"
hashed = hash_password(password)
print(f"Password verified: {verify_password(hashed, password)}")
```

---

## 🚀 Next Steps

### 📈 Advanced Topics to Explore:
1. **Cloud Computing** - AWS, Azure, Google Cloud
2. **Machine Learning** - AI algorithms and models
3. **Cybersecurity** - Advanced threat protection
4. **Mobile Development** - iOS and Android apps
5. **DevOps** - CI/CD pipelines and automation

### 💡 Practice Projects:
- Build a personal website
- Create a simple database application
- Implement sorting algorithms
- Design a basic network protocol
- Develop a security tool

### 📚 Recommended Learning Path:
```
Intermediate Fundamentals
         ↓
Advanced Programming
         ↓
Specialized Tracks:
• Web Development
• Data Science
• Systems Programming
• Mobile Development
```
